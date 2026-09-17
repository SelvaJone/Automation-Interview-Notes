# Playwright Interview Notes

## 1. Browser, BrowserContext, Page

### Browser

Represents the browser application such as Chromium, Firefox, or WebKit.

### BrowserContext

Represents an isolated browser session/profile.

Each context can have separate:

* Cookies
* Local storage
* Session storage
* Authentication state

### Page

Represents a browser tab.

### Hierarchy

```text
Browser
   │
   └── BrowserContext
          │
          ├── Page 1
          ├── Page 2
          └── Page 3
```

### Example

```javascript
const browser = await chromium.launch();

const context = await browser.newContext();

const page = await context.newPage();
```

In Playwright Test, the `page` fixture is normally provided automatically:

```javascript
test('example', async ({ page }) => {
    await page.goto('https://example.com');
});
```

---

# 2. Locators

A locator identifies an element on the page.

### Basic locator

```javascript
const username = page.locator('#user-name');

await username.fill('standard_user');
```

### CSS selectors

```javascript
page.locator('#username');      // ID
page.locator('.username');      // Class
page.locator('button');         // Tag
```

### Common Playwright locators

```javascript
page.locator('#username');

page.getByRole('button', { name: 'Login' });

page.getByText('Welcome');

page.getByLabel('Username');

page.getByPlaceholder('Enter username');
```

### Important distinction

```javascript
'#username'
```

is a selector string.

```javascript
page.locator('#username')
```

creates a Playwright Locator.

---

# 3. Click, Fill and Type

## click()

Used to click an element.

```javascript
await page.locator('#login').click();
```

## fill()

Clears the existing value and enters text.

```javascript
await page.locator('#username').fill('Selva');
```

## type()

Types characters into the field without automatically clearing existing text.

```javascript
await page.locator('#username').type('Selva');
```

### Easy memory

```text
fill()  → clear + enter
type()  → type into existing field
click() → click element
```

---

# 4. Assertions

Assertions verify that the application behaves as expected.

Import:

```javascript
import { test, expect } from '@playwright/test';
```

### Common assertions

```javascript
await expect(page).toHaveTitle('Dashboard');

await expect(page).toHaveURL('https://example.com/dashboard');

await expect(page.locator('#welcome')).toBeVisible();

await expect(page.locator('#message'))
    .toHaveText('Login successful');

await expect(page.locator('#username'))
    .toHaveValue('Selva');

await expect(page.locator('#submit')).toBeEnabled();

await expect(page.locator('#submit')).toBeDisabled();

await expect(page.locator('#checkbox')).toBeChecked();
```

### Pattern

```javascript
await expect(locator).assertion();
```

### Selenium comparison

Selenium:

```java
Assert.assertEquals(actualTitle, "Dashboard");
```

Playwright:

```javascript
await expect(page).toHaveTitle('Dashboard');
```

---

# 5. Dropdowns

For a standard HTML `<select>` element, use `selectOption()`.

### HTML

```html
<select id="country">
    <option value="us">United States</option>
    <option value="ca">Canada</option>
    <option value="mx">Mexico</option>
</select>
```

### Select by value

```javascript
await page.locator('#country').selectOption('us');
```

### Select by visible label

```javascript
await page.locator('#country')
    .selectOption({ label: 'Canada' });
```

### Important

`selectOption()` is designed for a real HTML `<select>` element.

Custom dropdowns built with buttons/divs require normal locator interactions.

### Value vs label

```text
value = "firefox"
label = "Firefox"
```

Example:

```javascript
await page.locator('#browser').selectOption('firefox');
```

or:

```javascript
await page.locator('#browser')
    .selectOption({ label: 'Firefox' });
```

---

# 6. Alerts / Dialogs

Playwright calls browser alerts **dialogs**.

Common dialog types:

```text
alert
confirm
prompt
```

### Handle a dialog

```javascript
page.on('dialog', async dialog => {
    console.log(dialog.message());

    await dialog.accept();
});
```

Then trigger the dialog:

```javascript
await page.locator('#delete').click();
```

### Dismiss dialog

```javascript
await dialog.dismiss();
```

### Enter text into a prompt

```javascript
await dialog.accept('Selva');
```

### Get dialog message

```javascript
dialog.message();
```

### Get dialog type

```javascript
dialog.type();
```

### Important

Set up the dialog handler **before** triggering the action that opens the dialog.

### Flow

```text
Click button
     ↓
Browser opens dialog
     ↓
Playwright detects dialog event
     ↓
Callback executes
     ↓
Accept / Dismiss
```

---

# 7. Frames / iFrames

An iframe is a webpage embedded inside another webpage.

Playwright provides `frameLocator()`.

### HTML

```html
<iframe id="login-frame">
    <input id="username">
    <button id="login">Login</button>
</iframe>
```

### Playwright

```javascript
const loginFrame = page.frameLocator('#login-frame');

await loginFrame.locator('#username').fill('Selva');

await loginFrame.locator('#login').click();
```

### Payment example

```javascript
const paymentFrame = page.frameLocator('#payment-frame');

await paymentFrame.locator('#card-number')
    .fill('123456789');

await paymentFrame.locator('#cvv')
    .fill('123');
```

### Selenium comparison

Selenium:

```java
driver.switchTo().frame("payment-frame");

driver.findElement(By.id("card-number"))
      .sendKeys("123456789");

driver.switchTo().defaultContent();
```

Playwright:

```javascript
const frame = page.frameLocator('#payment-frame');

await frame.locator('#card-number')
    .fill('123456789');
```

Playwright can interact with the frame through `frameLocator()` without explicitly switching into and out of the frame.

---

# 8. Tabs / Multiple Pages

In Playwright, a `Page` represents a browser tab.

### Hierarchy

```text
Browser
   │
   └── BrowserContext
          │
          ├── Page 1
          └── Page 2
```

### Open a new page

```javascript
const newPagePromise =
    page.context().waitForEvent('page');

await page.locator('#new-tab').click();

const newPage = await newPagePromise;
```

Now interact with the new tab:

```javascript
await newPage.locator('#username').fill('Selva');

console.log(newPage.url());
```

### Why waitForEvent()?

The click causes the new page to be created.

We start waiting for the event before clicking:

```text
Wait for new page
       ↓
Click button
       ↓
New page opens
       ↓
Get new Page object
```

### Important

```javascript
const newPage = await newPagePromise;
```

The `await` is required to resolve the Promise.

---

# 9. Popups

A popup is a new page opened as a result of an action on the current page.

Use:

```javascript
page.waitForEvent('popup')
```

### Example

```javascript
const popupPromise =
    page.waitForEvent('popup');

await page.locator('#open-popup').click();

const popup = await popupPromise;
```

Interact with the popup:

```javascript
await popup.locator('#username').fill('Selva');

console.log(popup.url());
```

### Popup pattern

```text
Wait for popup
      ↓
Click element
      ↓
Popup opens
      ↓
Get popup Page
      ↓
Interact with popup
```

### Popup vs new page

```javascript
page.waitForEvent('popup');
```

is commonly used when the current page opens a popup.

```javascript
page.context().waitForEvent('page');
```

can be used to wait for a new page in the browser context.

---

# 10. Screenshots

## Current page screenshot

```javascript
await page.screenshot({
    path: 'screenshot.png'
});
```

## Full-page screenshot

```javascript
await page.screenshot({
    path: 'fullpage.png',
    fullPage: true
});
```

## Element screenshot

```javascript
await page.locator('#login-form')
    .screenshot({
        path: 'login-form.png'
    });
```

### Easy memory

```text
Page screenshot
→ page.screenshot()

Element screenshot
→ locator.screenshot()
```

### Interview question

**How do you take a full-page screenshot in Playwright?**

```javascript
await page.screenshot({
    path: 'fullpage.png',
    fullPage: true
});
```

### Selenium comparison

Selenium:

```java
File src = ((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.FILE);
```

Playwright:

```javascript
await page.screenshot({
    path: 'screenshot.png'
});
```

---

# 11. File Upload

Playwright uses `setInputFiles()` for file uploads.

### HTML

```html
<input type="file" id="upload">
```

### Upload a file

```javascript
await page.locator('#upload')
    .setInputFiles('files/test.pdf');
```

### Multiple files

```javascript
await page.locator('#documents').setInputFiles([
    'files/resume.pdf',
    'files/coverletter.pdf'
]);
```

### Interview point

**How do you upload a file in Playwright?**

> Use `setInputFiles()` on the file input element.

---

# 12. File Download

Playwright uses the `download` event.

### Basic pattern

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.locator('#download').click();

const download = await downloadPromise;

await download.saveAs('files/result.pdf');
```

### Download flow

```text
Wait for download
       ↓
Click Download
       ↓
Get Download object
       ↓
Save file
```

### Interview point

**How do you download and save a file in Playwright?**

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.locator('#download').click();

const download = await downloadPromise;

await download.saveAs('files/result.pdf');
```

---

# 13. Waits / Auto-Waiting

Playwright has built-in **auto-waiting**.

For example:

```javascript
await page.locator('#login').click();
```

Playwright automatically waits for the element to be ready for the action.

It can wait for conditions such as:

* Element exists
* Element is visible
* Element is enabled
* Element is stable/interactable

This means explicit waits are needed less often than in Selenium.

### Explicit wait

```javascript
await page.locator('#message').waitFor();
```

### Wait for visible

```javascript
await page.locator('#message').waitFor({
    state: 'visible'
});
```

Common states:

```text
attached
visible
hidden
detached
```

### Assertion-based waiting

Often, assertions are preferred in tests because they both wait and verify:

```javascript
await expect(page.locator('#success'))
    .toBeVisible();
```

For example:

```javascript
await expect(page.locator('#submit'))
    .toBeEnabled();
```

### Important interview point

**Does Playwright require explicit waits like Selenium?**

> Playwright provides built-in auto-waiting for actions and assertions, so explicit waits are needed less often. When a specific condition requires it, Playwright provides methods such as `waitFor()` and assertion-based waiting.

---

# Quick Interview Cheat Sheet

| Concept            | Playwright                               |
| ------------------ | ---------------------------------------- |
| Browser tab        | `Page`                                   |
| Browser session    | `BrowserContext`                         |
| Locator            | `page.locator()`                         |
| Click              | `click()`                                |
| Enter text         | `fill()`                                 |
| Type text          | `type()`                                 |
| Assertion          | `expect()`                               |
| Dropdown           | `selectOption()`                         |
| Alert              | `page.on('dialog')`                      |
| iFrame             | `frameLocator()`                         |
| New tab/page       | `waitForEvent('page')`                   |
| Popup              | `waitForEvent('popup')`                  |
| Screenshot         | `page.screenshot()`                      |
| Element screenshot | `locator.screenshot()`                   |
| Upload             | `setInputFiles()`                        |
| Download           | `waitForEvent('download')`               |
| Save download      | `download.saveAs()`                      |
| Explicit wait      | `waitFor()`                              |
| Auto-waiting       | Built into Playwright actions/assertions |

---

# Key Playwright Patterns

## Login

```javascript
await page.locator('#username').fill('Selva');
await page.locator('#password').fill('Test123');
await page.locator('#login').click();
```

## Assertion

```javascript
await expect(page.locator('#welcome'))
    .toBeVisible();
```

## Dropdown

```javascript
await page.locator('#country')
    .selectOption('us');
```

## Dialog

```javascript
page.on('dialog', async dialog => {
    await dialog.accept();
});

await page.locator('#delete').click();
```

## Frame

```javascript
const frame = page.frameLocator('#login-frame');

await frame.locator('#username').fill('Selva');
```

## New Tab

```javascript
const pagePromise =
    page.context().waitForEvent('page');

await page.locator('#new-tab').click();

const newPage = await pagePromise;
```

## Popup

```javascript
const popupPromise =
    page.waitForEvent('popup');

await page.locator('#open-popup').click();

const popup = await popupPromise;
```

## Screenshot

```javascript
await page.screenshot({
    path: 'screenshot.png'
});
```

## Upload

```javascript
await page.locator('#upload')
    .setInputFiles('files/test.pdf');
```

## Download

```javascript
const downloadPromise =
    page.waitForEvent('download');

await page.locator('#download').click();

const download = await downloadPromise;

await download.saveAs('files/result.pdf');
```

## Wait / Verify

```javascript
await expect(page.locator('#success'))
    .toBeVisible();
```
