# Technology Mapping

## Purpose

This document maps the logical architecture to a practical implementation stack without forcing a single vendor-specific solution.

The goal is to help implementation start with a clean separation between:
- control plane
- execution plane
- state storage
- retrieval and summarization
- observability and operations

## Mapping principle

Choose technologies that support these architectural requirements first:
- stateless orchestrator runs
- durable event history
- fast ticket read views
- explicit worker dispatch
- bounded context assembly
- strong observability
- safe retries and recovery

## Logical component to technical responsibility mapping

## 1. UI / Intent Entry Layer

### Logical role
Capture user intent and attachments.

### Technical options
- web app frontend
- chat-style UI
- internal operations dashboard
- API gateway for external systems

### Implementation notes
The UI does not need deep orchestration logic. It only needs to:
- collect message and metadata
- attach correlation IDs
- show ticket state and outputs
- expose clarification questions and progress

## 2. Request Intake / Intent Normalization

### Logical role
Turn raw request into structured work input.

### Technical options
- lightweight backend service
- workflow entrypoint service
- model-backed normalization step

### Implementation notes
This layer should persist the raw request before normalization where practical.

## 3. Ticketing Layer

### Logical role
Represent work and current state.

### Technical options
- relational database for ticket current view
- document store if schema flexibility is needed

### Recommended default
Use a relational database first.

Why:
- ticket queries are structured
- ownership and dependency filters are common
- operational reporting is easier
- consistency is simpler than with purely document-first designs

## 4. Event Store

### Logical role
Append-only history of state changes.

### Technical options
- append-only table in relational DB for MVP
- log/stream platform for higher scale
- workflow engine history if it supports durable replay cleanly

### Recommended default
Start with an append-only events table in the same operational database for MVP, then split later if scale requires it.

## 5. Orchestrator Runtime

### Logical role
Run control decisions from fresh state.

### Technical options
- stateless backend service invoking model calls
- workflow orchestrator with task handlers
- job-driven control loop runner

### Recommended properties
- horizontally scalable
- trigger-driven
- idempotent run handling
- no dependence on sticky session memory

## 6. Context Builder

### Logical role
Assemble bounded context packages.

### Technical options
- retrieval service over relational state + artifact metadata
- vector index for long-form artifacts and summaries
- hybrid search over structured and semantic data

### Recommended default
Use hybrid retrieval:
- structured DB queries for tickets/events/policies
- semantic retrieval for large documents and artifact bodies

## 7. Worker Execution Plane

### Logical role
Run bounded specialized tasks.

### Technical options
- queue-based workers
- workflow engine activities
- container jobs
- serverless tasks for short workloads

### Recommended default
Use queue-based workers with explicit worker types.

Why:
- simple dispatch model
- clean failure isolation
- easy horizontal scale
- explicit retry behavior

## 8. Artifact Store

### Logical role
Persist outputs.

### Technical options
- object storage
- Git repository for versioned documents
- relational metadata + object content storage

### Recommended default
Separate artifact metadata from artifact content.

Store:
- metadata in DB
- document/blob content in object or versioned repository store

## 9. Summary Store

### Logical role
Persist versioned summaries with provenance.

### Technical options
- relational table for summary metadata and bodies
- document store if summaries become very large
- vector index for semantic retrieval of summaries

### Recommended default
Relational table plus vector indexing for search.

## 10. Messaging / Trigger Transport

### Logical role
Signal orchestrator runs and worker starts.

### Technical options
- message queue
- event bus
- workflow scheduler
- cron + DB scan for simple systems

### Recommended default
Use a queue for worker dispatch and a scheduler/queue combo for orchestrator triggers.

## 11. Observability Stack

### Logical role
Track health, traces, logs, and metrics.

### Technical options
- metrics platform
- centralized logging
- distributed tracing
- error tracking system

### Required visibility
- run lifecycle
- ticket lifecycle
- worker failure patterns
- summary freshness lag
- queue depth
- token and tool usage

## Suggested baseline stack profile

This is an example profile, not a mandate.

### MVP-friendly profile
- UI: web app or internal dashboard
- API/backend: stateless service
- ticket + event store: relational database
- worker dispatch: queue
- artifact content: object store or repo-backed docs
- summaries: relational + semantic index
- observability: logs + metrics + traces

### Scale-up profile
- separate event streaming infrastructure
- dedicated retrieval service
- specialized worker pools by workload class
- policy engine service
- stronger workflow orchestration runtime

## Model-layer mapping

### Orchestrator model profile
Prefer a model profile optimized for:
- structured decision making
- instruction following
- stable routing and decomposition
- tool-calling discipline

### Worker model profile
Use worker-specific model profiles depending on task:
- drafting
- analysis
- classification
- summarization
- validation

### Important rule
Do not assume the same model profile is optimal for orchestrator and workers.

## Storage mapping recommendation

## Minimum storage set
- `tickets`
- `events`
- `artifacts`
- `summaries`
- `agent_runs`
- `context_packages`
- `policies`

## Search / retrieval mapping

### Structured retrieval
Use for:
- ticket status
- ownership
- dependencies
- blocker state
- event recency

### Semantic retrieval
Use for:
- long-form architecture docs
- prior outputs
- requirement notes
- artifact similarity

## Deployment guidance

### Separate control plane from worker plane
This makes it easier to:
- scale workers independently
- protect orchestrator reliability
- apply stricter permissions to worker classes

### Prefer stateless services for orchestrator entrypoints
This supports horizontal scale and safe restart behavior.

## Anti-mappings to avoid

- storing all memory only in one chat transcript
- making the UI the source of truth for work state
- using summaries as the only retrievable memory source
- mixing worker output storage with unversioned temporary logs
- letting one universal worker own all task types by default

## Implementation order recommendation

1. Request intake
2. Ticket + event persistence
3. Orchestrator trigger loop
4. One worker type
5. Artifact publishing
6. Summary generation
7. Reconciliation jobs
8. Observability hardening

## Key invariant

Technology choices are replaceable. The architecture is preserved as long as stateless orchestration, durable state, bounded context, and explicit worker contracts remain intact.
