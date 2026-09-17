# Selenium Screenshots

## 1. What is a Screenshot in Selenium?

A screenshot captures the current browser screen during test execution.

Screenshots are useful for:

* Debugging failed tests
* Reporting
* Evidence for defects
* Capturing application behavior
* CI/CD test results

In Selenium, screenshots are handled using the `TakesScreenshot` interface.

---

# 2. Basic Screenshot

### Code

```java
import java.io.File;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.WebDriver;

File source = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);

File destination = new File("screenshots/homepage.png");

source.renameTo(destination);
```

### Explanation

```java
(TakesScreenshot) driver
```

Casts the WebDriver object to `TakesScreenshot`.

```java
getScreenshotAs(OutputType.FILE)
```

Captures the screenshot and returns it as a file.

```java
new File("screenshots/homepage.png")
```

Defines where the screenshot should be saved.

---

# 3. Recommended Way to Save Screenshot

Using `Files.copy()` is more reliable than `renameTo()`.

```java
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
```

---

# 4. Create a Screenshot Utility

In a real automation framework, we normally create a reusable utility method.

### ScreenshotUtil.java

```java
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
```

---

# 5. Using the Screenshot Utility

```java
ScreenshotUtil.takeScreenshot(driver, "login-page");
```

This creates:

```text
screenshots/
└── login-page.png
```

Another example:

```java
ScreenshotUtil.takeScreenshot(driver, "product-page");
```

Result:

```text
screenshots/
├── login-page.png
└── product-page.png
```

---

# 6. Screenshot in a TestNG Test

```java
@Test
public void loginTest() {

    driver.get("https://www.saucedemo.com/");

    ScreenshotUtil.takeScreenshot(driver, "login-page");

    // Test steps
}
```

---

# 7. Screenshot After Test Failure

This is a very common interview question.

We can capture a screenshot automatically when a TestNG test fails.

One approach is using `ITestResult`.

### Example

```java
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
```

In a framework, the WebDriver is usually managed through a shared `DriverManager` or `BaseTest`, rather than passing it directly through the listener constructor.

The important interview concept is:

> **Capture a screenshot when the test fails so that the failure can be investigated later.**

---

# 8. Element Screenshot

Selenium can also capture a screenshot of a specific element.

Example:

```java
WebElement loginButton = driver.findElement(
        By.id("login-button")
);

File source = loginButton.getScreenshotAs(OutputType.FILE);

Path destination = Path.of(
        "screenshots",
        "login-button.png"
);

Files.copy(source.toPath(), destination);
```

This captures only the element instead of the entire browser viewport.

---

# 9. Fu
