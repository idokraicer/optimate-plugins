# fireberry-app-builder

A Claude Code plugin for building and deploying custom Fireberry CRM apps using the platform SDK, CLI, and Design System.

## What it does

- **Scaffold** apps with `@fireberry/cli` (Vite + React)
- **Build** record, side-menu, and global-menu components
- **Integrate** `@fireberry/sdk` for iframe communication and context access
- **Style** with `@fireberry/ds` Design System (260+ icons, semantic colors, themed components)
- **Debug** locally with hot module replacement
- **Deploy** to Fireberry servers
- **Wrap** SDK with `fireberry-api-client` for QueryBuilder, caching, and batch operations

## Prerequisites

```bash
bun add -g @fireberry/cli
fireberry init
```

## Usage

The skill activates when building Fireberry platform apps. It guides scaffolding, component configuration, SDK setup, design system usage, local development, and deployment workflows.

## Included References

- **SKILL.md** — Complete app building guide with CLI commands, SDK API, Design System components, deployment workflow, and common patterns
