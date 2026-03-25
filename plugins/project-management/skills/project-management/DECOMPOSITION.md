# Project Decomposition Guide

## Overview

These are the 5 default categories for decomposing project characterization documents into actionable tasks. The agent should adapt this framework to each project — skip categories that are not relevant, and add custom categories based on what the characterization doc describes.

Each category becomes a **main item** on the tasks board, with **subitems** for specific tasks within that category. The templates and rules below guide how to name items, structure subitems, and write updates.

---

## Category 1: קבלת פרטי גישה (Access Details)

**Purpose:** Collect all system access credentials and connections needed before work can begin.

**Rule:** Create one subitem per system mentioned in the characterization doc.

**Status:** Leave the status empty until the PM confirms that access exists.

**Update template:**
- List of systems needing access
- Credentials or permissions needed for each system
- Contact person responsible for providing access (if known)

**Example item name:** `קבלת פרטי גישה והתחברות למערכות`

**Example subitems:**
- `גישה ל-Fireberry`
- `גישה ל-Make.com`
- `גישה למערכת הנהלת חשבונות`

---

## Category 2: תשתית מערכת (System Infrastructure)

**Purpose:** Build or configure the data objects and fields required by the project.

**Rule:** Break down the characterization doc into individual objects. For each object, make a key decision:
- **If the object already exists:** Create a subitem named `הוספת שדות לאובייקט X` (add fields to object X)
- **If the object is new:** Create a subitem named `בניית אובייקט X` (build object X)

**UX ordering:** If multiple objects are involved, add a separate subitem `סידור UX` to cover layout, views, and user experience considerations.

**Update template per subitem:** Use a checklist of fields to add/configure, edge cases, and relationships to other objects (see HTML templates section below).

**Example item name:** `בניית תשתית מערכת`

**Example subitems:**
- `בניית אובייקט לקוחות`
- `בניית אובייקט דוחות שנתיים`
- `סידור UX`

---

## Category 3: אוטומציות ואינטגרציות (Automations & Integrations)

**Purpose:** Define and implement all automations and third-party integrations.

**Rule:** Create one subitem per automation or integration identified in the characterization doc.

**Detail each subitem with:**
- Trigger: what initiates the automation
- Action: what happens as a result
- Systems connected: which platforms are involved
- Data flow: what data is passed between systems

**Testing:** Add a separate subitem `הרצות וריאנטים` for testing variations and edge cases before finalizing.

**Update template:** Use the automation checklist template (see HTML templates section below) including edge cases and failure scenarios with potential solutions.

**Example item name:** `אוטומציות ואינטגרציות`

**Example subitems:**
- `אוטומציה - שליחת מייל ללקוח חדש`
- `אינטגרציה - סנכרון מערכת הנהלת חשבונות`
- `הרצות וריאנטים`

---

## Category 4: רכישת מערכות (System Procurement)

**Purpose:** Handle purchasing or setting up any systems or licenses the project requires.

**Rule:** Only create this category if the characterization doc mentions systems that need to be purchased or set up from scratch.

**Split into two groups:**
- Existing systems that need license upgrades or renewals
- New systems that need to be acquired and onboarded

**Example item name:** `רכישת מערכות`

**Example subitems:**
- `רכישת רישיון Make.com`
- `הקמת חשבון Fireberry`

---

## Category 5: דאשבורדים (Dashboards)

**Purpose:** Design and build reporting dashboards based on project requirements.

**Rule:** Create one subitem per dashboard mentioned or implied in the characterization doc.

**Update template per subitem:**
- What data to display
- Filters and grouping options
- Access permissions (who can view)

**Example item name:** `דאשבורדים`

**Example subitems:**
- `דאשבורד מעקב לקוחות`
- `דאשבורד ביצועי צוות`

---

## Dynamic Categories

Some projects will have requirements that do not fit neatly into the 5 default categories above. When the agent identifies such requirements, it should **propose them to the PM as additional main items** before proceeding.

Common examples of dynamic categories:
- `הדרכות` — training sessions for the client or end users
- `מיגרציית נתונים` — migrating existing data from legacy systems
- `בדיקות` — structured testing phases (UAT, regression, etc.)

Always surface these as suggestions, not decisions — the PM approves the final scope.

---

## Update Format Templates

Use these HTML templates when writing updates to monday.com subitems.

### Template: Field Checklist Update (for תשתית מערכת subitems)

```html
<p><b>שדות נדרשים:</b></p>
<ul class='checklist'>
  <li class='checklist_task' dir='rtl'>FIELD_NAME - FIELD_TYPE - DESCRIPTION</li>
</ul>
<p><b>מקרי קצה:</b></p>
<ul class='checklist'>
  <li class='checklist_task' dir='rtl'>EDGE_CASE - SUGGESTED_SOLUTION</li>
</ul>
<p><b>הערות UX:</b></p>
<ul class='checklist'>
  <li class='checklist_task' dir='rtl'>UX_NOTE</li>
</ul>
```

### Template: Automation Update (for אוטומציות ואינטגרציות subitems)

```html
<p><b>תיאור:</b> DESCRIPTION</p>
<p><b>טריגר:</b> TRIGGER</p>
<p><b>פעולה:</b> ACTION</p>
<p><b>מערכות מעורבות:</b> SYSTEMS</p>
<p><b>מקרי קצה ופתרונות אפשריים:</b></p>
<ul class='checklist'>
  <li class='checklist_task' dir='rtl'>EDGE_CASE - SOLUTION</li>
</ul>
```

---

## Assumption Flagging

When the agent makes an assumption about a requirement that was not explicitly stated in the characterization doc, it must prefix the relevant update line with `[הנחה]`.

**Format:**
```html
<li class='checklist_task' dir='rtl'>[הנחה] ASSUMPTION_TEXT</li>
```

**Example:**
```html
<li class='checklist_task' dir='rtl'>[הנחה] שדה מסוג dropdown - ניתן לשנות לטקסט חופשי</li>
```

This ensures the PM can quickly review all assumptions and confirm or correct them before implementation begins.
