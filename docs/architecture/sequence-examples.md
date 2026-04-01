# Sequence Examples

## Purpose

This document provides concrete end-to-end examples of how the orchestrator-agent platform should behave.

These examples are intentionally implementation-agnostic. They show control flow, state changes, and decision boundaries.

## Scenario 1: Simple documentation request

### User intent
"Write an architecture overview for the agent orchestrator."

### Sequence
1. User sends request from UI.
2. Request Intake assigns `correlation_id` and stores the raw request.
3. Intent Normalizer creates structured intent.
4. Ticketing Layer creates a root ticket:
   - objective: produce architecture overview
   - status: `new`
5. Event `ticket_created` is emitted.
6. Orchestrator trigger `new_request` starts.
7. Context Builder assembles:
   - root ticket
   - current policy set
   - relevant prior artifacts if any
8. Orchestrator determines that no decomposition is needed.
9. Dispatcher selects `documentation_worker`.
10. Child ticket is created and assigned.
11. Worker runs with bounded context.
12. Worker publishes architecture document artifact.
13. Worker emits `worker_completed`.
14. Orchestrator reconciles child output.
15. Parent ticket moves to `done`.
16. Root summary is refreshed.
17. UI receives completion response with artifact link.

### Key properties proven
- simple task can stay small
- orchestrator does not over-plan
- artifact and summary are linked to the ticket

## Scenario 2: Multi-step architecture package

### User intent
"Create complete architecture docs for the orchestrator platform."

### Sequence
1. User submits request.
2. Root ticket is created.
3. Orchestrator inspects request and decides decomposition is required.
4. Planner creates child tickets such as:
   - overview
   - components
   - flows
   - data model
   - ADR package
5. Dispatcher routes:
   - documentation tasks to `documentation_worker`
   - summary tasks to `summarization_worker`
6. Workers execute in parallel where dependencies allow.
7. Each worker publishes artifacts and terminal events.
8. Orchestrator monitors child completion and blocker state.
9. If all acceptance criteria are met, parent moves to `review` and then `done`.
10. Root summary and project artifact index are refreshed.

### Key properties proven
- one request can fan out safely
- child tickets remain bounded
- parent closure depends on explicit reconciliation

## Scenario 3: Clarification required

### User intent
"Build the full system for my team."

### Sequence
1. User request is received.
2. Root ticket is created.
3. Orchestrator inspects request and finds ambiguity:
   - unclear target domain
   - unclear scope
   - unclear delivery expectation
4. Orchestrator chooses `request_clarification` instead of unsafe planning.
5. Clarification question set is generated.
6. Root ticket moves to `blocked` or remains `triaged` depending on policy.
7. UI displays clarification questions.
8. User answers.
9. New input is linked to the same root ticket family.
10. Orchestrator reruns with updated context.

### Key properties proven
- ambiguity is handled as state, not as chat drift
- the system does not silently guess on material uncertainty

## Scenario 4: Worker timeout and recovery

### User intent
"Produce a full architecture package."

### Sequence
1. Orchestrator assigns a documentation child ticket.
2. Worker starts but does not complete before timeout.
3. Recovery job detects expired worker run.
4. Worker run is marked failed or uncertain.
5. Event `worker_failed` with category `timeout` is emitted.
6. Orchestrator trigger starts.
7. Orchestrator inspects:
   - retry policy
   - partial artifacts
   - worker history
8. Orchestrator decides one of:
   - retry same worker
   - reroute to another worker type
   - split ticket into smaller tickets
   - mark blocked
9. Ticket family remains consistent and visible.

### Key properties proven
- timeouts do not corrupt ticket state
- recovery is based on durable state, not guesswork

## Scenario 5: Stale summary correction

### Situation
A root ticket summary exists, but many new events have occurred.

### Sequence
1. Summary freshness scan detects a gap between latest source event and latest summary.
2. Summary is marked `stale`.
3. Summarization worker regenerates the summary from source event range and artifacts.
4. New summary version is stored.
5. Old summary remains referenced for audit history.
6. Orchestrator uses the fresh summary on next run.

### Key properties proven
- memory is managed explicitly
- summary staleness is a known operational state

## Scenario 6: Unsafe worker recommendation

### Situation
A worker returns output plus a recommendation to take an action outside allowed scope.

### Sequence
1. Worker completes with artifact and recommendation.
2. Policy Guard inspects the recommended next action.
3. Action is found to be out of allowed worker scope.
4. Recommendation is stored as data but not executed.
5. Orchestrator decides whether to escalate, ignore, or convert it into a reviewed ticket.

### Key properties proven
- worker output is advisory unless policy allows direct action
- policy remains above worker reasoning

## Sequence summary

Across all scenarios, the critical execution shape is the same:

```text
UI request
  -> request intake
  -> root ticket
  -> orchestrator run
  -> bounded context
  -> control decision
  -> child ticket(s)
  -> worker run(s)
  -> artifacts/events
  -> orchestrator reconciliation
  -> summary refresh
  -> UI update
```

## Key invariant

Even complex flows should look like controlled ticket evolution, not like one giant opaque conversation.
