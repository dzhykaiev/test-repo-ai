# Failure and Recovery

## Purpose

This document defines how the system behaves when things go wrong.

The orchestrator-agent system must assume failures are normal:
- workers fail
- tools fail
- network calls fail
- summaries become stale
- processes restart unexpectedly
- duplicate triggers happen

The architecture must survive these failures without losing control of work state.

## Failure model

### Failure domains
- request intake failure
- orchestrator run failure
- worker execution failure
- event persistence failure
- ticket projection lag
- artifact publish failure
- summary refresh failure
- scheduler / trigger failure

## Core recovery principle

**No critical progress may exist only in volatile runtime memory.**

Everything needed for continuation must be persisted as state, events, or artifacts.

## Recovery strategy by layer

## 1. Request intake

### Risks
- duplicate request submission
- partial normalization
- request accepted but ticket not created

### Controls
- correlation IDs
- idempotent intake handling
- append `request_received` event before deeper processing where practical
- compensating action if ticket creation fails

## 2. Orchestrator run

### Risks
- crash mid-decision
- duplicate run on same trigger
- stale read before write

### Controls
- run IDs
- optimistic concurrency on ticket writes
- decision commit only through transaction-safe steps
- replay-safe trigger handling

## 3. Worker execution

### Risks
- timeout
- partial artifact generation
- tool failure
- terminal status not published

### Controls
- explicit run state
- worker heartbeat or checkpoint where useful
- terminal event required for completion
- recovery scan for in-flight workers with expired deadlines

## 4. Event store

### Risks
- write failure
- duplicate append
- ordering issues

### Controls
- append acknowledgments
- idempotency keys for duplicate-sensitive events
- monotonic timestamps or sequence IDs where available
- rebuildable read models

## 5. Materialized ticket views

### Risks
- lag behind event stream
- corruption
- stale status exposure

### Controls
- projection lag metrics
- ability to rebuild from event store
- visible freshness indicators
- orchestrator checks for projection staleness on critical flows

## 6. Artifact store

### Risks
- artifact uploaded but not linked
- artifact link created but content missing
- duplicate conflicting versions

### Controls
- content hash
- version metadata
- ticket linkage required
- publish as two-phase process when needed: store then link

## 7. Summary store

### Risks
- stale summaries
- bad summarization
- summary references wrong source window

### Controls
- source event range tracking
- freshness monitoring
- summary quality status
- invalidation path for known-bad summaries

## Recovery flows

## Flow A: Orchestrator crashed mid-run

1. Recovery job identifies a `started` orchestrator run with no terminal state.
2. Read target ticket version and emitted events.
3. If no decision was committed, safely rerun.
4. If partial commit occurred, reconcile from persisted state and continue.

## Flow B: Worker timed out

1. Ticket remains `in_progress` but worker deadline expires.
2. Recovery job marks worker run as failed or uncertain.
3. Emit timeout event.
4. Orchestrator decides retry, reroute, or block.

## Flow C: Ticket stuck with no owner activity

1. Reconciliation finds stale ticket age beyond threshold.
2. Inspect latest event and open blockers.
3. Trigger orchestrator recovery run.
4. Reassign, retry, or escalate.

## Flow D: Projection lag detected

1. Monitoring detects ticket view lag behind event stream.
2. For critical decisions, orchestrator reads source events directly.
3. Projection rebuild or catch-up job runs.
4. Health alert remains open until lag returns below threshold.

## Flow E: Summary known stale

1. Freshness gap crosses policy threshold.
2. Summary marked `stale`.
3. Refresh job regenerates summary from event range and linked artifacts.
4. New version supersedes old summary.

## Retry policies

## Safe to retry
- transient tool failure
- network timeout
- temporary dependency unavailable

## Unsafe to retry blindly
- ambiguous partial side effects
- policy violation
- malformed request
- repeated low-confidence output with same context

## Reconciliation jobs

Recommended recurring jobs:
- stale ticket scan
- in-flight run timeout scan
- orphan artifact scan
- summary freshness scan
- projection lag scan
- duplicate trigger scan

## Human recovery hooks

Operators should be able to:
- invalidate a summary
- cancel a run
- reopen a ticket
- reassign ownership
- force reconciliation
- freeze a ticket family

## Failure observability

Track at minimum:
- failed worker runs by type
- orchestrator rerun rate
- projection lag
- stale ticket count
- stale summary count
- retry exhaustion count
- orphan artifact count

## Key invariant

The system is recoverable only if all meaningful state transitions are externalized from the model runtime into durable system records.
