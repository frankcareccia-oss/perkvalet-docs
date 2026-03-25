# PerkValet Ownership & Billing Authority Contract v1.0

## Status
Draft → Ready for implementation

---

# 1. Purpose

Define authoritative rules for:
- Merchant ownership
- Merchant administration
- Billing authority (AP Clerk)
- Recovery scenarios (lost owner, acquisition, turnover)

This contract prevents orphaned merchants and ensures continuous operational and billing control.

---

# 2. Core Principles

## 2.1 Merchant Must Never Be Orphaned

A merchant must always have at least one recovery-capable user:

- owner
- merchant_admin

### Rule

```
merchant MUST HAVE ≥ 1 user with role ∈ [owner, merchant_admin]
```

Hard backend enforcement.

---

## 2.2 Billing Must Always Have Fallback

Billing authority precedence:

```
ap_clerk (if active)
else owner
else merchant_admin
```

System must never reach a state where invoices cannot be paid.

---

## 2.3 Role Separation

| Role             | Responsibility                     | Recovery Authority |
|------------------|------------------------------------|--------------------|
| owner            | ultimate merchant authority        | YES                |
| merchant_admin   | operational + backup authority     | YES                |
| ap_clerk         | billing only                       | NO                 |
| store roles      | store-level operations             | NO                 |

---

## 2.4 Store is NOT Source of Authority

- Store primary contact ≠ owner
- Store assignment ≠ authority
- Merchant control must exist independent of store assignments

---

# 3. Identity Model

## 3.1 Merchant-Level Identity

Authoritative control exists at MerchantUser level:

- owner
- merchant_admin
- ap_clerk

## 3.2 Store-Level Identity

Derived, non-authoritative:

- StoreUser assignments
- Primary contact

---

# 4. Ownership Transfer (First-Class Operation)

Ownership transfer is NOT a simple edit.
It is a controlled lifecycle operation.

## 4.1 Required Steps

1. Select new owner (existing or create new)
2. Decide fate of current owner:
   - remove
   - demote to merchant_admin (recommended)
   - retain temporarily
3. Billing transition:
   - keep current AP Clerk
   - assign new AP Clerk
   - remove AP Clerk (fallback applies)
4. Validate rules

## 4.2 Validation Rules

Block operation if:

- No remaining owner/admin
- No billing fallback path

---

# 5. AP Clerk Model

## 5.1 Characteristics

- Optional role
- Billing authority only
- Does NOT count toward recovery

## 5.2 Constraints (V1)

- Max 1 active ap_clerk per merchant
- Can be replaced or removed freely

## 5.3 Behavior

If ap_clerk is removed:

- Billing automatically falls back to owner/merchant_admin

---

# 6. Admin Realm Responsibilities (pv_admin)

All recovery and structural authority flows belong here.

## 6.1 Capabilities

- Create Merchant Admin
- Promote user to owner / merchant_admin
- Transfer ownership
- Recover orphaned merchant
- Replace AP Clerk during transfer

## 6.2 Guarantees

Admin must always be able to:

- Re-establish merchant access
- Repair broken identity states

---

# 7. Merchant Realm Responsibilities

Allowed for owner / merchant_admin:

- Manage merchant users
- Assign roles (within rules)
- Manage AP Clerk
- Manage store assignments

---

# 8. AP Clerk Permissions

Allowed:

- View invoices
- Pay invoices
- View billing data

Not allowed:

- Ownership transfer
- Role elevation
- Merchant recovery

---

# 9. Backend Enforcement Rules

## 9.1 Prevent Orphan State

Reject operations that would result in:

```
0 users with role ∈ [owner, merchant_admin]
```

## 9.2 Billing Continuity

Ensure at least one of:

- active ap_clerk
- active owner
- active merchant_admin

## 9.3 Role Restrictions

Block ap_clerk from:

- role elevation
- ownership change

---

# 10. UI Contract

## 10.1 Role-Based Visibility

Do NOT render restricted actions for unauthorized roles.

### Hidden for ap_clerk

- Create Merchant Admin
- Promote to Owner/Admin
- Transfer Ownership
- Remove/Demote Admin

## 10.2 Principle

```
If user cannot execute action → action must not be visible
```

---

# 11. Recovery Scenario Coverage

System must support:

- Owner deleted
- Owner leaves company
- Merchant acquisition
- Billing contact changes
- Admin account lost

---

# 12. Immediate Implementation Priorities

## Phase 1 (Critical)

- Admin can create merchant_admin
- Backend blocks zero-admin state

## Phase 2

- AP Clerk management rules
- Billing fallback enforcement

## Phase 3

- Ownership transfer flow (UI + backend)

---

# 13. Summary

This contract ensures:

- No merchant becomes inaccessible
- Billing always functions
- Roles are clearly separated
- Ownership changes are controlled

---

End of Contract

