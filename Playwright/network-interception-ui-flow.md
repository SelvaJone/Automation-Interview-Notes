# Playwright Network Interception – UI to API Flow

## 1. Important Concept

In real-world UI testing, a user performs an action in the browser, and the application sends an API request in the background.

For example:

```text
User
 ↓
Clicks "Create Customer"
 ↓
Application processes the click
 ↓
Application sends POST /customers
 ↓
Playwright intercepts the request
 ↓
Modify / Continue / Mock / Abort
 ↓
Application receives response
```

The important point is:

> `page.route()` does not trigger the API request. The UI action or application code triggers the request. `page.route()` intercepts it.

---

# 2. UI Action Triggers the API Request

Suppose the application has:

```html
<button id="create-customer">
    Create Customer
</button>
```

The test performs:

```javascript
await page.locator('#create-customer').click();
```

The application may internally execute something similar to:

```javascript
fetch('/customers', {
    method: 'POST',
    body: JSON.stringify({
        name: 'Selva',
        email: 'selva@test.com'
    })
});
```

We don't normally write this `fetch()` in our UI test.

The **application itself** makes the API call.

---

# 3. Playwright Intercepts the API Request

Before clicking the button, we register a route:

```javascript
await page.route(
    '**/customers',
    async route => {
        // Intercepted request
    }
);
```

Now Playwright watches for requests matching:

```text
**/customers
```

When the application sends:

```text
POST /customers
```

Playwright intercepts it.

---

# 4. Complete Example

```javascript
import { test } from '@playwright/test';

test('modify API request', async ({ page }) => {

    // 1. Register network interception
    await page.route(
        '**/customers',
        async route => {

            // 2. Read the request body
            const postData = route.request().postDataJSON();

            // 3. Modify the request body
            postData.email = 'updated@test.com';

            // 4. Continue with modified request
            await route.continue({
                headers: {
                    ...route.request().headers(),
                    'x-test-environment': 'qa'
                },
                postData: JSON.stringify(postData)
            });
        }
    );

    // 5. UI action triggers the API request
    await page.locator('#create-customer').click();
});
```

---

# 5. Step-by-Step Execution

## Step 1 – Register the Route

```javascript
await page.route(
    '**/customers',
    async route => {
        // interception logic
    }
);
```

Playwright is now waiting for a matching request.

Think:

> "If the application calls `/customers`, catch that request."

---

## Step 2 – Perform the UI Action

```javascript
await page.locator('#create-customer').click();
```

The test clicks the button.

The application responds to the click and sends its API request.

---

## Step 3 – Application Sends API Request

The application sends something like:

```json
{
    "name": "Selva",
    "email": "selva@test.com"
}
```

to:

```text
POST /customers
```

---

## Step 4 – Playwright Intercepts the Request

Because we registered:

```javascript
await page.route('**/customers', ...)
```

Playwright catches the request.

The callback executes:

```javascript
async route => {
    // interception logic
}
```

---

## Step 5 – Read the Request Body

```javascript
const postData = route.request().postDataJSON();
```

Now `postData` contains:

```json
{
    "name": "Selva",
    "email": "selva@test.com"
}
```

---

## Step 6 – Modify the Request

```javascript
postData.email = 'updated@test.com';
```

Now the request body becomes:

```json
{
    "name": "Selva",
    "email": "updated@test.com"
}
```

---

## Step 7 – Modify the Header

```javascript
headers: {
    ...route.request().headers(),
    'x-test-environment': 'qa'
}
```

This preserves the existing headers and adds:

```text
x-test-environment: qa
```

---

## Step 8 – Continue the Request

```javascript
await route.continue({
    headers: {
        ...route.request().headers(),
        'x-test-environment': 'qa'
    },
    postData: JSON.stringify(postData)
});
```

The modified request is now sent to the **real server**.

---

# 6. Complete Flow

```text
┌─────────────────────┐
│       Test          │
└──────────┬──────────┘
           │
           │ click()
           ↓
┌─────────────────────┐
│    Application      │
└──────────┬──────────┘
           │
           │ POST /customers
           ↓
┌─────────────────────┐
│ Playwright Route    │
│     Intercepts      │
└──────────┬──────────┘
           │
           │ Read request
           ↓
┌─────────────────────┐
│ Modify headers/body │
└──────────┬──────────┘
           │
           │ continue()
           ↓
┌─────────────────────┐
│    Real Server      │
└─────────────────────┘
```

---

# 7. `page.route()` vs `click()`

These have completely different responsibilities.

### `page.route()`

```javascript
await page.route('**/customers', ...);
```

Means:

> Intercept a matching network request.

### `click()`

```javascript
await page.locator('#create-customer').click();
```

Means:

> Perform a UI action.

The click may cause the application to make an API request.

---

# 8. Why Do We Register `route()` Before `click()`?

Correct:

```javascript
await page.route(
    '**/customers',
    async route => {
        // interception
    }
);

await page.locator('#create-customer').click();
```

Incorrect approach:

```javascript
await page.locator('#create-customer').click();

await page.route(
    '**/customers',
    async route => {
        // too late
    }
);
```

The API request could already have happened before Playwright started intercepting it.

### Easy Rule

> Register the interceptor first, then perform the action that triggers the request.

---

# 9. Three Common Scenarios

## Scenario 1 – Continue Original Request

```javascript
await page.route(
    '**/customers',
    async route => {
        await route.continue();
    }
);
```

Flow:

```text
UI click
 ↓
API request
 ↓
Intercept
 ↓
Continue unchanged
 ↓
Real server
```

---

## Scenario 2 – Modify Request

```javascript
await page.route(
    '**/customers',
    async route => {

        const postData = route.request().postDataJSON();

        postData.email = 'updated@test.com';

        await route.continue({
            postData: JSON.stringify(postData)
        });
    }
);
```

Flow:

```text
UI click
 ↓
API request
 ↓
Intercept
 ↓
Modify request
 ↓
Real server
```

---

## Scenario 3 – Mock Response

```javascript
await page.route(
    '**/customers',
    async route => {

        await route.fulfill({
            status: 201,
            contentType: 'application/json',
            body: JSON.stringify({
                id: 101,
                name: 'Selva',
                email: 'selva@test.com'
            })
        });
    }
);
```

Flow:

```text
UI click
 ↓
API request
 ↓
Intercept
 ↓
Mock response
 ↓
Application
```

The real server is not called.

---

# 10. `continue()` vs `fulfill()` vs `abort()`

| Method                  | What happens?                        |
| ----------------------- | ------------------------------------ |
| `route.continue()`      | Original request goes to real server |
| `route.continue({...})` | Modified request goes to real server |
| `route.fulfill()`       | Playwright returns a mocked response |
| `route.abort()`         | Request is blocked                   |

### Easy Memory

```text
continue()         → Let it go
continue(options)  → Modify it and let it go
fulfill()          → Give my response
abort()            → Stop it
```

---

# 11. Common Request Modifications

## Modify Header

```javascript
await route.continue({
    headers: {
        ...route.request().headers(),
        'Authorization': 'Bearer new-token'
    }
});
```

---

## Add Custom Header

```javascript
await route.continue({
    headers: {
        ...route.request().headers(),
        'x-test-environment': 'qa'
    }
});
```

---

## Modify URL

```javascript
const url = new URL(route.request().url());

url.searchParams.set('category', 'laptop');

await route.continue({
    url: url.toString()
});
```

---

## Modify POST Body

```javascript
const postData = route.request().postDataJSON();

postData.email = 'updated@test.com';

await route.continue({
    postData: JSON.stringify(postData)
});
```

---

# 12. UI + Network Interception Example

A realistic test might look like this:

```javascript
test('create customer with modified API request', async ({ page }) => {

    await page.route(
        '**/customers',
        async route => {

            const postData = route.request().postDataJSON();

            postData.email = 'qa@test.com';

            await route.continue({
                headers: {
                    ...route.request().headers(),
                    'x-test-environment': 'qa'
                },
                postData: JSON.stringify(postData)
            });
        }
    );

    await page.goto('https://example.com');

    await page.locator('#create-customer').click();

    await expect(
        page.locator('#success-message')
    ).toBeVisible();
});
```

Here we are testing the complete flow:

```text
UI
 ↓
Click button
 ↓
Application
 ↓
API request
 ↓
Playwright intercepts
 ↓
Modify request
 ↓
Real API
 ↓
Response
 ↓
UI
 ↓
Verify result
```

---

# 13. API Testing vs Network Interception

These should not be confused.

### API Testing

We directly call the API:

```javascript
const response = await request.post(
    'https://api.example.com/customers',
    {
        data: {
            name: 'Selva',
            email: 'selva@test.com'
        }
    }
);
```

Flow:

```text
Test
 ↓
API
 ↓
Server
```

### Network Interception

The application makes the API call:

```text
Test
 ↓
UI click
 ↓
Application
 ↓
API request
 ↓
Playwright intercepts
 ↓
Server / Mock response
```

---

# 14. Interview Answer

### Question:

**How can you modify an API request triggered by a UI action in Playwright?**

### Answer:

> First, I register a route using `page.route()` before performing the UI action. When the UI action triggers the API request, Playwright intercepts it. I can inspect the request using `route.request()`, modify headers, URL, or POST data, and then use `route.continue()` to send the modified request to the real server.

Example:

```javascript
await page.route(
    '**/customers',
    async route => {

        const postData = route.request().postDataJSON();

        postData.email = 'updated@test.com';

        await route.continue({
            postData: JSON.stringify(postData)
        });
    }
);

await page.locator('#create-customer').click();
```

---

# 15. Key Interview Point

Remember this sentence:

> **The UI action triggers the API request; `page.route()` intercepts the request.**

This is one of the most important concepts to understand when combining **UI automation and network interception** in Playwright.

---

# Quick Revision

```text
page.route()
     ↓
Register interceptor
     ↓
UI action
     ↓
Application sends API request
     ↓
Playwright intercepts
     ↓
┌──────────────┬──────────────┬──────────────┐
│              │              │              │
continue()   fulfill()      abort()
│              │              │
Real API      Mock API      Block
│
└── continue(options)
        ↓
   Modified API request
```

### Remember

```text
click()                         → UI action / trigger
page.route()                    → Intercept
route.request()                 → Inspect request
route.request().headers()       → Get headers
route.request().postDataJSON()  → Get JSON body

route.continue()                → Send original request
route.continue({...})           → Modify and send request
route.fulfill()                 → Mock response
route.abort()                   → Block request
```
