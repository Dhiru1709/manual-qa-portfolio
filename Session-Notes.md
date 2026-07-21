# Exploratory Testing — Session Notes

**Tester:** Dhirendra Sinha
**Approach:** Session-Based Exploratory Testing (SBET)

---

## Session 001 — Trade Entry Flow — Edge Cases & Timing Issues

**Charter:** Explore trade creation flow focusing on unexpected user behaviour, rapid actions, and timing-based edge cases
**Duration:** 90 minutes
**Sprint:** Sprint 14
**Risk Area:** High — trade entry is core financial workflow

### Mission
Find defects that scripted test cases would not catch. Focus on unusual input combinations, rapid user actions, browser behaviour, and session timing.

### Areas Explored & Findings

**1. Rapid Form Submission**
- Clicked Submit button multiple times rapidly before response loaded
- Finding: Duplicate trades created — no duplicate guard on rapid double-click
- Raised: BUG-007 (High) — duplicate trade guard missing on submit button
- Recommended fix: Disable submit button after first click until response received

**2. Browser Back Button During Save**
- Started trade submission, hit browser Back before response came back
- Finding: Trade saved in backend but UI shows previous screen with no confirmation
- User has no indication whether trade was saved or not — they may submit again
- Raised: BUG-008 (Medium)

**3. Copy-Paste with Leading/Trailing Spaces**
- Pasted " 100 " (with spaces) into quantity field
- Finding: Accepted as-is and saved as string — downstream calculation failed silently
- Raised: BUG-009 (High) — input not trimmed before validation

**4. Tab Navigation Order**
- Navigated entire form using Tab key only
- Finding: Tab order skips the instrument dropdown — keyboard-only users cannot reach it
- Raised: BUG-010 (Medium) — accessibility issue

**5. Session Expiry During Form Fill**
- Started filling form, left session idle for 30 minutes, then submitted
- Finding: Form submitted but data lost — session expired mid-save
- No warning to user about session expiry while filling form
- Raised: BUG-011 (High) — should warn user before session expires

**6. Long Text in Notes Field**
- Pasted 600 characters into notes field (max = 500)
- Finding: Frontend allowed it, API rejected it, but error message was generic: "Something went wrong"
- User doesn't know what failed or how to fix it
- Raised: BUG-012 (Low) — improve error message specificity

### Summary
- Bugs found: 6
- Critical: 0 | High: 3 | Medium: 2 | Low: 1
- Most valuable finding: Silent duplicate trade creation on double-click
- Observation: Timing-based and rapid action testing is very productive on this module

---

## Session 002 — API Boundary & Security Testing

**Charter:** Explore API endpoints with boundary values, malformed payloads, SQL injection, and concurrent requests
**Duration:** 60 minutes
**Tool:** Postman

### Mission
Find API vulnerabilities and edge cases not covered by scripted test cases.

### Areas Explored & Findings

**1. Extremely Large Payload**
- Sent notes field with 10,000 characters
- Finding: API accepted it — no server-side max length validation on notes
- DB stored truncated value silently — no error returned
- Raised: BUG-013 (Medium)

**2. SQL Injection Attempt**
- Sent `trade_id = '1' OR '1'='1'` in GET request query parameter
- Finding: API returned ALL trades — SQL injection vulnerability confirmed
- Raised: BUG-014 (Critical) — flagged immediately to dev lead, hotfix required

**3. Missing Content-Type Header**
- Sent POST without `Content-Type: application/json`
- Finding: API returned 500 Internal Server Error instead of 415 Unsupported Media Type
- Stack trace partially visible in response — internal details exposed
- Raised: BUG-015 (Medium)

**4. Concurrent Duplicate Requests**
- Sent 10 simultaneous POST requests with identical trade data using Postman Runner
- Finding: 3 duplicate trades created — race condition in trade creation
- Raised: BUG-016 (High)

**5. Expired Token**
- Reused token after session expired
- Finding: API returned 200 OK with valid data — expired tokens not invalidated server-side
- Raised: BUG-017 (Critical)

### Summary
- Bugs found: 5
- Critical: 2 | High: 1 | Medium: 2
- Most impactful: SQL injection (BUG-014) and expired token reuse (BUG-017) — both escalated immediately
- Note: Security testing on financial APIs should be a mandatory part of every release cycle

---

## Session 003 — Settlements Calculation Edge Cases

**Charter:** Explore settlement calculation accuracy for edge case trade combinations
**Duration:** 45 minutes

### Findings

**1. SELL direction settlement positive instead of negative**
- Expected: -18,000.00 for SELL 200 @ 90.00
- Actual: +18,000.00
- Raised: BUG-003 (High) — documented in Bug Reports

**2. Fractional price rounding inconsistency**
- Qty=150, Price=72.333 — UI showed 10,849.95 but DB stored 10,849.9
- Rounding rule not documented — raised as question to product team
- Business rule clarified: round to 2 decimal places using HALF_UP

**3. Settlement date on public holiday**
- Confirmed trade on Friday before a public holiday Monday
- Settlement date calculated as T+2 = Monday (holiday) instead of T+3 = Tuesday
- Raised: BUG-018 (High) — business day calendar not excluding public holidays
