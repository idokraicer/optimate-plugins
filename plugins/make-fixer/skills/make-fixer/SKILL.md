---
name: make-fixer
description: Fetch, analyze, edit, build, and push Make.com (formerly Integromat) scenario blueprints via the make-fixer CLI. Use when the user mentions Make.com scenarios, blueprints, modules, automation workflows, routers, webhooks, operations cost, or wants to create, debug, optimize, or improve any Make.com automation. Also use when the user asks about Make.com inline functions, expressions, formatDate, parseDate, ifempty, switch, get, map, or any Make.com function syntax — load FUNCTIONS_REFERENCE.md for the complete verified reference. Covers blueprint JSON structure, module mapper formats, variable patterns (Set/Get), error handlers, filters, cost optimization, and community best practices (BEST_PRACTICES.md). This skill and its bundled sub-files are the primary source of truth for all Make.com work.
user-invocable: true
---

# Make.com Scenario Editor & Builder

You can fetch, edit, validate, and push Make.com scenario blueprints using the `make-fixer` CLI. You can also design new scenarios from scratch through collaborative brainstorming.

## Bundled Reference Files — Load On Demand

This skill includes sub-files with deep reference material. **Read them when needed:**

- **[FUNCTIONS_REFERENCE.md](FUNCTIONS_REFERENCE.md)** — Complete catalog of all Make.com inline functions (70+), date/time formatting/parsing tokens, filter operators, data types, and type coercion rules. **Load this file whenever:**
  - The user asks about any Make.com function, expression syntax, or formula
  - You need to write or verify an inline expression in a blueprint
  - You need date formatting tokens, filter operator codes, or type coercion behavior
  - You're unsure whether a function exists (this file is the canonical list — if it's not here, it doesn't exist)

- **[BEST_PRACTICES.md](BEST_PRACTICES.md)** — Error handling patterns, debugging workflows, testing strategies, naming conventions, architecture patterns (webhook chains, queue-based processing), production settings, AI integration patterns, cost optimization checklist, and data store design. **Load this file whenever:**
  - Designing a new scenario from scratch
  - Reviewing or improving an existing scenario's architecture
  - Adding error handlers or debugging a failing scenario
  - The user asks about Make.com best practices, patterns, or production readiness

## Installation

Before first use, check if `make-fixer` is available by running `make-fixer --version`. If the command is not found, install it:

```bash
git clone https://github.com/idokraicer/make-fixer.git ~/.make-fixer
cd ~/.make-fixer && bun install && bun link
```

This registers the `make-fixer` command globally. Requires [Bun](https://bun.sh) (`curl -fsSL https://bun.sh/install | bash`).

## Setup

The API token is stored globally at `~/.make-fixer/.env` and works from any directory.

To configure, run: `make-fixer login --token <YOUR_TOKEN>`

**IMPORTANT:** Always use `--token <token>` flag. Do NOT run bare `make-fixer login` — the interactive prompt does not work inside Claude Code.

Optionally set a custom base URL: `make-fixer login --token <token> --base-url https://us1.make.com` (default: `https://eu1.make.com`). Running login again updates the existing token.

If a command fails with "MAKE_API_TOKEN not found", ask the user for their token and run `make-fixer login --token <token>`.

## Workflow

<HARD-GATE>
Do NOT edit the blueprint, push changes, or take any implementation action until you have presented a plan of changes and the user has approved it. This applies to EVERY scenario regardless of perceived simplicity.
</HARD-GATE>

**URLs are supported directly.** You can pass a full Make.com URL to `-s` — the zone and scenario ID are auto-detected:
```bash
make-fixer fetch -s "https://eu2.make.com/1490053/scenarios/8268971/edit"
```
This works with all zones (eu1, eu2, us1, us2, etc.). Bare scenario IDs still work and use the globally configured zone.

### Determine Mode

Based on what the user provides, choose the right starting point:

- **Existing scenario** (ID or URL provided): Start at Phase 1 — Gather Context
- **New scenario idea** (description of what they want to build): Start at Phase 2 — Brainstorm & Clarify
- **Vague request** ("help me with my automations", "I need a Make scenario"): Start at Phase 2 — Brainstorm & Clarify

### Phase 1: Gather Context (existing scenarios)

When the user provides a scenario ID or URL — immediately fetch and analyze:

```bash
make-fixer fetch -s <ID-or-URL> && make-fixer analyze -s <ID> --local
```

Then **read** the blueprint file (`blueprints/<id>.json`) to understand the scenario structure: what modules exist, how they connect, what the scenario does, and what issues were found.

Present a brief summary to the user:
- Scenario name and what it does (1-2 sentences)
- Module count and flow structure (linear, branching, etc.)
- Issues found by `analyze` (if any)
- **Proactive improvement suggestions** (see below)

#### Proactive Improvement Suggestions

After analyzing a scenario, always look for and suggest improvements — even if the user didn't ask. Present these as observations, not demands:

- **Cost optimization**: Modules that could be consolidated, redundant steps, logic that could be combined into fewer modules. Every module costs operations.
- **Missing error handlers**: Modules making external API calls without `onerror` handlers
- **Fragile patterns**: Hardcoded values that should use variables, missing `ifempty` fallbacks on fields that could be null
- **Naming**: Modules with default names that don't describe what they do
- **Structure**: Overly complex routing that could be simplified, unnecessary intermediate modules
- **Reliability**: Missing retry logic, no fallback paths for critical operations

Frame suggestions as: "I also noticed a few things that could be improved — want me to address any of these along with your main request?"

### Phase 2: Brainstorm & Clarify

Ask questions **one at a time** to understand what the user wants to achieve. Prefer multiple-choice questions when possible (using the AskUserQuestion tool), but open-ended is fine too.

**For existing scenarios**, focus on:
- **What** they want to change and **why**
- Which modules or parts of the flow are involved
- Whether they want to fix existing issues, add new functionality, or restructure
- Any constraints (e.g. "don't touch module X", "must keep the existing webhook")
- Whether they also want to address any of the proactive improvement suggestions

**For new scenarios**, focus on:
- **What triggers the scenario?** (webhook, schedule, watching a service, etc.)
- **What's the end goal?** What should happen when the scenario runs successfully?
- **What services/APIs are involved?** (CRM, email, spreadsheets, HTTP APIs, etc.)
- **What data flows between steps?** What inputs does each step need?
- **Error handling requirements?** What should happen when something fails?
- **Volume & frequency?** How often does this run? How many records per run?
- **Edge cases?** What if a field is empty? What if an API is down?

Use the analysis results and blueprint structure to ask **informed** questions. Reference specific modules by their name/ID when asking about existing scenarios.

**Do NOT batch questions.** One question per message. If a topic needs more exploration, break it into multiple questions.

**Keep asking until you have a clear picture.** Don't rush to propose changes. It's better to ask one more question than to propose the wrong thing.

### Phase 3: Propose Approaches

Once you understand what needs to happen:

1. **Propose 2-3 approaches** when there are meaningful alternatives
   - Describe each approach briefly
   - List trade-offs (cost in operations, complexity, reliability, maintainability)
   - Lead with your recommendation and explain why
2. For straightforward changes, present a single clear plan
3. List the specific edits: which modules change, what gets added/removed, what fields are modified
4. For new scenarios, describe the full flow: trigger → processing → output, with module types
5. **Always consider cost**: can this be done with fewer modules? Can steps be combined?

Get explicit user approval before proceeding.

### Phase 4: Edit and Validate

Only after the user approves your plan:

1. **Look up module types** if needed: `make-fixer apps <query>` and `make-fixer modules <app>` to find correct module type strings
2. **Edit** the blueprint file directly using your Edit tool
3. **Validate** changes: `make-fixer validate -s <id>` — shows diff vs remote (added/removed/modified modules, issue count)
4. Present the validation results to the user
5. If validation reveals issues, fix them and re-validate

### Phase 5: Push

1. **Always ask the user** for explicit confirmation before pushing
2. **Push** when confirmed: `make-fixer push -s <id> --yes`
3. Confirm success to the user

### Anti-Patterns

**"This Is Too Simple To Need Questions"** — Even "just add an error handler" requires understanding: which modules? what retry settings? should it break or ignore? A quick question prevents wasted work. The questions can be short for simple tasks, but you MUST ask before editing.

**"I'll just add all the modules"** — More modules = more cost. Always look for ways to consolidate. Three similar HTTP calls might be replaceable with one parameterized call inside an iterator. Question every module: is this strictly necessary?

**"The user said what they want, I can just build it"** — The user described their goal, not the design. You need to understand the design before building. Ask about edge cases, error handling, and data flow before proposing a plan.

## Blueprint Structure

A blueprint is a JSON object:

```json
{
  "name": "Scenario Name",
  "flow": [
    {
      "id": 1,
      "module": "gateway:CustomWebHook",
      "version": 1,
      "mapper": { ... },
      "metadata": {
        "designer": { "x": 0, "y": 0, "name": "Custom Name" }
      },
      "onerror": [
        { "id": 2, "module": "builtin:Break", "mapper": { "retry": true, "count": 3, "interval": 60 } }
      ],
      "routes": [
        { "flow": [ ... nested modules ... ] }
      ]
    }
  ]
}
```

### Key fields

- **id**: Unique integer. Never reuse. New modules must use the next available ID (shown by `fetch` and `validate` commands).
- **module**: Module type string like `gmail:sendEmail`, `powerlink:plquery`, `http:ActionSendData`
- **mapper**: Module configuration (API URLs, field mappings, query parameters)
- **metadata.designer.name**: Custom display name in the Make.com UI
- **onerror**: Error handler array. Standard break handler: `[{ "id": N, "module": "builtin:Break", "version": 1, "mapper": { "retry": true, "count": 3, "interval": 1 } }]`. The `interval` is in **minutes** (1 = 1 minute, 60 = 1 hour). When `mapper` is omitted, Make.com defaults to `count: 3, interval: 15`.
- **routes**: Array of route objects, each containing a `flow` array (for router modules)
- **filter**: Filter condition between modules. Format: `{ "name": "Filter Name", "conditions": [[{ "a": "{{1.field}}", "b": "value", "o": "equal" }]] }`. Inner array = AND conditions, outer array = OR groups. Available operators: `equal`, `notEqual`, `greater`, `less`, `greaterOrEqual`, `lessOrEqual`, `text:equal`, `text:notEqual`, `text:startsWith`, `text:endsWith`, `text:contains`, `text:notContains`, `text:matches` (regex), `exist`, `notExist`, `array:contains`, `array:notContains`

### Module types

Excluded from checks (utility): `builtin:*`, `gateway:*`, `json:*`, `tools:*`, `util:*`, `flow:*`, `code:*`

Common types:
- Triggers: `gateway:CustomWebHook`, `google-sheets:watchRows`
- API calls: `http:ActionSendData`, `http:MakeRequest`
- CRM: `powerlink:plquery`, `powerlink:createObject`, `powerlink:updateObject`
- Communication: `gmail:sendEmail`, `slack:sendMessage`
- Routing: `flow:Router` (has `routes` array)

### Expression syntax

Expressions use `{{...}}` delimiters inside module mapper fields.

**Critical rules:**
- **Arguments are separated by semicolons `;`** — NEVER use commas. Example: `{{if(1.status = "active"; "Yes"; "No")}}`
- **Module output references:** `{{moduleId.fieldName}}` — e.g. `{{1.phone}}`, `{{3.data.email}}`
- **Nested data:** use dot notation — `{{1.body.results.0.name}}`
- **Array indexing starts at 1** (not 0) when using `get()` — `{{get(1.items; 1)}}` returns first item
- **Functions can be nested:** `{{upper(substring(1.name; 0; 1))}}` — uppercase first letter
- **`now` and `timestamp` are variables, NOT functions** — write `{{now}}` not `{{now()}}`
- **Comparison operators in expressions:** `=`, `!=`, `<`, `>`, `<=`, `>=`
- **Logical operators:** `and`, `or`, `not`
- **Arithmetic:** `+`, `-`, `*`, `/`, `%`

### Variable references

Modules reference other modules' output with `{{moduleId.fieldName}}`:
- `{{1.phone}}` — field `phone` from module #1
- `{{ifempty(1.phone; 0121231234)}}` — with fallback

### `ifempty` fallback values

`ifempty` is used to prevent null/empty values from breaking API calls or queries. The fallback must:
1. **Match the data type of the field** — an email field gets a fake email, a phone field gets a fake phone number, etc.
2. **Never actually match a real record** — the purpose is to let the query run and return zero results.
3. **Never use quoted strings for numeric-like fields** — quotes break most APIs. Write `0121231234` not `"0121231234"`.
4. **Be a realistic-looking value** — not random gibberish, not empty strings, not single characters.

Examples:
- Email field: `{{ifempty(6.email; nomatch@invalid.test)}}`
- Phone field: `{{ifempty(6.phone; 0121231234)}}`
- ID/UUID field: generate a random UUID for each `ifempty` — e.g. `{{ifempty(6.id; f47ac10b-58cc-4372-a567-0e02b2c3d479)}}`. Never reuse the same UUID across different `ifempty` calls.

### Set/Get Variables

There are 4 variable modules — use them correctly:

**Set Variable** (`util:SetVariable2`) — sets a single variable:
```json
{ "mapper": { "name": "myVar", "scope": "roundtrip", "value": "{{1.someField}}" } }
```

**Set Multiple Variables** (`util:SetVariables`) — sets multiple variables at once:
```json
{ "mapper": { "variables": [{ "name": "x", "value": "{{1.field1}}" }, { "name": "y", "value": "{{1.field2}}" }], "scope": "roundtrip" } }
```

**Get Variable** (`util:GetVariable2`) — retrieves a single variable:
```json
{ "mapper": { "name": "myVar" } }
```

**Get Multiple Variables** (`util:GetVariables`) — retrieves multiple variables. **Variables are plain strings, NOT objects:**
```json
{ "mapper": { "variables": ["x", "y"] } }
```

**Variable lifetime (`scope`):**
- `"roundtrip"` = **One Cycle** — persists for one cycle of the scenario
- `"execution"` = **One Execution** — persists for the entire scenario execution

**Important behavior:** Variables with `"roundtrip"` scope do NOT reset between iterator iterations. They persist until explicitly overwritten. If you need per-iteration isolation, use a different pattern (e.g., inline expressions or aggregators).

**Referencing variable output in downstream modules:**
- `util:GetVariables` (multiple): outputs use variable names — `{{moduleId.myVarName}}`
- `util:GetVariable2` (single): output is always named `value` — `{{moduleId.value}}`

**Cost:** Prefer multi-variable modules. `util:SetVariables` with 5 vars = **1 operation**. Five `util:SetVariable2` = **5 operations**.

**Variable naming:** Case-sensitive (`"myVar"` and `"myvar"` are different). Literal strings only — expressions NOT supported in name fields. Typos between Set and Get cause **silent null returns** (no error thrown).

**When to use Set/Get Variables vs inline expressions:**
- Use variables when you need to **pass data between different routes** (after a Router)
- Use variables when a value is **computed once but referenced many times** across distant modules
- Use **inline expressions** (`{{1.field}}`) when data flows linearly — direct module references are simpler, cheaper (no extra module), and clearer

### Inline functions reference

<HARD-RULE>
**NEVER fabricate, guess, or invent inline functions.** Only use functions listed below. If you're unsure whether a function exists, do NOT use it. There is no `dateDifference`, `startOfDay`, `endOfDay`, `toBoolean`, `toNumber`, `log`, `ln`, or `values` function. Using a non-existent function will cause the scenario to fail.
</HARD-RULE>

**General functions:**

| Function | Syntax | Returns |
|---|---|---|
| `get` | `get(object_or_array; path)` | Value at path (dot notation, 1-based arrays) |
| `if` | `if(expression; value1; value2)` | value1 if true, else value2 |
| `ifempty` | `ifempty(value1; value2)` | value1 if not empty, else value2 |
| `switch` | `switch(expr; val1; res1; [val2; res2; ...]; [else])` | Result matching first value |
| `omit` | `omit(collection; key1; [key2; ...])` | Collection without specified keys |
| `pick` | `pick(collection; key1; [key2; ...])` | Collection with only specified keys |
| `coalesce` | `coalesce(value1; value2; [...])` | First non-null/non-empty value |

**String functions:**

| Function | Syntax | Returns |
|---|---|---|
| `ascii` | `ascii(text; [removeDiacritics])` | ASCII-only string |
| `base64` | `base64(text)` | Base64-encoded string |
| `capitalize` | `capitalize(text)` | First char uppercased |
| `contains` | `contains(text; searchString)` | Boolean |
| `decodeURL` | `decodeURL(text)` | URL-decoded string |
| `encodeURL` | `encodeURL(text)` | URL-encoded string |
| `escapeHTML` | `escapeHTML(text)` | HTML-escaped string |
| `indexOf` | `indexOf(text; searchString; [start])` | Position (number) |
| `join` | `join(array; separator)` | Joined string |
| `length` | `length(text_or_buffer)` | Character count or buffer bytes |
| `lower` | `lower(text)` | Lowercase string |
| `md5` | `md5(text)` | MD5 hex hash |
| `replace` | `replace(text; search; replacement)` | Modified string (supports regex) |
| `replaceEmojiCharacters` | `replaceEmojiCharacters(text; replacement)` | Emoji-free string |
| `sha1` | `sha1(text; [encoding]; [key])` | SHA-1 hash |
| `sha256` | `sha256(text; [encoding]; [key]; [keyEncoding])` | SHA-256 hash |
| `sha512` | `sha512(text; [encoding]; [key]; [keyEncoding])` | SHA-512 hash |
| `split` | `split(text; separator)` | Array of strings |
| `startcase` | `startcase(text)` | Title Case String |
| `stripHTML` | `stripHTML(text)` | Plain text (HTML removed) |
| `substring` | `substring(text; start; [length])` | Substring |
| `toBinary` | `toBinary(value; [encoding])` | Binary data |
| `toString` | `toString(value)` | String representation |
| `trim` | `trim(text)` | Whitespace-trimmed string |
| `upper` | `upper(text)` | UPPERCASE string |

**Math functions:**

| Function | Syntax | Returns |
|---|---|---|
| `abs` | `abs(number)` | Absolute value |
| `average` | `average(value1; ...) or average(array)` | Mean |
| `ceil` | `ceil(number)` | Rounded up integer |
| `floor` | `floor(number)` | Rounded down integer |
| `formatNumber` | `formatNumber(number; decimalPoints; [decSep]; [thousSep])` | Formatted string |
| `max` | `max(value1; ...) or max(array)` | Maximum value |
| `median` | `median(array)` | Median value |
| `min` | `min(value1; ...) or min(array)` | Minimum value |
| `mod` | `mod(dividend; divisor)` | Remainder |
| `parseNumber` | `parseNumber(text; [decimalSeparator])` | Number |
| `pow` | `pow(base; exponent)` | Power result |
| `round` | `round(number; [decimalPlaces])` | Rounded number |
| `sqrt` | `sqrt(number)` | Square root |
| `sum` | `sum(value1; ...) or sum(array)` | Total |
| `trunc` | `trunc(number; [decimalPlaces])` | Truncated number |

**Math variables (no parentheses):** `random` (float 0–1), `pi` (3.14159...)

**Date/time functions:**

| Function | Syntax | Returns |
|---|---|---|
| `formatDate` | `formatDate(date; format; [timezone])` | Formatted date string |
| `parseDate` | `parseDate(text; format; [timezone])` | Date object |
| `addDays` | `addDays(date; number)` | Date (negative subtracts) |
| `addHours` | `addHours(date; number)` | Date |
| `addMinutes` | `addMinutes(date; number)` | Date |
| `addMonths` | `addMonths(date; number)` | Date |
| `addSeconds` | `addSeconds(date; number)` | Date |
| `addYears` | `addYears(date; number)` | Date |
| `setDate` | `setDate(date; number)` | Date (day of month 1–31) |
| `setDay` | `setDay(date; number_or_name)` | Date (day of week, 0=Sun) |
| `setHour` | `setHour(date; number)` | Date (0–23) |
| `setMinute` | `setMinute(date; number)` | Date (0–59) |
| `setMonth` | `setMonth(date; number_or_name)` | Date (1–12) |
| `setSecond` | `setSecond(date; number)` | Date (0–59) |
| `setYear` | `setYear(date; number)` | Date |

**Date/time variables (no parentheses):** `now` (current datetime), `timestamp` (Unix seconds)

**Common format tokens for `formatDate`/`parseDate`:** `YYYY` (year), `MM` (month 01–12), `DD` (day 01–31), `HH` (24h hours), `hh` (12h hours), `mm` (minutes), `ss` (seconds), `X` (Unix timestamp), `ddd`/`dddd` (weekday short/full)

**Array functions:**

| Function | Syntax | Returns |
|---|---|---|
| `add` | `add(array; value1; [...])` | Array with added values |
| `contains` | `contains(array; value)` | Boolean |
| `deduplicate` | `deduplicate(array)` | Unique primitives |
| `distinct` | `distinct(array; [key])` | Unique by key (for objects) |
| `first` | `first(array)` | First element |
| `flatten` | `flatten(array; [depth])` | Flattened array |
| `join` | `join(array; separator)` | String |
| `keys` | `keys(object)` | Array of key names |
| `last` | `last(array)` | Last element |
| `length` | `length(array)` | Count |
| `map` | `map(array; key; [filterKey]; [filterValues])` | Extracted values |
| `merge` | `merge(array1; array2; [...])` | Merged array |
| `remove` | `remove(array; value1; [...])` | Array without values |
| `reverse` | `reverse(array)` | Reversed array |
| `shuffle` | `shuffle(array)` | Randomized array |
| `slice` | `slice(array; start; [end])` | Sub-array (1-based) |
| `sort` | `sort(array; [order]; [key])` | Sorted (asc/desc/asc ci/desc ci) |
| `toArray` | `toArray(collection)` | Array of {key, value} pairs |
| `toCollection` | `toCollection(array; keyProp; valueProp)` | Collection object |

**Special values:** `true`, `false`, `null`, `emptystring` (forces empty string vs null)

## Rules

1. **Never reuse module IDs.** Check the "Next ID" shown by `fetch` or `validate`.
2. **Always validate before pushing.** Run `make-fixer validate -s <id>` to check your changes.
3. **Always ask the user before pushing.** Show them what changed and get explicit confirmation.
4. **Preserve the `idSequence` field** if present — it is server-managed.
5. **Error handlers need their own unique IDs** — they are separate modules in the `onerror` array.
6. **Position modules visually** using `metadata.designer.x` and `metadata.designer.y`. Increment `x` by ~300 for each subsequent module.

## Common Operations

### Add error handler to a module
Edit the module's `onerror` field:
```json
"onerror": [{ "id": NEXT_ID, "module": "builtin:Break", "version": 1, "mapper": { "retry": true, "count": 3, "interval": 1 } }]
```

### Rename a module
Set `metadata.designer.name`:
```json
"metadata": { "designer": { "x": 300, "y": 0, "name": "Descriptive Name" } }
```

### Add a new module to the flow
Insert into the `flow` array at the desired position with a unique ID.

### Add a route
Add an object with a `flow` array to a router module's `routes` array. **Filters go on the first module of each route's `flow` array** — NOT on the router itself. The router's mapper is always `null`.

### Quoting strings inside expressions
Inside `{{...}}`, use single quotes to avoid JSON escaping issues:
```json
{ "field": "{{ifempty(1.name; 'Unknown')}}" }
```

## Cost Optimization Principles

- **Every module run costs operations and credits.** Always build scenarios with the minimum number of modules needed.
- **"Name a run" (`builtin:NameARun`) is the only free module.** Use it freely for labeling — it has zero cost impact. All other modules count.
- Consolidate logic where possible. Prefer fewer modules doing more over many modules doing little.
- When proposing approaches, always include the operation cost comparison (e.g. "Approach A: 5 modules per run, Approach B: 3 modules per run").
- Actively question every module: is this strictly necessary? Can it be combined with another step?

## $ARGUMENTS

- **Scenario ID or URL provided**: Start Phase 1 (fetch and gather context) immediately, then proceed to brainstorming/clarifying questions.
- **Description of what to build**: Start Phase 2 (brainstorm & clarify) immediately — ask questions to understand the full picture before proposing a design.
- **Both ID and description**: Fetch first, then use the description to inform your clarifying questions — but still go through all phases.
- In all cases, go through the workflow phases. Never skip to editing.
