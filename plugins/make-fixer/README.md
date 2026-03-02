# make-fixer

A Claude Code plugin for editing and building Make.com scenario blueprints.

## What it does

- **Fetch** scenario blueprints from Make.com via API
- **Analyze** scenarios for issues and optimization opportunities
- **Edit** blueprints directly using Claude's editing tools
- **Validate** changes against the remote version
- **Push** updated blueprints back to Make.com

## Prerequisites

The `make-fixer` CLI must be installed:

```bash
git clone https://github.com/idokraicer/make-fixer.git ~/.make-fixer
cd ~/.make-fixer && bun install && bun link
```

Requires [Bun](https://bun.sh).

## Setup

Configure your Make.com API token:

```bash
make-fixer login --token <YOUR_TOKEN>
```

Optionally set a custom base URL:

```bash
make-fixer login --token <token> --base-url https://us1.make.com
```

## Usage

Once installed, just mention a Make.com scenario ID or URL in Claude Code and the skill will automatically activate. You can also invoke it directly:

```
/make-fixer:make-fixer 4227637
```

## Included References

- **SKILL.md** — Main skill with workflow, blueprint structure, and rules
- **BEST_PRACTICES.md** — Error handling, debugging, testing, naming, architecture patterns
- **FUNCTIONS_REFERENCE.md** — Complete catalog of 70+ inline functions, data types, filter operators, and date/time tokens
