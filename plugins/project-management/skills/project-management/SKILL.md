---
name: project-management
description: Use when creating monday.com tasks from a project characterization document, updating existing monday.com tasks, or querying the monday.com board for Optimate project management
---

# Optimate Project Management

This skill helps Optimate PMs decompose project characterization documents into structured monday.com tasks, update existing tasks, or query board status. All output is in Hebrew.

## Bundled Reference Files — Load On Demand

This skill includes sub-files with deep reference material. **Read them when needed:**

- **[MONDAY_API.md](MONDAY_API.md)** — Load when making any monday.com API call. Contains GraphQL mutations, queries, column value formats, auth setup, and curl templates.

- **[DECOMPOSITION.md](DECOMPOSITION.md)** — Load when analyzing a characterization doc for task creation. Contains the 5 default decomposition categories, sub-decomposition rules, Hebrew naming conventions, and HTML update templates.

- **[OPTIMATE_CONTEXT.md](OPTIMATE_CONTEXT.md)** — Load when suggesting implementation approaches or estimating hours. Contains Optimate's business context and estimation guidelines.

- **systems/** directory — When the characterization doc mentions specific systems, load matching files from this directory:
  - `systems/fireberry.md` — Fireberry CRM patterns and edge cases
  - `systems/make.md` — Make.com automation patterns and cost considerations
  - `systems/monday.md` — Monday.com board/column patterns
  - Additional system files may be added over time. If no matching file exists for a mentioned system, note it and work with general knowledge.

## Setup

On first use:

1. Check if `~/.project-management/.env` exists with `MONDAY_API_TOKEN`
2. If not, ask the user for their monday.com API token
3. Store it:
   ```bash
   mkdir -p ~/.project-management && echo "MONDAY_API_TOKEN=<token>" > ~/.project-management/.env
   ```
4. Load the token:
   ```bash
   source ~/.project-management/.env
   ```
5. Run column discovery (see MONDAY_API.md) on board `9116155603` and note the column IDs for future use

## Entry Point

When the skill is invoked, ask the PM:

**"מה תרצה לעשות?"**

1. **יצירת משימות** מתוך מסמך אפיון (Create tasks from characterization doc)
2. **עדכון/ניהול** משימות קיימות (Update/manage existing tasks)
3. **שאילתת** הבורד לסטטוס ומידע (Query the board)

---

## Flow 1: Task Creation from Characterization Doc

This is the core flow.

### Step 1 — Receive the doc

Accept any format: Google Docs (use gws-docs skill if available), PDF, Word (.docx), or plain text pasted into chat.

### Step 2 — Parse and analyze

Read the document thoroughly. Identify all requirements, systems mentioned, and scope.

### Step 3 — Load system knowledge

Scan the doc for system mentions (Fireberry, Make.com, Monday.com, etc.). Load matching files from `systems/` directory. If no matching file exists, note the system and work with general knowledge.

### Step 4 — Load decomposition guide

Read DECOMPOSITION.md for the 5 default categories and templates.

### Step 5 — Load business context

Read OPTIMATE_CONTEXT.md for estimation guidelines.

### Step 6 — Ask PM for bulk settings

In a single question, ask:

- מי צריך להיות מוקצה למשימות? (Who should be assigned? Supports "all to X" or per-category)
- מהו הפרויקט הרלוונטי? (What's the linked project? Search projects board `7109422573`)
- יש אילוצי דדליינים? (Any deadline constraints?)

The PM can give sweeping answers: "הכל לעדן", "אינטגרציות עד 1/5/2027".

### Step 7 — Identify categories

From the 5 defaults in DECOMPOSITION.md, determine which apply to this project. Propose any additional custom categories if the doc warrants them.

### Step 8 — Category-by-category brainstorming

For each relevant category:

1. Present the proposed breakdown: item name, subitems, suggested hour estimates
2. For each subitem, suggest: implementation approach (opinionated based on system knowledge), edge cases, UX considerations
3. Flag all assumptions with `[הנחה]` prefix
4. Ask PM to approve, adjust, or skip
5. On approval: load MONDAY_API.md and create the item, subitems, and updates with checklists via API
6. Report created item IDs back to PM

### Step 9 — Summary

After all categories, present a summary table of everything created: item names, subitem counts, assigned persons, deadlines.

---

## Flow 2: Update/Manage Existing Tasks

1. Ask PM what they want to update
2. Load MONDAY_API.md
3. Query board `9116155603` to find matching items/subitems
4. Present findings in Hebrew for confirmation
5. Execute changes via `change_multiple_column_values` mutation
6. Support bulk operations: "סמן הכל כ-Done", "העבר את כל המשימות של דאשבורדים לסער"

---

## Flow 3: Query Board

1. Ask PM what they want to know
2. Load MONDAY_API.md
3. Query board `9116155603`
4. Present readable summary in Hebrew
5. Support filters: by status, assignee, project, category

---

## Key Behaviors

- **Hebrew output** — All task names, updates, and checklists in Hebrew
- **Brief yet complete** — Updates should contain everything an implementer needs: requirement, suggested approach, edge cases, UX notes
- **Opinionated but transparent** — Suggest implementation approaches based on loaded system knowledge. Always flag assumptions with `[הנחה]`
- **Bulk-friendly** — Support sweeping PM answers ("all tasks to Eden", "integrations due 1/5/2027")
- **Progressive creation** — Create tasks in monday.com as each category is approved, not all at the end
- **RTL support** — Use `dir='rtl'` in HTML update content for proper Hebrew rendering
