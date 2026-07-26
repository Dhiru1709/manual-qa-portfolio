# Enterprise Test Cases

## Project Information

| Field | Details |
|--------|---------|
| Project | UGI Energy Services |
| Module | Customer Management |
| Application | Web Application |
| Test Type | Functional & Regression |
| Prepared By | Dhirendra Sinha |

---

# Test Case Summary

| Total Test Cases | 10 |
|------------------|----|
| Functional | 6 |
| Validation | 2 |
| Negative | 2 |

---

## TC_001 - Verify user can create a new customer

| Field | Details |
|--------|---------|
| Test Case ID | TC_001 |
| Priority | High |
| Type | Functional |

### Preconditions

- User is logged into the application.
- User has permission to create customers.

### Test Steps

1. Navigate to Customer Management.
2. Click **New Customer**.
3. Enter all mandatory fields.
4. Click **Save**.

### Expected Result

- Customer is created successfully.
- Success message is displayed.
- Customer appears in the customer list.

---

## TC_002 - Verify mandatory field validation

| Field | Details |
|--------|---------|
| Test Case ID | TC_002 |
| Priority | High |
| Type | Validation |

### Test Steps

1. Open Customer Creation page.
2. Leave mandatory fields blank.
3. Click **Save**.

### Expected Result

- Required field validation messages are displayed.
- Customer record is not created.

---

## TC_003 - Verify duplicate customer cannot be created

| Field | Details |
|--------|---------|
| Test Case ID | TC_003 |
| Priority | High |
| Type | Negative |

### Test Steps

1. Create a customer using an existing Customer ID.
2. Save the record.

### Expected Result

- System prevents duplicate record creation.
- Appropriate validation message is displayed.

---

## TC_004 - Verify customer information can be updated

| Field | Details |
|--------|---------|
| Test Case ID | TC_004 |
| Priority | Medium |
| Type | Functional |

### Test Steps

1. Search an existing customer.
2. Update editable fields.
3. Save changes.

### Expected Result

- Updated information is saved successfully.
- Modified values appear after refresh.

---

## TC_005 - Verify customer search functionality

| Field | Details |
|--------|---------|
| Test Case ID | TC_005 |
| Priority | Medium |
| Type | Functional |

### Test Steps

1. Enter Customer Name or Customer ID.
2. Click Search.

### Expected Result

- Matching customer records are displayed.

---

## TC_006 - Verify inactive customers are displayed correctly

| Field | Details |
|--------|---------|
| Test Case ID | TC_006 |
| Priority | Medium |
| Type | Functional |

### Expected Result

- Inactive customers are identified correctly according to filter selection.

---

## TC_007 - Verify invalid email format validation

| Field | Details |
|--------|---------|
| Test Case ID | TC_007 |
| Priority | Medium |
| Type | Validation |

### Test Steps

1. Enter an invalid email address.
2. Save the customer.

### Expected Result

- Validation message is displayed.
- Record is not saved.

---

## TC_008 - Verify unauthorized user cannot create customers

| Field | Details |
|--------|---------|
| Test Case ID | TC_008 |
| Priority | High |
| Type | Security / Authorization |

### Expected Result

- User without required permissions cannot access customer creation functionality.

---

## TC_009 - Verify customer record is stored in database

| Field | Details |
|--------|---------|
| Test Case ID | TC_009 |
| Priority | High |
| Type | Database Validation |

### Validation

- Verify the customer record is successfully stored in the database.
- Ensure all submitted values match the application.

### Expected Result

- Database values match UI values.

---

## TC_010 - Verify customer creation API updates application

| Field | Details |
|--------|---------|
| Test Case ID | TC_010 |
| Priority | High |
| Type | API Validation |

### Validation

- Verify successful API response.
- Verify customer record appears in UI.
- Verify customer data is stored correctly in database.

### Expected Result

- API, UI and database remain consistent.
