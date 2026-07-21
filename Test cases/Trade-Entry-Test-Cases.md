# Test Cases — Trade Entry Module

**Module:** Trade Entry
**Prepared by:** Dhirendra Sinha
**Environment:** QA
**Tool:** TestRail

---

## TC-001 to TC-005 — Login & Authentication

| TC ID | Title | Steps | Expected Result | Priority | Status |
|---|---|---|---|---|---|
| TC-001 | Valid login | 1. Enter valid username and password 2. Click Login | Dashboard loads. User role displayed correctly | P1 | Pass |
| TC-002 | Invalid password | 1. Enter valid username 2. Enter wrong password 3. Click Login | Error: "Invalid credentials". No dashboard access | P1 | Pass |
| TC-003 | Empty username and password | 1. Leave both fields blank 2. Click Login | Validation messages shown for both fields | P2 | Pass |
| TC-004 | Account lockout after failed attempts | 1. Enter wrong password 5 times | Account locked. Error message displayed. Admin notified | P1 | Pass |
| TC-005 | Session timeout | 1. Login 2. Leave idle for 30 minutes 3. Try to perform action | Redirected to login page. Session expired message shown | P1 | Pass |

---

## TC-010 to TC-020 — Trade Creation — Happy Path

| TC ID | Title | Steps | Expected Result | Priority | Status |
|---|---|---|---|---|---|
| TC-010 | Create trade with all mandatory fields | 1. Navigate to Trade Entry 2. Fill instrument, quantity, price, direction, trade date 3. Submit | Trade created. Unique Trade ID assigned. Status = PENDING | P1 | Pass |
| TC-011 | Create trade with optional fields | 1. Fill mandatory + optional fields (notes, reference ID) 2. Submit | Trade created. All fields saved correctly in DB | P2 | Pass |
| TC-012 | Verify trade saved in database | 1. Submit trade 2. Run SQL: `SELECT * FROM trades WHERE trade_id = 'TRD-XXXX'` | All submitted values match DB record exactly | P1 | Pass |
| TC-013 | Trade status transition PENDING to CONFIRMED | 1. Create trade 2. Authorise trade | Status changes from PENDING to CONFIRMED. Audit log entry created | P1 | Pass |
| TC-014 | Risk position updated after trade creation | 1. Create trade 2. Navigate to Risk module | Trade position reflected in Risk dashboard with correct values | P1 | Pass |
| TC-015 | Settlement record created after confirmation | 1. Confirm trade 2. Navigate to Settlements 3. Run SQL: `SELECT * FROM settlements WHERE trade_id = 'X'` | Settlement record created with correct calculated amount | P1 | Pass |

---

## TC-021 to TC-035 — Trade Creation — Negative Scenarios

| TC ID | Title | Input | Expected Result | Priority | Status |
|---|---|---|---|---|---|
| TC-021 | Submit with missing instrument | Instrument field empty | Validation error: "Instrument is required" | P1 | Pass |
| TC-022 | Submit with missing trade date | Trade date field empty | Validation error: "Trade date is required" | P1 | Pass |
| TC-023 | Negative quantity | Quantity = -50 | Validation error: "Quantity must be greater than 0" | P1 | Pass |
| TC-024 | Zero quantity | Quantity = 0 | Validation error: "Quantity must be greater than 0" | P1 | Pass |
| TC-025 | Invalid date format | Trade date = "abc" | Validation error: "Invalid date format" | P2 | Pass |
| TC-026 | Past date beyond allowed range | Trade date = 2 years ago | Validation error: "Trade date out of allowed range" | P2 | Pass |
| TC-027 | Duplicate trade submission | Submit same trade twice rapidly | Second submission rejected: "Duplicate trade detected" | P1 | Pass |
| TC-028 | XSS in text field | `<script>alert('xss')</script>` in notes | Input sanitised. Script not executed. Stored as plain text | P1 | Pass |
| TC-029 | SQL injection in trade ID field | `1' OR '1'='1` in trade ID | Input rejected or sanitised. No data exposed | P1 | Pass |
| TC-030 | Exceed quantity limit | Quantity = 10,000,001 (above max) | Validation error: "Quantity exceeds maximum allowed limit" | P2 | Pass |

---

## TC-040 to TC-050 — Trade Modification

| TC ID | Title | Steps | Expected Result | Priority | Status |
|---|---|---|---|---|---|
| TC-040 | Modify PENDING trade quantity | 1. Open PENDING trade 2. Update quantity 3. Save | Quantity updated. Audit log entry created with old and new values | P1 | Pass |
| TC-041 | Modify CONFIRMED trade | 1. Open CONFIRMED trade 2. Attempt to edit quantity | Edit option disabled or error: "Confirmed trades cannot be modified" | P1 | Pass |
| TC-042 | Cancel PENDING trade | 1. Open PENDING trade 2. Click Cancel 3. Confirm cancellation | Status = CANCELLED. Trade moved to cancelled list. Audit log updated | P1 | Pass |
| TC-043 | Cancel CONFIRMED trade | 1. Open CONFIRMED trade 2. Attempt cancellation | Cancellation blocked or requires additional authorisation | P1 | Pass |
| TC-044 | Audit log accuracy after modification | 1. Modify trade 2. Check audit log | Old value, new value, timestamp, and user recorded correctly | P2 | Pass |

---

## TC-060 to TC-070 — Boundary Value Testing

| TC ID | Title | Input | Expected Result | Priority |
|---|---|---|---|---|
| TC-060 | Minimum valid quantity | 1 | Trade created successfully | P2 |
| TC-061 | Maximum valid quantity | 10,000,000 | Trade created successfully | P2 |
| TC-062 | Max + 1 quantity | 10,000,001 | Validation error — exceeds limit | P2 |
| TC-063 | Minimum valid price | 0.01 | Trade created successfully | P2 |
| TC-064 | Price with 4 decimal places | 85.1234 | Accepted or rounded per business rule | P2 |
| TC-065 | Trade date = today | Today's date | Trade created successfully | P1 |
| TC-066 | Trade date = tomorrow | Tomorrow's date | Accepted or rejected per business rule | P2 |
| TC-067 | Max length notes field | 500 characters | Accepted — stored correctly | P2 |
| TC-068 | Notes field exceeding max | 501 characters | Truncated or validation error shown | P2 |

---

## TC-080 to TC-090 — Integration Testing

| TC ID | Title | Steps | Expected Result | Priority |
|---|---|---|---|---|
| TC-080 | Trade data flows to Risk module | 1. Submit trade 2. Check Risk positions | Position updated with correct quantity and exposure | P1 |
| TC-081 | Settlement amount calculated correctly | 1. Confirm trade (qty=100, price=85.50) 2. Check settlement | Settlement amount = 8,550.00. DB confirms same | P1 |
| TC-082 | Multiple trades aggregate in Risk | 1. Submit 3 trades same instrument 2. Check Risk | Total position = sum of all 3 trade quantities | P1 |
| TC-083 | Cancelled trade removed from Risk | 1. Cancel a PENDING trade 2. Check Risk positions | Position reduced by cancelled trade quantity | P1 |
| TC-084 | DB integrity across all 3 modules | 1. Submit and confirm trade 2. Run cross-module SQL validation | trades, risk_positions, settlements all consistent | P1 |
