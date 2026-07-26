# SQL Validation

This section demonstrates how SQL is used during functional, API, and regression testing to verify backend data integrity and ensure consistency between the application, APIs, and database.

The queries below are representative examples created for portfolio purposes and do not contain proprietary database structures or business data.

---

# Database Tools

- Microsoft SQL Server
- SQL Server Management Studio (SSMS)

---

# SQL Validation Objectives

- Verify data saved from the application.
- Validate API transactions in the database.
- Ensure data integrity.
- Verify updates and deletions.
- Support defect investigation.

---

# Validation Scenarios

## Scenario 1 - Verify Customer Creation

### Objective

Verify that a newly created customer record exists in the database.

```sql
SELECT CustomerID,
       CustomerName,
       Status
FROM Customers
WHERE CustomerID = 'CUST001';
```

### Validation

- Customer record exists.
- Status is Active.
- Customer name matches the application.

---

## Scenario 2 - Verify Customer Update

### Objective

Verify updated customer information is correctly stored.

```sql
SELECT CustomerName,
       Email,
       PhoneNumber
FROM Customers
WHERE CustomerID = 'CUST001';
```

### Validation

- Updated values match the UI.
- No unexpected data changes.

---

## Scenario 3 - Verify Duplicate Customers

### Objective

Ensure duplicate Customer IDs are not present.

```sql
SELECT CustomerID,
       COUNT(*)
FROM Customers
GROUP BY CustomerID
HAVING COUNT(*) > 1;
```

### Validation

No duplicate records should be returned.

---

## Scenario 4 - Verify API Transaction

### Objective

Validate that an API request successfully updates the database.

```sql
SELECT CustomerID,
       ModifiedDate
FROM Customers
WHERE CustomerID = 'CUST001';
```

### Validation

- Record exists.
- Modification timestamp is updated.
- Values match API response.

---

## Scenario 5 - Verify Active Customers

### Objective

Retrieve all active customer records.

```sql
SELECT CustomerID,
       CustomerName
FROM Customers
WHERE Status = 'Active';
```

### Validation

Only active customer records should be returned.

---

# SQL Validation Workflow

```
Application

↓

API Request

↓

Database Update

↓

SQL Validation

↓

Result Verification
```

---

# Best Practices

- Validate critical business data after every transaction.
- Compare UI, API, and database values.
- Use SQL during defect investigation.
- Verify inserts, updates, and deletes.
- Validate data before closing defects.

---

# Deliverables

- SQL Validation Queries
- Database Verification Notes
- Data Integrity Checks
- API-to-Database Validation
