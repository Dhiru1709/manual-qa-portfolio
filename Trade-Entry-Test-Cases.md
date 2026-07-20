# Test Cases — Trade Entry Module

**Module:** Trade Entry
**Prepared by:** Dhirendra Sinha
**Environment:** QA

---

## Login & Authentication

| TC ID | Test Case | Steps | Expected Result | Priority |
|---|---|---|---|---|
| TC_001 | Valid login | 1. Enter valid username and password 2. Click Login | User lands on dashboard | P1 |
| TC_002 | Invalid password | 1. Enter valid username 2. Enter wrong password 3. Click Login | Error message: "Invalid credentials" | P1 |
| TC_003 | Empty fields | 1. Leave username and password blank 2. Click Login | Validation message shown for both fields | P2 |
| TC_004 | Session timeout | 1. Login 2. Leave session idle for X minutes | User automatically logged out | P1 |
| TC_005 | Brute force lockout | 1. Enter wrong password 5 times | Account locked, error message displayed | P1 |

---

## Trade Creation — Happy Path

| TC ID | Test Case | Steps | Expected Result | Priority |
|---|---|---|---|---|
| TC_010 | Create single trade | 1. Navigate to Trade Entry 2. Fill all mandatory fields 3. Submit | Trade created with unique Trade ID, status = PENDING | P1 |
| TC_011 | Create trade with optional fields | 1. Fill mandatory + optional fields 2. Submit | Trade created with all fields saved correctly | P2 |
| TC_012 | Verify trade in database | 1. Submit trade 2. Query DB: `SELECT * FROM trades WHERE trade_id = 'X'` | Trade record exists with correct values | P1 |
| TC_013 | Trade confirmation email | 1. Submit trade 2. Check registered email | Confirmation email received within 2 minutes | P2 |

---

## Trade Creation — Negative Scenarios

| TC ID | Test Case | Steps | Expected Result | Priority |
|---|---|---|---|---|
| TC_020 | Submit with empty mandatory fields | 1. Leave mandatory fields blank 2. Click Submit | Validation errors shown for each empty field | P1 |
| TC_021 | Invalid trade quantity | 1. Enter negative quantity 2. Submit | Error: "Quantity must be greater than 0" | P1 |
| TC_022 | Invalid date format | 1. Enter date as text string 2. Submit | Error: "Invalid date format" | P2 |
| TC_023 | Duplicate trade submission | 1. Submit trade 2. Immediately submit same trade again | Error: "Duplicate trade detected" or second trade rejected | P1 |
| TC_024 | Special characters in text fields | 1. Enter `<script>alert('xss')</script>` in name field 2. Submit | Input sanitized, no script execution | P1 |

---

## Trade Modification

| TC ID | Test Case | Steps | Expected Result | Priority |
|---|---|---|---|---|
| TC_030 | Modify PENDING trade | 1. Open PENDING trade 2. Update quantity 3. Save | Trade updated, audit log entry created | P1 |
| TC_031 | Modify CONFIRMED trade | 1. Open CONFIRMED trade 2. Attempt to edit | Edit restricted, error message shown | P1 |
| TC_032 | Cancel PENDING trade | 1. Open PENDING trade 2. Click Cancel 3. Confirm | Trade status = CANCELLED, visible in cancelled list | P1 |

---

## Integration — Risk Module

| TC ID | Test Case | Steps | Expected Result | Priority |
|---|---|---|---|---|
| TC_040 | Trade reflects in Risk | 1. Submit trade 2. Navigate to Risk module 3. Check position | Trade position visible in Risk dashboard | P1 |
| TC_041 | DB validation — Risk table | 1. Submit trade 2. Query: `SELECT * FROM risk_positions WHERE trade_id = 'X'` | Risk record created with correct values | P1 |

---

## Boundary Value Testing

| TC ID | Test Case | Input | Expected Result | Priority |
|---|---|---|---|---|
| TC_050 | Min quantity | 1 | Trade created successfully | P2 |
| TC_051 | Max quantity | 999999 | Trade created successfully | P2 |
| TC_052 | Zero quantity | 0 | Validation error | P1 |
| TC_053 | Max+1 quantity | 1000000 | Validation error: exceeds limit | P2 |
