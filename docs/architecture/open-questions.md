# Open Questions

## Purpose

This document captures the unresolved design questions that should be answered before or during implementation.

Not all of them block the MVP, but many of them deserve explicit decisions or future ADRs.

## 1. Ticket granularity

### Question
How small should child tickets be by default?

### Why it matters
If tickets are too large, workers become overloaded. If tickets are too small, orchestration overhead explodes.

### Needs decision on
- target size of one worker task
- when to decompose vs keep direct execution
- max child count per parent

## 2. Root ticket continuity

### Question
When the user asks for a follow-up, should the system:
- reopen the existing root ticket,
- create a new child under the same root family,
- or create a new root linked to the original?

### Why it matters
This affects traceability, UI simplicity, and summary scope.

## 3. Worker taxonomy

### Question
Which worker types are first-class in v1, and which should remain routing variants inside a smaller set?

### Why it matters
Too many worker types make dispatch and observability noisy. Too few make contracts vague.

## 4. Summary refresh policy

### Question
When exactly should summaries be refreshed?

### Candidate strategies
- after every terminal child ticket
- after N events
- on parent state transition
- on freshness threshold breach only

### Why it matters
This affects cost, latency, and memory quality.

## 5. Confidence policy

### Question
How should low-confidence worker results affect control flow?

### Options
- always require review
- retry with same worker
- reroute to another worker
- convert to clarification request

## 6. Human review thresholds

### Question
Which action categories require explicit human approval in early phases?

### Why it matters
This is essential for safe rollout and affects product behavior.

## 7. Context retrieval strategy

### Question
What exact retrieval mix should be used for orchestrator runs?

### Candidate inputs
- live ticket state only
- ticket state + recent events
- ticket state + summaries + selective artifact retrieval
- hybrid semantic + structured retrieval

## 8. Artifact authority

### Question
Which artifacts count as canonical deliverables versus intermediate drafts?

### Why it matters
This affects UI presentation, summary generation, and closure criteria.

## 9. Ticket closure policy

### Question
What exact evidence is required before a ticket can move to `done`?

### Candidate requirements
- all acceptance criteria checked
- all mandatory artifacts published
- no open blocker remains
- parent reconciliation completed

## 10. Blocker taxonomy depth

### Question
How detailed should blocker categories be in v1?

### Why it matters
Too coarse reduces operational clarity. Too detailed increases complexity early.

## 11. Retry strategy design

### Question
Should retries be owned entirely by the orchestrator, partially automated by the worker runtime, or split by failure category?

### Why it matters
This affects determinism and debugging.

## 12. Queueing and concurrency limits

### Question
What are the concurrency limits per:
- root ticket family
- worker type
- project scope

### Why it matters
Without explicit limits, fan-out can become unstable.

## 13. Summary invalidation policy

### Question
Who or what can mark a summary `invalid`?

### Candidates
- orchestrator automatically
- validation worker
- human operator
- policy engine

## 14. UI detail level

### Question
How much of the internal ticket graph should the default UI expose?

### Why it matters
Too much detail overwhelms users. Too little makes the system feel opaque.

## 15. External system integration boundary

### Question
In the first implementation, should the system only manage internal artifacts, or also create external tickets/issues/tasks automatically?

### Why it matters
External side effects raise the bar for safety, approval, and idempotency.

## 16. Policy engine maturity

### Question
Should policies be hardcoded in early phases or modeled as versioned external policy records from day one?

### Why it matters
This affects implementation speed versus future governance.

## 17. Long-term memory scope

### Question
Should there be a project-level memory beyond ticket and summary scope, and if so, what qualifies for entry into it?

### Why it matters
This is where many systems accidentally reintroduce prompt-bloat thinking under another name.

## 18. Model selection strategy

### Question
Should the orchestrator use one stable model profile while workers vary by task, or should both be dynamically selected?

### Why it matters
This affects routing complexity, cost, and predictability.

## 19. Validation ownership

### Question
Who is responsible for validating worker outputs before parent closure?

### Candidates
- orchestrator directly
- dedicated validation worker
- human review for certain classes

## 20. Initial ADR candidates

Recommended next ADRs:
- ADR for ticket closure rules
- ADR for summary refresh policy
- ADR for clarification handling model
- ADR for worker taxonomy v1
- ADR for external side-effect authorization

## Key principle

Unresolved questions are normal. What matters is making them explicit before they become hidden behavior in the system.
