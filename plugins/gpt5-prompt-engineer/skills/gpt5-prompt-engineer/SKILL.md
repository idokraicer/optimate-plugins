---
name: gpt5-prompt-engineer
description: Use when constructing, improving, or reviewing prompts for GPT-5.2 or GPT-5.4 AI agents. Trigger on requests to create agent prompts, optimize existing prompts, or fix prompt issues targeting OpenAI GPT-5 models.
---

# GPT-5 Prompt Engineer

Construct and improve GPT-5.2/5.4 agent prompts using the CTCO framework and GPT-5-specific best practices.

## Two Modes

**Create:** User describes what the agent should do. Generate a full prompt in an `.md` file using the CTCO template. Ask 1-2 clarifying questions first if the description is ambiguous.

**Improve:** User points to an existing `.md` prompt file and describes issues. Read the file, apply best practices respecting existing structure, edit in place.

Both modes end with a self-review and a Rationale written to a separate companion file.

## CTCO Template (new prompts)

Structure every new prompt with these four sections:

```
## Context
Who the agent is, what domain it operates in, what it has access to.
Factual scope only — no persona fluff.

## Task
What the agent must do. Concrete, specific actions.
Imperative language: "Extract...", "Generate...", "Respond with..."

## Constraints
Explicit boundaries: what NOT to do, scope limits, verbosity limits,
output length, forbidden behaviors.
MUST include: "Do EXACTLY and ONLY what is requested."

## Output
Exact format: JSON schema, markdown structure, bullet count,
sentence limits. Include examples when format isn't obvious.
```

Every statement must be actionable and testable. If a sentence uses words like "appropriately", "thoroughly", or "well" — rewrite it with specifics.

For existing prompts in Improve mode: apply the same principles but do NOT force this structure. Respect the prompt's existing organization.

## GPT-5 Best Practices

Apply these as judgment calls during both Create and Improve:

1. **No persona fluff** — strip "You are an expert/world-class/seasoned..." phrasing. Replace with factual scope: "You operate on customer support tickets for SaaS products." GPT-5 treats persona adjectives as noise.

2. **Explicit length constraints** — every prompt MUST specify output length. Use sentence count, bullet count, word limit, or JSON field limits. "Keep responses under 4 sentences unless the user asks for detail" is good. "Respond helpfully" is not.

3. **Scope discipline** — explicitly state: "Do EXACTLY and ONLY what is requested. Do not add features, UI elements, suggestions, or behaviors not specified." GPT-5 is disciplined but still benefits from explicit boundaries.

4. **Ambiguity handling** — instruct the agent what to do when requirements are unclear. Pick one: (a) ask 1-3 clarifying questions, OR (b) present 2-3 labeled interpretations with stated assumptions. Never leave this unspecified.

5. **Structured output** — when the agent produces data, provide the exact schema or shape. Required vs optional fields must be explicit. Use `null` for missing fields, never guess.

6. **Conditional framing** — avoid absolutes ("always", "guaranteed", "never fails"). Use "Based on provided context..." or "If X is available..." when uncertainty exists.

7. **Long-context anchoring** — only when the prompt will process large inputs (>10k tokens). Require the agent to: outline relevant sections, restate user constraints, anchor claims to specific document parts.

8. **Tool efficiency** — describe each tool in 1-2 sentences. Encourage parallel calls for independent operations. Require verification for high-impact actions (billing, deletions, infrastructure).

9. **Agentic updates** — brief status updates (1-2 sentences) only at phase transitions. No narration of routine tool calls. Include concrete outcomes.

## Anti-Patterns Checklist

After generating or editing a prompt, review against this checklist. Flag violations with suggested fixes.

| Anti-Pattern | Look for |
|---|---|
| Persona fluff | "expert", "world-class", "seasoned", "highly skilled" |
| Missing output format | No concrete description of expected response shape |
| Missing length constraint | No word/sentence/bullet limits anywhere |
| Vague instructions | "handle appropriately", "be thorough", "respond well" |
| Scope creep invitation | No boundary on what the agent should NOT do |
| Absolute claims | Unqualified "always", "never", "guaranteed" |
| Redundant repetition | Same instruction restated multiple ways — GPT-5 doesn't need reinforcement |
| Missing ambiguity strategy | No instruction for unclear inputs |
| Hallucination risk | No grounding instruction for claims or data |
| Over-prompting | Micromanaging things GPT-5 already does well by default |

Present findings as a short list. Apply fixes. If no issues found, state that and move on.

## Workflow

### Create Mode
1. User describes what the agent should do
2. Ask 1-2 clarifying questions if description is ambiguous
3. Generate prompt using CTCO template + best practices
4. Write the `.md` file
5. Run anti-patterns checklist, apply fixes
6. Write rationale to a companion file (see Rationale Section below)

### Improve Mode
1. User points to `.md` file and describes issue or improvement
2. Read the file
3. Run anti-patterns checklist — flag existing issues alongside requested changes
4. Apply improvements using best practices, respect existing structure
5. Edit file in place
6. Write/update rationale in the companion file

### Rationale Section

**Write rationale to a SEPARATE companion file, never inside the prompt file.** If the prompt is `agent-prompt.md`, write the rationale to `agent-prompt.rationale.md`. This prevents GPT-5 from processing meta-commentary about itself (e.g., "removed persona fluff — GPT-5 treats this as noise") which can confuse or distract the agent.

The prompt file must be clean and ready to use as-is — copy it directly into a system prompt with zero editing.

Example companion file (`agent-prompt.rationale.md`):
```markdown
# Rationale for agent-prompt.md
- Removed persona fluff ("expert AI assistant") — GPT-5 treats this as noise
- Added 3-bullet limit to output — prevents verbose responses
- Added scope constraint — agent was generating unrequested suggestions
```
