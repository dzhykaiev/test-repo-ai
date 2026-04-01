# ADR-0002: Use Event-Sourced Task State with Materialized Ticket Views

## Status
Accepted

## Context

The system must preserve a trustworthy history of how work evolved while also serving fast operational reads for the orchestrator.

A mutable ticket table alone is insufficient because it makes it hard to explain:
- why state changed
- which worker produced which outcome
- when blockers appeared
- how summaries were derived

## Decision

The system will store task evolution as immutable events and maintain current ticket state through materialized views.

### Canonical write model
Append-only event stream.

### Canonical read model
Materialized ticket views optimized for operational reads.

## Consequences

### Positive
- full audit trail
- easy replay and reconstruction
- clearer debugging of orchestration behavior
- better support for summary generation

### Negative
- additional infrastructure complexity
- projection consistency must be managed carefully
- developers must think in events, not only records

## Example event types
- ticket_created
- ticket_updated
- ticket_assigned
- dependency_added
- worker_started
- worker_completed
- blocker_detected
- blocker_cleared
- artifact_published
- summary_refreshed
- ticket_closed

## Rejected alternative

### Alternative: mutable ticket rows only
Rejected because it weakens traceability and makes recovery and debugging harder.

## Follow-up implications

- event schemas must be versioned
- projections must be rebuildable
- every status transition must map to at least one event
- summaries should reference source event ranges
