# Operations Runbook

## Purpose

This runbook defines the operational rules that keep the orchestrator reliable over time.

## Operating stance

The orchestrator is always-on as a **service role**, but it should not be implemented as a single infinitely growing conversation.

Operationally, “always on” means:
- always able to accept new triggers
- always able to rebuild fresh context
- always able to continue from persisted state
- always observable and recoverable

## Golden rules

### 1. Never rely on hidden conversational memory
Every run must be resumable from system state.

### 2. Never allow unbounded child spawning
Set hard policies for max depth, max breadth, and retry counts.

### 3. Never let workers own orchestration policy
Workers execute. Orchestrator decides.

### 4. Never lose provenance
Every artifact, summary, and state change must link back to ticket IDs and source events.

### 5. Always separate control failure from task failure
A worker failing a task must not destabilize the orchestrator.

## Trigger sources

The orchestrator should react to:
- new user request
- worker completion
- worker failure
- timeout / SLA breach
- blocker escalation
- scheduled reconciliation
- summary refresh need

## Recommended background jobs

### Reconciliation job
Runs periodically to detect:
- stale tickets
- missing owners
- overdue blockers
- hanging retries
- child tickets with closed parents

### Summary refresh job
Runs when:
- event volume exceeds threshold
- artifact count changes materially
- parent ticket state changes meaningfully

### Cleanup job
Runs periodically to archive:
- obsolete intermediate artifacts
- superseded summaries
- expired temporary worker traces

## Retry policy

### Worker retry
Use bounded retries with reason-aware backoff.

Suggested approach:
- transient tool failure: retry
- missing required input: do not retry automatically
- policy violation: do not retry automatically
- low confidence: escalate or reroute

### Orchestrator retry
The orchestrator may retry a failed control action only if the action is idempotent or transaction-safe.

## Escalation policy

Escalate when:
- blocker age exceeds threshold
- retry budget is exhausted
- ambiguity blocks safe planning
- policy conflict exists
- external dependency remains unavailable

Escalation targets may include:
- user clarification
- human reviewer
- alternate worker type
- deferred maintenance queue

## Health signals

### Required metrics
- orchestration cycle time
- ticket lead time
- number of blocked tickets
- number of stale summaries
- worker failure rate by type
- average retries per ticket
- orphan ticket count

### Required alerts
- orchestrator trigger backlog
- repeated worker crash loop
- summary store lag
- unresolved blocker age threshold
- event ingestion lag

## Safe shutdown and recovery

### Shutdown assumptions
At any time, the current process may stop unexpectedly.

### Therefore
- ticket state changes must be persisted transactionally
- events must be append-only and durable
- worker start and finish must be observable
- orchestrator runs should be re-entrant

### Recovery steps
1. Rebuild current ticket views from event store if needed.
2. Detect in-flight worker tickets.
3. Mark uncertain executions for reconciliation.
4. Re-trigger orchestrator for unresolved active tickets.

## Human override

The system must allow human operators to:
- pause ticket families
- reassign tickets
- cancel retries
- override status
- pin reference artifacts
- mark summaries as invalid

## Operational anti-patterns

- one permanent agent session as the only memory store
- direct worker-to-worker uncontrolled delegation
- state mutation without events
- free-form manual status changes with no audit trail
- summaries replacing canonical ticket state
