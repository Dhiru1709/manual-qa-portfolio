# API Testing Strategy

**Project:** Enterprise Trading Platform (ETRM)
**Prepared by:** Dhirendra Sinha

---

## Objective

Define the approach, coverage, and validation strategy for REST API testing across the Trade Entry, Risk, and Settlements APIs.

---

## APIs in Scope

| API | Method | Endpoint | Description |
|---|---|---|---|
| Trade Submission | POST | /api/v3/trades/submit | Create new trade |
| Get Trade | GET | /api/v3/trades/{id} | Retrieve trade by ID |
| Update Trade | PUT | /api/v3/trades/{id} | Modify existing trade |
| Cancel Trade | DELETE | /api/v3/trades/{id} | Cancel a trade |
| Get Risk Position | GET | /api/v3/risk/positions | Get risk positions |
| Get Settlement | GET | /api/v3/settlements/{trade_id} | Get settlement for trade |

---

## Validation Checklist Per Endpoint

### Functional Validations
- Correct HTTP status codes (200, 201, 400, 401, 403, 404, 500)
- Response body structure matches API contract
- All expected fields present with correct data types
- Business logic validated — calculations, status transitions, limits

### Data Validations
- Mandatory fields enforced — 400 returned when missing
- Field format validations — dates, numbers, enums
- Boundary values — min/max quantity, price ranges
- SQL verification — DB record matches API response

### Security Validations
- 401 returned with no/invalid token
- 403 returned with insufficient permissions
- No sensitive data exposed in error messages
- SQL injection attempts rejected

### Error Handling
- Meaningful error messages returned
- Correct error codes for each failure scenario
- No stack traces or internal details exposed in response

---

## Tools

- **Postman** — manual API testing, collection organisation, environment variables
- **REST Assured (Java)** — automated API regression tests integrated into CI/CD

---

## Environment Variables (Postman)

```
base_url = https://api.tradingplatform.com
auth_token = {{generated_per_session}}
trade_id = {{created_dynamically}}
```
