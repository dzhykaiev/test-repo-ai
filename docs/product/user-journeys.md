# User Journeys

## Purpose

This document describes how the system should feel from the user side.

The architecture is only successful if the UI experience stays understandable while the backend remains structured and reliable.

## Product principle

The user should interact with one coherent system, even though internally the system may create many tickets and worker runs.

The user should not be forced to understand the full internal agent graph unless they want operational detail.

## Journey 1: Submit a simple request

### User goal
Ask the system to produce one artifact.

### User flow
1. User opens the UI.
2. User types a request.
3. User submits the request.
4. UI immediately confirms that the request has been accepted.
5. UI shows a top-level work item with status.
6. Later, UI shows completion and artifact links.

### What the user should see
- acknowledgment
- current status
- estimated phase or current stage
- final artifact
- concise explanation of what happened

### What should stay mostly hidden
- low-level worker dispatch details
- internal event IDs
- unnecessary system noise

## Journey 2: Submit a complex request

### User goal
Ask for a multi-step result, such as a full architecture package.

### User flow
1. User submits a broad request.
2. UI acknowledges and creates one visible root work item.
3. UI may show that the task is being broken into subtasks.
4. User sees progress grouped under one main request.
5. When subtasks complete, artifacts appear progressively.
6. The root request finishes when all required outcomes are complete.

### What the user should see
- one root request status
- optional expandable subtask list
- blockers if they matter to the user
- artifact list as outputs become available

## Journey 3: Clarification needed

### User goal
Answer missing questions so work can continue.

### User flow
1. User submits ambiguous request.
2. System responds with specific clarification questions.
3. UI marks the request as waiting for input.
4. User answers directly in the same request thread or request panel.
5. System resumes planning and execution.

### UX requirement
Clarification should feel like part of the same request, not like a brand-new disconnected task.

## Journey 4: Follow progress

### User goal
Understand what is happening without reading internal logs.

### User flow
1. User opens a request detail page.
2. UI shows:
   - current status
   - latest update
   - open blockers
   - produced artifacts
   - recent major decisions
3. User may expand details to inspect subtask progress.

### UX requirement
Default view should stay simple.
Advanced detail should be optional.

## Journey 5: Failure or delay

### User goal
Understand why work is delayed and what happens next.

### User flow
1. A worker times out or a blocker appears.
2. UI updates the main request status.
3. UI explains the blocker category in human-readable form.
4. UI shows next action, for example:
   - retrying
   - waiting for user input
   - rerouted to another worker
   - escalated for review

### UX requirement
The system should never look like it silently froze.

## Journey 6: Review outputs incrementally

### User goal
Consume partial outputs before the entire request is finished.

### User flow
1. User submits a large request.
2. Some child tickets complete before others.
3. UI exposes completed artifacts immediately.
4. Root request remains open until the package is complete.

### UX requirement
Progressive delivery should be possible without corrupting the root state model.

## Journey 7: Reopen or extend work

### User goal
Ask for changes after initial completion.

### User flow
1. User opens a completed request.
2. User asks for revision or extension.
3. System creates either:
   - a new child ticket under same root family, or
   - a follow-up root ticket linked to the original
4. UI preserves traceability to prior outputs.

### UX requirement
The system should show continuity of work, not treat every follow-up as unrelated.

## User-visible status model

Recommended simple statuses for UI:
- `Received`
- `Planning`
- `Working`
- `Waiting`
- `Reviewing`
- `Done`
- `Cancelled`

These may map from richer internal statuses.

## User-visible explanations

Every status should support a short explanation.

Examples:
- `Planning` — breaking the request into the next steps
- `Working` — specialized agents are producing outputs
- `Waiting` — blocked on your input or an external dependency
- `Reviewing` — outputs exist and are being reconciled

## UI design implications

### The UI should support
- one root request view
- expandable subtasks
- artifact list
- blocker explanations
- clarification input
- recent activity timeline

### The UI should avoid
- exposing raw event stream by default
- exposing all worker internals as first-class user concepts
- mixing completed and failed subtasks without explanation

## Product invariant

From the user perspective, the system should feel like one reliable operator that stays on top of the work, even though internally it is a ticketed orchestration platform.
