# Test Cases — Settlements Module

**Module:** Settlements
**Prepared by:** Dhirendra Sinha

---

## Overview

Settlements module calculates and manages financial settlement amounts for confirmed trades. Accuracy here is critical — incorrect settlement values directly impact financial reporting.

---

## TC-S001 to TC-S010 — Settlement Calculation Validation

| TC ID | Title | Input | Expected Result | Priority |
|---|---|---|---|---|
| TC-S001 | Basic settlement calculation | Qty=100, Price=85.50 | Settlement amount = 8,550.00 | P1 |
| TC-S002 | Settlement with fees applied | Qty=100, Price=85.50, Fee=1% | Settlement = 8,550.00 + 85.50 = 8,635.50 | P1 |
| TC-S003 | Settlement for SELL direction | Qty=200, Price=90.00, SELL | Settlement = -18,000.00 (negative for sell) | P1 |
| TC-S004 | Settlement for fractional price | Qty=150, Price=72.333 | Settlement = 10,849.95 — verify rounding rule | P2 |
| TC-S005 | DB validation of settlement amount | Confirm trade, run SQL | `SELECT settlement_amount FROM settlements WHERE trade_id = 'X'` matches UI | P1 |
| TC-S006 | Settlement status after confirmation | Confirm trade | Settlement status = PENDING_PAYMENT | P1 |
| TC-S007 | Settlement date assigned correctly | Confirm trade on 15-Jun | Settlement date = T+2 = 17-Jun (business days) | P1 |
| TC-S008 | Multiple settlements for same counterparty | 3 trades same counterparty | Each settlement separate. Net position report available | P2 |
| TC-S009 | Settlement recalculates on trade amendment | Amend confirmed trade price | Settlement amount updated. Old value in audit log | P1 |
| TC-S010 | Cancelled trade settlement voided | Cancel trade after settlement created | Settlement status = VOIDED. Amount = 0 in active settlements | P1 |

---

## SQL Validation Queries Used

```sql
-- Verify settlement amount
SELECT trade_id, settlement_amount, settlement_date, status
FROM settlements
WHERE trade_id = 'TRD-2047';

-- Cross-check with trade details
SELECT t.trade_id, t.quantity, t.price, t.direction,
       (t.quantity * t.price) AS expected_settlement,
       s.settlement_amount AS actual_settlement
FROM trades t
JOIN settlements s ON t.trade_id = s.trade_id
WHERE t.trade_id = 'TRD-2047';

-- Check for null settlement amounts
SELECT trade_id FROM settlements WHERE settlement_amount IS NULL;

-- Verify no duplicate settlements
SELECT trade_id, COUNT(*) FROM settlements
GROUP BY trade_id HAVING COUNT(*) > 1;
```
