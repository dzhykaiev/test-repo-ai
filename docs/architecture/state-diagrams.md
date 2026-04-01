# State Diagrams

## Purpose

This document describes the major lifecycle states in text form.

The diagrams are written as state transition guides for:
- tickets
- agent runs
- blockers
- summaries

## 1. Ticket lifecycle

### Primary ticket states
- `new`
- `triaged`
- `planned`
- `assigned`
- `in_progress`
- `blocked`
- `review`
- `done`
- `cancelled`

### Standard transitions

```text
new -> triaged
triaged -> planned
planned -> assigned
assigned -> in_progress
in_progress -> review
review -> done
```

### Alternate transitions

```text
triaged -> blocked
planned -> blocked
assigned -> blocked
in_progress -> blocked
blocked -> assigned
blocked -> in_progress
triaged -> cancelled
planned -> cancelled
review -> in_progress
review -> blocked
```

### Meaning of states

#### `new`
Ticket exists but has not yet been evaluated.

#### `triaged`
The request is understood enough for basic routing and prioritization.

#### `planned`
Execution approach exists, possibly including decomposition into child tickets.

#### `assigned`
An owner exists and execution is ready to start.

#### `in_progress`
A worker or human is actively executing the ticket.

#### `blocked`
Execution cannot continue safely due to dependency, missing input, or policy constraint.

#### `review`
The expected output exists and must be reconciled or validated.

#### `done`
Acceptance criteria are satisfied.

#### `cancelled`
The ticket will not be pursued further.

## 2. Parent-child ticket family state guidance

### Parent ticket should not move to `done` unless
- required child tickets are terminal
- acceptance criteria are satisfied at parent level
- blocker state is resolved or intentionally accepted

### Parent ticket may remain in `review` while
- child outputs are complete
- final synthesis or validation is pending

### Parent ticket may move back from `review` to `in_progress` when
- validation fails
- new required child tickets are created
- output quality is insufficient

## 3. Agent run lifecycle

### Run states
- `started`
- `completed`
- `failed`
- `cancelled`

### Standard transitions

```text
started -> completed
started -> failed
started -> cancelled
```

### Notes
- there should be exactly one terminal state per run
- duplicate terminal writes should be prevented or handled idempotently

## 4. Worker execution sub-states

These may be internal and optional, but useful for observability.

- `queued`
- `starting`
- `executing`
- `publishing_artifacts`
- `completed`
- `failed`
- `timed_out`

### Example transition

```text
queued -> starting -> executing -> publishing_artifacts -> completed
queued -> starting -> executing -> timed_out
queued -> cancelled
```

## 5. Blocker lifecycle

### Blocker states
- `open`
- `acknowledged`
- `resolved`

### Standard transitions

```text
open -> acknowledged
acknowledged -> resolved
open -> resolved
```

### Notes
- a ticket may stay `blocked` while a blocker is `acknowledged`
- blocker resolution should trigger ticket reevaluation

## 6. Summary lifecycle

### Summary quality states
- `draft`
- `trusted`
- `stale`
- `invalid`

### Standard transitions

```text
draft -> trusted
trusted -> stale
stale -> trusted
trusted -> invalid
stale -> invalid
```

### Meaning

#### `draft`
Generated but not yet accepted as good retrieval material.

#### `trusted`
Safe for regular retrieval and orchestration use.

#### `stale`
No longer current relative to source events or artifacts.

#### `invalid`
Known to be wrong, unsafe, or superseded by critical correction.

## 7. Clarification request state pattern

Clarification can be modeled as:
- a `blocked` ticket with `missing_input` blocker
- or a dedicated clarification child ticket

### Example pattern

```text
triaged -> blocked
blocked -> triaged
triaged -> planned
```

## 8. Recovery state pattern

### Timeout recovery

```text
in_progress -> blocked
blocked -> assigned
assigned -> in_progress
```

### Validation failure recovery

```text
review -> in_progress
in_progress -> review
review -> done
```

## 9. Terminal state guidance

### Terminal ticket states
- `done`
- `cancelled`

### Terminal run states
- `completed`
- `failed`
- `cancelled`

### Non-terminal states
Everything else must remain eligible for reconciliation.

## 10. State discipline rules

1. No ticket should skip directly from `new` to `done` unless policy allows a truly trivial direct-complete path.
2. No ticket should remain `in_progress` indefinitely without fresh events.
3. No parent should be `done` with unresolved mandatory children.
4. No summary marked `stale` should be preferred over fresher live state.
5. No run should have multiple terminal outcomes.

## Key invariant

States exist to make control decisions explicit. If a human operator cannot look at a ticket family and understand what phase it is in, the state model is too weak.
