# Playwright Network Interception

## 1. What is Network Interception?

**Network interception** means Playwright catches a network request before it reaches the server.

This allows us to:

* Inspect requests
* Continue requests to the real server
* Modify requests
* Mock API responses
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

---

# 4. `route.continue()`

`route.continue()` allows the intercepted request to continue to the real server.

```javascript
await route.continue();
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

---

# 5. Request Modification

Network interception can also be used to **modify a request before sending it to the server**.

We use:

```javascript
route.continue({
    ...
});
```

This is different from simply:

```javascript
await route.continue();
```

The second version sends the original request unchanged.

---

# 6. Modify Request Headers

Suppose the application sends:

```text
Authorization: Bearer old-token
```

We can replace it with another token.

```javascript
await page.route(
    '**/customers',
    async route => {

        await route.continue({
            headers: {
                ...route.request().headers(),
                'Authorization': 'Bearer new-token'
            }
        });

    }
);
```

### Important

```javascript
...route.request().headers()
```

keeps the existing headers.

Then:

```javascript
'Authorization': 'Bearer new-token'
```

replaces the Authorization header.

### Flow

```text
Original Request
Authorization: Bearer old-token
        ↓
Playwright intercepts
        ↓
Modify Authorization
        ↓
Authorization: Bearer new-token
        ↓
Real Server
```

---

# 7. Modify a Custom Header

Suppose we want to add a custom header:

```text
x-test-environment: qa
```

Example:

```javascript
await page.route(
    '**/customers',
    async route => {

        await route.continue({
            headers: {
                ...route.request().headers(),
                'x-test-environment': 'qa'
            }
        });

    }
);
```

The original headers are preserved, and the new header is added.

---

# 8. Modify Query Parameters

Suppose the application calls:

```text
https://example.com/products?category=phone
```

We can modify the query parameter before sending the request.

```javascript
await page.route(
    '**/products?*',
    async route => {

        const url = new URL(route.request().url());

        url.searchParams.set('category', 'laptop');

        await route.continue({
            url: url.toString()
        });

    }
);
```

### What happens?

Original:

```text
/products?category=phone
```

Modified:

```text
/products?category=laptop
```

The modified request is then sent to the real server.

---

# 9. Modify POST Request Data

We can also modify the POST request body.

Suppose the application sends:

```json
{
    "name": "Selva",
    "email": "selva@test.com"
}
```

We can change the email before sending it.

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

### Flow

```text
Application sends

{
    "name": "Selva",
    "email": "selva@test.com"
}

        ↓

Playwright intercepts

        ↓

Modify email

        ↓

{
    "name": "Selva",
    "email": "updated@test.com"
}

        ↓

Real Server
```

---

# 10. Modify Multiple Parts of a Request

We can modify headers and POST data in the same interception.

```javascript
await page.route(
    '**/customers',
    async route => {

        const postData = route.request().postDataJSON();

        postData.email = 'test@example.com';

        await route.continue({
            headers: {
                ...route.request().headers(),
                'x-test-environment': 'qa'
            },
            postData: JSON.stringify(postData)
        });

    }
);
```

This allows us to change multiple parts of the request before it reaches the server.

---

# 11. `continue()` vs Modified `continue()`

### Original request

```javascript
await route.continue();
```

The request goes to the server unchanged.

### Modified request

```javascript
await route.continue({
    headers: {
        ...route.request().headers(),
        'x-test': 'qa'
    }
});
```

The request goes to the server with the modified header.

---

# 12. `continue()` vs `fulfill()` vs `abort()`

| Method                    | Purpose                                |
| ------------------------- | -------------------------------------- |
| `route.continue()`        | Send original request to real server   |
| `route.continue({ ... })` | Modify request and send to real server |
| `route.fulfill()`         | Return a mocked response               |
| `route.abort()`           | Block the request                      |

### Easy Memory Trick

```text
continue()          → Let it go
continue(options)   → Modify it and let it go
fulfill()           → Give my response
abort()             → Stop it
```

---

# 13. Mocking an API Response

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

The real API is not called.

---

# 14. Mocking a Server Error

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

This is useful for testing how the application handles backend failures.

---

# 15. `route.abort()`

`route.abort()` blocks the network request.

```javascript
await page.route(
    '**/analytics',
    async route => {
        await route.abort();
    }
);
```

---

# 16. Blocking Ads

```javascript
await page.route(
    '**/ads/**',
    async route => {
        await route.abort();
    }
);
```

---

# 17. Why Request Modification is Useful

Request modification is useful when we want to test different request conditions without changing the application code.

Examples:

### Change authentication

```text
Authorization: Bearer test-token
```

### Add test headers

```text
x-test-environment: qa
```

### Change query parameters

```text
category=phone
        ↓
category=laptop
```

### Change request body

```text
email: old@test.com
        ↓
email: new@test.com
```

---

# 18. Complete Request Modification Example

```javascript
import { test } from '@playwright/test';

test('modify API request', async ({ page }) => {

    await page.route(
        '**/customers',
        async route => {

            const postData = route.request().postDataJSON();

            postData.email = 'updated@test.com';

            await route.continue({
                headers: {
                    ...route.request().headers(),
                    'x-test-environment': 'qa'
                },
                postData: JSON.stringify(postData)
            });

        }
    );

    // Application action that sends POST /customers
    // await page.locator('#create-customer').click();

});
```

---

# 19. Important Interview Question

### How do you modify a request in Playwright?

**Answer:**

> We can intercept the request using `page.route()` and use `route.continue()` with options such as `headers`, `url`, or `postData` to modify the request before sending it to the server.

Example:

```javascript
await route.continue({
    headers: {
        ...route.request().headers(),
        'x-test': 'qa'
    }
});
```

---

# 20. Important Interview Question

### Can Playwright modify request headers?

**Answer:**

Yes. We can use `route.continue()` and provide modified headers.

```javascript
await route.continue({
    headers: {
        ...route.request().headers(),
        'Authorization': 'Bearer new-token'
    }
});
```

---

# 21. Important Interview Question

### Can Playwright modify POST request data?

**Answer:**

Yes. We can read the existing POST body using:

```javascript
route.request().postDataJSON()
```

modify it, and pass it back using:

```javascript
route.continue({
    postData: JSON.stringify(postData)
});
```

---

# 22. Quick Revision

```text
page.route()
      ↓
Intercept request
      ↓
 ┌───────────────────────────────┐
 │                               │
continue()              continue(options)
 │                               │
Original request          Modified request
 │                               │
 └──────────────┬────────────────┘
                ↓
           Real Server


fulfill()
    ↓
Mock Response


abort()
    ↓
Block Request
```

### Remember

```text
page.route()                    → Intercept
route.request()                 → Inspect
route.request().url()           → Get URL
route.request().method()        → Get HTTP method
route.request().headers()       → Get headers
route.request().postDataJSON()  → Get JSON request body

route.continue()                → Send original request
route.continue({...})           → Modify and send request
route.fulfill()                 → Mock response
route.abort()                   → Block request
```
