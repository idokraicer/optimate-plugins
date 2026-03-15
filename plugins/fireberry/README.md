# fireberry

A Claude Code plugin for working with the Fireberry CRM API using the `fireberry-api-client` TypeScript library.

## What it does

- **Query** records using the fluent QueryBuilder API with auto-pagination
- **CRUD** operations with proper error handling
- **Batch** operations with auto-chunking (20 records per batch)
- **Metadata** inspection - objects, fields, dropdown values
- **Schema generation** - TypeScript interfaces from live API metadata
- **ERD generation** - Mermaid diagrams from schema relationships

## Prerequisites

```bash
bun add fireberry-api-client
```

## Usage

The skill activates automatically when working with Fireberry CRM data. It guides initialization (API mode, SDK mode, or Hybrid mode), query construction, batch operations, and error handling patterns.

## Included References

- **SKILL.md** — Complete integration guide with code patterns, best practices, and object type reference
