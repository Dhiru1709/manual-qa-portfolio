# Bug Reports

**Project:** Enterprise Trading Platform
**Reported by:** Dhirendra Sinha
**Tool:** JIRA

---

## BUG-001 | Critical | Silent Data Corruption During Trade Mid-Flow Cancellation

**Summary:** Modifying a trade mid-flow and then cancelling causes settlement values to save as zero with no error message

**Severity:** Critical
**Priority:** P1
**Status:** Fixed & Verified
**Module:** Trade Entry / Settlements
**Found During:** Exploratory Testing

### Environment
- Application Version: 2.4.1
- Environment: QA
- Browser: Chrome 124
- OS: Windows 11

### Steps to Reproduce
1. Login with valid credentials
2. Navigate to Trade Entry
3. Create a new trade — Instrument: CRUDE_OIL, Qty: 100, Price: 85.50, Direction: BUY
4. Submit trade (Trade ID assigned: TRD-2047, Status: PENDING)
5. Click Edit on TRD-2047 — change quantity from 100 to 250
6. While the save request is in progress (network tab shows pending), click Cancel on the edit screen
7. Navigate to Settlements module and search for TRD-2047

### Expected Result
Settlement amount shows either 8,550.00 (original: 100 × 85.50) or 21,375.00 (modified: 250 × 85.50) — one consistent, correct value.

### Actual Result
Settlement amount shows 0.00. No error message displayed anywhere. Data silently corrupted.

### Evidence

**SQL Query Run After Reproduction:**
```sql
SELECT trade_id, quantity, price, settlement_amount
FROM settlements
WHERE trade_id = 'TRD-2047';
```

**Result:**
```
trade_id   | quantity | price | settlement_amount
TRD-2047   | NULL     | 85.50 | 0.00
```

Expected: quantity = 100 or 250, settlement_amount = 8550.00 or 21375.00

### Root Cause Analysis
Race condition between the save transaction and cancel action. The cancel interrupts the save mid-write, leaving an incomplete record with NULL quantity and zero settlement amount. The application has no rollback mechanism for this scenario.

### Fix Applied
Added transaction lock on save operation — cancel button disabled until save completes. Added rollback handling if cancel fires during save.

### Fix Verification
- Reproduced original scenario 10 times after fix — no data corruption observed
- SQL confirms correct values saved in all 10 runs
- Boundary tested: cancel immediately after save starts, cancel after save completes — both handled correctly
- Closed after full verification ✅

---

## BUG-002 | High | Trade Submission API Returns 200 for Missing Mandatory Fields

**Summary:** POST /api/v3/trades/submit returns HTTP 200 OK even when mandatory field trade_date is missing from request body

**Severity:** High
**Priority:** P1
**Status:** Fixed & Verified
**Module:** Trade Entry API
**Found During:** API Testing

### Environment
- API Version: v3.2
- Environment: QA
- Tool: Postman

### Steps to Reproduce
1. Open Postman
2. Set method to POST, URL to `/api/v3/trades/submit`
3. Add Authorization header with valid token
4. Send request body WITHOUT trade_date field:

```json
{
  "instrument": "CRUDE_OIL",
  "quantity": 100,
  "price": 85.50,
  "direction": "BUY",
  "trader_id": "TRD_USR_001"
}
```
5. Send request and observe response

### Expected Result
```
HTTP 400 Bad Request
Body: { "error": "trade_date is required" }
```

### Actual Result
```
HTTP 200 OK
Body: { "status": "success", "trade_id": "TRD-TEST-001" }
```

Trade created in database with NULL trade_date.

### Evidence

**DB Verification:**
```sql
SELECT trade_id, trade_date FROM trades WHERE trade_id = 'TRD-TEST-001';
-- Result: trade_date = NULL
```

### Impact
Trades with NULL trade_date cause downstream settlement date calculation to fail silently. Financial reporting shows incorrect settlement dates. Data integrity compromised across two modules.

### Root Cause
Server-side validation for trade_date was missing. Frontend validation existed but was easily bypassed via direct API call.

### Fix Verification
- Missing trade_date now returns HTTP 400 with correct error message ✅
- Tested all other mandatory fields — all validated correctly ✅
- Closed ✅

---

## BUG-003 | High | Settlement Amount Incorrect for SELL Direction Trades

**Summary:** Settlement amount calculated as positive value for SELL direction trades — should be negative to reflect outflow

**Severity:** High
**Priority:** P1
**Status:** Fixed & Verified
**Module:** Settlements

### Steps to Reproduce
1. Create trade: Instrument = CRUDE_OIL, Qty = 200, Price = 90.00, Direction = SELL
2. Confirm trade
3. Navigate to Settlements module
4. Note settlement amount

### Expected Result
Settlement amount = -18,000.00 (negative — representing cash outflow for SELL)

### Actual Result
Settlement amount = +18,000.00 (positive — same as BUY direction)

### Evidence
```sql
SELECT trade_id, direction, quantity, price, settlement_amount
FROM trades t JOIN settlements s ON t.trade_id = s.trade_id
WHERE t.direction = 'SELL';
-- All SELL trades showing positive settlement amounts
```

### Root Cause
Settlement calculation formula did not account for trade direction. Formula was: `quantity × price` for all trades regardless of direction.

### Fix
Formula updated to: `quantity × price × (direction = 'BUY' ? 1 : -1)`

### Fix Verification
- SELL trades now show negative settlement amounts ✅
- BUY trades unaffected ✅
- Cross-checked 20 historical SELL trades — all corrected ✅

---

## BUG-004 | Medium | Pagination Breaks on Last Page of Trade List

**Summary:** Clicking Next on the last page of trade list shows blank screen

**Severity:** Medium
**Priority:** P2
**Status:** Fixed & Verified
**Module:** Trade List UI

### Steps to Reproduce
1. Navigate to Trade List
2. Go to the last page using pagination controls
3. Click Next button

### Expected Result
Button disabled OR stays on same page — no action

### Actual Result
Blank white screen. Console error: `Cannot read property 'data' of undefined`

### Root Cause
Frontend did not handle empty API response for page beyond last page — tried to render undefined data.

### Fix Verification
Next button now disabled on last page ✅
