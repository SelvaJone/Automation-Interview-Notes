# Playwright API Automation – Complete Guide

## 1. Introduction

This document explains the structure and concepts used in a Playwright JavaScript API Automation framework.

The framework uses:

* Playwright
* JavaScript
* Node.js
* npm
* REST APIs
* Custom Playwright fixtures
* Environment variables
* JSON test data
* Git
* GitHub
* GitHub Actions

---

# 2. Project Structure

A typical API automation project can look like this:

```text
APIAutomation/
│
├── tests/
│   └── 24MMMXVehicleCreation.spec.js
│
├── fixtures/
│   └── apiFixture.js
│
├── utils/
│   ├── testDataHelper.js
│   └── 24MMMXpayloadBuilder.js
│
├── test-data/
│   └── ...
│
├── output/
│   └── createdVins.json
│
├── playwright.config.js
├── package.json
├── package-lock.json
├── .env
├── .gitignore
└── README.md
```

---

# 3. What Is the Project?

The entire folder is the **Playwright project**.

For example:

```text
APIAutomation/
```

This is the project.

Inside the project, different folders have different responsibilities.

---

# 4. Test Directory

In Playwright configuration:

```js
testDir: './tests'
```

This tells Playwright:

> Look inside the `tests` folder for test files.

Example:

```text
APIAutomation/
└── tests/
    └── 24MMMXVehicleCreation.spec.js
```

Therefore:

* `APIAutomation` → project
* `tests` → test directory
* `24MMMXVehicleCreation.spec.js` → test file
* `test()` → individual test
* `test.describe()` → group of related tests

---

# 5. What Is a Test File?

A test file contains one or more test cases.

Example:

```js
test('Create vehicle', async () => {
    // test steps
});

test('Get vehicle', async () => {
    // test steps
});
```

A test file can contain multiple tests.

---

# 6. What Is test()?

`test()` defines an individual test case.

Example:

```js
test('POST Vehicle API', async () => {
    // test steps
});
```

Here:

```text
POST Vehicle API
```

is the test name.

---

# 7. What Is test.describe()?

`test.describe()` groups related tests together.

Example:

```js
test.describe('Vehicle API tests', () => {

    test('POST Vehicle API', async () => {
        // test
    });

    test('GET Vehicle API', async () => {
        // test
    });

});
```

Think of it like:

```text
Test Suite
│
├── Test 1
├── Test 2
└── Test 3
```

`describe()` is the container/group.

`test()` represents an individual test.

---

# 8. What Is a Test Suite?

A test suite is a collection of related test cases.

For example:

```text
24MM MX Vehicle Creation API Test Suite
│
├── POST 24MM MX ADF API
└── POST 24MM MX FDF API
```

In Playwright, this can be created using:

```js
test.describe('24MM MX Vehicle creation API tests', () => {

    test('POST 24MM MX ADF API', async () => {
    });

    test('POST 24MM MX FDF API', async () => {
    });

});
```

---

# 9. Serial Tests

Example:

```js
test.describe.serial('24MM MX Vehicle creation API tests', () => {

    test('POST ADF', async () => {
    });

    test('POST FDF', async () => {
    });

});
```

`serial` means the tests in that group are executed sequentially.

For example:

```text
ADF
 ↓
FDF
```

The second test does not start until the first test finishes.

This can be useful when one test depends on the previous test.

---

# 10. Playwright Configuration

Example:

```js
// @ts-check

import { defineConfig, devices } from '@playwright/test';
import dotenv from 'dotenv';

dotenv.config();

export default defineConfig({

    testDir: './tests',

    fullyParallel: true,

    forbidOnly: !!process.env.CI,

    retries: process.env.CI ? 2 : 0,

    workers: process.env.CI ? 1 : undefined,

    reporter: 'html',

    use: {
        trace: 'on-first-retry',
        baseURL: 'https://api.example.com',
    },

    projects: [
        {
            name: 'chromium',
            use: {
                ...devices['Desktop Chrome']
            }
        },

        {
            name: 'firefox',
            use: {
                ...devices['Desktop Firefox']
            }
        },

        {
            name: 'webkit',
            use: {
                ...devices['Desktop Safari']
            }
        },

        {
            name: 'api',
            use: {}
        }
    ]
});
```

---

# 11. testDir

```js
testDir: './tests'
```

This tells Playwright where the test files are located.

Example:

```text
APIAutomation/
└── tests/
    ├── login.spec.js
    ├── vehicle.spec.js
    └── subscription.spec.js
```

---

# 12. fullyParallel

```js
fullyParallel: true
```

This allows Playwright to run tests in parallel where possible.

Parallel execution can reduce total execution time.

Example:

```text
Test 1 ──────────>
Test 2 ──────────>
Test 3 ──────────>
```

Instead of:

```text
Test 1 ──>
          Test 2 ──>
                    Test 3 ──>
```

However, if tests have dependencies, use:

```js
test.describe.serial()
```

---

# 13. forbidOnly

```js
forbidOnly: !!process.env.CI
```

This is especially useful for CI/CD.

Suppose we have:

```js
test.only('POST Vehicle API', async () => {
});
```

Locally, Playwright will run only that test.

This is useful while debugging.

But accidentally pushing `.only` to GitHub can cause problems.

Therefore:

```js
forbidOnly: !!process.env.CI
```

means:

```text
Local:
CI = false
forbidOnly = false
test.only is allowed

GitHub Actions:
CI = true
forbidOnly = true
test.only is NOT allowed
```

If `.only` exists during CI, Playwright fails the test run instead of running only that test.

---

# 14. test.only

Example:

```js
test.only('POST Vehicle API', async () => {
});
```

This tells Playwright:

> Run only this test.

Useful for debugging.

Do not leave it in committed code.

---

# 15. Retries

Configuration:

```js
retries: process.env.CI ? 2 : 0
```

Means:

```text
Local:
0 retries

CI:
2 retries
```

If a test fails in CI, Playwright can retry it.

Example:

```text
Attempt 1 → Failed
Attempt 2 → Failed
Attempt 3 → Passed
```

---

# 16. Workers

Configuration:

```js
workers: process.env.CI ? 1 : undefined
```

This means:

```text
CI:
1 worker

Local:
Playwright decides the number of workers
```

A worker is a process used by Playwright to execute tests.

---

# 17. Reporter

Example:

```js
reporter: 'html'
```

Playwright generates an HTML report.

After execution:

```bash
npx playwright show-report
```

This opens the Playwright HTML report.

---

# 18. Trace

Configuration:

```js
trace: 'on-first-retry'
```

Playwright collects a trace when a test is retried for the first time.

Trace information can help debug:

* API requests
* UI actions
* errors
* screenshots
* timing
* test execution

---

# 19. baseURL

Example:

```js
use: {
    baseURL: 'https://api.example.com'
}
```

Instead of writing:

```js
await request.post('https://api.example.com/v1/vehicle');
```

we can potentially use:

```js
await request.post('/v1/vehicle');
```

The base URL provides the common starting point.

---

# 20. Projects

Playwright allows multiple projects.

Example:

```js
projects: [

    {
        name: 'chromium',
        use: {
            ...devices['Desktop Chrome']
        }
    },

    {
        name: 'firefox',
        use: {
            ...devices['Desktop Firefox']
        }
    },

    {
        name: 'webkit',
        use: {
            ...devices['Desktop Safari']
        }
    },

    {
        name: 'api',
        use: {}
    }

]
```

Projects allow us to separate execution configurations.

Example:

```text
chromium → UI tests
firefox  → UI tests
webkit   → UI tests
api      → API tests
```

---

# 21. Running a Specific Project

Run API tests:

```bash
npx playwright test --project=api
```

Run Chromium tests:

```bash
npx playwright test --project=chromium
```

Run Firefox:

```bash
npx playwright test --project=firefox
```

---

# 22. API Testing with Playwright

Playwright supports API testing through request contexts.

Example:

```js
const response = await apiCallContext.post('/v1/vehicle', {
    data: payload
});
```

The request sends:

```text
HTTP POST
```

to:

```text
/v1/vehicle
```

with:

```text
payload
```

as the request body.

---

# 23. HTTP Methods

Common HTTP methods:

```text
GET
POST
PUT
PATCH
DELETE
```

## GET

Retrieve information.

```js
const response = await api.get('/vehicles');
```

## POST

Create information.

```js
const response = await api.post('/vehicles', {
    data: payload
});
```

## PUT

Replace/update information.

```js
const response = await api.put('/vehicles/123', {
    data: payload
});
```

## PATCH

Partially update information.

```js
const response = await api.patch('/vehicles/123', {
    data: payload
});
```

## DELETE

Delete information.

```js
const response = await api.delete('/vehicles/123');
```

---

# 24. API Response

Example:

```js
const response = await apiCallContext.post(endpoint, {
    data: payload
});
```

We can check the HTTP status:

```js
console.log(response.status());
```

Example:

```text
201
```

---

# 25. Status Code Validation

Instead of checking only one status:

```js
expect(response.status()).toBe(201);
```

we can validate a successful 2xx response:

```js
expect(response.status()).toBeGreaterThanOrEqual(200);
expect(response.status()).toBeLessThan(300);
```

This accepts successful responses such as:

```text
200
201
202
204
```

---

# 26. Response Body

Get the response as text:

```js
const body = await response.text();

console.log(body);
```

For JSON:

```js
const body = await response.json();

console.log(body);
```

Example:

```js
expect(body.status).toBe('SUCCESS');
```

---

# 27. Request Payload

A payload is the data sent to an API.

Example:

```js
const payload = {
    vin: '123456789',
    model: 'Prius',
    year: 2027
};
```

Send it:

```js
const response = await api.post('/vehicle', {
    data: payload
});
```

---

# 28. JSON Test Data

Instead of hardcoding data inside the test, test data can be stored separately.

Example:

```text
test-data/
└── adfdata.json
```

Example JSON:

```json
[
    {
        "model": "Prius",
        "year": 2027
    },
    {
        "model": "RAV4",
        "year": 2027
    }
]
```

The test reads the data and creates payloads.

This makes the framework data-driven.

---

# 29. Data-Driven Testing

Example:

```js
const records = readTestData(
    Generation,
    Region,
    Brand,
    'adfdata.json'
);

for (const record of records) {

    const payload = buildPayload(record);

    const response = await api.post('/vehicle', {
        data: payload
    });

}
```

If there are 10 records:

```text
Record 1 → API
Record 2 → API
Record 3 → API
...
Record 10 → API
```

---

# 30. Utility Functions

Utility functions contain reusable logic.

Example:

```text
utils/
├── testDataHelper.js
└── payloadBuilder.js
```

Instead of writing payload creation logic inside every test, we can create:

```js
function buildVehiclePayload(record) {
    return {
        vin: record.vin,
        model: record.model
    };
}
```

Then:

```js
const payload = buildVehiclePayload(record);
```

Benefits:

* Reusable
* Easier maintenance
* Cleaner tests
* Less duplicate code

---

# 31. Custom Fixtures

A fixture provides reusable setup or objects to tests.

Example:

```js
const { test } = require('../fixtures/apiFixture');
```

Then:

```js
test('POST Vehicle API', async ({ authSetup }) => {

    const {
        apiCallContext,
        Generation,
        Region,
        Brand
    } = authSetup;

});
```

The fixture provides:

```text
authSetup
│
├── Generation
├── Region
├── Brand
├── apiCallContext
├── endpoint information
└── authentication/setup
```

---

# 32. Why Use Fixtures?

Without fixtures, every test might need to create:

```text
API context
Authentication
Environment
Headers
Tokens
Endpoints
Configuration
```

Fixtures allow us to centralize this setup.

Example:

```text
Fixture
   │
   ├── Authentication
   ├── API context
   ├── Environment
   └── Endpoints
          │
          ↓
       Test Case
```

This makes tests easier to maintain.

---

# 33. Environment Variables

Environment variables allow us to change configuration without changing source code.

Example:

```env
Generation=17cyplus
Region=US
Brand=LEX
```

Read them in JavaScript:

```js
process.env.Generation
process.env.Region
process.env.Brand
```

Example:

```js
const generation = process.env.Generation;
```

---

# 34. dotenv

Install:

```bash
npm install dotenv
```

Load `.env`:

```js
import dotenv from 'dotenv';

dotenv.config();
```

Then:

```js
process.env.Generation
```

can access the value.

---

# 35. Why Use Environment Variables?

Suppose we have:

```text
DEV
QA
STAGE
PROD
```

We do not want to modify test code every time.

Instead:

```text
Environment
     ↓
Environment Variables
     ↓
Playwright
     ↓
Tests
```

For example:

```text
QA:
BASE_URL=https://qa.example.com

STAGE:
BASE_URL=https://stage.example.com
```

---

# 36. Different Environment Files

A framework may use:

```text
.env.qa
.env.stage
.env.prod
```

Example:

```text
.env.qa
BASE_URL=https://qa.example.com

.env.stage
BASE_URL=https://stage.example.com
```

However, secrets should not be committed to GitHub.

---

# 37. GitHub Secrets

For CI/CD, sensitive information should be stored in GitHub Secrets.

Examples:

```text
API_USERNAME
API_PASSWORD
API_TOKEN
CLIENT_SECRET
```

GitHub Actions can access them using:

```yaml
${{ secrets.API_TOKEN }}
```

Do not hardcode:

```js
const token = 'my-secret-token';
```

---

# 38. Test Types

Common test types:

```text
Smoke
Sanity
Regression
Integration
API
UI
End-to-End
```

## Smoke

Small set of critical tests.

Purpose:

> Is the application basically working?

Example:

```text
Login
Create vehicle
Get vehicle
```

## Sanity

Focused testing of a particular change or feature.

## Regression

Larger test suite covering existing functionality.

Example:

```text
All vehicle APIs
All subscription APIs
All payment APIs
All major workflows
```

---

# 39. Running Different Test Types

One approach is to use tags.

Example:

```js
test('Create vehicle', {
    tag: '@smoke'
}, async () => {
});
```

Then run:

```bash
npx playwright test --grep @smoke
```

Regression:

```bash
npx playwright test --grep @regression
```

Sanity:

```bash
npx playwright test --grep @sanity
```

---

# 40. Recommended Test Organization

Example:

```text
tests/
│
├── smoke/
│   ├── vehicleSmoke.spec.js
│   └── subscriptionSmoke.spec.js
│
├── sanity/
│   └── vehicleSanity.spec.js
│
└── regression/
    ├── vehicleRegression.spec.js
    ├── subscriptionRegression.spec.js
    └── paymentRegression.spec.js
```

Another approach is to keep tests organized by feature and use tags.

Example:

```text
tests/
├── vehicle/
├── subscription/
├── payment/
└── customer/
```

---

# 41. Complete API Test Example

```js
const { expect } = require('@playwright/test');

const {
    readTestData
} = require('../utils/testDataHelper');

const {
    buildVehiclePayload
} = require('../utils/payloadBuilder');

const {
    test
} = require('../fixtures/apiFixture');


test.describe.serial('Vehicle API tests', () => {

    test('POST Vehicle API', async ({ authSetup }) => {

        const {
            Generation,
            Region,
            Brand,
            apiCallContext,
            endpoint
        } = authSetup;

        const records = readTestData(
            Generation,
            Region,
            Brand,
            'data.json'
        );

        for (const record of records) {

            const payload =
                buildVehiclePayload(record);

            console.log(
                'Request:',
                JSON.stringify(payload, null, 2)
            );

            const response =
                await apiCallContext.post(endpoint, {
                    data: payload
                });

            console.log(
                'Status:',
                response.status()
            );

            console.log(
                'Body:',
                await response.text()
            );

            expect(response.status())
                .toBeGreaterThanOrEqual(200);

            expect(response.status())
                .toBeLessThan(300);
        }
    });

});
```

---

# 42. Understanding the Test Flow

The test execution flow is:

```text
Playwright starts
      ↓
playwright.config.js
      ↓
Environment variables loaded
      ↓
Project selected
      ↓
Test file discovered
      ↓
Fixture executed
      ↓
Authentication/setup
      ↓
Test data loaded
      ↓
Payload created
      ↓
API request sent
      ↓
Response received
      ↓
Status validated
      ↓
Response validated
      ↓
Test result
```

---

# 43. Local Execution

Install dependencies:

```bash
npm install
```

or:

```bash
npm ci
```

Run all tests:

```bash
npx playwright test
```

Run API tests:

```bash
npx playwright test --project=api
```

Run one file:

```bash
npx playwright test tests/vehicle.spec.js
```

Run one test:

```bash
npx playwright test tests/vehicle.spec.js -g "POST Vehicle API"
```

---

# 44. Debugging

Run Playwright in debug mode:

```bash
npx playwright test --debug
```

For UI tests, you can also use:

```js
await page.pause();
```

For API testing, use:

```js
console.log(response.status());
console.log(await response.text());
```

Also log the request payload:

```js
console.log(JSON.stringify(payload, null, 2));
```

---

# 45. Git

Git is used for source-code version control.

Basic flow:

```text
Working Directory
       ↓
git add
       ↓
Staging Area
       ↓
git commit
       ↓
Local Repository
       ↓
git push
       ↓
GitHub
```

---

# 46. Initialize Git

From the project directory:

```bash
git init
```

Check status:

```bash
git status
```

---

# 47. .gitignore

Create:

```text
.gitignore
```

Example:

```gitignore
node_modules/
.env
.env.*
playwright-report/
test-results/
output/
```

This prevents files from being committed accidentally.

---

# 48. Why node_modules Should Not Be Committed

`node_modules` can contain thousands of files.

Instead of committing it, commit:

```text
package.json
package-lock.json
```

Anyone can then run:

```bash
npm ci
```

to recreate `node_modules`.

---

# 49. Git Add

Add project files:

```bash
git add .
```

Check:

```bash
git status
```

---

# 50. Git Commit

Create a commit:

```bash
git commit -m "Initial Playwright API automation framework"
```

---

# 51. Create GitHub Repository

Create a repository in GitHub.

Example:

```text
APIAutomation
```

Then connect the local project:

```bash
git remote add origin <repository-url>
```

Check:

```bash
git remote -v
```

---

# 52. Push to GitHub

Set main branch:

```bash
git branch -M main
```

Push:

```bash
git push -u origin main
```

After this, the project is available in the GitHub repository.

---

# 53. Important Security Rule

Never commit sensitive information such as:

```text
Passwords
API tokens
Access tokens
Client secrets
Private keys
Production credentials
Sensitive company data
```

Do not do this:

```js
const token = '123456-secret-token';
```

Instead use environment variables:

```js
const token = process.env.API_TOKEN;
```

---

# 54. Company Code and GitHub

If the automation project contains company source code, APIs, test data, credentials, internal URLs, or proprietary information:

> Check your company's security and repository policy before pushing it to a personal GitHub repository.

A private GitHub repository does not automatically mean company data is approved for external storage.

---

# 55. GitHub Actions

GitHub Actions is a CI/CD automation platform.

It can automatically execute tests when:

```text
Code is pushed
       ↓
GitHub Actions starts
       ↓
Node.js installed
       ↓
Dependencies installed
       ↓
Playwright tests executed
       ↓
Results generated
```

---

# 56. GitHub Actions Workflow

Create:

```text
.github/
└── workflows/
    └── playwright.yml
```

Example:

```yaml
name: Playwright API Tests

on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

jobs:

  test:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 24

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run API tests
        run: npx playwright test --project=api
```

---

# 57. What Does npm ci Do?

```bash
npm ci
```

means:

> Install the dependencies defined in `package-lock.json`.

It is commonly used in CI/CD.

The flow is:

```text
package.json
      +
package-lock.json
      ↓
npm ci
      ↓
node_modules
```

---

# 58. Where Is the Application Code?

This is an important distinction.

In an API automation project, your GitHub repository usually contains the:

```text
Automation code
```

It does not necessarily contain the actual application/API implementation.

For example:

```text
Application
    ↓
Remote server/API
    ↓
https://api.example.com

Automation project
    ↓
Playwright tests
    ↓
API requests
```

The Playwright project sends requests to the deployed application.

---

# 59. Environment in GitHub Actions

The environment can be configured using environment variables.

Example:

```yaml
env:
  BASE_URL: https://qa.example.com
  Generation: 17cyplus
  Region: US
  Brand: LEX
```

Then JavaScript can read:

```js
process.env.BASE_URL
process.env.Generation
process.env.Region
process.env.Brand
```

---

# 60. QA and STAGE Environments

We can maintain different configurations.

Example:

```text
QA
│
├── BASE_URL
├── Generation
├── Region
└── Brand

STAGE
│
├── BASE_URL
├── Generation
├── Region
└── Brand
```

The test code remains the same.

Only the environment configuration changes.

---

# 61. Example GitHub Actions QA Environment

```yaml
name: Playwright QA Tests

on:
  workflow_dispatch:

jobs:

  test:

    runs-on: ubuntu-latest

    env:
      BASE_URL: https://qa.example.com
      Generation: 17cyplus
      Region: US
      Brand: LEX

    steps:

      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 24

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright
        run: npx playwright install --with-deps

      - name: Run API tests
        run: npx playwright test --project=api
```

---

# 62. Using GitHub Secrets

For sensitive values:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

Then JavaScript:

```js
const token = process.env.API_TOKEN;
```

The secret should be stored in:

```text
GitHub
   ↓
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
```

---

# 63. Test Type + Environment

A workflow can support both environment and test type.

Example:

```yaml
workflow_dispatch:

  inputs:

    environment:
      description: "Environment"
      required: true
      default: "qa"

    testType:
      description: "Test Type"
      required: true
      default: "smoke"
```

Conceptually:

```text
Environment
    ↓
QA / STAGE

Test Type
    ↓
Smoke / Sanity / Regression

        ↓

Playwright
        ↓
Tests
```

---

# 64. Example Test Type Commands

Smoke:

```bash
npx playwright test --grep @smoke
```

Sanity:

```bash
npx playwright test --grep @sanity
```

Regression:

```bash
npx playwright test --grep @regression
```

API project:

```bash
npx playwright test --project=api
```

API smoke:

```bash
npx playwright test --project=api --grep @smoke
```

---

# 65. Recommended Framework Architecture

A clean API automation framework can follow:

```text
APIAutomation/
│
├── tests/
│   ├── smoke/
│   ├── sanity/
│   └── regression/
│
├── fixtures/
│   └── apiFixture.js
│
├── utils/
│   ├── testDataHelper.js
│   ├── payloadBuilder.js
│   └── logger.js
│
├── test-data/
│   ├── qa/
│   └── stage/
│
├── output/
│
├── .github/
│   └── workflows/
│       └── playwright.yml
│
├── playwright.config.js
├── package.json
├── package-lock.json
├── .gitignore
└── README.md
```

---

# 66. Framework Responsibilities

## tests/

Contains test scenarios.

```text
What are we testing?
```

---

## fixtures/

Contains reusable setup.

```text
How do we prepare the test?
```

Examples:

```text
Authentication
API context
Headers
Environment
Endpoints
```

---

## utils/

Contains reusable helper functions.

```text
What logic can be reused?
```

Examples:

```text
Payload builders
Test-data readers
Logging
Common functions
```

---

## test-data/

Contains external test data.

```text
What data should we use?
```

---

## playwright.config.js

Contains framework-level configuration.

```text
How should Playwright run?
```

---

## .github/workflows/

Contains CI/CD workflows.

```text
When and where should the tests run?
```

---

# 67. Complete Execution Architecture

The overall architecture can be visualized as:

```text
                 GitHub Actions
                       │
                       ↓
              Playwright Config
                       │
                       ↓
                  Environment
                       │
                       ↓
                   Fixtures
                       │
          ┌────────────┴────────────┐
          ↓                         ↓
      Test Data                Authentication
          │                         │
          └────────────┬────────────┘
                       ↓
                 Payload Builder
                       │
                       ↓
                  API Request
                       │
                       ↓
                Application API
                       │
                       ↓
                  API Response
                       │
                       ↓
                 Assertions
                       │
                       ↓
                  Test Result
                       │
                       ↓
                HTML Report
```

---

# 68. Important Interview Questions

## What is Playwright?

Playwright is an automation framework for testing web applications and APIs.

It supports:

* Chromium
* Firefox
* WebKit
* API testing
* Network interception
* Multiple tabs
* Frames
* File upload/download
* Parallel execution
* Trace viewer
* CI/CD

---

## What is a fixture?

A fixture provides reusable test setup and resources.

Example:

```js
test('API test', async ({ authSetup }) => {
});
```

Here `authSetup` is a custom fixture.

---

## What is test.describe()?

It groups related test cases.

```js
test.describe('Vehicle APIs', () => {
});
```

---

## What is test.describe.serial()?

It runs tests in the group sequentially.

```js
test.describe.serial('Vehicle APIs', () => {
});
```

---

## What is test.only()?

It runs only the selected test locally.

```js
test.only('Create vehicle', async () => {
});
```

It should not be committed to CI.

---

## What is forbidOnly?

```js
forbidOnly: !!process.env.CI
```

It prevents accidental `.only` usage in CI.

---

## What is baseURL?

A common URL used as the starting point for requests/pages.

```js
baseURL: 'https://api.example.com'
```

---

## What is npm ci?

It installs dependencies using the lock file.

```bash
npm ci
```

It is commonly used in CI/CD.

---

## What is GitHub Actions?

A CI/CD platform that automatically runs workflows such as:

```text
Build
Test
Report
Deploy
```

---

# 69. Important Commands

Install dependencies:

```bash
npm install
```

CI installation:

```bash
npm ci
```

Run all tests:

```bash
npx playwright test
```

Run API tests:

```bash
npx playwright test --project=api
```

Run smoke tests:

```bash
npx playwright test --grep @smoke
```

Debug:

```bash
npx playwright test --debug
```

Open report:

```bash
npx playwright show-report
```

Git status:

```bash
git status
```

Git add:

```bash
git add .
```

Git commit:

```bash
git commit -m "Add API automation tests"
```

Git push:

```bash
git push
```

---

# 70. Final Framework Concept

The most important concept to remember is:

```text
CONFIG
   ↓
ENVIRONMENT
   ↓
FIXTURE
   ↓
TEST DATA
   ↓
PAYLOAD
   ↓
API REQUEST
   ↓
API RESPONSE
   ↓
ASSERTION
   ↓
REPORT
```

And for CI/CD:

```text
Developer
    ↓
Git
    ↓
GitHub
    ↓
GitHub Actions
    ↓
npm ci
    ↓
Playwright
    ↓
API Tests
    ↓
Results
    ↓
Report
```

---

# 71. Key Takeaways

### Project

```text
APIAutomation
```

is the overall automation project.

### Test directory

```text
tests/
```

contains test files.

### Test file

```text
*.spec.js
```

contains test cases.

### Test suite/group

```js
test.describe()
```

groups related tests.

### Individual test

```js
test()
```

defines one test scenario.

### Fixture

Provides reusable setup and resources.

### Utility

Contains reusable helper logic.

### Test data

Contains data used by the tests.

### Configuration

Controls how Playwright executes.

### Environment variables

Allow the same code to run against different environments.

### Git

Tracks code changes.

### GitHub

Stores the repository.

### GitHub Actions

Runs automation automatically in CI/CD.

---

# 72. Recommended Learning Order

For Playwright API automation, learn in this order:

```text
1. JavaScript basics
        ↓
2. Playwright basics
        ↓
3. API testing
        ↓
4. Request / Response
        ↓
5. Assertions
        ↓
6. Test data
        ↓
7. Fixtures
        ↓
8. Environment variables
        ↓
9. Playwright config
        ↓
10. Tags
        ↓
11. Smoke / Sanity / Regression
        ↓
12. Parallel execution
        ↓
13. Debugging
        ↓
14. Trace
        ↓
15. Git
        ↓
16. GitHub
        ↓
17. GitHub Actions
        ↓
18. CI/CD
```

---

# 73. One-Line Interview Summary

> I use Playwright with JavaScript for API automation, where tests are organized by feature and test type, reusable setup is handled through custom fixtures, test data and payload creation are separated into utilities, environment-specific values are managed through environment variables, and the tests can be executed locally or through GitHub Actions as part of CI/CD.
