# Selenium Automation Framework

A test automation framework I built to reduce manual regression effort on a large enterprise application. The project started as a way to automate repetitive test cycles that were taking 2–3 days every sprint — after setting this up, we got that down to a few hours.

Built with Java and Selenium WebDriver, following Page Object Model so the test logic stays clean and the page interactions are easy to update when the UI changes.

---

## Why I Built This

We had a growing regression suite that was eating up most of the sprint just to execute manually. I started by automating the most repetitive flows — login, trade entry, and status checks — and expanded from there. The goal was always to free up time for exploratory and edge case testing, not to replace it.

---

## Tech Stack

- Java
- Selenium WebDriver
- TestNG
- Maven
- WebDriverManager

---

## Project Structure

```text
selenium-framework/
├── src/
│   ├── main/java/
│   │   ├── base/          # WebDriver setup and teardown
│   │   ├── pages/         # Page Object classes
│   │   └── utils/         # Helpers — config reader, Excel reader, wait utilities
│   └── test/java/
│       └── tests/         # TestNG test classes
├── config/
│   └── config.properties  # Environment URLs, browser, credentials
├── test-data/
│   └── TestData.xlsx      # Data-driven test inputs
├── testng.xml
└── pom.xml
```

---

## Key Design Decisions

**Page Object Model** — Each page has its own class with locators and actions. When a locator breaks (which happens often in active development), I update one place instead of hunting through every test.

**Data-Driven Testing** — Test data lives in Excel files outside the code. This made it easy to add new data combinations without touching the test logic, and non-technical team members could update test data themselves.

**Config via Properties File** — Environment URLs, browser choice, and credentials are all in `config.properties`. Switching between QA and staging is a one-line change, not a code change.

**WebDriverManager** — No manual driver downloads or version mismatch issues. It handles the driver setup automatically, which saved a lot of setup headaches especially when onboarding.

---

## How to Run

```bash
# Run full regression suite
mvn clean test

# Run a specific TestNG suite
mvn clean test -DsuiteXmlFile=testng.xml

# Run specific test group
mvn clean test -Dgroups=smoke
```

---

## Login Page Object

```java
public class LoginPage {
    private WebDriver driver;

    private By usernameField = By.id("username");
    private By passwordField = By.id("password");
    private By loginButton = By.id("login-btn");
    private By errorMessage = By.className("error-msg");

    public LoginPage(WebDriver driver) {
        this.driver = driver;
    }

    public void login(String username, String password) {
        driver.findElement(usernameField).sendKeys(username);
        driver.findElement(passwordField).sendKeys(password);
        driver.findElement(loginButton).click();
    }

    public String getErrorMessage() {
        return driver.findElement(errorMessage).getText();
    }

    public boolean isErrorDisplayed() {
        return driver.findElements(errorMessage).size() > 0;
    }
}
```

---

## Data-Driven Test — Login Flow

```java
@Test(dataProvider = "loginData")
public void testLogin(String username, String password, String expectedResult) {
    LoginPage loginPage = new LoginPage(driver);
    loginPage.login(username, password);

    if (expectedResult.equals("success")) {
        Assert.assertTrue(dashboardPage.isDisplayed(), "Login failed for: " + username);
    } else {
        Assert.assertTrue(loginPage.isErrorDisplayed(), "Error not shown for invalid login");
    }
}

@DataProvider(name = "loginData")
public Object[][] getLoginData() {
    return ExcelReader.getData("TestData.xlsx", "Login");
}
```

---

## What's Next

A few things I want to add when I get time:

- Parallel execution across browsers using TestNG's parallel config
- Extent Reports for better visual reporting after each run
- GitHub Actions integration for automatic runs on every PR
- Cross-browser testing — currently Chrome only, want to add Firefox and Edge

These are in progress — the core framework is stable and running in CI via Jenkins right now.

---

## Notes

The framework covers UI regression for the main application flows. API test automation is handled separately using REST Assured — that repo is linked in my profile if you want to look at both together.
