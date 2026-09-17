Selenium Screenshots
1. What is a Screenshot in Selenium?

A screenshot captures the current browser screen during test execution.

Screenshots are useful for:

Debugging failed tests
Reporting
Evidence for defects
Capturing application behavior
CI/CD test results

In Selenium, screenshots are handled using the TakesScreenshot interface.

2. Basic Screenshot
Code
import java.io.File;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;

File source = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

File destination = new File("screenshots/homepage.png");

source.renameTo(destination);
Explanation
(TakesScreenshot) driver

Casts the WebDriver object to TakesScreenshot.

getScreenshotAs(OutputType.FILE)

Captures the screenshot and returns it as a file.

new File("screenshots/homepage.png")

Defines where the screenshot should be saved.

3. Recommended Way to Save Screenshot

Using Files.copy() is more reliable than renameTo().

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;

File source = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

Path destination = Path.of("screenshots/homepage.png");

Files.copy(source.toPath(), destination);
4. Create a Screenshot Utility

In a real automation framework, we normally create a reusable utility method.

ScreenshotUtil.java
package com.selva.selenium.utils;

import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;

import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;

public class ScreenshotUtil {

    public static void takeScreenshot(WebDriver driver, String fileName) {

        File source = ((TakesScreenshot) driver)
                .getScreenshotAs(OutputType.FILE);

        Path destination = Path.of("screenshots", fileName + ".png");

        try {
            Files.createDirectories(destination.getParent());
            Files.copy(source.toPath(), destination);

            System.out.println("Screenshot saved: " + destination);

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
5. Using the Screenshot Utility
ScreenshotUtil.takeScreenshot(driver, "login-page");

This creates:

screenshots/
└── login-page.png

Another example:

ScreenshotUtil.takeScreenshot(driver, "product-page");

Result:

screenshots/
├── login-page.png
└── product-page.png
6. Screenshot in a TestNG Test
@Test
public void loginTest() {

    driver.get("https://www.saucedemo.com/");

    ScreenshotUtil.takeScreenshot(driver, "login-page");

    // Test steps
}
7. Screenshot After Test Failure

This is a very common interview question.

We can capture a screenshot automatically when a TestNG test fails.

One approach is using ITestResult.

Example
import org.testng.ITestListener;
import org.testng.ITestResult;
import org.openqa.selenium.WebDriver;

public class TestListener implements ITestListener {

    private WebDriver driver;

    public TestListener(WebDriver driver) {
        this.driver = driver;
    }

    @Override
    public void onTestFailure(ITestResult result) {

        String testName = result.getName();

        ScreenshotUtil.takeScreenshot(driver, testName + "-failure");
    }
}

In a framework, the WebDriver is usually managed through a shared DriverManager or BaseTest, rather than passing it directly through the listener constructor.

The important interview concept is:

Capture a screenshot when the test fails so that the failure can be investigated later.

8. Element Screenshot

Selenium can also capture a screenshot of a specific element.

Example:

WebElement loginButton = driver.findElement(
        By.id("login-button")
);

File source = loginButton.getScreenshotAs(OutputType.FILE);

Path destination = Path.of(
        "screenshots",
        "login-button.png"
);

Files.copy(source.toPath(), destination);

This captures only the element instead of the entire browser viewport.

9. Full Page Screenshot

Selenium 4 supports full-page screenshots with browsers that support the relevant screenshot capability.

Example:

File source = ((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.FILE);

Path destination = Path.of(
        "screenshots",
        "full-page.png"
);

Files.copy(source.toPath(), destination);

For exact full-page behavior, support can depend on the browser and driver implementation.

In interviews, it is safer to distinguish:

Viewport screenshot → visible browser area
Element screenshot → specific element
Full-page screenshot → entire page when supported
10. Screenshot as Base64

Selenium can also return a screenshot as a Base64 string.

String screenshot = ((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.BASE64);

This can be useful for:

Reports
Embedding screenshots
Sending screenshot data to another system
11. Screenshot as Bytes

Selenium can also return screenshot data as bytes.

byte[] screenshot = ((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.BYTES);

This is useful when another API or reporting framework expects byte data.

12. Screenshot Utility with Timestamp

A timestamp can prevent screenshots from overwriting each other.

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public static String getTimestamp() {

    DateTimeFormatter formatter =
            DateTimeFormatter.ofPattern("yyyyMMdd_HHmmss");

    return LocalDateTime.now().format(formatter);
}

Usage:

String fileName =
        "login-failure-" + getTimestamp();

ScreenshotUtil.takeScreenshot(driver, fileName);

Example:

login-failure-20260917_141500.png
13. Recommended Framework Structure

A Selenium automation project could have:

seleniumtestngpractise/
│
├── src/
│   └── test/
│       └── java/
│           └── com/
│               └── selva/
│                   └── selenium/
│                       ├── pages/
│                       │   ├── LoginPage.java
│                       │   ├── ProductPage.java
│                       │   └── CartPage.java
│                       │
│                       ├── tests/
│                       │   └── LoginTest.java
│                       │
│                       └── utils/
│                           └── ScreenshotUtil.java
│
├── screenshots/
│
├── pom.xml
└── testng.xml
14. Interview Questions
Q1. How do you take a screenshot in Selenium?
Answer

Selenium provides the TakesScreenshot interface. I cast the WebDriver to TakesScreenshot and use getScreenshotAs() to capture the screenshot.

Example:

File screenshot =
        ((TakesScreenshot) driver)
                .getScreenshotAs(OutputType.FILE);
Q2. What is TakesScreenshot?

TakesScreenshot is a Selenium interface that provides screenshot functionality.

TakesScreenshot screenshotDriver =
        (TakesScreenshot) driver;

Then:

screenshotDriver.getScreenshotAs(OutputType.FILE);
Q3. What are the different OutputType options?

Common options are:

OutputType.FILE
OutputType.BYTES
OutputType.BASE64
FILE

Returns the screenshot as a file.

getScreenshotAs(OutputType.FILE);
BYTES

Returns screenshot data as bytes.

getScreenshotAs(OutputType.BYTES);
BASE64

Returns screenshot data as a Base64 string.

getScreenshotAs(OutputType.BASE64);
Q4. Can Selenium take a screenshot of an element?

Yes.

WebElement element = driver.findElement(By.id("login-button"));

element.getScreenshotAs(OutputType.FILE);
Q5. When do you capture screenshots?

Typical situations:

Test failure
Important checkpoints
Before/after a critical operation
Defect investigation
Reporting

In an automation framework, I generally prefer automatic screenshots on test failure.

15. Selenium Screenshot vs Playwright Screenshot
Feature	Selenium	Playwright
Full-page/browser screenshot	TakesScreenshot	page.screenshot()
Element screenshot	element.getScreenshotAs()	locator.screenshot()
File output	OutputType.FILE	path option
Base64	OutputType.BASE64	Buffer/Base64 handling
Bytes	OutputType.BYTES	Buffer
Failure screenshots	TestNG listener	Playwright configuration/hooks
Selenium
File screenshot =
        ((TakesScreenshot) driver)
                .getScreenshotAs(OutputType.FILE);
Playwright
await page.screenshot({
    path: 'screenshots/homepage.png'
});
16. Important Interview Point

Remember this pattern:

WebDriver
   ↓
TakesScreenshot
   ↓
getScreenshotAs()
   ↓
OutputType
   ↓
FILE / BYTES / BASE64

The most important Selenium screenshot syntax to remember is:

File screenshot =
        ((TakesScreenshot) driver)
                .getScreenshotAs(OutputType.FILE);
17. Quick Revision
Entire browser screenshot
((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.FILE);
Element screenshot
element.getScreenshotAs(OutputType.FILE);
Base64
((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.BASE64);
Bytes
((TakesScreenshot) driver)
        .getScreenshotAs(OutputType.BYTES);
Interview answer

I use Selenium's TakesScreenshot interface and getScreenshotAs() method to capture screenshots. In my framework, I can integrate screenshot capture with TestNG listeners so that screenshots are automatically captured when a test fails.
