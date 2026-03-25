# Optimate Business Context

## Who is Optimate

- Systems integrator that builds, implements, and integrates various platforms for clients
- No default tech stack — each project's characterization doc drives which systems are involved
- Works across CRM, automation, ERP, accounting, and custom solutions

## How Optimate Works

- Projects start with a characterization document from the client or internal PM
- The characterization is decomposed into structured tasks on monday.com (board 9116155603)
- Tasks are grouped by project via a linked project column (projects board 7109422573)
- Implementation follows the task structure: infrastructure first, then automations, then dashboards

## Task Estimation Guidelines

- The agent should suggest hour estimates based on complexity but always defer to PM
- Simple object (few fields, no relations): 1-2 hours
- Complex object (many fields, lookups, permissions): 3-5 hours
- Simple automation (single trigger-action): 1-2 hours
- Complex automation (multiple conditions, error handling): 3-6 hours
- Dashboard: 2-4 hours depending on data sources
- These are starting points — the PM adjusts based on experience

## When Making Suggestions

- Load system-specific knowledge files from `systems/` when a system is mentioned in the characterization
- Be opinionated about implementation approaches based on loaded system knowledge
- Always flag assumptions with `[הנחה]` prefix
- Suggest edge cases the PM might not have considered
- Recommend UX patterns based on system conventions
