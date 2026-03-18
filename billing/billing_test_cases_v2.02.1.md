# PerkValet Billing Service — Acceptance Test Cases
Version: v2.02.1  
Thread: F — Billing Service Behavior  
Status: LOCKED (tool-agnostic)  
Source of Truth: BILLING_SERVICE_PLAYBOOK_v2.02.1

This document defines acceptance test cases for Billing Service behavior. It is independent of any QA platform.
Use these IDs (TC-F-xx) consistently in code comments, logs, bugs, and future QA tools.

---

## Test Case Index

| ID      | Name                                               |
|---------|----------------------------------------------------|
| TC-F-01 | Invoice generated on billing cycle                 |
| TC-F-02 | Invoice lifecycle transitions follow allowed rules |
| TC-F-03 | Late fee applied after grace period                |
| TC-F-04 | Payments update invoice balance and status         |
| TC-F-05 | Issued invoices are immutable                      |

---

## TC-F-01 Invoice generated on billing cycle

### Purpose
Verify that an invoice is automatically generated for an active Billing Account when the billing cycle boundary is reached.

### Preconditions
- BillingAccount exists and is ACTIVE
- BillingPolicy is assigned to the BillingAccount (global default or merchant override)
- No invoice exists for the current billing cycle period
- System clock is at or past the configured billing cycle boundary

### Trigger
- Billing cycle job executes (scheduled or manually invoked)

### Expected Results
- Exactly one invoice is created for the BillingAccount for that billing period
- Invoice is linked to the correct BillingAccount / Merchant
- Billing period start/end are correct for the cycle
- Invoice initial status is OPEN (issued) OR DRAFT (if your implementation stages before issuing) — must match Playbook rules
- Invoice totals are consistent with generated line items (no float math)

### Rejection / Failure Conditions
- Duplicate invoice created for the same (billingAccountId, periodStart, periodEnd, generationVersion)
- Invoice created for an inactive/suspended/archived account when not allowed
- Incorrect period boundaries or totals
- Invoice created without required linkages (billingAccountId/merchantId)

---

## TC-F-02 Invoice lifecycle transitions follow allowed rules

### Purpose
Verify invoice status transitions comply with the Billing Service state machine.

### Preconditions
- An issued invoice exists with status OPEN and a non-zero outstanding balance
- Due date (dueAt) is configured and can be evaluated vs current time

### Trigger
- Status normalization runs (as part of billing job or on-demand)

### Steps
1. Evaluate invoice before due date with balance > 0
2. Evaluate invoice after due date with balance > 0
3. Apply payments to reduce balance to 0 and re-evaluate

### Expected Results
- Before due date: status remains OPEN
- After due date and unpaid: status becomes PAST_DUE
- When balance reaches 0: status becomes PAID
- Invalid transitions are rejected (e.g., PAID → OPEN, VOID → OPEN)

### Rejection / Failure Conditions
- Status does not update when conditions are met
- Status updates inconsistently across repeated evaluations (non-idempotent)
- Invoice becomes PAID with non-zero balance, or PAST_DUE with zero balance

---

## TC-F-03 Late fee applied after grace period

### Purpose
Verify late fee is applied according to effective Billing Policy once an invoice becomes overdue, and is never duplicated.

### Preconditions
- Invoice exists in OPEN or PAST_DUE with non-zero outstanding balance
- Invoice dueAt has passed (and grace rules, if configured, have elapsed)
- Late-fee policy is enabled (flat fee or percentage)

### Trigger
- Late-fee evaluation runs

### Expected Results
- Late fee is applied per Billing Policy configuration (flat or % of outstanding balance per Playbook)
- Late fee is recorded as an append-only financial event (per your playbook rules)
- Late fee is applied exactly once per eligible invoice per policy rules (idempotent under reruns)

### Rejection / Failure Conditions
- Late fee applied before dueAt/grace period ends
- Late fee duplicated on rerun / retry
- Late fee amount does not match policy
- Late fee changes existing issued line items (must be append-only behavior per Playbook)

---

## TC-F-04 Payments update invoice balance and status

### Purpose
Verify successful payments update invoice balance and status correctly; duplicates do not double-apply.

### Preconditions
- Invoice exists with status OPEN or PAST_DUE and a non-zero outstanding balance
- Payment posting mechanism exists (manual simulation or provider callback)

### Trigger
- Payment transitions to a “successful/settled” state and is applied to invoice

### Steps
1. Apply partial payment (less than outstanding)
2. Verify balance decreased and status is correct
3. Apply additional payment to reach full settlement
4. Verify balance is 0 and status becomes PAID
5. Re-submit the same payment event/idempotency key

### Expected Results
- Partial payment increases amountPaid (or equivalent) and reduces balance correctly
- Status remains OPEN (if not overdue) or PAST_DUE (if overdue) until fully paid
- Full payment sets balance to 0 and status to PAID
- Duplicate payment submission does not double-apply (idempotent)
- Overpayment behavior follows Playbook (strict reject or handled via explicit credit policy if later added)

### Rejection / Failure Conditions
- Payment applies to wrong invoice/billing account
- Duplicate payment increments amountPaid twice
- Invoice becomes PAID with non-zero balance
- Overpayment produces negative balance without explicit credit policy

---

## TC-F-05 Issued invoices are immutable

### Purpose
Verify that issued invoices cannot be edited or deleted; only approved append-only events are allowed.

### Preconditions
- Invoice exists in OPEN or PAST_DUE with one or more line items
- Actor has access to attempt edits (admin/API client)

### Steps
1. Attempt to edit an existing invoice line item (description/amount)
2. Attempt to delete an existing invoice line item
3. Attempt to modify invoice header totals directly (subtotal/total)
4. Attempt to delete the invoice
5. (Optional) Run late-fee evaluation on an overdue invoice

### Expected Results
- Steps 1–4 are rejected with a clear error (locked/immutable)
- No changes are persisted to the issued invoice or existing line items
- Optional step 5: late fee behavior follows Playbook and is append-only (does not mutate existing issued line items)

### Rejection / Failure Conditions
- Any issued invoice line item can be edited/deleted
- Totals can be changed directly post-issue
- Invoice can be deleted without an audit-preserving void mechanism
