# Selenium Automation Framework — Overview

**Language:** Java
**Framework:** Selenium WebDriver + TestNG
**Pattern:** Page Object Model (POM) + Data-Driven Testing
**CI/CD:** Jenkins nightly pipeline
**Prepared by:** Dhirendra Sinha

---

## Framework Structure

```
selenium-framework/
├── src/
│   ├── main/java/
│   │   ├── base/
│   │   │   └── BaseTest.java          # WebDriver setup/teardown
│   │   ├── pages/
│   │   │   ├── LoginPage.java         # Login page object
│   │   │   ├── TradeDashboardPage.java
│   │   │   └── TradeEntryPage.java    # Trade entry page object
│   │   └── utils/
│   │       ├── ConfigReader.java      # Read properties file
│   │       ├── ExcelReader.java       # Read test data from Excel
│   │       └── WaitHelper.java        # Custom wait utilities
│   └── test/java/
│       └── tests/
│           ├── LoginTests.java
│           └── TradeEntryTests.java
├── test-data/
│   └── TradeTestData.xlsx
├── config/
│   └── config.properties
├── testng.xml
└── pom.xml
```

---

## Why This Architecture

**Page Object Model** separates test logic from UI interaction. When a locator changes, only the page class needs updating — not every test.

**Data-Driven Testing** using Excel files allows running the same test with multiple data sets without code duplication — critical for financial data validation.

**TestNG** provides parallel execution, grouping, and detailed reporting out of the box.

---

## Sample Code — BaseTest.java

```java
public class BaseTest {
    protected WebDriver driver;

    @BeforeMethod
    public void setUp() {
        String browser = ConfigReader.getProperty("browser");
        if (browser.equalsIgnoreCase("chrome")) {
            WebDriverManager.chromedriver().setup();
            driver = new ChromeDriver();
        }
        driver.manage().window().maximize();
        driver.manage().timeouts().implicitlyWait(Duration.ofSeconds(10));
        driver.get(ConfigReader.getProperty("base.url"));
    }

    @AfterMethod
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}
```

---

## Sample Code — TradeEntryPage.java (Page Object)

```java
public class TradeEntryPage {
    private WebDriver driver;

    // Locators
    private By instrumentDropdown = By.id("instrument-select");
    private By quantityField = By.id("quantity-input");
    private By priceField = By.id("price-input");
    private By directionDropdown = By.id("direction-select");
    private By submitButton = By.id("submit-trade-btn");
    private By successMessage = By.className("success-notification");
    private By tradeIdDisplay = By.id("trade-id-display");

    public TradeEntryPage(WebDriver driver) {
        this.driver = driver;
    }

    public void selectInstrument(String instrument) {
        Select select = new Select(driver.findElement(instrumentDropdown));
        select.selectByVisibleText(instrument);
    }

    public void enterQuantity(String quantity) {
        WebElement field = driver.findElement(quantityField);
        field.clear();
        field.sendKeys(quantity);
    }

    public void enterPrice(String price) {
        WebElement field = driver.findElement(priceField);
        field.clear();
        field.sendKeys(price);
    }

    public void selectDirection(String direction) {
        Select select = new Select(driver.findElement(directionDropdown));
        select.selectByVisibleText(direction);
    }

    public void clickSubmit() {
        driver.findElement(submitButton).click();
    }

    public String getSuccessMessage() {
        return driver.findElement(successMessage).getText();
    }

    public String getTradeId() {
        return driver.findElement(tradeIdDisplay).getText();
    }
}
```

---

## Sample Code — TradeEntryTests.java (Data-Driven)

```java
public class TradeEntryTests extends BaseTest {

    @Test(dataProvider = "tradeData")
    public void testTradeCreation(String instrument, String quantity,
                                   String price, String direction,
                                   String expectedResult) {
        LoginPage loginPage = new LoginPage(driver);
        loginPage.login(
            ConfigReader.getProperty("username"),
            ConfigReader.getProperty("password")
        );

        TradeEntryPage tradePage = new TradeEntryPage(driver);
        tradePage.selectInstrument(instrument);
        tradePage.enterQuantity(quantity);
        tradePage.enterPrice(price);
        tradePage.selectDirection(direction);
        tradePage.clickSubmit();

        if (expectedResult.equals("SUCCESS")) {
            Assert.assertTrue(
                tradePage.getSuccessMessage().contains("Trade submitted"),
                "Trade creation failed for: " + instrument
            );
            String tradeId = tradePage.getTradeId();
            Assert.assertFalse(tradeId.isEmpty(), "Trade ID not generated");
        } else {
            // Verify validation error shown
            ErrorPage errorPage = new ErrorPage(driver);
            Assert.assertTrue(
                errorPage.isErrorDisplayed(),
                "Expected error not shown for: " + instrument
            );
        }
    }

    @DataProvider(name = "tradeData")
    public Object[][] getTradeData() {
        return ExcelReader.getData("TradeTestData.xlsx", "TradeEntry");
    }
}
```

---

## Sample Code — REST Assured API Test

```java
public class TradeApiTests {

    private static final String BASE_URL = "https://api.tradingplatform.com";
    private static String authToken;

    @BeforeClass
    public static void setup() {
        RestAssured.baseURI = BASE_URL;
        authToken = AuthHelper.getToken();
    }

    @Test
    public void testValidTradeSubmission() {
        String requestBody = "{"
            + "\"trade_date\": \"2024-06-15\","
            + "\"instrument\": \"CRUDE_OIL\","
            + "\"quantity\": 100,"
            + "\"price\": 85.50,"
            + "\"direction\": \"BUY\","
            + "\"trader_id\": \"TRD_USR_001\""
            + "}";

        Response response = given()
            .header("Authorization", "Bearer " + authToken)
            .header("Content-Type", "application/json")
            .body(requestBody)
            .when()
            .post("/api/v3/trades/submit")
            .then()
            .statusCode(200)
            .body("status", equalTo("success"))
            .body("trade_id", notNullValue())
            .time(lessThan(2000L))
            .extract().response();

        String tradeId = response.jsonPath().getString("trade_id");
        Assert.assertFalse(tradeId.isEmpty(), "Trade ID should not be empty");
    }

    @Test
    public void testMissingMandatoryField() {
        String requestBody = "{"
            + "\"instrument\": \"CRUDE_OIL\","
            + "\"quantity\": 100,"
            + "\"price\": 85.50,"
            + "\"direction\": \"BUY\""
            + "}";

        given()
            .header("Authorization", "Bearer " + authToken)
            .header("Content-Type", "application/json")
            .body(requestBody)
            .when()
            .post("/api/v3/trades/submit")
            .then()
            .statusCode(400)
            .body("error", containsString("trade_date is required"));
    }

    @Test
    public void testUnauthorizedRequest() {
        given()
            .header("Content-Type", "application/json")
            .body("{\"instrument\": \"CRUDE_OIL\"}")
            .when()
            .post("/api/v3/trades/submit")
            .then()
            .statusCode(401);
    }
}
```

---

## Jenkins Pipeline (Jenkinsfile)

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps { git 'https://github.com/Dhiru1709/selenium-framework' }
        }
        stage('Build') {
            steps { sh 'mvn clean compile' }
        }
        stage('Run Tests') {
            steps { sh 'mvn test -Dsuite=regression' }
        }
        stage('Publish Report') {
            steps {
                publishHTML([
                    reportDir: 'target/surefire-reports',
                    reportFiles: 'index.html',
                    reportName: 'Test Report'
                ])
            }
        }
    }
    post {
        failure { emailext subject: 'Test Run Failed', body: '${BUILD_LOG}', to: 'team@company.com' }
    }
}
```
