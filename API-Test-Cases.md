# API Testing — Sample Test Cases

**Tool:** Postman / REST Assured
**Prepared by:** Dhirendra Sinha

---

## Trade Submission API

**Endpoint:** `POST /api/v3/trades/submit`
**Base URL:** `https://api.tradingplatform.com`

---

### TC_API_001 — Valid Trade Submission

**Type:** Happy Path
**Priority:** P1

**Request:**
```json
{
  "trade_date": "2024-06-15",
  "instrument": "CRUDE_OIL",
  "quantity": 100,
  "price": 85.50,
  "direction": "BUY",
  "trader_id": "TRD_USR_001"
}
```

**Expected Response:**
```json
{
  "status": "success",
  "trade_id": "TRD-XXXX",
  "message": "Trade submitted successfully"
}
```

**Status Code:** 200 OK
**Validations:**
- Status code is 200
- Response body contains `trade_id`
- `trade_id` format matches `TRD-[0-9]{4}`
- Response time < 2000ms
- DB query confirms trade record created

---

### TC_API_002 — Missing Mandatory Field

**Type:** Negative
**Priority:** P1

**Request (trade_date missing):**
```json
{
  "instrument": "CRUDE_OIL",
  "quantity": 100,
  "price": 85.50,
  "direction": "BUY",
  "trader_id": "TRD_USR_001"
}
```

**Expected Response:**
```json
{
  "status": "error",
  "message": "trade_date is required"
}
```

**Status Code:** 400 Bad Request
**Validations:**
- Status code is 400
- Error message clearly identifies missing field
- No trade record created in DB

---

### TC_API_003 — Invalid Quantity (Negative Value)

**Type:** Negative / Boundary
**Priority:** P1

**Request:**
```json
{
  "trade_date": "2024-06-15",
  "instrument": "CRUDE_OIL",
  "quantity": -50,
  "price": 85.50,
  "direction": "BUY",
  "trader_id": "TRD_USR_001"
}
```

**Expected:** 400 Bad Request — `"quantity must be greater than 0"`

---

### TC_API_004 — Unauthorized Request (No Token)

**Type:** Security
**Priority:** P1

**Request:** Valid body, no Authorization header

**Expected:** 401 Unauthorized
**Validations:**
- Status code is 401
- No trade created in DB
- Error message does not expose internal system details

---

### TC_API_005 — Get Trade by ID

**Endpoint:** `GET /api/v3/trades/{trade_id}`

**Request:** `GET /api/v3/trades/TRD-2047`

**Expected Response:**
```json
{
  "trade_id": "TRD-2047",
  "status": "CONFIRMED",
  "instrument": "CRUDE_OIL",
  "quantity": 100,
  "price": 85.50
}
```

**Validations:**
- Status code 200
- All fields present and correct data type
- Values match DB: `SELECT * FROM trades WHERE trade_id = 'TRD-2047'`

---

### TC_API_006 — Get Non-Existent Trade

**Request:** `GET /api/v3/trades/TRD-9999`

**Expected:** 404 Not Found
**Validations:**
- Status code 404
- Error message: `"Trade not found"`
- Response does not expose DB structure

---

## SQL Validation Queries

```sql
-- Verify trade created correctly
SELECT trade_id, instrument, quantity, price, status, created_at
FROM trades
WHERE trade_id = 'TRD-2047';

-- Verify risk position updated
SELECT * FROM risk_positions WHERE trade_id = 'TRD-2047';

-- Verify settlement record created
SELECT * FROM settlements WHERE trade_id = 'TRD-2047';

-- Check for duplicate trades
SELECT trade_id, COUNT(*) as count
FROM trades
GROUP BY trade_id
HAVING COUNT(*) > 1;
```
