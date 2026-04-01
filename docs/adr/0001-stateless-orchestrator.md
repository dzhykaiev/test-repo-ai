# ADR-0001: Use a Stateless Orchestrator Per Run

## Status
Accepted

## Context

The system needs a primary agent that can continuously manage incoming work, create tickets, route work to specialized agents, and remain reliable over long operational periods.

A single long-running conversational orchestrator appears attractive, but it introduces major risks:
- prompt inflation
- stale assumptions
- hidden memory dependence
- difficult recovery after interruption
- low auditability of decision basis

## Decision

The primary orchestrator will be stateless per run.

Each orchestrator invocation will rebuild its decision context from durable system state, including:
- current ticket state
- relevant event slices
- latest summaries
- linked artifacts
- operating policies

The orchestrator will not depend on a continuously accumulated conversation transcript as its primary memory.

## Consequences

### Positive
- clean context on every invocation
- easier recovery and replay
- lower risk of prompt contamination
- better observability and auditability
- simpler scaling of orchestrator execution

### Negative
- requires explicit context-building infrastructure
- summary management becomes a first-class subsystem
- some latency is added for retrieval and assembly

## Rejected alternative

### Alternative: one long-lived orchestrator session
Rejected because it creates operational fragility and memory drift over time.

## Follow-up implications

- context builder is mandatory
- ticket and event stores are mandatory
- summary versioning is mandatory
- trigger-driven orchestration is preferred over session-bound orchestration
