---
name: arch-debate-prep
description: >
  This skill should be used when the user asks to "arch-debate prep",
  "arch debate prep", "prepare an arch-debate", "brainstorm prep",
  "format for brainstorm", "structure my idea for debate",
  "turn this into a debate prompt", "frame this problem for debate",
  or provides a casual problem description that needs formatting into
  a structured /arch-debate prompt. Converts informal input into
  well-structured debate prompts with BAB framing and constraint calibration.
version: 1.0.0
---

# Arch-Debate Prep — Informal Input to Structured Prompt

Transform casual or dictated problem descriptions into structured prompts
optimized for the `/arch-debate` skill.
Apply BAB framing (Before-After-Bridge) and constraint calibration to maximize
proposal differentiation.

## Decomposition Pipeline

Extract these components from raw user input:

| Component | Extract | Example |
|-----------|---------|---------|
| Core question | Reframe as "How might we..." decision question | "How might we unify error handling across all 5 layers?" |
| Current pain (Before) | What is wrong now, what triggered this | "Some layers throw, some return Results, no consistency" |
| Desired state (After) | What good looks like | "Users see actionable errors, devs get debug context" |
| Hard constraints | Non-negotiable technical requirements | "TypeScript strict, must work with DI in bootstrap.ts" |
| Soft preferences | Nice-to-haves, separate from hard constraints | "Prefer minimal dependencies" |
| Solution hints | Ideas the user already has — extract and isolate | "Maybe use a Result monad" |
| Success criteria | How to judge if a proposal is good | "Zero bare throw statements, all errors have exit codes" |

## Codebase Exploration

After decomposition, explore the codebase to ground the prompt in reality:

- Verify file paths mentioned by the user exist (use Glob)
- Discover additional relevant files the user did not mention (use Grep for related symbols)
- Read CLAUDE.md or project docs for implicit constraints (architecture rules, conventions)

Include only confirmed paths in the output. Never guess file paths.

## Validation Rules

Before outputting, check each rule. Maximum 1 clarifying question total. If the input
needs more than 1 question, ask the single most important one. Do not interrogate the user.

| Check | Risk if violated | Action |
|-------|-------------------|--------|
| Core question is a decision, not a symptom | "Error handling is messy" (symptom) | Reframe: "How might we choose a unified error handling strategy?" |
| Hard constraints >= 2 (including codebase-derived) | Debate will be unfocused | Count constraints from codebase exploration first, then ask 1 question if still under 2 |
| Hard constraints <= 7 | Over-constrained, reduces divergence | Suggest prioritizing top 3-5 |
| No solution embedded in problem statement | "We should use event sourcing for X" anchors proposers | Move to "Initial thinking" section |
| Problem length ~40-80 words | Too short = vague, too long = anchoring | Trim or expand as needed |
| Success criteria present | No way to evaluate proposals | Infer from constraints or ask 1 question |

## Output Format

Present the formatted prompt in a fenced code block ready to copy:

````markdown
/arch-debate

Problem: {Core question framed as decision.
1-2 sentences of Before/After context explaining why this matters now.}

Constraints:
- {Hard constraint 1}
- {Hard constraint 2}
- {Hard constraint 3}

Success criteria:
- {What a good solution achieves}

Relevant files: {comma-separated confirmed paths}
````

If solution hints were extracted from the user's input, append them separately
to prevent anchoring the proposers:

````markdown
> Initial thinking (for context, not to anchor proposals):
> {User's preliminary ideas, separated from the problem}
````

## Transformation Examples

These examples are illustrative. Adapt all file paths and constraints to the current project's codebase.

### Example 1: Symptom reframing + constraint discovery

**Raw input:**
> "error handling is a mess, some places throw some don't, users see ugly stack traces"

**Formatted output:**
```
/arch-debate

Problem: How might we implement a unified error handling strategy across all layers?
Currently some layers throw exceptions, others return error values, and users
see raw stack traces instead of actionable messages. Need consistency that serves
both end-users (readable errors) and developers (debug context).

Constraints:
- TypeScript strict mode, no `any`
- Must use existing error class hierarchy from src/errors/
- JSON output mode must produce structured errors to stderr
- Must work with DI pattern in src/bootstrap.ts

Success criteria:
- Zero bare `throw new Error()` in codebase
- All errors have error codes and exit codes
- Users never see stack traces without --verbose

Relevant files: src/errors/, src/commands/, src/services/, src/bootstrap.ts
```

### Example 2: Solution hint extraction

**Raw input:**
> "maybe we should use sqlite for caching, the api is slow"

**Formatted output:**
```
/arch-debate

Problem: How might we implement a local caching layer to reduce API call latency?
Repeated requests to the external API add significant latency.
Need a caching strategy that balances freshness with responsiveness.

Constraints:
- Must not add heavy runtime dependencies
- Cache invalidation when data changes on server
- Works offline for previously fetched data

Success criteria:
- Repeated reads served locally without API call
- Stale data is detectable and refreshable

Relevant files: src/services/, src/api/

> Initial thinking (for context, not to anchor proposals):
> User mentioned SQLite as a possible approach
```

### Example 3: Overly specific input with embedded solution

**Raw input:**
> "we need to add Redis caching with a 5 minute TTL to all service calls, it should
> use ioredis and invalidate on writes"

**Formatted output:**
```
/arch-debate

Problem: How might we add a caching layer to service calls to reduce API latency?
Every service call hits the external API directly. Frequently accessed data
could be served from cache to improve responsiveness.

Constraints:
- Must integrate with existing service interface pattern
- Write operations must invalidate relevant cache entries
- Cache expiry needed to prevent serving stale data

Success criteria:
- Read-heavy operations measurably faster on repeated access
- No stale data served after a write operation
- Cache failures degrade gracefully to direct API calls

Relevant files: src/services/, src/api/, src/config/

> Initial thinking (for context, not to anchor proposals):
> User suggested Redis with ioredis, 5-minute TTL, invalidate-on-write strategy
```
