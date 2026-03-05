# RFC Template

## General Information

Affected and new services:

Launch date:

Stakeholders:

Product manager:

Tech lead:

Task link:

## Context and Problem

### Terminology

List terms used in this RFC, especially ambiguous or domain-specific ones.

### Problem Statement

Briefly describe why changes are being made and what problem this RFC addresses.

### Functional Requirements

What capabilities, features, and use cases must be implemented.

### Non-Functional Requirements

Characteristics not directly related to functionality but critical for quality, performance, reliability, security, etc.

### Current State

If applicable, describe how the system works today.

Use Mermaid and PlantUML for diagrams.

## Architectural Decisions

### Considered Alternatives

Consider more than one solution. Pros and cons of each. Attach component and sequence diagrams where they clearly demonstrate how the solution works.

### Rationale for Selection

Justify why this particular option was chosen. Explain why other options were rejected.

## New Services

What functions the service will perform. Define boundaries early to set the development direction and prevent monolith growth. Justify why the task cannot be solved without a new service.

Specify criticality tier. If new or existing services become Tier A — justify why it cannot be avoided.

Choose frameworks for new services.

## Service Interactions

Who calls what, how, and why. What new entry/exit points appeared.

Attach detailed component and sequence diagrams.

Estimate changes in incoming system traffic. Estimate changes in inter-service traffic.

### API

OpenAPI / Protobuf descriptions for API endpoints.

### Events

New events and changes to existing events.

## Periodic Processes

List periodic processes (cron, periodic, etc.). Describe logic in detail. Add sequence diagrams if helpful. Specify run frequency and/or trigger conditions.

## Data Storage

If schemas change across multiple databases in different services, list them all.

### Table Schemas

Data storage schema (table structures, collections, etc.).
What indexes will be created.
What relationships exist between tables.
Any periodic processes within the storage layer.

### Database Type and Flavor

### Data Volume and Growth Rate Estimates

### Data Replication to DWH

### Data Archival and Cleanup

## Resource Consumption

Describe expected changes in resources (CPU, RAM, Disk, Network) for services (new and existing ones that will interact).

If calculation is not possible, note the need for load testing.

## Reliability and Fault Tolerance

Scaling capabilities. For example, potential for vertical scaling.

Possible failure points and consequences. Availability of fallback scenarios.

Protection mechanisms: graceful degradation, retry policy, circuit-breaker.

Recovery process for full/partial failure.

## Security and Compliance

Changes to public API.

Related to creating/modifying authentication/authorization processes.

Whether personal data processing is planned.

## Fraud Prevention

Whether the work involves payments or card data.

## Custom Metrics, Alerts, Logs, and Operations

Only non-default metrics and alerts planned to be added.

If applicable, what logs will be written to simplify diagnostics or track the launch process.

## Launch Plan

Describe the migration plan or new feature launch plan in detail.

## Risks

Potential risks and security concerns related to the new functionality.

## Future Plans
