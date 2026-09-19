No file chosen
Library



Search library

New


Suggested

Folders

Images

All

Name
Last activity


Playwright-Fixtures.md
Modified 1m ago



Playwright-Authentication-storageState.md
Modified 5m ago



Pasted markdown(1)
Modified 2w ago



Pasted markdown
Modified 2w ago



50a2d3e0-a536-4160-9233-0206cbdd79c6.mp3
Modified 3w ago



Executive_Bio_Functional_Validation_Engineer.docx
Modified 3w ago



Playwright_Interview_Preparation_Master_Guide.docx
Modified 3w ago



JavaScript_Coding_Master_Guide.txt
Modified 4w ago



JavaScript_Coding_Master_Guide.txt
Modified 4w ago



Java_Coding_Exercises.txt
Modified 4w ago



Java_Coding_Exercises.txt
Modified 4w ago



JavaScript-Coding-Interview-Master-Guide.docx
Modified 1mo ago




image-gen-1.png
Modified 1mo ago



Modified 1mo ago




image(20260806-191418).png
Modified 1mo ago




image(20260806-191406).png
Modified 1mo ago




image(20260806-191231).png
Modified 1mo ago




image(20260806-191047).png
Modified 1mo ago




image(22).png
Modified 1mo ago




image(21).png
Modified 1mo ago


Library
/
Playwright-Fixtures

Share
Playwright Fixtures
1. What is a Fixture?
A fixture is a reusable, ready-to-use object or setup provided by
Playwright Test.

Fixtures help us prepare the environment required by a test and clean it
up after the test finishes.

Examples of built-in Playwright fixtures:

page

browser

context

request

2. Built-in Fixtures
page
Represents a browser tab/page.

Example:

test('open page', async ({ page }) => {

    await page.goto('https://www.saucedemo.com/');

});
browser
Represents the browser instance.

test('browser test', async ({ browser }) => {

    const context = await browser.newContext();

});
context
Represents an isolated browser session.

test('context test', async ({ context }) => {

    const page = await context.newPage();

});
request
Used for API testing.

test('API test', async ({ request }) => {

    const response = await request.get('https://example.com/api/users');

});
3. Fixture Injection
Playwright automatically provides fixtures when we request them in the
test function.

Example:

test('my test', async ({ page }) => {

    await page.goto('https://www.saucedemo.com/');

});
Here:

{ page }
means Playwright provides the page fixture to the test.

This is called fixture injection.

4. Fixture Lifecycle
A fixture generally follows this lifecycle:

Start test
    ↓
Create required fixtures
    ↓
Fixture setup
    ↓
Give fixture to test
    ↓
Run test
    ↓
Fixture cleanup
    ↓
Test finished
5. Why Use Custom Fixtures?
Built-in fixtures are useful, but real automation frameworks often need
custom setup.

For example:

Login

Test data creation

Database setup

API authentication

Common navigation

Logging

Cleanup

Instead of repeating these steps in every test, we can create a custom
fixture.

6. Creating a Custom Fixture
Create a fixture file at the project root:

playwright-fixtures-practice/
├── fixtures.js
├── playwright.config.js
├── package.json
├── package-lock.json
└── tests/
    └── fixture.spec.js
Example fixtures.js:

import { test as base } from '@playwright/test';

export const test = base.extend({

    myFixture: async ({ page }, use) => {

        console.log('Fixture setup');

        await use(page);

        console.log('Fixture cleanup');
    }
});
7. Understanding base.extend()
This:

base.extend()
allows us to create or extend Playwright fixtures.

Example:

export const test = base.extend({
We are creating our own version of Playwright's test function with a
custom fixture.

8. Using the Custom Fixture
Test file:

import { test } from '../fixtures.js';

test('custom fixture test', async ({ myFixture }) => {

    console.log('Test is running');

});
The test requests:

{ myFixture }
Playwright creates the fixture and provides it to the test.

9. Understanding use()
This is one of the most important fixture concepts.

myFixture: async ({ page }, use) => {

    console.log('Fixture setup');

    await use(page);

    console.log('Fixture cleanup');
}
Think of:

await use(page);
as:

Give page to the test.

The code before use() is setup.

The code after use() is cleanup.

Fixture starts
     ↓
Setup code
     ↓
await use(page)
     ↓
Test runs
     ↓
Cleanup code
10. Custom Fixture Example
import { test as base } from '@playwright/test';

export const test = base.extend({

    myFixture: async ({ page }, use) => {

        console.log('=== Fixture setup ===');

        await page.goto('https://www.saucedemo.com/');

        await use(page);

        console.log('=== Fixture cleanup ===');
    }
});
Test:

import { test } from '../fixtures.js';

test('custom fixture test', async ({ myFixture }) => {

    console.log('=== Test is running ===');

});
The fixture navigates to the application before the test starts.

11. Fixture Dependency
A custom fixture can depend on another fixture.

For example:

page
 ↓
myFixture
 ↓
loginFixture
 ↓
Test
Example:

loginFixture: async ({ myFixture }, use) => {

    await myFixture.getByPlaceholder('Username')
        .fill('standard_user');

    await myFixture.getByPlaceholder('Password')
        .fill('secret_sauce');

    await myFixture.getByRole('button', { name: 'Login' })
        .click();

    await use(myFixture);
}
Here:

{ myFixture }
is passed into loginFixture.

12. Fixture Dependency Chain
The dependency chain is:

page
 ↓
myFixture
 ↓
loginFixture
 ↓
Test
Playwright creates the dependencies in the correct order.

13. Setup and Cleanup Order
Suppose we have:

page
 ↓
myFixture
 ↓
loginFixture
 ↓
Test
Setup happens in dependency order:

myFixture setup
       ↓
loginFixture setup
       ↓
Test
Cleanup happens in reverse order:

Test
 ↓
loginFixture cleanup
 ↓
myFixture cleanup
This is important when fixtures depend on one another.

14. Automatic Fixtures
A fixture normally runs only when a test requests it.

For example:

test('example', async ({ myFixture }) => {
});
But we can create an automatic fixture.

Example:

logger: [async ({}, use) => {

    console.log('=== Before Test ===');

    await use();

    console.log('=== After Test ===');

}, { auto: true }]
The important part is:

{ auto: true }
This means the fixture runs automatically without the test explicitly
requesting it.

15. Normal Fixture vs Automatic Fixture
Normal fixture
test('test', async ({ myFixture }) => {
});
The test must request:

{ myFixture }
Automatic fixture
{ auto: true }
The fixture runs automatically.

Test starts
   ↓
Automatic fixture setup
   ↓
Test runs
   ↓
Automatic fixture cleanup
16. Example of an Automatic Logger
import { test as base } from '@playwright/test';

export const test = base.extend({

    logger: [async ({}, use) => {

        console.log('=== Before Test ===');

        await use();

        console.log('=== After Test ===');

    }, { auto: true }]
});
Now every test using this custom test automatically gets the logger
behavior.

17. Test vs Worker
You may see output like:

Running 3 tests using 3 workers
Test
A test is an individual test case.

Example:

test('login test', async ({ page }) => {
});
That is one test.

Worker
A worker is a Playwright process that executes tests.

If Playwright says:

Running 3 tests using 3 workers
it means Playwright can execute three test tasks in parallel using three
worker processes.

18. Parallel Execution
Multiple workers allow tests to run in parallel.

For example:

Worker 1 → Test 1
Worker 2 → Test 2
Worker 3 → Test 3
This can reduce total execution time.

In your fixture practice project, you may also see tests executed
across:

Chromium
Firefox
WebKit
when those projects are configured in playwright.config.js.

19. Complete Fixture Flow
A simplified flow looks like this:

Test starts
     ↓
Playwright creates required fixtures
     ↓
Fixture setup
     ↓
Fixture dependency setup
     ↓
Test receives fixture
     ↓
Test executes
     ↓
Fixture cleanup
     ↓
Test finishes
For dependent fixtures:

page
 ↓
myFixture setup
 ↓
loginFixture setup
 ↓
Test
 ↓
loginFixture cleanup
 ↓
myFixture cleanup
20. Built-in vs Custom Fixtures
Built-in Fixture Custom Fixture

Provided by Playwright Created by us
page myFixture
browser loginFixture
context logger
request testData
Ready to use Designed for project needs

21. Why Fixtures Are Useful in Real Projects
Fixtures help keep tests clean.

Instead of:

test('test 1', async ({ page }) => {

    // login
    // navigate
    // create data
    // test
    // cleanup
});
We can move common setup into fixtures:

Fixture
 ↓
Login
 ↓
Test data
 ↓
Test
Then the actual test focuses on the behavior being tested.

22. Interview Question
What are fixtures in Playwright?
Good answer:

Fixtures are reusable, ready-to-use objects provided by Playwright
Test that handle setup and cleanup for test dependencies. Examples
include page, browser, context, and request.

23. Interview Question
What is base.extend()?
Answer:

base.extend() is used to create or extend Playwright fixtures. It
allows us to define custom fixtures for common setup, test data,
authentication, logging, and cleanup.

24. Interview Question
What is use() in a fixture?
Answer:

use() passes the fixture value to the test. Code before use() is
generally used for setup, while code after use() is used for cleanup
or teardown.

25. Interview Question
What is an automatic fixture?
Answer:

An automatic fixture is a fixture configured with auto: true. It
runs automatically for the applicable tests without the test
explicitly requesting the fixture.

26. Interview Question
Why use custom fixtures?
Answer:

Custom fixtures allow us to centralize reusable setup and cleanup
logic, such as login, test data creation, API authentication, logging,
and database preparation. This reduces duplicate code and makes tests
easier to maintain.

27. Key Concepts to Remember
Remember these concepts:

Built-in fixture
      ↓
Custom fixture
      ↓
base.extend()
      ↓
use()
      ↓
Setup
      ↓
Test
      ↓
Cleanup
Important examples:

{ page }
base.extend()
await use(page)
{ auto: true }
28. Quick Interview Summary
If asked about Playwright fixtures, remember:

Fixtures provide reusable test dependencies.

Playwright has built-in fixtures such as page, browser,
context, and request.

Custom fixtures are created using base.extend().

use() passes the fixture to the test.

Code before use() is setup; code after use() is cleanup.

Fixtures can depend on other fixtures.

auto: true creates an automatic fixture.

Workers execute tests, and multiple workers enable parallel
execution.

Overall pattern:

Fixture
   ↓
Setup
   ↓
Provide dependency
   ↓
Test
   ↓
Cleanup

