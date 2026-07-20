# Exploratory Testing — Session Notes

**Tester:** Dhirendra Sinha
**Approach:** Session-Based Exploratory Testing (SBET)

---

## Session 001 — Trade Entry Flow

**Charter:** Explore trade creation flow focusing on edge cases and unexpected user behavior
**Duration:** 90 minutes
**Date:** June 2024

### Mission
Find defects that scripted test cases would not catch — focus on unusual input combinations, rapid actions, and unexpected navigation patterns.

### Areas Explored

**1. Rapid Form Submission**
- Clicked Submit button multiple times rapidly before response
- Found: Duplicate trades created — no duplicate guard on rapid clicks
- Severity: High — raised as BUG-004

**2. Browser Back Button During Save**
- Started trade submission, hit browser Back before response
- Found: Trade saved in backend but UI shows previous screen with no confirmation
- User has no way to know if trade was saved
- Severity: Medium — raised as BUG-005

**3. Copy-Paste Behavior**
- Pasted text with leading/trailing spaces into quantity field
- Found: " 100 " (with spaces) accepted and saved as string — caused calculation errors downstream
- Severity: High — raised as BUG-006

**4. Tab Navigation Order**
- Navigated form using Tab key only
- Found: Tab order skips the instrument dropdown — keyboard users cannot reach it
- Severity: Medium — accessibility issue

**5. Session During Long Form Fill**
- Started filling form, left session idle for 20 minutes, then submitted
- Found: Form submitted successfully but data lost — session expired mid-save
- User got no warning about session expiry while filling form
- Severity: High — raised as BUG-007

### Bugs Found This Session: 5
### Notes
Rapid action testing is very effective for this module. Most interesting bugs came from timing and unexpected navigation patterns rather than invalid data inputs.

---

## Session 002 — API Boundary Testing

**Charter:** Explore API endpoints with boundary values, malformed payloads, and unexpected inputs
**Duration:** 60 minutes
**Date:** July 2024

### Mission
Find API vulnerabilities that are not covered by happy path test cases.

### Areas Explored

**1. Extremely Large Payload**
- Sent request body with 10,000 character string in notes field
- Found: API accepted it — no max length validation on notes field
- DB stored truncated value silently
- Severity: Medium

**2. SQL Injection in Query Parameters**
- Sent `trade_id = '1' OR '1'='1'` in GET request
- Found: API returned all trades — SQL injection vulnerability
- Severity: Critical — raised immediately as BUG-008

**3. Missing Content-Type Header**
- Sent POST request without `Content-Type: application/json`
- Found: API returned 500 Internal Server Error instead of 415 Unsupported Media Type
- Severity: Low

**4. Concurrent API Calls**
- Sent 10 simultaneous POST requests with same trade data
- Found: 3 duplicate trades created — race condition in trade creation endpoint
- Severity: High — raised as BUG-009

### Bugs Found This Session: 4 (1 Critical)
### Notes
SQL injection finding was the most impactful. Boundary and concurrency testing should be standard practice for financial APIs.
