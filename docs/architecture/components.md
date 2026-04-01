# Components

## 1. UI / Intent Entry Layer

### Responsibility
Accept free-form user requests and convert them into normalized intents.

### Outputs
- raw user message
- normalized request
- correlation ID
- priority hints
- optional project or domain scope

### Notes
The UI should not decide how work is decomposed. Its role is to capture intent and metadata.

## 2. Intent Normalizer

### Responsibility
Transform ambiguous user input into a structured request object.

### Typical fields
- request_id
- actor
- requested_outcome
- constraints
- urgency
- scope
- referenced artifacts
- referenced tickets

### Why it matters
This removes ambiguity early and prevents the orchestrator from re-parsing the same natural language repeatedly.

## 3. Ticketing Layer

### Responsibility
Represent work as durable ticket objects.

### Ticket fields
- ticket_id
- parent_ticket_id
- title
- objective
- status
- owner_type
- owner_id
- priority
- dependencies
- acceptance_criteria
- artifact_links
- created_at
- updated_at
- due_at
- blocked_reason

### Status model
Recommended states:
- `new`
- `triaged`
- `planned`
- `assigned`
- `in_progress`
- `blocked`
- `review`
- `done`
- `cancelled`

## 4. Orchestrator

### Responsibility
Act as the control-plane brain.

### It should do
- inspect current state
- decide whether work needs decomposition
- create child tickets
- assign tickets to worker types
- merge worker outcomes
- decide next actions
- detect blockers and escalation conditions
- maintain system flow

### It should not do
- hold the entire project memory in prompt form
- perform heavy domain execution by default
- directly mutate state without emitting events

## 5. Context Builder

### Responsibility
Construct the minimum valid context for each orchestrator or worker invocation.

### Inputs
- current ticket
- parent / sibling ticket summaries
- recent events
- latest summaries
- linked artifacts
- policy set
- task-specific domain references

### Output
A bounded context package with provenance.

## 6. Policy Guard

### Responsibility
Enforce operating constraints before actions are taken.

### Example rules
- max decomposition depth
- max concurrent workers per parent ticket
- retry limits
- escalation thresholds
- tool access controls
- human approval gates

## 7. Planner / Decomposer

### Responsibility
Turn a ticket into a feasible execution graph.

### Output options
- direct completion by orchestrator
- one child ticket
- multiple child tickets in sequence
- multiple child tickets in parallel
- defer and request clarification

## 8. Dispatcher

### Responsibility
Select the right worker type for a child ticket.

### Worker examples
- research agent
- architecture writer
- documentation agent
- issue preparation agent
- validation agent
- summarization agent

### Dispatch criteria
- task type
- required tools
- confidence requirements
- latency target
- cost ceiling

## 9. Worker Agents

### Responsibility
Perform bounded, specialized execution.

### Worker contract
Input:
- ticket
- bounded context
- constraints
- expected deliverable

Output:
- result summary
- produced artifacts
- structured findings
- follow-up recommendations
- completion status
- emitted events

### Important property
Workers are ephemeral. They should be safely restartable and disposable.

## 10. Event Store

### Responsibility
Capture immutable state transitions and important observations.

### Example events
- ticket_created
- ticket_assigned
- worker_started
- worker_completed
- blocker_detected
- artifact_published
- summary_refreshed
- ticket_closed

## 11. Ticket Store

### Responsibility
Provide the materialized current state of every ticket.

### Why separate from event store
The event stream is ideal for auditability. The ticket store is ideal for fast operational reads.

## 12. Summary Store

### Responsibility
Store compressed, versioned summaries of long histories and artifact sets.

### Rule
Summaries are helpers for retrieval efficiency, not replacements for canonical state.

## 13. Artifact Store

### Responsibility
Hold outputs such as documents, plans, drafts, structured datasets, and references.

## 14. Observability and Audit Layer

### Responsibility
Expose system health and decision traceability.

### Metrics
- active tickets
- blocked tickets
- average orchestration latency
- worker failure rate
- stale summary count
- retry volume
- decomposition depth distribution
