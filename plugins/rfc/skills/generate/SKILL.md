---
name: generate
description: >
  This skill should be used when the user asks to "generate an RFC",
  "write an RFC", "create an RFC document", "draft an RFC",
  "prepare an RFC for review", "architecture RFC", "rfc generate",
  "help me write an RFC", "fill out RFC template", "RFC for a new feature",
  "RFC for migration", "document architecture decision",
  or wants to produce a structured RFC document for architecture review.
  Guides the user through an interactive interview to gather all necessary
  information, then generates a complete RFC using a standardized template.
version: 1.0.0
---

# RFC Document Generation

Generate structured RFC (Request for Comments) documents for architecture review.
The RFC serves as a pre-development checkpoint — a structured way to describe the problem,
proposed solution, and implementation details before writing code.

A good RFC prevents duplicated work, surfaces architectural weaknesses early,
and aligns the team on technical direction.

## Process Overview

Three phases:

1. **Gather context** — understand the problem and scope through user interview
2. **Explore codebase** — analyze existing code to ground the RFC in reality
3. **Generate RFC** — produce the document section by section

## Phase 1: Gather Context

Start by understanding what the user wants to build or change. Ask targeted questions
to collect enough information for the RFC. Adapt based on what the user has already provided.

### Required information (must have before generating):

- **Problem statement** — what problem is being solved and why now
- **Proposed solution** — at least a high-level idea of the approach
- **Scope** — what services/systems are affected

### Gate: do NOT proceed to Phase 2 or 3 until all three required items above are present.

If the user's input is missing any of the three required items, respond ONLY with
clarifying questions — do not generate an RFC or even a draft. The whole point of the
interview phase is to avoid generating an RFC full of assumptions. An RFC built on
guesses is worse than no RFC at all because it looks authoritative while being wrong.

Ask 3-5 targeted questions in a single round. Focus on the missing required items first,
then on the most impactful gaps from the optional list below. Wait for the user to respond
before moving on. Maximum 2-3 interview rounds total.

### Information to collect if not provided:

- Stakeholders, tech lead, product manager
- Launch timeline
- Task/ticket link
- Current system behavior (if modifying existing functionality)
- Non-functional requirements (latency, throughput, availability targets)

If the user provides a detailed description that covers all three required items,
skip straight to codebase exploration. Don't ask questions the user has already answered.

## Phase 2: Explore Codebase

Before generating, explore the project to understand existing architecture.

### Preferred: use `feature-dev:code-explorer` agent

If the `feature-dev:code-explorer` agent is available, delegate codebase exploration to it.
Spawn the agent with a prompt describing what you need to understand:
- Current architecture and service boundaries
- Existing data flows and API endpoints relevant to the RFC
- Patterns and conventions used in the project
- Any existing similar functionality that could be reused or affected

Example spawn prompt:
```
Analyze the codebase to gather context for an RFC about {feature description}.
I need to understand:
1. Current architecture of {affected area} — services, modules, key files
2. Existing API endpoints and data flows related to {domain}
3. Database schemas and storage patterns in use
4. Any existing functionality that overlaps with {proposed change}
Focus on architectural patterns, not implementation details.
```

### Fallback: native tools via subagents

If `feature-dev:code-explorer` is not available, use subagents (Agent tool with
`subagent_type: "Explore"`) or explore directly:

- Use Glob to find relevant services, modules, and configuration files
- Use Grep to trace existing data flows and API endpoints
- Read key files to understand current architecture patterns
- Check for existing similar functionality to avoid duplication

This grounds the RFC in reality rather than assumptions. Reference specific files
and code patterns in the generated RFC where relevant.

## Phase 3: Generate RFC

Read the template from **`references/rfc-template.md`** and the writing guide
from **`references/rfc-guide.md`** before generating.

### Generation rules:

1. **Use the template structure exactly.** Don't invent new sections or reorder them.

2. **Skip irrelevant sections gracefully.** Mark them as "N/A — {brief reason}"
   rather than omitting them entirely. This shows the author considered the section.

3. **Always provide at least two alternatives** in the "Considered Alternatives"
   section, even if one is clearly better. Compare them with specific pros/cons.

4. **Include Mermaid diagrams** for:
   - Component diagram showing service interactions (always)
   - Sequence diagram for key request flows (always)
   - ER diagram for database schema (when data storage changes)

5. **Quantify everything possible.** Replace vague statements with numbers:
   - Bad: "The service will handle high load"
   - Good: "Expected ~500 RPS at launch, growing to ~2000 RPS within 6 months"

6. **Be explicit about failure modes.** For each new interaction or dependency,
   describe what happens when it fails.

7. **Write in the language the user communicates in.** If the user writes in Russian,
   generate the RFC in Russian. If in English — in English.

### Section priorities:

Not all sections need equal depth. Focus effort based on what matters most
for the specific change:

| Change Type | High Priority Sections | Can Be Brief |
|-------------|----------------------|--------------|
| New service | New Services, Service Interactions, Data Storage, Reliability | Fraud, Periodic Processes |
| New feature in existing service | Problem Statement, Alternatives, API, Launch Plan | New Services |
| Data migration | Data Storage, Launch Plan, Risks, Reliability | New Services, Fraud |
| Performance optimization | Current State, Alternatives, Resource Consumption, Metrics | New Services, Security |

## Output

Save the generated RFC as a markdown file. Suggest a filename based on the feature:
`rfc-{feature-name}.md` in the project root or a docs directory if one exists.

After generating, offer to:
- Review specific sections in more detail
- Add more alternatives to the architectural decisions
- Expand diagrams or add new ones
- Iterate on any section the user is not satisfied with
