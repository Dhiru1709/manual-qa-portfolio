# Selenium Automation Framework

This repository contains a sample Selenium automation framework built using Java and TestNG following the Page Object Model (POM) design pattern.

The framework demonstrates how I organize reusable automation scripts for UI regression testing while keeping the code maintainable and scalable.

> **Note:** This framework is a representative implementation created to showcase automation practices. It does not contain proprietary company code or client-specific test scripts.

---

# Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Java |
| Automation Tool | Selenium WebDriver |
| Test Framework | TestNG |
| Build Tool | Maven |
| Design Pattern | Page Object Model (POM) |
| Version Control | Git |
| CI/CD | GitHub Actions |

---

# Framework Structure

```
Automation_Framework
│
├── src
│   ├── main
│   │   ├── pages
│   │   ├── utils
│   │   └── config
│   │
│   └── test
│       ├── base
│       ├── tests
│       └── data
│
├── screenshots
├── reports
├── testng.xml
├── pom.xml
└── README.md
```

---

# Framework Features

- Page Object Model (POM)
- Reusable WebDriver setup
- Explicit Wait implementation
- Centralized configuration
- Cross-browser support (Chrome & Edge)
- Screenshot capture on failure
- TestNG execution
- Maven build support
- GitHub Actions ready

---

# Sample Test Flow

```
Launch Browser

↓

Login

↓

Navigate to Module

↓

Perform Action

↓

Validate Result

↓

Logout

↓

Close Browser
```

---

# Sample Test Scenarios

- Verify user login
- Verify customer creation
- Verify customer update
- Verify search functionality
- Verify logout
- Verify validation messages

---

# Framework Design Principles

- Reusable page objects
- Maintainable test scripts
- Clear separation of test logic and page elements
- Easy addition of new test cases
- Reduced code duplication

---

# Execution

### Run all tests

```bash
mvn clean test
```

### Run TestNG suite

```bash
mvn test
```

---

# Future Enhancements

- Data-Driven Testing
- Parallel Execution
- Extent Reports
- Docker Integration
- Selenium Grid
- BrowserStack Integration

---

# Deliverables

- Selenium Framework
- Sample Test Scripts
- Page Objects
- TestNG Suite
- Execution Reports
