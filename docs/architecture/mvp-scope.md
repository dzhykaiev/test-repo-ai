# MVP Scope

## Purpose

This document defines the smallest version of the system that still validates the architecture.

The MVP should prove that:
- the orchestrator can stay context-clean
- user requests can become tickets
- tickets can spawn bounded workers
- outputs can be persisted and reconciled
- the system can recover from common failures

## MVP objective

Build the first production-capable slice of the orchestration platform with the minimum number of moving parts.

## MVP success criteria

The MVP is successful if it can:
- accept a request from UI
- create a root ticket
- decompose into child tickets when needed
- dispatch at least one specialized worker type
- publish artifacts and summaries
- recover from worker failure or timeout
- present current work state to the user

## In scope for MVP

## 1. Request intake
- receive user message
- assign correlation ID
- normalize request into structured intent

## 2. Ticketing
- create root ticket
- create child tickets
- update ticket status
- maintain parent-child relationships

## 3. Event history
- append key events
- support replay for debugging and recovery

## 4. Orchestrator
- trigger on new request
- trigger on worker completion/failure
- build bounded context
- choose direct complete / decompose / block / clarify / assign

## 5. Workers
At least these worker types:
- documentation worker
- summarization worker

Optional early third worker:
- validation worker

## 6. Artifact publishing
- persist documents
- link artifacts to tickets

## 7. Summary generation
- root ticket summary
- ticket summary

## 8. Recovery basics
- detect timed-out workers
- rerun reconciliation
- mark stale summaries

## 9. Minimal observability
- logs
- metrics for worker failures and blocked tickets
- basic run tracing

## Out of scope for MVP

- complex multi-tenant governance
- advanced fine-grained policy engine
- autonomous worker-to-worker delegation
- cost optimization routing across many model vendors
- very deep dependency graphs
- automatic prioritization across many business units
- large-scale streaming event platform
- full human operations console with all override features

## Recommended MVP architecture choices

### Simplicity-first choices
- single relational DB for tickets, events, summaries, and metadata
- queue-based worker dispatch
- one stateless orchestrator service
- one UI entry channel
- object or repo-backed storage for artifacts

### Why this is enough
This proves the architecture without prematurely distributing every subsystem.

## Minimal worker contract for MVP

Each worker must:
- accept one ticket
- accept bounded context
- produce one result summary
- optionally publish artifacts
- emit one terminal event

## Minimal summary strategy for MVP

Generate summaries only for:
- root ticket
- child ticket on completion

Avoid overbuilding project-wide memory until ticket-level flows are stable.

## Minimum policies for MVP

- max child tickets per parent
- max retry count
- worker timeout
- approval needed for sensitive actions
- no worker spawning by workers

## Minimum reconciliation jobs for MVP

- stale in-progress ticket scan
- timed-out worker scan
- stale summary scan

## MVP failure cases that must be handled

- worker timeout
- duplicate request submission
- artifact publish failure
- orchestrator rerun on same trigger
- summary stale or invalid

## MVP demo scenarios

## Scenario 1: Simple doc request
User asks for architecture document.
System creates root ticket.
Orchestrator dispatches documentation worker.
Worker publishes doc.
Orchestrator closes ticket.

## Scenario 2: Multi-step request
User asks for architecture package.
Orchestrator creates child tickets for overview, ADR, and summary.
Workers complete outputs.
Parent ticket moves to done.

## Scenario 3: Failure and recovery
Worker times out.
Recovery scan marks run failed.
Orchestrator reroutes or retries.
Ticket family remains consistent.

## Exit criteria for MVP

Move beyond MVP only when:
- ticket lifecycle is stable
- worker contracts are consistent
- summary freshness is manageable
- recovery paths work in practice
- observability reveals reliable system health

## Key invariant

The MVP must validate the architecture pattern, not just produce outputs. If the first version requires a giant hidden prompt to work, it is not a real MVP of this architecture.
