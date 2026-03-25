# PerkValet Role & Permission Model
## Version 2.00
Status: Locked (pending implementation)

---

## 1. Core Principle

PerkValet separates three distinct concepts:

### Platform Role
Who the user is within PerkValet (internal system)

### Merchant Role
Who the user is within a merchant organization

### Store Assignment & Permission
Where the user works and what they can do at that store

---

## 2. Platform Roles (PV Internal)

- pv_admin
- pv_support
- pv_qa
- pv_ar_clerk

**Example:**  
A support agent helping a merchant → `pv_support`

---

## 3. Merchant Roles (Organizational Identity)

- owner  
- merchant_admin  
- merchant_ap_clerk  
- merchant_employee  

**Examples:**

- Owner → business owner  
- Merchant Admin → regional manager  
- AP Clerk → accounting  
- Merchant Employee → cashier / staff  

**Rule:** Merchant roles define identity, NOT store permissions.

---

## 4. Store Model

### 4A. Store Assignment

Determines if a person works at a store.

**Example:**  
Mary works at Store #1 but not Store #2.

---

### 4B. Store Permissions

Defines what a person can do at a store.

### StorePermissionLevel (enum)

- store_admin
- store_subadmin
- pos_access

**Examples:**

- store_admin → store manager  
- store_subadmin → assistant manager  
- pos_access → cashier  

**Rule:** Store permissions define capabilities, NOT identity.

---

## 5. Merchant Users Page

Manages:
- identity
- merchant role

Does NOT manage:
- store assignment
- store permissions

---

## 6. Store Page (Team & Access)

Manages:

### Assignment
- add employee to store
- remove employee from store

### Permissions
- assign:
  - store_admin
  - store_subadmin
  - pos_access

### Primary Contact
- assign/remove store contact

---

## 7. Canonical Workflow

### Cashier

1. Create user  
2. Role = merchant_employee  
3. Assign to store  
4. Permission = pos_access  

---

### Store Manager

1. Create user  
2. Assign to store  
3. Permission = store_admin  

---

## 8. Key Rules

- Merchant role ≠ Store permission  
- Store assignment ≠ Permission  
- POS is a permission, not a role  
- No fallback to admin  
- No mixing roles and permissions  

---

## 9. Schema Alignment

### MerchantRole
- includes merchant_employee
- excludes store_admin and store_subadmin

### StorePermissionLevel

- store_admin
- store_subadmin
- pos_access

---

## 10. Final Model

| Layer | Question | Example |
|------|--------|--------|
| Merchant | Who are you? | Employee |
| Store | Do you work here? | Yes |
| Permission | What can you do? | POS |

---

## Summary

Mary is a merchant employee, assigned to Store #1, with POS access.
