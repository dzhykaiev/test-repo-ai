# Execution Flows

## Flow 1: New request from UI

1. User submits a request in the UI.
2. The request receives a correlation ID.
3. Intent Normalizer creates a structured request.
4. Ticketing Layer creates a parent ticket.
5. An event `ticket_created` is emitted.
6. Orchestrator is triggered.
7. Context Builder assembles a bounded decision context.
8. Orchestrator decides one of the following:
   - complete immediately
   - request clarification
   - create child tickets
   - mark blocked
9. Events and ticket state are persisted.

## Flow 2: Ticket decomposition

1. Orchestrator reads parent ticket objective and constraints.
2. Planner determines execution graph.
3. For each required subtask, a child ticket is created.
4. Child tickets inherit traceability links to parent ticket.
5. Dispatcher assigns each child ticket to a worker class.
6. Events `ticket_created` and `ticket_assigned` are emitted.

### Decomposition rules
- keep child tickets outcome-oriented
- keep tasks independently completable where possible
- avoid deep recursive trees unless strictly necessary
- cap decomposition depth with policy

## Flow 3: Worker execution

1. Dispatcher selects a worker type.
2. Context Builder creates a task-specific worker context.
3. Worker executes bounded work.
4. Worker emits progress or milestone events if needed.
5. Worker publishes artifacts and structured output.
6. Worker marks ticket as `review`, `done`, or `blocked`.
7. Event `worker_completed` or `blocker_detected` is emitted.
8. Orchestrator is triggered again.

## Flow 4: Parent reconciliation

1. Orchestrator receives a worker completion signal.
2. It reads updated child ticket states.
3. It decides whether:
   - the parent can be completed
   - additional child tickets are needed
   - another specialized worker is needed
   - a blocker must be escalated
4. Parent state is updated.
5. Relevant summaries are refreshed.

## Flow 5: Clarification loop

1. Orchestrator detects insufficient information.
2. It does not guess silently when ambiguity is material.
3. It creates a clarification ticket or responds to the user with precise questions.
4. Parent ticket remains in `blocked` or `triaged` state.
5. New user input reactivates planning.

## Flow 6: Blocker handling

1. Worker or orchestrator detects blocker.
2. Blocker is recorded structurally with cause category.
3. Ticket state becomes `blocked`.
4. Escalation policy decides one of:
   - wait for dependency
   - ask user for clarification
   - reroute to another worker
   - retry later
   - mark failed with explanation

### Recommended blocker categories
- missing_input
- missing_artifact
- external_tool_failure
- dependency_wait
- permission_denied
- policy_violation
- low_confidence

## Flow 7: Periodic health reconciliation

1. A scheduled orchestrator run scans system health.
2. It finds:
   - stale tickets
   - overdue blocked tickets
   - orphan child tickets
   - outdated summaries
   - failed worker retries
3. The orchestrator creates recovery or maintenance tickets.

## Flow 8: Summary refresh

1. When enough new events accumulate, summary refresh is triggered.
2. Summarization agent compacts event sequences and artifact deltas.
3. New summary version is stored with references to source range.
4. Old summaries are retained for auditability.

## State transition guidance

Typical happy path:

`new -> triaged -> planned -> assigned -> in_progress -> review -> done`

Blocker path:

`in_progress -> blocked -> assigned -> in_progress -> review -> done`

Cancellation path:

`triaged -> cancelled`

## Anti-flows to avoid

- worker directly changing unrelated tickets
- orchestrator re-reading full historical chat on every run
- unbounded spawning of child agents
- tickets without owners
- artifacts without ticket links
- summaries without provenance
