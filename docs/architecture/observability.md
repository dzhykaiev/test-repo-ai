# Observability

## Purpose

This document defines how to observe, debug, and operate the orchestrator-agent system in production.

Without strong observability, a multi-agent system becomes operationally opaque very quickly.

## Observability goals

- understand current system health
- understand ticket progress and blockers
- understand why the orchestrator made a decision
- understand which workers fail and why
- detect stale or inconsistent memory layers
- support fast recovery and audit

## Observability pillars

## 1. Logs

Capture structured logs for:
- request intake
- orchestration runs
- worker runs
- event writes
- artifact publication
- summary refresh
- policy violations
- recovery actions

### Required log fields
- timestamp
- level
- correlation_id
- root_ticket_id
- ticket_id
- run_id
- actor_type
- actor_id
- event_type or action_type
- status
- error_code if applicable

## 2. Metrics

Track quantitative health indicators.

### Core metrics
- requests received
- active root tickets
- tickets by status
- blocked ticket count
- average orchestration latency
- average worker latency by type
- worker failure rate by type
- retry count by category
- stale summary count
- projection lag
- queue depth
- orphan artifact count

### Recommended dimensions
- worker_type
- project or domain scope
- trigger_type
- priority level
- policy profile

## 3. Traces

Use distributed tracing or equivalent causal linking for:
- request -> ticket -> orchestrator run -> worker run -> artifact -> summary -> user response

### Trace value
Tracing is especially important because one user request may fan out into multiple worker runs and summaries.

## 4. Audit records

Separate from operational logs where needed.

Audit trail should capture:
- sensitive actions
- approval-gated actions
- manual overrides
- summary invalidations
- policy changes

## Dashboards

## 1. Control plane dashboard

Show:
- orchestrator run rate
- orchestration latency
- failed runs
- duplicate trigger handling rate
- pending reconciliation jobs

## 2. Ticket health dashboard

Show:
- ticket counts by state
- blocked tickets by age
- stale in-progress tickets
- parent tickets with failed child tickets
- oldest unresolved root tickets

## 3. Worker health dashboard

Show:
- runs by worker type
- failure rate by worker type
- timeout rate
- average retries
- average artifact output volume

## 4. Memory health dashboard

Show:
- stale summaries
- summary refresh rate
- projection lag
- mismatch alerts between summary freshness and live event recency

## Alerting

## High-priority alerts
- request intake unavailable
- orchestrator trigger backlog growing rapidly
- repeated orchestrator run failures
- projection lag above safe threshold
- worker crash loop for a core worker type
- stale blocker beyond SLA

## Medium-priority alerts
- summary refresh lag
- rising retry volume
- orphan artifacts detected
- abnormal ticket cancellation spike

## Debugging workflows

## Workflow 1: Why is a ticket stuck?
1. Inspect current ticket state.
2. Inspect latest event history.
3. Inspect open blocker record.
4. Inspect last orchestrator decision.
5. Inspect latest worker run.

## Workflow 2: Why did the orchestrator route this task?
1. Inspect orchestrator run record.
2. Inspect context package references.
3. Inspect applied policy IDs.
4. Inspect decision summary and emitted events.

## Workflow 3: Why is the system feeling stale?
1. Check summary freshness dashboard.
2. Check projection lag.
3. Compare latest events with latest summaries.
4. Run reconciliation or summary refresh.

## Required correlation keys

Everything important should be joinable by:
- `correlation_id`
- `root_ticket_id`
- `ticket_id`
- `run_id`

## Practical guidance

### For orchestrator runs
Persist a compact decision record every time.

### For worker runs
Persist terminal state and reason, even when output is empty due to failure.

### For summaries
Persist source coverage range and freshness status.

## Anti-patterns

- logs without correlation IDs
- worker output with no run tracking
- dashboards that show only output volume but not blocker age
- summaries that cannot be tied back to source events
- success metrics with no failure categorization

## Key invariant

If operators cannot explain the current state of work from logs, metrics, traces, and records, the system is not production-ready.
