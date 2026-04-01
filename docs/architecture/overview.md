# Architecture Overview

## Problem statement

A long-lived agent that continuously receives requests through a UI will eventually degrade if it relies on raw conversational context as its main memory. Prompt inflation causes drift, stale assumptions, repeated work, and brittle reasoning.

The target architecture must allow a primary agent to:

- accept new work continuously
- remain stable over long periods
- stay aware of the current state of the system
- coordinate multiple specialized agents
- preserve a clean reasoning context on every run

## Architectural answer

The system is designed as a **stateless orchestrator with durable memory and ephemeral workers**.

The orchestrator is executed in runs. Each run rebuilds a compact operating context from structured state stores rather than from raw dialogue history.

## Top-level principles

### 1. Orchestrator is a control plane
The primary agent should make routing, decomposition, prioritization, and completion decisions. It should not spend most of its time doing deep domain work.

### 2. Workers are data plane actors
Specialized agents execute bounded work: research, drafting, analysis, classification, synthesis, or external tool workflows.

### 3. Ticket is the canonical work object
Every significant request becomes a ticket. The ticket graph captures parent-child relationships, ownership, status, dependencies, and deliverables.

### 4. Events are immutable facts
Changes to work state are recorded as events. Current state is derived from the event stream plus compact materialized views.

### 5. Summaries are generated artifacts, not memory truth
Summaries help compress history, but they are subordinate to source facts and current state.

### 6. Context is assembled per decision
No agent should receive the entire project history by default. Context is built specifically for the current decision or task.

## Logical architecture

```text
UI / API
  -> Intent Normalizer
  -> Ticketing Layer
  -> Orchestrator Run
       -> Context Builder
       -> Policy Guard
       -> Planner / Decomposer
       -> Dispatcher
       -> State Reader / Writer
       -> Summary Reader
  -> Worker Agents
       -> Tools / Knowledge / Artifact Generation
  -> Event Store
  -> Ticket Store
  -> Artifact Store
  -> Summary Store
  -> Observability / Audit
```

## Required system properties

### Freshness
The orchestrator must always reconstruct its view from current persisted state.

### Bounded context windows
Each agent invocation receives only relevant state slices.

### Recoverability
Any interrupted run should be resumable from persisted state.

### Traceability
Every decision must be explainable from tickets, events, policies, and outputs.

### Isolation
Worker failures must not corrupt orchestrator state.

### Deterministic control behavior
Orchestration policy should be as deterministic as practical, even if worker outputs are probabilistic.

## Invariants

1. No untracked work exists outside tickets.
2. No ticket state changes without an event.
3. The orchestrator never depends on a hidden long-running prompt history.
4. Workers are disposable and replaceable.
5. Summaries are versioned and attributable.
6. Final user-visible state is always reconstructable.

## Recommended execution model

- Trigger orchestrator on new user request
- Trigger orchestrator on worker completion
- Trigger orchestrator on SLA timeout or blocker timeout
- Trigger orchestrator on schedule for health reconciliation

This produces a system that stays active without forcing one continuously bloated conversation thread.
