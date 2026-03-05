---
name: arch-debate
description: >
  This skill should be used when the user asks to "run a brainstorm",
  "run a structured debate", "compare architectural approaches",
  "play devil's advocate on a design", "strategy session",
  "evaluate competing solutions", "trade-off analysis",
  "pros and cons of different approaches", "arch-debate",
  "help me decide between approaches", "architecture review",
  or wants to spawn an Agent Teams session for exploring a problem
  from multiple angles. Orchestrates 3 teammates through a structured
  debate protocol based on Analysis of Competing Hypotheses methodology.
version: 1.0.0
---

# Structured Architectural Debate via Agent Teams

Orchestrate a structured debate via Agent Teams. Two proposers independently develop
competing solutions; a validator stress-tests both using disconfirmation logic
(the stronger proposal is the one with the least evidence against it, not the most
evidence for it).

## Prerequisites

Verify that `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set before proceeding.
If not enabled, inform the user with setup instructions for their Claude Code
settings (`.claude/settings.json` in the project root or `~/.claude/settings.json` globally):

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

Halt the session until the environment variable is confirmed.

## Lead Role

The lead acts as **coordinator only**. Never generate solutions. Only delegate,
moderate, and synthesize. Use delegate mode throughout the session.

## Input Handling

Before spawning the team, gather from the user:

- **Problem statement** — what decision needs to be made
- **Constraints** — tech stack, timeline, team size, compatibility requirements
- **Codebase context** — which files/modules are relevant (read them before spawning)

If the problem statement is vague, ask 1-2 targeted clarifying questions. No more than 2.
If still unclear after 2 questions, do not spawn the team — ask the user to refine.

## Team Roles

Spawn exactly 3 teammates. Full spawn prompts are in **`references/session-protocol.md`**.

| Role | Name | Thinking Style | Focus |
|------|------|---------------|-------|
| Proposer Alpha | `alpha` | Conservative, pragmatic | Proven patterns, maintainability, incremental adoption |
| Proposer Beta | `beta` | Exploratory, unconventional | Novel approaches, long-term scalability, challenging assumptions |
| Validator | `validator` | Skeptical, systematic, fair | Disconfirming evidence, failure modes, hidden assumptions |

Key constraint: Alpha and Beta MUST propose fundamentally different approaches, not
variations of the same idea. If proposals converge, instruct the Validator to flag it. Convergence
toward the same solution is a valid signal — document why both arrived at the same
conclusion.

## Session Phases

Three core phases + one optional refinement:

| Phase | Mode | What Happens |
|-------|------|-------------|
| 1. Research | Parallel | Alpha and Beta independently explore the problem, read codebase, form proposals. No cross-talk. Each produces a structured proposal. |
| 2. Cross-Examine | Interactive | Validator poses 3-5 disconfirming questions per proposal. Alpha and Beta critique each other: identify the strongest point and weakest assumption. 2 rounds max. |
| 3. Refine (optional) | Parallel | Only if Validator requests it (low confidence). Each proposer revises addressing strongest criticisms. |
| 4. Synthesize | Lead only | Lead reads everything and produces the synthesis document. |

**Adaptive break rules:**
- Validator may skip Phase 3 if Phase 2 produced clear differentiation and sufficient evidence.
- Terminate Cross-Examine early if a point stalls for 2+ exchanges on the same argument.
- Log stalled points as Open Questions in the synthesis.

For step-by-step orchestration instructions, consult **`references/session-protocol.md`**.

## Structured Proposal Format

Instruct each proposer to output using this structure. Include the format in spawn prompts:

```markdown
## Proposal: {name}

### Approach
{2-3 sentence summary}

### Key Design Decisions
- Decision 1: {what and why}
- Decision 2: {what and why}

### Implementation Outline
- {file/module}: {what changes}

### Assumptions
- {explicitly stated assumptions the proposal depends on}

### Trade-offs and Risks
- {honest weaknesses and what could go wrong}
```

## Synthesis Output Format

Produce this document at the end of Phase 4:

```markdown
## Architectural Debate: {problem}

### Problem Statement
{1-2 sentences restating the decision}

### Areas of Agreement
{Points both proposals share — establish common ground first}

### Solution A: {name}
**Approach**: {summary}
**Strengths**: {from debate evidence}
**Risks and disconfirming evidence**: {what undermines this proposal}

### Solution B: {name}
**Approach**: {summary}
**Strengths**: {from debate evidence}
**Risks and disconfirming evidence**: {what undermines this proposal}

### Key Debate Points
{3-5 most important arguments from cross-examination}

### Recommendation
**Preferred**: {A, B, or hybrid with specific elements}
**Confidence**: {high | medium | low}
**Rationale**: {based on which proposal had less disconfirming evidence}

### Open Questions
{Unresolved concerns needing further investigation}

### Conditions to Revisit
{What changes would invalidate this recommendation}
```

## Failure Mode Prevention

| Risk | Mitigation |
|------|-----------|
| Sycophantic collapse (proposer abandons position without reason) | Require explicit evidence citation when changing position |
| Degeneration of Thought (repetitive arguments, no new insights) | Cap at 2 cross-examination rounds; use opponent feedback, not self-reflection |
| Echo chamber (both proposers converge on shared bias) | Different personas enforced; Validator flags convergence |
| Persuasion over truth (eloquent but weak arguments win) | Validator weights evidence quality over rhetoric; structured proposal format |
| Debate deadlock (same point argued repeatedly) | Lead intervenes after 2 exchanges on same point; log as Open Question |

## Additional Resources

- **`references/session-protocol.md`** — Complete spawn prompts for all 3 teammates, step-by-step phase orchestration messages, and edge case handling
