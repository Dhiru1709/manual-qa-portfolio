# API Test Cases — Ngenue Web API

**Platform:** nGenue ETRM (Energy Trading & Risk Management)
**Tool:** Postman / REST Assured
**Prepared by:** Dhirendra Sinha
**Base URL:** Internal nGenue API Gateway

---

## Authentication

All requests require:
```
Authorization: Bearer {{auth_token}}
Content-Type: application/json
Accept: application/json
```

---

## 1. Billing API

### TC-BILL-001 — Get Billing Data (Valid Customer)
**Endpoint:** GET /api/Billing/DownloadBillingData.Get
**Type:** Happy Path | Priority: P1

**Request:**
```
GET /api/Billing/DownloadBillingData.Get?customerId=CUST-001&billPeriod=2024-06
```

**Expected Response (HTTP 200):**
```json
{
  "customerId": "CUST-001",
  "billPeriod": "2024-06",
  "totalAmount": 12450.75,
  "lineItems": [...]
}
```

**Validations:**
- Status code = 200
- totalAmount is numeric and greater than 0
- billPeriod matches requested period
- Response time < 3000ms
- DB: `SELECT total_amount FROM billing WHERE customer_id = 'CUST-001' AND bill_period = '2024-06'` matches response

---

### TC-BILL-002 — Get Billing Data (Invalid Customer ID)
**Type:** Negative | Priority: P1

**Request:**
```
GET /api/Billing/DownloadBillingData.Get?customerId=INVALID-999
```

**Expected:** HTTP 404 — "Customer not found"

---

### TC-BILL-003 — Get Billing Tree Data
**Endpoint:** GET /api/Billing/DownloadBillingTreeData.Get
**Type:** Happy Path | Priority: P2

**Validations:**
- Returns hierarchical billing structure
- Parent and child records linked correctly
- DB cross-check: billing tree matches customer hierarchy

---

### TC-BILL-004 — Get Billing Tree Data (Flat)
**Endpoint:** GET /api/Billing/DownloadBillingTreeData.Get (flat)
**Type:** Happy Path | Priority: P2

**Validations:**
- Returns flat list format instead of tree
- Same total records as tree format
- No orphaned child records

---

## 2. Cash Receipts API

### TC-CR-001 — Get Cash Receipts (Valid Date Range)
**Endpoint:** GET /api/CashReceipts/CashReceiptsData.Get
**Type:** Happy Path | Priority: P1

**Request:**
```
GET /api/CashReceipts/CashReceiptsData.Get?startDate=2024-01-01&endDate=2024-06-30
```

**Expected Response (HTTP 200):**
```json
{
  "receipts": [
    {
      "receiptId": "RCP-001",
      "amount": 5000.00,
      "receiptDate": "2024-01-15",
      "customerId": "CUST-001"
    }
  ],
  "totalCount": 45
}
```

**Validations:**
- All receipt dates fall within requested range
- Amount values are positive numerics
- DB: `SELECT COUNT(*) FROM cash_receipts WHERE receipt_date BETWEEN '2024-01-01' AND '2024-06-30'` matches totalCount

---

### TC-CR-002 — Cash Receipts with Invalid Date Range
**Type:** Negative | Priority: P1

**Request:** startDate = 2024-06-30, endDate = 2024-01-01 (end before start)

**Expected:** HTTP 400 — "endDate must be after startDate"

---

## 3. Customer Data API

### TC-CUST-001 — Get Customer List
**Endpoint:** GET /api/CustomerData/Customers
**Type:** Happy Path | Priority: P1

**Validations:**
- Status code = 200
- Returns array of customers
- Each customer has: customerId, customerName, status, serviceAddress
- No null customer IDs in response
- DB count matches: `SELECT COUNT(*) FROM customers WHERE status = 'ACTIVE'`

---

### TC-CUST-002 — Get Single Customer by ID
**Endpoint:** GET /api/CustomerData/Customers/{customerId}
**Type:** Happy Path | Priority: P1

**Request:** GET /api/CustomerData/Customers/CUST-001

**Validations:**
- Correct customer returned
- All fields populated
- DB: `SELECT * FROM customers WHERE customer_id = 'CUST-001'` matches response

---

### TC-CUST-003 — Customer with Concurrent Billing Request
**Type:** Integration | Priority: P1

**Steps:**
1. GET customer details for CUST-001
2. GET billing data for CUST-001 using same customerId
3. Verify customer name in billing matches customer details response

**Validates cross-API data consistency**

---

## 4. Daily Volumes API

### TC-DV-001 — Get Daily Volumes (Valid Date Range)
**Endpoint:** GET /api/DailyVolumes/DailyVolumes
**Type:** Happy Path | Priority: P1

**Request:**
```
GET /api/DailyVolumes/DailyVolumes?startDate=2024-06-01&endDate=2024-06-30
```

**Expected Response:**
```json
{
  "volumes": [
    {
      "date": "2024-06-01",
      "commodity": "NATURAL_GAS",
      "volumeMCF": 1250.50,
      "locationId": "LOC-001"
    }
  ]
}
```

**Validations:**
- Volume values are non-negative
- Dates fall within requested range
- DB: `SELECT SUM(volume_mcf) FROM daily_volumes WHERE volume_date = '2024-06-01'` matches API sum

---

### TC-DV-002 — Daily Volumes — Missing Required Parameter
**Type:** Negative | Priority: P1

**Request:** GET /api/DailyVolumes/DailyVolumes (no date parameters)

**Expected:** HTTP 400 — "startDate and endDate are required"

---

## 5. Market Prices API

### TC-MP-001 — Get Market Prices
**Endpoint:** GET /api/MarketPrices/MarketPrice
**Type:** Happy Path | Priority: P1

**Request:**
```
GET /api/MarketPrices/MarketPrice?commodity=NATURAL_GAS&priceDate=2024-06-15
```

**Expected Response:**
```json
{
  "commodity": "NATURAL_GAS",
  "priceDate": "2024-06-15",
  "spotPrice": 2.85,
  "unit": "MMBtu",
  "source": "NYMEX"
}
```

**Validations:**
- spotPrice is positive numeric
- priceDate matches requested date
- unit is a valid enum value
- DB: `SELECT spot_price FROM market_prices WHERE commodity = 'NATURAL_GAS' AND price_date = '2024-06-15'` matches

---

### TC-MP-002 — Market Price for Weekend/Holiday
**Type:** Edge Case | Priority: P2

**Request:** priceDate = Saturday or public holiday

**Expected:** Either previous business day price returned with note, OR HTTP 404 "No price available for this date"
**Not Acceptable:** HTTP 500 or null price returned without explanation

---

## 6. Meter Read API

### TC-MR-001 — Get Meter Read Data
**Endpoint:** GET /api/MeterRead/MeterRead
**Type:** Happy Path | Priority: P1

**Request:**
```
GET /api/MeterRead/MeterRead?meterId=MTR-001&readDate=2024-06-30
```

**Expected Response:**
```json
{
  "meterId": "MTR-001",
  "readDate": "2024-06-30",
  "readValue": 45230.5,
  "previousRead": 44100.0,
  "consumption": 1130.5,
  "unit": "MCF"
}
```

**Validations:**
- consumption = readValue - previousRead (arithmetic check)
- readValue > previousRead (consumption should be positive)
- DB: `SELECT read_value, consumption FROM meter_reads WHERE meter_id = 'MTR-001' AND read_date = '2024-06-30'` matches

---

### TC-MR-002 — Meter Read Consumption Calculation Accuracy
**Type:** Data Validation | Priority: P1

**Steps:**
1. Get meter read for MTR-001 on 2024-06-30
2. Extract readValue and previousRead from response
3. Calculate expected consumption: readValue - previousRead
4. Assert consumption in response matches calculation

**This test catches silent calculation errors**

---

## 7. Retail Nominations API

### TC-RN-001 — Get Retail Nominations
**Endpoint:** GET /api/RetailNominations/RetailNominations.Get
**Type:** Happy Path | Priority: P1

**Validations:**
- Status code = 200
- Each nomination has: nominationId, customerId, commodity, volume, nominationDate, status
- No null nomination IDs
- DB: nomination count matches `SELECT COUNT(*) FROM retail_nominations WHERE status = 'ACTIVE'`

---

### TC-RN-002 — Nomination Volume Matches Daily Volume
**Type:** Integration | Priority: P1

**Steps:**
1. Get retail nomination for customer CUST-001 for 2024-06-01
2. Get daily volume for same customer and date
3. Assert nominated volume matches delivered volume (within acceptable tolerance)

**Validates nomination-to-delivery accuracy**

---

## 8. Weather Zones API

### TC-WZ-001 — Get Weather Zones
**Endpoint:** GET /api/WeatherZones/WeatherZones
**Type:** Happy Path | Priority: P2

**Validations:**
- Returns list of weather zones
- Each zone has: zoneId, zoneName, state, HDDBase, CDDBase
- No duplicate zone IDs

---

## 9. Utility Transactions API

### TC-UT-001 — Get Utility Transactions
**Endpoint:** GET /api/UtilityTransactions/UtilityTransactions
**Type:** Happy Path | Priority: P1

**Validations:**
- Transaction amounts are non-null
- Transaction dates within valid range
- DB: `SELECT SUM(transaction_amount) FROM utility_transactions` matches API total

---

## Cross-API Integration Test Cases

### TC-INT-001 — End-to-End: Customer → Nomination → Volume → Billing
**Priority:** P1 | Type: Integration

**Steps:**
1. GET /api/CustomerData/Customers/CUST-001 — verify customer exists
2. GET /api/RetailNominations/RetailNominations.Get?customerId=CUST-001 — get nomination
3. GET /api/DailyVolumes/DailyVolumes?customerId=CUST-001 — get delivered volumes
4. GET /api/Billing/DownloadBillingData.Get?customerId=CUST-001 — get bill
5. Assert: bill amount is consistent with volume × market price

**SQL Verification:**
```sql
-- Verify end-to-end data consistency
SELECT
  c.customer_name,
  rn.nominated_volume,
  dv.delivered_volume,
  mp.spot_price,
  (dv.delivered_volume * mp.spot_price) AS expected_bill,
  b.total_amount AS actual_bill
FROM customers c
JOIN retail_nominations rn ON c.customer_id = rn.customer_id
JOIN daily_volumes dv ON c.customer_id = dv.customer_id
JOIN market_prices mp ON dv.volume_date = mp.price_date
JOIN billing b ON c.customer_id = b.customer_id
WHERE c.customer_id = 'CUST-001'
AND dv.volume_date = '2024-06-30';
```

---

### TC-INT-002 — Meter Read Drives Billing Accuracy
**Priority:** P1 | Type: Integration

**Steps:**
1. GET meter read consumption for CUST-001 for June 2024
2. GET billing total for CUST-001 for June 2024
3. GET market price for June 2024
4. Assert: billing total ≈ consumption × market price (within rounding tolerance)

**This is the most critical financial accuracy test**
