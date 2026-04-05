# gpt5-prompt-engineer

A Claude Code plugin for constructing and improving GPT-5.2/5.4 agent prompts.

## What it does

- **Create** new GPT-5 prompts from scratch using the CTCO (Context/Task/Constraints/Output) framework
- **Improve** existing prompts by applying GPT-5-specific best practices
- **Review** prompts against a 10-point anti-patterns checklist
- **Document** changes with a rationale section explaining design decisions

## Usage

Invoke the skill manually when building or optimizing GPT-5 agent prompts:

```
/gpt5-prompt-engineer
```

### Create Mode

Describe what your agent should do and the skill generates a full CTCO-structured prompt in a `.md` file.

### Improve Mode

Point to an existing `.md` prompt file and describe the issue — the skill reads, reviews, and edits in place.

## GPT-5 Best Practices Applied

- No persona fluff — factual scope only
- Explicit length constraints on every prompt
- Scope discipline ("Do EXACTLY and ONLY what is requested")
- Structured output with schemas
- Ambiguity handling strategies
- Sequential discount/escalation patterns
- Tool efficiency (parallel calls, minimal descriptions)
- Anti-patterns self-review after every edit

## Output

Every prompt includes a `## Rationale` section explaining key design decisions. This section is for your reference — not sent to GPT-5.
