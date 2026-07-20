# Bug Reports — Sample Defects

**Project:** Enterprise Trading Platform
**Reported by:** Dhirendra Sinha
**Tool:** JIRA

---

## BUG-001 | Critical | Trade Modification Causes Silent Data Corruption

**Summary:** Modifying a trade mid-flow causes incorrect settlement values to be saved without any error message

**Severity:** Critical
**Priority:** P1
**Status:** Fixed & Verified
**Module:** Trade Entry / Settlements

### Environment
- Application Version: 2.4.1
- Environment: QA
- Browser: Chrome 124

### Steps to Reproduce
1. Login with valid credentials
2. Create a new trade and submit (Trade ID: TRD-2047)
3. Before trade is confirmed, click Edit and modify the quantity from 100 to 250
4. While save is in progress, click Cancel on the same screen
5. Navigate to Settlements module

### Expected Result
Trade quantity remains 100 (original value) OR 250 (modified value) — one consistent value saved, with appropriate status.

### Actual Result
Settlement module shows quantity as 0. No error message displayed. Data silently corrupted.

### Evidence
```sql
-- Query run after reproduction
SELECT trade_id, quantity, settlement_amount
FROM settlements
WHERE trade_id = 'TRD-2047';

-- Result: quantity = 0, settlement_amount = 0
-- Expected: quantity = 100 or 250
```

### Root Cause
Race condition between save and cancel operations — cancel interrupts the save transaction mid-write, leaving an incomplete record.

### Fix Verification
- Reproduced fix in QA environment
- Ran same scenario 10 times — no data corruption observed
- SQL query confirms correct values saved
- Closed after full verification

---

## BUG-002 | High | API Returns 200 for Invalid Payload

**Summary:** Trade submission API returns HTTP 200 even when mandatory fields are missing in the request body

**Severity:** High
**Priority:** P1
**Status:** Fixed & Verified
**Module:** Trade Entry API

### Environment
- API Version: v3.2
- Environment: QA
- Tool: Postman

### Steps to Reproduce
1. Open Postman
2. Send POST request to `/api/v3/trades/submit`
3. Remove `trade_date` field from request body
4. Send request

### Expected Result
HTTP 400 Bad Request with error message: `"trade_date is required"`

### Actual Result
HTTP 200 OK returned. Trade created in database with NULL trade_date.

### Evidence
**Request Body Sent:**
```json
{
  "trade_id": "TRD-TEST-001",
  "quantity": 100,
  "instrument": "CRUDE_OIL"
}
```

**Response Received:**
```json
{
  "status": "success",
  "trade_id": "TRD-TEST-001"
}
```

**DB Query:**
```sql
SELECT trade_id, trade_date FROM trades WHERE trade_id = 'TRD-TEST-001';
-- Result: trade_date = NULL
```

### Root Cause
Missing server-side validation for `trade_date` field. Frontend validation was present but API had no validation layer.

### Fix Verification
- Retested with missing `trade_date` — now returns HTTP 400 with correct error message
- Tested all other mandatory fields — all validated correctly
- Closed after full verification

---

## BUG-003 | Medium | Pagination Breaks on Last Page of Trade List

**Summary:** Clicking Next on the last page of the trade list throws a blank screen instead of staying on current page

**Severity:** Medium
**Priority:** P2
**Status:** Fixed & Verified
**Module:** Trade List UI

### Steps to Reproduce
1. Navigate to Trade List
2. Go to the last page using pagination controls
3. Click the Next button

### Expected Result
Button should be disabled OR stay on the same page with no action

### Actual Result
Blank white screen displayed. Console error: `Cannot read property 'data' of undefined`

### Evidence
Screenshot attached in JIRA.
Console error captured.

### Root Cause
Frontend was not handling empty API response for page beyond last page — tried to render undefined data.
