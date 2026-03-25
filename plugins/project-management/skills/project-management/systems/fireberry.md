# Fireberry CRM

- **What it is:** CRM platform. Optimate builds custom objects, fields, views, and automations on it.
- **Typical task patterns:**
  - Building an object: create object → add fields → set field types (text, dropdown, lookup, formula) → configure permissions → set up views → order UX
  - Fields to consider: always include created/modified dates, status dropdown, owner lookup
  - Lookups: define relationships between objects early — they affect UX and reporting
- **Common edge cases:**
  - Dropdown vs. free text: prefer dropdowns for data consistency, free text only when values are truly unpredictable
  - Lookup direction: decide which object "owns" the relationship
  - Formula fields: can break if referenced fields change — document dependencies
  - Permission model: plan who can see/edit each object early
- **UX conventions:**
  - Group related fields in sections
  - Put most-used fields at the top
  - Required fields should be minimal to reduce friction
