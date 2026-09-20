# Playwright Authentication, StorageState, Fixtures and Test

This document shows a complete Playwright setup using:

* `playwright.config.js`
* Authentication setup project
* `storageState`
* Custom `loginFixture`
* Built-in `page` fixture
* Test using the custom fixture
* Chromium, Firefox, and WebKit projects

---

# 1. Project Structure

Our project can look like this:

```text
playwright-fixture-project/
│
├── tests/
│   └── products.spec.js
│
├── auth.setup.js
├── fixtures.js
├── playwright.config.js
├── auth.json
├── package.json
└── node_modules/
```

### Important files

| File                   | Purpose                         |
| ---------------------- | ------------------------------- |
| `playwright.config.js` | Playwright configuration        |
| `auth.setup.js`        | Logs in and creates `auth.json` |
| `auth.json`            | Stores authentication state     |
| `fixtures.js`          | Creates our custom fixture      |
| `products.spec.js`     | Actual test                     |

---

# 2. Complete Flow

The overall flow is:

```text
auth.setup.js
      │
      │ Login
      ▼
auth.json
      │
      │ storageState
      ▼
Browser Context
      │
      │ built-in page fixture
      ▼
page
      │
      │ custom fixture
      ▼
loginFixture
      │
      ▼
Test
```

In simple words:

```text
Login
  ↓
Save authentication
  ↓
auth.json
  ↓
Create authenticated browser context
  ↓
Playwright creates page
  ↓
Custom fixture receives page
  ↓
Test receives custom fixture
```

---

# 3. playwright.config.js

This is the main configuration file.

```javascript
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({

  testDir: './tests',

  workers: 1,

  use: {
    baseURL: 'https://www.saucedemo.com',
    trace: 'on-first-retry',
  },

  projects: [

    // 1. Authentication setup project
    {
      name: 'setup',
      testMatch: /auth\.setup\.js/,
    },

    // 2. Chromium
    {
      name: 'chromium',

      use: {
        ...devices['Desktop Chrome'],
        storageState: 'auth.json',
      },

      dependencies: ['setup'],
    },

    // 3. Firefox
    {
      name: 'firefox',

      use: {
        ...devices['Desktop Firefox'],
        storageState: 'auth.json',
      },

      dependencies: ['setup'],
    },

    // 4. WebKit
    {
      name: 'webkit',

      use: {
        ...devices['Desktop Safari'],
        storageState: 'auth.json',
      },

      dependencies: ['setup'],
    },
  ],
});
```

---

# 4. Understanding the Setup Project

This section is important.

```javascript
{
  name: 'setup',
  testMatch: /auth\.setup\.js/,
}
```

This tells Playwright:

> Find and run `auth.setup.js` as the setup project.

Then:

```javascript
dependencies: ['setup']
```

means:

> This browser project depends on the setup project.

For example:

```javascript
{
  name: 'chromium',
  dependencies: ['setup']
}
```

means:

```text
setup
  ↓
chromium
```

Firefox:

```text
setup
  ↓
firefox
```

WebKit:

```text
setup
  ↓
webkit
```

So when we run:

```bash
npx playwright test
```

Playwright understands that setup must run first.

---

# 5. auth.setup.js

This file performs the login.

```javascript
import { test as setup } from '@playwright/test';

setup('authenticate', async ({ page }) => {

  // Open application
  await page.goto('/');

  // Enter username
  await page.getByPlaceholder('Username').fill('standard_user');

  // Enter password
  await page.getByPlaceholder('Password').fill('secret_sauce');

  // Click Login
  await page.getByRole('button', { name: 'Login' }).click();

  // Save authentication state
  await page.context().storageState({
    path: 'auth.json'
  });
});
```

---

# 6. What Happens Inside auth.setup.js?

Let's understand this line:

```javascript
setup('authenticate', async ({ page }) => {
```

The `page` here is a **built-in Playwright fixture**.

We did not create it.

Playwright provides it automatically.

So:

```javascript
async ({ page }) => {
```

means:

> Playwright, please give me the built-in `page` fixture.

Then:

```javascript
await page.goto('/');
```

opens:

```text
https://www.saucedemo.com/
```

because our config contains:

```javascript
baseURL: 'https://www.saucedemo.com'
```

---

# 7. Login

The setup performs the actual login:

```javascript
await page.getByPlaceholder('Username').fill('standard_user');

await page.getByPlaceholder('Password').fill('secret_sauce');

await page.getByRole('button', { name: 'Login' }).click();
```

After successful login, the browser has authentication information.

We then save it:

```javascript
await page.context().storageState({
  path: 'auth.json'
});
```

---

# 8. What is page.context()?

This is an important concept.

The hierarchy is:

```text
Browser
   │
   └── Browser Context
          │
          └── Page
```

For example:

```javascript
page
```

belongs to:

```javascript
page.context()
```

So:

```javascript
page.context()
```

means:

> Give me the browser context that owns this page.

Then:

```javascript
page.context().storageState()
```

means:

> Get/save the authentication state of this browser context.

---

# 9. auth.json

After `auth.setup.js` runs, Playwright creates:

```text
auth.json
```

Conceptually:

```text
auth.setup.js
      │
      │ Login
      ▼
Browser Context
      │
      │ storageState()
      ▼
auth.json
```

The file contains authentication-related browser state such as cookies and local storage.

We don't normally edit this file manually.

---

# 10. How storageState is Used

Look at the Chromium project:

```javascript
{
  name: 'chromium',

  use: {
    ...devices['Desktop Chrome'],
    storageState: 'auth.json',
  },

  dependencies: ['setup'],
}
```

This line is important:

```javascript
storageState: 'auth.json'
```

It tells Playwright:

> When creating the browser context for this project, load the authentication state from `auth.json`.

Therefore:

```text
auth.json
    ↓
Browser Context
    ↓
Page
```

The page starts with the saved authentication state.

---

# 11. Firefox and WebKit

We do the same thing for Firefox:

```javascript
{
  name: 'firefox',

  use: {
    ...devices['Desktop Firefox'],
    storageState: 'auth.json',
  },

  dependencies: ['setup'],
}
```

And WebKit:

```javascript
{
  name: 'webkit',

  use: {
    ...devices['Desktop Safari'],
    storageState: 'auth.json',
  },

  dependencies: ['setup'],
}
```

Therefore:

```text
                auth.setup.js
                     │
                     ▼
                  auth.json
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
      Chromium     Firefox    WebKit
```

One setup creates the authentication state.

All three browser projects can reuse it.

---

# 12. fixtures.js

Now we create our custom fixture.

```javascript
import { test as base } from '@playwright/test';

export const test = base.extend({

  loginFixture: async ({ page }, use) => {

    console.log('=== Authenticated fixture is running ===');

    await page.goto('/');

    await use(page);
  }

});
```

---

# 13. Understanding the Custom Fixture

This line is the most important:

```javascript
loginFixture: async ({ page }, use) => {
```

There are two important things here:

```javascript
{ page }
```

and:

```javascript
use
```

---

# 14. Where Does page Come From?

The `page` comes from Playwright's **built-in fixture**.

Playwright already provides fixtures such as:

```text
browser
context
page
request
```

We are using:

```javascript
page
```

inside our custom fixture.

So:

```javascript
async ({ page }, use) => {
```

means:

> Playwright, give my custom fixture the built-in `page` fixture.

---

# 15. How Does the page Become Authenticated?

Remember our configuration:

```javascript
storageState: 'auth.json'
```

Playwright uses that configuration when creating the browser context.

Therefore:

```text
auth.json
     ↓
Authenticated Browser Context
     ↓
Built-in page fixture
     ↓
Custom loginFixture
```

The custom fixture doesn't directly read `auth.json`.

It receives a `page` that belongs to the context where `storageState` has already been applied.

---

# 16. What Does use(page) Mean?

Our fixture contains:

```javascript
await use(page);
```

This means:

> Make this page available to the test as `loginFixture`.

So:

```javascript
loginFixture
```

is basically the page we passed through:

```javascript
use(page)
```

Conceptually:

```text
Built-in page
      │
      ▼
loginFixture
      │
      ▼
Test
```

---

# 17. Test File

Now create:

```text
tests/products.spec.js
```

Code:

```javascript
import { test } from '../fixtures.js';
import { expect } from '@playwright/test';

test('verify Products page', async ({ loginFixture }) => {

  console.log('=== Test is running ===');

  await expect(
    loginFixture.getByText('Products')
  ).toBeVisible();

});
```

---

# 18. How Does the Test Get loginFixture?

Look at:

```javascript
test('verify Products page', async ({ loginFixture }) => {
```

We are asking Playwright:

> Give me the custom fixture called `loginFixture`.

That fixture was created here:

```javascript
export const test = base.extend({

  loginFixture: async ({ page }, use) => {

    await page.goto('/');

    await use(page);
  }

});
```

Therefore:

```text
Test
 │
 │ requests loginFixture
 ▼
loginFixture
 │
 │ receives page
 ▼
Built-in page
 │
 │ belongs to authenticated context
 ▼
storageState
 │
 ▼
auth.json
```

---

# 19. Complete Execution Flow

Now let's put everything together.

When we execute:

```bash
npx playwright test
```

Playwright follows the project dependencies.

### Step 1 — Setup starts

```text
setup project
```

Playwright runs:

```text
auth.setup.js
```

---

### Step 2 — Login

The setup uses the built-in:

```javascript
page
```

and performs:

```text
Open SauceDemo
      ↓
Enter username
      ↓
Enter password
      ↓
Click Login
```

---

### Step 3 — Save authentication

This executes:

```javascript
await page.context().storageState({
  path: 'auth.json'
});
```

Result:

```text
auth.json
```

is created.

---

### Step 4 — Chromium starts

Chromium has:

```javascript
storageState: 'auth.json'
```

So Playwright creates an authenticated browser context.

```text
auth.json
    ↓
Chromium Context
    ↓
Page
```

---

### Step 5 — Custom fixture starts

The custom fixture receives the built-in page:

```javascript
loginFixture: async ({ page }, use) => {
```

Then:

```javascript
await page.goto('/');
```

Then:

```javascript
await use(page);
```

---

### Step 6 — Test starts

The test receives:

```javascript
loginFixture
```

and executes:

```javascript
await expect(
  loginFixture.getByText('Products')
).toBeVisible();
```

---

# 20. Complete Execution Diagram

The entire process is:

```text
                    npx playwright test
                            │
                            ▼
                     Setup Project
                            │
                            ▼
                     auth.setup.js
                            │
                            ▼
                         Login
                            │
                            ▼
                       auth.json
                            │
              ┌─────────────┼─────────────┐
              │             │             │
              ▼             ▼             ▼
          Chromium       Firefox        WebKit
              │             │             │
              ▼             ▼             ▼
       storageState    storageState   storageState
              │             │             │
              ▼             ▼             ▼
       Browser Context Browser Context Browser Context
              │             │             │
              ▼             ▼             ▼
           page          page          page
              │             │             │
              ▼             ▼             ▼
       loginFixture   loginFixture   loginFixture
              │             │             │
              ▼             ▼             ▼
            Test          Test          Test
```

---

# 21. Important Difference: storageState vs Fixture

These two things have different responsibilities.

## storageState

```javascript
storageState: 'auth.json'
```

Its job is:

> Restore authentication state.

It works at the **browser context level**.

---

## Fixture

```javascript
loginFixture: async ({ page }, use) => {
```

Its job is:

> Provide or prepare something for the test.

In our example, it provides an authenticated page to the test.

---

# 22. Does the Fixture Perform Login?

In this design:

**No.**

The login happens in:

```text
auth.setup.js
```

The authentication is saved to:

```text
auth.json
```

The context loads:

```text
auth.json
```

The fixture receives:

```text
page
```

The test receives:

```text
loginFixture
```

So we don't need to log in again inside the fixture.

---

# 23. Avoid Duplicate Login

A common mistake is doing this:

```javascript
loginFixture: async ({ page }, use) => {

  await page.goto('/');

  await page.fill('#username', 'standard_user');
  await page.fill('#password', 'secret_sauce');
  await page.click('#login');

  await use(page);
}
```

while also using:

```javascript
storageState: 'auth.json'
```

That means authentication is being handled twice.

```text
storageState
     ↓
Already authenticated

       +

Fixture
     ↓
Login again
```

Usually this is unnecessary.

A cleaner design is:

```text
Setup → Login
          ↓
      auth.json
          ↓
   storageState
          ↓
   Authenticated page
          ↓
      Fixture
          ↓
        Test
```

---

# 24. Built-in Fixture vs Custom Fixture

### Built-in fixture

Playwright provides it:

```javascript
page
```

Example:

```javascript
test('example', async ({ page }) => {
  await page.goto('/');
});
```

We don't create `page`.

---

### Custom fixture

We create it:

```javascript
loginFixture
```

Example:

```javascript
export const test = base.extend({

  loginFixture: async ({ page }, use) => {
    await use(page);
  }

});
```

Then the test can use:

```javascript
test('example', async ({ loginFixture }) => {
});
```

---

# 25. The Key Concept

Remember this chain:

```text
Playwright built-in fixture
          │
          ▼
        page
          │
          ▼
    custom fixture
          │
          ▼
   loginFixture
          │
          ▼
        test
```

And authentication is applied earlier:

```text
auth.json
    │
    ▼
storageState
    │
    ▼
Browser Context
    │
    ▼
page
```

Combining both:

```text
auth.json
    │
    ▼
storageState
    │
    ▼
Authenticated Browser Context
    │
    ▼
Built-in page fixture
    │
    ▼
Custom loginFixture
    │
    ▼
Test
```

---

# 26. Files Summary

## playwright.config.js

Responsible for:

```text
Projects
Base URL
Browser configuration
storageState
Setup dependency
```

## auth.setup.js

Responsible for:

```text
Login
Save authentication state
Create auth.json
```

## auth.json

Responsible for:

```text
Stored authentication state
```

## fixtures.js

Responsible for:

```text
Custom fixture
Receive built-in page
Provide page to test
```

## products.spec.js

Responsible for:

```text
Actual test validation
```

---

# 27. Interview Answer

If an interviewer asks:

**"How do storageState and fixtures work together in Playwright?"**

A good answer is:

> `storageState` is used to save and restore authentication state such as cookies and local storage. We can create the authentication state in a setup project and save it to a JSON file. The browser projects load that file through `storageState`, so their browser contexts start authenticated. A custom fixture can then depend on Playwright's built-in `page` fixture and provide that authenticated page to the test. This avoids performing the login repeatedly in every test.

---

# 28. One-Line Mental Model

Remember this:

```text
Setup logs in → auth.json saves login → storageState restores login → page receives authenticated context → fixture provides page → test uses fixture
```

That is the complete relationship between:

```text
Config
+
Setup
+
storageState
+
Built-in page fixture
+
Custom fixture
+
Test
```
