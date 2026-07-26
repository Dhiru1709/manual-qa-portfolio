# API Testing

This section demonstrates my approach to validating REST APIs as part of end-to-end testing for enterprise web applications.

The focus is on ensuring that APIs function correctly, return accurate data, handle invalid requests gracefully, and maintain consistency between the API, application UI, and database.

---

# Tools Used

- Postman
- REST Assured (Java)
- Swagger
- SQL Server

---

# API Validation Checklist

The following validations are performed during API testing:

- Request Payload Validation
- Response Body Validation
- HTTP Status Code Verification
- Response Time Verification
- Authentication Validation
- Authorization Validation
- Error Message Validation
- JSON Schema Validation
- API to Database Validation

---

# Test Scenarios

## Scenario 1 - Create Customer

**Method**

POST

**Objective**

Verify that a new customer can be created successfully using the API.

### Validation

- Status Code should be **201 Created**
- Customer ID should be generated
- Response body should contain submitted data
- Customer record should be available in the application
- Customer record should exist in the database

---

## Scenario 2 - Retrieve Customer

**Method**

GET

**Objective**

Verify that customer information is returned correctly.

### Validation

- Status Code should be **200 OK**
- Response data should match database values
- Response time should be within acceptable limits

---

## Scenario 3 - Update Customer

**Method**

PUT

**Objective**

Verify customer information is updated successfully.

### Validation

- Updated values are reflected in API response
- Updated values are displayed in UI
- Updated values are stored in database

---

## Scenario 4 - Delete Customer

**Method**

DELETE

**Objective**

Verify customer record is deleted successfully.

### Validation

- Status Code should indicate successful deletion
- Record is no longer accessible
- Database record is removed or marked inactive

---

# Negative Test Scenarios

- Invalid Authentication Token
- Missing Mandatory Fields
- Invalid Customer ID
- Duplicate Customer Creation
- Invalid Request Payload
- Unsupported HTTP Method
- Invalid Content Type

---

# API Validation Flow

```
Request

↓

API

↓

Response Validation

↓

Application UI

↓

Database Validation
```

---

# Common Status Codes Verified

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 201 | Resource Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 409 | Conflict |
| 500 | Internal Server Error |

---

# Best Practices Followed

- Validate both positive and negative scenarios.
- Verify response payload against business requirements.
- Cross-check critical data with the database.
- Reuse environment variables for different environments.
- Maintain reusable API collections for regression testing.

---

# Deliverables

- API Test Scenarios
- Postman Collection
- Environment Variables
- API Validation Report
