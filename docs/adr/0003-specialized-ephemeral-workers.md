# ADR-0003: Use Specialized Ephemeral Workers for Task Execution

## Status
Accepted

## Context

The system must execute heterogeneous tasks such as drafting, analysis, classification, validation, and research. Using the orchestrator to perform all work directly would overload its context and blur control responsibilities.

## Decision

Task execution will be handled by specialized, short-lived worker agents.

Each worker:
- receives one bounded task
- receives task-scoped context only
- returns structured output and artifacts
- can be retried, replaced, or cancelled independently

The orchestrator remains responsible for decomposition, routing, and reconciliation.

## Consequences

### Positive
- lower context load on the orchestrator
- better specialization by worker type
- safer failure isolation
- easier horizontal scaling
- simpler cost and latency control per task type

### Negative
- dispatch and worker contract design become critical
- more state transitions must be tracked
- reconciliation logic becomes more important

## Rejected alternative

### Alternative: a monolithic general-purpose agent does everything
Rejected because it couples orchestration with execution and increases context contamination risk.

## Worker contract requirements

Each worker invocation should define:
- ticket ID
- objective
- acceptance criteria
- allowed tools
- forbidden actions
- required output schema
- linked artifacts
- timeout and retry policy

## Follow-up implications

- dispatcher policy is required
- worker types must be explicit and versioned
- worker outputs should be normalized before parent reconciliation
