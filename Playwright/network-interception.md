# Playwright Network Interception

## 1. What is Network Interception?

**Network interception** means Playwright catches a network request before it reaches the server.

This allows us to:

* Inspect requests
* Continue requests to the real server
* Mock API responses
* Modify requests/responses
* Block requests
* Simulate server errors

### Simple Flow

```text
Application
     |
     | API Request
     ↓
Playwright intercepts request
     |
     ├── continue() → Real Server
     |
     ├── fulfill()  → Mock Response
     |
     └── abort()    → Block Request
```

### Simple Definition

> Playwright catches the request before it reaches the server, so we can decide what happens to it.

---

# 2. `page.route()`

The main Playwright method used for network interception is:

```javascript
await page.route();
```

### General Structure

```javascript
await page.route(
    'URL pattern',
    async route => {
        // What to do with the intercepted request
    }
);
```

There are two important parts:

```javascript
page.route(
    'URL pattern',
    callback
);
```

### URL Pattern

Example:

```javascript
'**/products'
```

This tells Playwright which requests to intercept.

### Callback

```javascript
async route => {
    // actions
}
```

The callback runs when Playwright intercepts a matching request.

`route` is the object representing the intercepted network request.

---

# 3. Inspect the Request

We can inspect information about the intercepted request.

### Get URL

```javascript
route.request().url()
```

### Get HTTP Method

```javascript
route.request().method()
```

Example:

```javascript
await page.route(
    '**/products',
    async route => {
        console.log(route.request().url());
        console.log(route.request().method());

        await route.continue();
    }
);
```

If the application sends:

```text
GET https://example.com/products
```

Playwright can print:

```text
https://example.com/products
GET
```

---

# 4. `route.continue()`

`route.continue()` allows the intercepted request to continue to the real server.

```javascript
await route.continue();
```

### Example

```javascript
await page.route(
    '**/products',
    async route => {
        console.log(route.request().url());
        console.log(route.request().method());

        await route.continue();
    }
);
```

### Flow

```text
Application
     ↓
API Request
     ↓
Playwright intercepts
     ↓
route.continue()
     ↓
Real Server
     ↓
Real Response
```

### When to use it?

Use `continue()` when you want to:

* Monitor requests
* Log requests
* Inspect requests
* Allow the real API call

---

# 5. `route.fulfill()`

`route.fulfill()` allows us to provide our own response instead of calling the real server.

This is commonly used for **mocking API responses**.

### Example

```javascript
await page.route(
    '**/products',
    async route => {
        await route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify({
                products: [
                    {
                        id: 1,
                        name: 'Laptop',
                        price: 1000
                    }
                ]
            })
        });
    }
);
```

The real API is not called.

Playwright returns our mocked response to the application.

### Flow

```text
Application
     ↓
API Request
     ↓
Playwright intercepts
     ↓
route.fulfill()
     ↓
Mock Response
     ↓
Application
```

---

# 6. Mocking a Simple Response

Example:

```javascript
await page.route(
    '**/products',
    async route => {
        await route.fulfill({
            status: 200,
            contentType: 'application/json',
            body: JSON.stringify({
                products: [
                    {
                        id: 1,
                        name: 'Phone'
                    }
                ]
            })
        });
    }
);
```

The application receives:

```json
{
    "products": [
        {
            "id": 1,
            "name": "Phone"
        }
    ]
}
```

---

# 7. Mocking a Server Error

We can also simulate server errors.

For example, HTTP `500 Internal Server Error`:

```javascript
await page.route(
    '**/products',
    async route => {
        await route.fulfill({
            status: 500,
            body: JSON.stringify({
                err: 'Internal Server Error'
            })
        });
    }
);
```

This is useful for testing how the application behaves when the backend is unavailable.

### Example Test Scenario

```text
User opens Products
        ↓
Application calls Products API
        ↓
Playwright intercepts request
        ↓
Playwright returns HTTP 500
        ↓
Application should show error message
```

We can then verify the UI:

```javascript
await expect(
    page.locator('#error-message')
).toBeVisible();
```

---

# 8. `route.abort()`

`route.abort()` blocks the network request.

```javascript
await route.abort();
```

### Example

```javascript
await page.route(
    '**/analytics',
    async route => {
        await route.abort();
    }
);
```

The request will not reach the server.

### Flow

```text
Application
     ↓
API Request
     ↓
Playwright intercepts
     ↓
route.abort()
     ↓
Request blocked
```

---

# 9. Blocking Ads

A common real-world example is blocking advertisement requests.

```javascript
await page.route(
    '**/ads/**',
    async route => {
        await route.abort();
    }
);
```

Any matching request will be blocked.

---

# 10. `continue()` vs `fulfill()` vs `abort()`

| Method             | Purpose                     |
| ------------------ | --------------------------- |
| `route.continue()` | Send request to real server |
| `route.fulfill()`  | Return a mocked response    |
| `route.abort()`    | Block the request           |

### Easy Memory Trick

```text
continue() → Let it go
fulfill()  → Give my response
abort()    → Stop it
```

---

# 11. Complete Example

```javascript
import { test, expect } from '@playwright/test';

test('mock products API', async ({ page }) => {

    await page.route(
        '**/products',
        async route => {

            await route.fulfill({
                status: 200,
                contentType: 'application/json',
                body: JSON.stringify({
                    products: [
                        {
                            id: 1,
                            name: 'Laptop',
                            price: 1000
                        },
                        {
                            id: 2,
                            name: 'Phone',
                            price: 500
                        }
                    ]
                })
            });

        }
    );

    await page.goto('https://example.com');

    // Continue UI test here
});
```

The Products API response is completely controlled by the test.

---

# 12. Why Network Interception is Useful

Network interception is useful when:

### 1. Backend is unavailable

You can mock the API response.

### 2. Need to test error scenarios

For example:

```text
500 Server Error
404 Not Found
401 Unauthorized
403 Forbidden
```

### 3. Need predictable test data

Instead of depending on changing production/test API data, return fixed data.

### 4. Test slow API behavior

You can simulate different network conditions.

### 5. Block unnecessary requests

For example:

```text
Ads
Analytics
Tracking
Third-party requests
```

---

# 13. Network Interception vs API Testing

These are related but different.

### API Testing

You directly send API requests:

```javascript
const response = await request.get(
    'https://api.example.com/products'
);
```

You are testing the API itself.

### Network Interception

You intercept a request made by the application:

```javascript
await page.route(
    '**/products',
    async route => {
        await route.fulfill({
            status: 200,
            body: JSON.stringify({
                products: []
            })
        });
    }
);
```

You are controlling or observing the application's network traffic.

### Simple Difference

```text
API Testing
    ↓
Test API directly

Network Interception
    ↓
Control/observe API calls made by the application
```

---

# 14. Important Interview Question

### What is network interception in Playwright?

**Answer:**

> Network interception in Playwright allows us to intercept HTTP requests and responses made by the application. We can inspect, continue, modify, mock, or abort those requests using `page.route()`.

---

# 15. Important Interview Question

### What is the difference between `route.continue()` and `route.fulfill()`?

**Answer:**

> `route.continue()` allows the request to proceed to the real server, while `route.fulfill()` provides a custom or mocked response without calling the real server.

---

# 16. Important Interview Question

### What does `route.abort()` do?

**Answer:**

> `route.abort()` prevents the intercepted request from reaching the server.

---

# 17. Most Important Code to Remember

### Intercept

```javascript
await page.route(
    '**/products',
    async route => {
        // action
    }
);
```

### Continue

```javascript
await route.continue();
```

### Mock

```javascript
await route.fulfill({
    status: 200,
    contentType: 'application/json',
    body: JSON.stringify({
        message: 'Mock response'
    })
});
```

### Block

```javascript
await route.abort();
```

---

# Quick Revision

```text
page.route()
      ↓
Intercept request
      ↓
 ┌───────────────┐
 │               │
continue()    fulfill()    abort()
 │               │            │
Real API      Mock API      Block
```

### Remember

```text
page.route()       → Intercept
route.request()   → Inspect
route.continue()  → Real request
route.fulfill()   → Mock response
route.abort()     → Block request
```
