# project-management

A Claude Code plugin for decomposing project characterization documents into structured monday.com tasks for Optimate project management.

## What it does

- **Create tasks** from characterization documents with category-by-category brainstorming
- **Update and manage** existing monday.com tasks
- **Query board** for project status
- **Hebrew output** with checklists in updates
- **System-specific knowledge** for Fireberry, Make.com, and Monday.com
- **Opinionated suggestions** with flagged assumptions

## Prerequisites

A monday.com API token with access to the relevant board.

## Usage

The skill activates when working with Optimate project characterization documents. It guides task decomposition, subitem creation, update formatting, and board queries.

## Installation

```
/plugin install project-management@optimate-plugins
```

## Included References

- **SKILL.md** — Orchestrator skill with task decomposition workflow
- **MONDAY_API.md** — monday.com GraphQL API reference
- **DECOMPOSITION.md** — Task decomposition methodology
- **OPTIMATE_CONTEXT.md** — Optimate-specific project context
- **systems/** — Knowledge files for Fireberry, Make.com, and Monday.com
