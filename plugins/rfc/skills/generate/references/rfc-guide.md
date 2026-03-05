# RFC Writing Guide

Best practices for filling out each RFC section.

## Table of Contents

1. [General Principles](#general-principles)
2. [Section-by-Section Guide](#section-by-section-guide)
3. [Common Mistakes](#common-mistakes)
4. [Diagrams](#diagrams)

## General Principles

- **Answer only relevant sections.** Not every RFC needs every section. Skip sections that don't apply, but explicitly mark them as "N/A" with a brief reason — this shows the author considered the section rather than overlooked it.
- **Write for the reviewer, not yourself.** The reader doesn't have your context. Explain domain-specific terms, link to existing documentation, and provide enough background for someone outside your team.
- **Quantify wherever possible.** "High load" means nothing. "~800 TPS incoming, ~100K events/day" gives reviewers something to evaluate.
- **Consider at least two alternatives.** Even if the choice seems obvious, documenting why you rejected the alternative strengthens the argument and catches blind spots.

## Section-by-Section Guide

### General Information

Fill in all metadata fields. This section is for traceability — someone reading this RFC in 6 months needs to know who to ask questions. Don't skip the task link.

### Problem Statement

The most important section. A good problem statement answers:
- What triggered this work? (incident, product requirement, tech debt)
- What happens if we do nothing?
- What's the scope boundary? (what this RFC does NOT cover)

Keep it 3-5 sentences. If you need more, the scope is probably too large — consider splitting into multiple RFCs.

### Functional vs Non-Functional Requirements

Separate these clearly. Functional = what the system does. Non-functional = how well it does it.

Non-functional examples: latency SLOs (p99 < 200ms), availability targets (99.9%), throughput requirements (handle 10K RPS), data retention policies.

### Current State

Critical for reviewers who don't know your system. Include:
- Architecture diagram of the current state
- Key data flows
- Known pain points that motivate the change

### Considered Alternatives

This is where most RFCs fall short. For each alternative provide:
- Brief description of the approach
- Pros (with specifics, not generic "simpler")
- Cons (with specifics, not generic "more complex")
- Why it was rejected (the key differentiator)

A comparison table works well for 3+ alternatives.

### New Services

Before proposing a new service, justify why existing services can't handle the responsibility. Address:
- Service boundaries and ownership
- Why this can't be a library/module in an existing service
- Criticality tier with justification
- Technology choice rationale

### Service Interactions

Map all communication paths. For each interaction specify:
- Protocol (HTTP/gRPC/message queue)
- Direction (who initiates)
- Payload (what data is exchanged)
- Failure behavior (what happens if the call fails)

Estimate traffic changes — both incoming to the system and between internal services. A new feature adding 100 RPS to a service currently handling 10 RPS is a 10x increase that needs attention.

### Data Storage

Include:
- DDL or schema definitions
- Index strategy with justification
- Expected data volume at launch and after 1 year
- Archival/cleanup strategy (data doesn't delete itself)
- Migration plan if modifying existing schemas

### Reliability and Fault Tolerance

For each failure point ask:
- What breaks if this component goes down?
- Can the system degrade gracefully?
- What's the recovery procedure?
- Is there a fallback or "kill switch"?

Document retry policies, circuit breakers, and timeout values — not just "we'll add retries".

### Launch Plan

Think in phases:
1. Feature flag / dark launch
2. Canary release (% of traffic)
3. Gradual rollout
4. Full rollout
5. Rollback plan if something goes wrong

## Common Mistakes

| Mistake | Example | Better Approach |
|---------|---------|-----------------|
| N+1 service calls | Querying a service per item in a list (1000 items = 1000 requests) | Use batch endpoints or local caching |
| Ignoring existing solutions | Building a feature that already exists in another team's service | Research existing capabilities first |
| Missing traffic estimates | "We'll handle the load" | Calculate expected RPS, data volume, growth rate |
| Single alternative | "We chose Kafka" (no comparison) | Compare at least 2 options with pros/cons |
| Vague non-functional requirements | "Must be fast and reliable" | "p99 latency < 200ms, 99.9% availability" |
| No rollback plan | Assuming the launch will succeed | Define rollback steps and trigger conditions |

## Diagrams

Use Mermaid or PlantUML. Every RFC should include at minimum:
- **Component diagram** — showing services and their interactions
- **Sequence diagram** — showing key request flows

For data-heavy designs, add:
- **ER diagram** — for database schema
- **Data flow diagram** — for showing how data moves through the system

Keep diagrams focused. A diagram showing everything is as useful as a diagram showing nothing. One diagram per key flow or interaction.
