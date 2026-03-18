\# PerkValet Merchant Team Scope

Status: Locked (v1.0)



Purpose:

Define what the Merchant Team page represents and explicitly prevent misuse as a general employee directory.



Scope:

Merchant UI, backend identity model, and future UI/UX decisions.



------------------------------------------------------------



1\. Core Definition



The Merchant Team page represents:



→ People who participate in PerkValet workflows



A Team member is a person who:



• has a PerkValet identity (User)

• has a merchant membership (MerchantUser)

• participates in at least one PV-managed workflow



------------------------------------------------------------



2\. Included Roles (Canonical)



owner

merchant\_admin

merchant\_ap\_clerk

store\_admin

pos\_employee



These roles define participation in:



• POS operations

• store management

• billing workflows

• merchant administration



------------------------------------------------------------



3\. In-Scope Users



The following users belong on the Merchant Team page:



• Merchant owners and admins

• AP clerks handling invoices

• Store admins managing store operations

• POS employees interacting with promotions

• Store primary contacts



------------------------------------------------------------



4\. Out of Scope (Strict)



The following must NOT be included:



• general employee directory

• staff not using PerkValet

• HR / payroll-only personnel

• warehouse or back-office staff unrelated to PV

• placeholder or "just in case" users



------------------------------------------------------------



5\. Key Principle



The Merchant Team page is:



→ a PerkValet participation roster



NOT:



→ a full employee directory



This distinction must be preserved across:



• UI design

• backend modeling

• onboarding flows

• future feature development



------------------------------------------------------------



6\. Identity Model Alignment



Each Team member must map to:



User

&nbsp; └── MerchantUser

&nbsp;       └── (optional) StoreUser



No "lightweight" or "non-auth" users should be created

to represent non-PV staff.



------------------------------------------------------------



7\. UX Requirement (Deferred)



The Merchant Team page must clearly communicate its scope.



Future UI addition:



Helper text or tooltip:



"This list includes only employees who use or manage PerkValet (POS, store operations, billing, or administration)."



Optional placements:



• page header helper text

• "Add Employee" modal

• empty state messaging



------------------------------------------------------------



8\. Anti-Pattern Warning



Do NOT introduce:



• mixed user types (PV + non-PV in same list)

• flags like "non-system user"

• placeholder identities without login intent

• role-less users



These patterns lead to:



• identity ambiguity

• permission confusion

• audit inconsistencies

• long-term schema complexity



------------------------------------------------------------



9\. Future Extension (Optional)



If merchants require a broader employee list:



Introduce a separate feature:



→ "Staff Directory"



This would be a different model, such as:



MerchantStaff (non-auth entity)



This must remain separate from:



User / MerchantUser / StoreUser



------------------------------------------------------------



10\. Enforcement Guidance



UI must:



• only allow creation of valid PV users

• require role selection from canonical list

• prevent empty or incomplete identities



Backend must:



• enforce role enum validation

• reject invalid role values

• maintain strict User → MerchantUser relationship



------------------------------------------------------------



11\. Relationship to Role Model



This document complements:



PerkValet Role \& Contact Model (v1.0)



Specifically:



• reinforces merchant role usage

• prevents misuse of identity layer

• preserves separation of concerns



------------------------------------------------------------



Thread Reference



Thread: Identity / Auth / Team UI Stabilization

Focus: Merchant User lifecycle + UI contract alignment

