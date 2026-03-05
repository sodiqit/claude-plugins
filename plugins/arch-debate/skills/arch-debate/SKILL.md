---
name: arch-debate
description: >
  This skill should be used when the user asks to "run a brainstorm",
  "run a structured debate", "compare architectural approaches",
  "play devil's advocate on a design", "strategy session",
  "evaluate competing solutions", "trade-off analysis",
  "pros and cons of different approaches", "arch-debate",
  "help me decide between approaches", "architecture review",
  "evaluate my architecture", "assess module boundaries",
  or wants to spawn an Agent Teams session for exploring a problem
  from multiple perspectives. Orchestrates 3 personalized architecture
  advisors (Kleppmann, Uncle Bob, Evans) through a structured debate
  protocol based on Analysis of Competing Hypotheses methodology.
version: 1.1.0
---

# Multi-Perspective Architecture Advisory via Agent Teams

Orchestrate a structured architecture advisory session via Agent Teams. Three personalized
advisors — each with a distinct cognitive framework — independently analyze the problem,
cross-examine each other's reasoning, and produce a synthesized recommendation.

The advisors challenge each other from their own frameworks (not generically), and the
lead applies ACH logic: the strongest recommendation is the one with the least disconfirming
evidence across all three perspectives.

## Prerequisites

Verify that `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` is set before proceeding.
If not enabled, inform the user with setup instructions for their Claude Code
settings (`.claude/settings.json` in the project root or `~/.claude/settings.json` globally):

```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```

Halt the session until the environment variable is confirmed.

## Lead Role

The lead acts as **coordinator only**. Never generate solutions or architectural opinions.
Only delegate, moderate, and synthesize. Use delegate mode throughout the session.

## Input Handling

Before spawning the team, gather from the user:

- **Problem statement** — what decision needs to be made
- **Constraints** — tech stack, timeline, team size, compatibility requirements
- **Codebase context** — which files/modules are relevant (read them before spawning)

If the problem statement is vague, ask 1-2 targeted clarifying questions. No more than 2.
If still unclear after 2 questions, do not spawn the team — suggest `/arch-debate-prep`.

## Session Mode

Determine the mode before spawning. This affects Phase 1 instructions:

| Mode | When | Behavior |
|------|------|----------|
| Architecture Review | User wants to evaluate existing code/design | All three analyze existing code through their lens, focus on problems and improvements |
| Module Boundary Design | User asks where to split modules, define responsibilities | Evans leads (bounded contexts), Kleppmann adds data flow constraints, Uncle Bob adds dependency direction |
| New Solution Design | User needs to design something new | All three propose approaches from their frameworks, then cross-examine |

If unclear, default to Architecture Review for existing code, New Solution Design for new features.

## Advisor Panel

Spawn exactly 3 advisors. Full persona profiles are in **`personas/`**. Spawn prompts are in **`references/session-protocol.md`**.

| Advisor | Name | Cognitive Entry Point | Focus |
|---------|------|----------------------|-------|
| Martin Kleppmann | `kleppmann` | Data flow and failure modes | Distributed systems, consistency models, data architecture, operational complexity |
| Uncle Bob | `uncle-bob` | Dependency direction and structural quality | SOLID, Clean Architecture, testability, the Dependency Rule |
| Eric Evans | `evans` | Domain model and ubiquitous language | Bounded contexts, aggregates, context maps, conceptual integrity |

Key design: these three enter problems from **fundamentally different cognitive angles**.
Even on the same problem, Kleppmann asks "where does the data flow?", Uncle Bob asks
"which way do the dependencies point?", and Evans asks "what does the domain expert call this?".
This diversity of entry points is the primary mechanism for avoiding Degeneration of Thought.

## Session Phases

Three core phases + one optional refinement:

| Phase | Mode | What Happens |
|-------|------|-------------|
| 1. Research | Parallel | All three advisors independently explore the problem, read codebase, produce analysis through their framework. No cross-talk. |
| 2. Cross-Examine | Interactive | Each advisor receives the other two's analyses. They identify strengths and blind spots from their own framework. 2 rounds max. |
| 3. Refine (optional) | Parallel | Only if lead determines low differentiation. Each advisor revises addressing strongest criticisms. |
| 4. Synthesize | Lead only | Lead reads everything and produces the synthesis document. |

**Adaptive break rules:**
- Skip Phase 3 if Phase 2 produced clear differentiation and sufficient evidence.
- Terminate Cross-Examine early if a point stalls for 2+ exchanges on the same argument.
- Log stalled points as Open Questions in the synthesis.

For step-by-step orchestration instructions, consult **`references/session-protocol.md`**.

## Structured Analysis Format

Instruct each advisor to output using this structure. Include the format in spawn prompts:

```markdown
## Analysis: {Advisor Name}'s Perspective

### Framework Applied
{Which of their core principles/frameworks they used — 1-2 sentences}

### Key Observations
- {Observation 1 through their specific lens, with evidence from codebase}
- {Observation 2 through their specific lens}

### Recommendation
{Specific, implementable architectural recommendation}

### Design Decisions
- Decision 1: {what and why, grounded in their framework}
- Decision 2: {what and why}

### Implementation Outline
- {file/module}: {what changes}

### Assumptions
- {explicitly stated assumptions the recommendation depends on}

### Trade-offs and Risks
- {honest weaknesses from their own perspective}

### What I Am Less Certain About
- {areas outside their primary expertise where they defer to others}
```

## Synthesis Output Format

Produce this document at the end of Phase 4:

```markdown
## Architecture Advisory Session: {problem}

### Problem Statement
{1-2 sentences restating the decision}

### Advisors
- **Martin Kleppmann** — analyzed through: data flows, consistency, failure modes
- **Uncle Bob** — analyzed through: dependency direction, SOLID, testability
- **Eric Evans** — analyzed through: domain model, bounded contexts, ubiquitous language

### Areas of Agreement
{Points where all three frameworks converge — high-confidence guidance}

### Perspective: Martin Kleppmann
**Framework applied**: {which principles}
**Key insight**: {the most important thing this perspective uniquely revealed}
**Recommendation**: {summary}
**Disconfirming evidence**: {what the other frameworks raised against this}

### Perspective: Uncle Bob
**Framework applied**: {which principles}
**Key insight**: {the most important thing this perspective uniquely revealed}
**Recommendation**: {summary}
**Disconfirming evidence**: {what the other frameworks raised against this}

### Perspective: Eric Evans
**Framework applied**: {which principles}
**Key insight**: {the most important thing this perspective uniquely revealed}
**Recommendation**: {summary}
**Disconfirming evidence**: {what the other frameworks raised against this}

### Key Debate Points
{3-5 most important arguments from cross-examination — genuine disagreements and how they resolved}

### Recommendation
**Preferred approach**: {integrated recommendation drawing from the strongest elements}
**Confidence**: {high | medium | low}
**Rationale**: {based on ACH: which elements had the least disconfirming evidence across all three frameworks}

### Framework-Specific Checklist
- [ ] Data flow and consistency requirements met (Kleppmann)
- [ ] Dependency Rule respected, core testable in isolation (Uncle Bob)
- [ ] Domain model reflects ubiquitous language, bounded contexts clear (Evans)

### Open Questions
{Unresolved concerns needing further investigation}

### Conditions to Revisit
{What changes would invalidate this recommendation}
```

## Failure Mode Prevention

| Risk | Mitigation |
|------|-----------|
| Persona collapse (advisor breaks character, gives generic advice) | Each persona has explicit anti-collapse instructions and behavioral constraints. Lead sends re-engagement prompt referencing specific cognitive entry point. |
| Degeneration of Thought (all converge to same generic advice) | Different cognitive ENTRY POINTS prevent this — Kleppmann starts with data flow, Uncle Bob with dependencies, Evans with domain language. Even on the same problem, they ask different first questions. |
| Sycophantic collapse (advisor abandons position without evidence) | Require explicit evidence citation when changing position. Each persona has "concede only with stronger evidence" in their rules. |
| One advisor dominates (problem clearly in one domain) | Synthesis explicitly requires extracting value from ALL three frameworks. Even when one is most relevant, others surface risks and blind spots. Framework-Specific Checklist ensures all three lenses represented. |
| Echo chamber (advisors converge on shared bias) | Three fundamentally different frameworks (data flow vs. dependency direction vs. domain model) make unconscious alignment unlikely. If convergence occurs, it is a strong signal — document why all three frameworks agree. |
| Debate deadlock (same point argued repeatedly) | Lead intervenes after 2 exchanges on same point; log as Open Question. |

## Additional Resources

- **`personas/martin-kleppmann.md`** — Full persona profile for Martin Kleppmann
- **`personas/uncle-bob.md`** — Full persona profile for Uncle Bob (Robert C. Martin)
- **`personas/eric-evans.md`** — Full persona profile for Eric Evans
- **`references/session-protocol.md`** — Complete spawn prompts, phase orchestration, and edge case handling
