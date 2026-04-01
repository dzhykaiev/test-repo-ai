# Orchestrator Control Loop

## Purpose

This document defines the runtime behavior of the primary orchestrator.

The orchestrator is the control plane. Its main job is to make safe, current, and traceable decisions while staying context-clean.

## Control loop goals

The orchestrator must:
- accept new work continuously
- react to worker outcomes
- reconcile incomplete work
- avoid prompt accumulation
- avoid uncontrolled delegation
- keep all work grounded in durable state

## Trigger model

A control loop run starts only from an explicit trigger.

### Supported triggers
- new user request
- worker completion
- worker failure
- blocker detected
- dependency cleared
- scheduled reconciliation
- stale summary detected
- manual human override

## Standard loop

1. Accept trigger.
2. Resolve target ticket scope.
3. Build bounded context.
4. Validate policy constraints.
5. Determine current execution phase.
6. Choose next control action.
7. Persist decision as state mutation plus events.
8. Emit downstream actions.
9. Return user-visible or system-visible result.

## Decision phases

## Phase 1: Intake and normalization

Questions:
- what exactly triggered this run?
- which root ticket is affected?
- is this a new request, reconciliation run, or worker outcome?

## Phase 2: State inspection

Read:
- target ticket state
- parent state
- direct child states
- open blockers
- latest summaries
- recent relevant events

Questions:
- is the ticket actionable?
- are dependencies satisfied?
- is there enough information to proceed?
- does the current summary appear stale?

## Phase 3: Control decision

Choose one of:
- close ticket
- decompose ticket
- assign ticket
- reassign ticket
- create clarification request
- block ticket
- retry ticket
- escalate ticket
- refresh summary
- do nothing except record health state

## Phase 4: Commit

Every decision must result in:
- ticket update if state changes
- at least one event
- optional downstream dispatch
- optional user-visible response

## Phase 5: Post-commit follow-up

Possible follow-ups:
- trigger worker
- trigger summary refresh
- schedule reconciliation
- notify UI
- escalate to human review

## Context discipline

The orchestrator must never start a run with inherited hidden conversational context from earlier runs.

Each run gets only:
- trigger metadata
- bounded state slice
- current policies
- relevant artifacts
- latest trusted summaries

## Routing logic

### Direct completion
Use when the ticket is a simple control decision or already has enough output to close.

### Decomposition
Use when the outcome clearly requires multiple bounded subtasks.

### Clarification
Use when ambiguity materially affects safe planning.

### Block / wait
Use when external dependency or missing input prevents useful progress.

### Retry / reroute
Use when the failure reason suggests another attempt or another worker type could succeed.

## Reconciliation logic

When a child worker completes:
- check whether acceptance criteria are satisfied
- check whether sibling tickets are still pending
- check whether a summary refresh is needed
- determine whether the parent can advance to `review` or `done`

When a worker fails:
- inspect error category
- apply retry policy
- decide whether to reroute, block, or escalate

## Concurrency rules

The orchestrator may manage many tickets, but each root ticket family should have controlled concurrency.

Recommended controls:
- max child tickets per parent
- max active workers per root ticket
- no duplicate worker assignment for same terminal goal without explicit reason
- optimistic locking or version checks on ticket updates

## Loop safety rules

1. Never spawn children without acceptance criteria.
2. Never assign a worker without explicit output expectations.
3. Never close a parent if required child tickets remain unresolved.
4. Never trust summaries over fresher live state.
5. Never retry indefinitely.

## Pseudologic

```text
on trigger:
  resolve target scope
  context = build_context(scope)
  enforce_policies(context)
  decision = decide_next_action(context)
  persist(decision)
  emit_followups(decision)
```

## Run outputs

Each orchestrator run should produce a compact decision record:
- run ID
- target ticket ID
- decision type
- rationale summary
- emitted events
- spawned workers
- user-visible message if applicable

## Key invariant

The orchestrator stays stable by behaving like a transaction-oriented decision engine, not like a permanently chatting super-agent.
