# Optimate Plugins

A Claude Code plugin marketplace for automation and integration tools.

## Installation

Add this marketplace to Claude Code:

```
/plugin marketplace add idokraicer/optimate-plugins
```

## Available Plugins

### make-fixer

Edit and build Make.com scenario blueprints directly from Claude Code.

```
/plugin install make-fixer@optimate-plugins
```

**Features:**
- Fetch, analyze, edit, validate, and push Make.com scenarios
- Complete function reference (70+ inline functions)
- Best practices for error handling, architecture, and cost optimization
- Automatic module type discovery and validation

**Prerequisites:** [make-fixer CLI](https://github.com/idokraicer/make-fixer) + [Bun](https://bun.sh)

### fireberry

Integration guide for the `fireberry-api-client` TypeScript library.

```
/plugin install fireberry@optimate-plugins
```

**Features:**
- Fluent QueryBuilder API with date helpers and auto-pagination
- CRUD, batch operations, streaming, metadata inspection
- Schema and ERD generation from live API
- Error handling patterns and object type reference

### fireberry-app-builder

Build and deploy custom Fireberry CRM apps with the platform SDK, CLI, and Design System.

```
/plugin install fireberry-app-builder@optimate-plugins
```

**Features:**
- CLI scaffolding (Vite + React)
- Component types: record, side-menu, global-menu
- `@fireberry/sdk` integration for iframe communication
- `@fireberry/ds` Design System (260+ icons, themed components)
- Local debugging with HMR
- Deployment workflow

## Contributing

To add a plugin to this marketplace, create a directory under `plugins/` following the [Claude Code plugin structure](https://code.claude.com/docs/en/plugins).

## License

MIT
