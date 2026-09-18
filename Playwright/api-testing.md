# Playwright API Testing – Interview Notes

## 1. What is API Testing?

API testing verifies the behavior of backend APIs without interacting with the application's UI.

Playwright provides the `request` fixture for API testing.

```javascript
import { test, expect } from '@playwright/test';

test('API test', async ({ request }) => {
    // API request
});
```

---

# 2. GET Request

GET is used to retrieve data.

```javascript
test('GET customer', async ({ request }) => {
    const response = await request.get(
        'https://api.example.com/customers/1'
    );

    await expect(response).toBeOK();
});
```

### Get response body

```javascript
const data = await response.json();

console.log(data);
console.log(data.id);
console.log(data.name);
console.log(data.email);
```

### Validate response

```javascript
expect(data.id).toBe(1);
expect(data.name).toBe('Selva');
```

---

# 3. POST Request

POST is generally used to create a new resource.

```javascript
test('Create customer', async ({ request }) => {
    const response = await request.post(
        'https://api.example.com/customers',
        {
            data: {
                name: 'Selva',
                email: 'selva@test.com'
            }
        }
    );

    expect(response.status()).toBe(201);
});
```

### Validate POST response

```javascript
const data = await response.json();

expect(data.name).toBe('Selva');
expect(data.email).toBe('selva@test.com');
```

---

# 4. PUT Request

PUT is generally used to update an existing resource.

```javascript
test('Update customer', async ({ request }) => {
    const response = await request.put(
        'https://api.example.com/customers/101',
        {
            data: {
                name: 'Selva',
                email: 'updated@test.com'
            }
        }
    );

    expect(response.status()).toBe(200);

    const data = await response.json();

    expect(data.email).toBe('updated@test.com');
});
```

---

# 5. DELETE Request

DELETE is used to delete a resource.

```javascript
test('Delete customer', async ({ request }) => {
    const response = await request.delete(
        'https://api.example.com/customers/101'
    );

    expect(response.status()).toBe(204);
});
```

A `204 No Content` response normally means the request was successful and there is no response body.

Therefore, we normally don't call:

```javascript
await response.json();
```

for a 204 response.

---

# 6. HTTP Status Codes

Common status codes:

| Status Code | Meaning                            |
| ----------- | ---------------------------------- |
| 200         | OK / Successful request            |
| 201         | Created                            |
| 204         | Successful request with no content |
| 400         | Bad Request                        |
| 401         | Unauthorized                       |
| 403         | Forbidden                          |
| 404         | Not Found                          |
| 500         | Internal Server Error              |

The exact status code depends on the API design.

---

# 7. response.status()

`response.status()` returns the HTTP status code.

```javascript
expect(response.status()).toBe(200);
```

Important:

```javascript
response.status()
```

not:

```javascript
response.statuscode()
```

---

# 8. response.json()

`response.json()` reads the response body as JSON.

```javascript
const data = await response.json();

console.log(data);
```

Example response:

```json
{
    "id": 101,
    "name": "Selva",
    "email": "selva@test.com"
}
```

We can access the values:

```javascript
console.log(data.id);
console.log(data.name);
console.log(data.email);
```

---

# 9. toBeOK()

Playwright provides:

```javascript
await expect(response).toBeOK();
```

This verifies that the response has a successful HTTP status.

Example:

```javascript
const response = await request.get(
    'https://api.example.com/customers'
);

await expect(response).toBeOK();
```

---

# 10. API Headers

HTTP headers provide additional information about an API request or response.

Common headers:

```text
Content-Type
Authorization
Accept
```

Example:

```javascript
const response = await request.get(
    'https://api.example.com/customers',
    {
        headers: {
            'Content-Type': 'application/json',
            'Authorization': 'Bearer abc123',
            'Accept': 'application/json'
        }
    }
);
```

### Header meanings

**Content-Type**

Specifies the format of the data being sent.

```text
Content-Type: application/json
```

**Accept**

Specifies the response format we want.

```text
Accept: application/json
```

**Authorization**

Provides authentication information.

```text
Authorization: Bearer abc123
```

---

# 11. Bearer Token Authentication

A common API authentication method is Bearer Token authentication.

```javascript
const token = 'myToken123';

const response = await request.get(
    'https://api.example.com/customers',
    {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    }
);

expect(response.status()).toBe(200);
```

### Template literals

JavaScript uses backticks and `${}` to insert a variable:

```javascript
const token = 'myToken123';

const authorization = `Bearer ${token}`;
```

The result is:

```text
Bearer myToken123
```

---

# 12. Login API → Get Token → Use Token

A real-world authentication flow often looks like:

```text
Login API
    ↓
Username + Password
    ↓
Authentication
    ↓
Access Token
    ↓
Use Token
    ↓
Call Protected API
```

### Step 1: Login

```javascript
const postResponse = await request.post(
    'https://api.example.com/login',
    {
        data: {
            username: 'Selva',
            password: 'Test123'
        }
    }
);
```

### Step 2: Read response

Suppose the login API returns:

```json
{
    "token": "abc123xyz"
}
```

Read the response:

```javascript
const loginData = await postResponse.json();
```

### Step 3: Extract token

```javascript
const token = loginData.token;
```

### Step 4: Use token

```javascript
const getResponse = await request.get(
    'https://api.example.com/customers',
    {
        headers: {
            'Authorization': `Bearer ${token}`
        }
    }
);
```

### Step 5: Validate

```javascript
expect(getResponse.status()).toBe(200);
```

---

# 13. Complete Authentication Example

```javascript
import { test, expect } from '@playwright/test';

test('Login and access customers API', async ({ request }) => {

    // Step 1: Login
    const postResponse = await request.post(
        'https://api.example.com/login',
        {
            data: {
                username: 'Selva',
                password: 'Test123'
            }
        }
    );

    // Step 2: Verify login
    expect(postResponse.status()).toBe(200);

    // Step 3: Read response
    const loginData = await postResponse.json();

    // Step 4: Extract token
    const token = loginData.token;

    // Step 5: Call protected API
    const getResponse = await request.get(
        'https://api.example.com/customers',
        {
            headers: {
                'Authorization': `Bearer ${token}`
            }
        }
    );

    // Step 6: Verify response
    expect(getResponse.status()).toBe(200);
});
```

---

# 14. When to Use `await`

Use `await` when a Playwright operation returns a Promise and we need its result before continuing.

Common examples:

```javascript
await request.get(...);
await request.post(...);
await request.put(...);
await request.delete(...);

await response.json();

await page.goto(...);
await page.locator('#login').click();
```

Usually no `await` is needed for synchronous operations such as:

```javascript
response.status();

page.url();

console.log(data);

expect(data.id).toBe(1);
```

For Playwright assertions that wait for a condition:

```javascript
await expect(response).toBeOK();
```

---

# 15. CRUD API Mapping

| Operation | HTTP Method | Common Status |
| --------- | ----------- | ------------- |
| Create    | POST        | 201           |
| Read      | GET         | 200           |
| Update    | PUT         | 200           |
| Delete    | DELETE      | 204           |

These are common conventions; actual APIs can use different status codes.

---

# 16. Interview Questions

### Q1. How do you perform API testing in Playwright?

Use Playwright's `request` fixture.

```javascript
test('GET API', async ({ request }) => {
    const response = await request.get(
        'https://api.example.com/customers'
    );

    expect(response.status()).toBe(200);
});
```

### Q2. How do you send JSON data in a POST request?

```javascript
await request.post(url, {
    data: {
        name: 'Selva',
        email: 'selva@test.com'
    }
});
```

### Q3. How do you validate the response status?

```javascript
expect(response.status()).toBe(200);
```

### Q4. How do you read a JSON response?

```javascript
const data = await response.json();
```

### Q5. How do you send an Authorization token?

```javascript
headers: {
    'Authorization': `Bearer ${token}`
}
```

### Q6. How do you get a token dynamically?

Call the login API, read the response, and extract the token:

```javascript
const loginData = await loginResponse.json();

const token = loginData.token;
```

### Q7. What is the difference between `response.status()` and `response.json()`?

```javascript
response.status()
```

returns the HTTP status code.

```javascript
response.json()
```

returns the response body as JSON.

---

# API Testing Quick Memory

```text
GET     → Read
POST    → Create
PUT     → Update
DELETE  → Delete

status() → Status code
json()   → Response body
headers  → Request metadata
Bearer   → Common token authentication

Login
  ↓
Get token
  ↓
Authorization header
  ↓
Call protected API
  ↓
Validate response
```
