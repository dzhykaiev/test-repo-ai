# API Contracts

## Purpose

This document defines the logical contracts between:
- UI and orchestration system
- orchestrator and ticketing subsystem
- orchestrator and worker agents
- workers and artifact / summary publication

These are architecture-level contracts, not implementation-specific HTTP schemas.

## Contract design principles

1. Every request carries a correlation ID.
2. Every execution path resolves to one or more tickets.
3. Every worker invocation is explicit and bounded.
4. Every completion returns structured status and outputs.
5. Every state mutation emits an event.

## 1. UI -> Request Intake

### Intent
Accept a free-form user request.

### Input contract
- `correlation_id`
- `actor_id`
- `message`
- `attachments[]`
- `scope`
- `priority_hint`
- `ui_metadata`

### Output contract
- `request_id`
- `root_ticket_id`
- `accepted_at`
- `normalized_intent_preview`
- `status` — `accepted`, `needs_clarification`, `rejected`

## 2. Request Intake -> Ticketing Layer

### Intent
Create or update the root ticket.

### Input contract
- `request_id`
- `normalized_intent`
- `requested_outcome`
- `constraints`
- `linked_artifacts`
- `priority_hint`

### Output contract
- `ticket_id`
- `root_ticket_id`
- `status`
- `created_event_id`

## 3. Trigger -> Orchestrator Run

### Trigger types
- `new_request`
- `worker_completed`
- `worker_failed`
- `blocker_detected`
- `scheduled_reconciliation`
- `summary_refresh_needed`

### Input contract
- `trigger_type`
- `trigger_ref_id`
- `root_ticket_id`
- `target_ticket_id`
- `correlation_id`

### Output contract
- `run_id`
- `next_actions[]`
- `decision_summary`
- `emitted_event_ids[]`

## 4. Orchestrator -> Context Builder

### Intent
Build a bounded context package.

### Input contract
- `run_id`
- `target_ticket_id`
- `context_type` — `orchestrator`, `worker`
- `worker_type` if applicable
- `retrieval_policy_id`
- `token_budget`

### Output contract
- `context_package_id`
- `included_ticket_ids[]`
- `included_summary_ids[]`
- `included_artifact_ids[]`
- `included_event_window`
- `freshness_warnings[]`

## 5. Orchestrator -> Planner / Decomposer

### Intent
Decide how to execute or decompose a ticket.

### Input contract
- `ticket`
- `context_package_id`
- `policies[]`

### Output contract
One of:
- `complete_directly`
- `create_child_tickets`
- `request_clarification`
- `mark_blocked`
- `defer`

### For `create_child_tickets`
Return:
- `child_tickets[]`
- `dependency_plan[]`
- `dispatch_hints[]`

## 6. Orchestrator -> Dispatcher

### Intent
Assign a child ticket to a worker class.

### Input contract
- `ticket_id`
- `task_type`
- `constraints`
- `required_tools`
- `latency_target`
- `cost_limit`

### Output contract
- `worker_type`
- `dispatch_reason`
- `execution_policy_id`

## 7. Orchestrator -> Worker Invocation

### Intent
Start bounded task execution.

### Input contract
- `ticket_id`
- `worker_type`
- `context_package_id`
- `objective`
- `acceptance_criteria[]`
- `allowed_tools[]`
- `forbidden_actions[]`
- `output_schema`
- `timeout_seconds`
- `retry_policy`

### Output contract
- `run_id`
- `started_at`
- `status` — `started`

## 8. Worker -> Ticket/Event System

### Intent
Publish worker progress or final outcome.

### Input contract
- `run_id`
- `ticket_id`
- `status` — `checkpoint`, `completed`, `failed`, `blocked`
- `result_summary`
- `structured_findings`
- `artifact_refs[]`
- `recommended_next_actions[]`
- `confidence`
- `error_info`

### Output contract
- `event_ids[]`
- `ticket_status`
- `review_required`

## 9. Worker -> Artifact Store

### Intent
Publish generated artifacts.

### Input contract
- `ticket_id`
- `artifact_type`
- `title`
- `content_ref`
- `format`
- `metadata`

### Output contract
- `artifact_id`
- `uri`
- `version`

## 10. Summary Refresh -> Summary Store

### Intent
Create or replace a summary.

### Input contract
- `summary_type`
- `scope_id`
- `source_event_range`
- `source_artifact_ids[]`
- `body`
- `open_questions[]`
- `open_risks[]`

### Output contract
- `summary_id`
- `quality_status`
- `supersedes_summary_id`

## 11. Orchestrator -> User Response Layer

### Intent
Respond back to UI after a control decision.

### Input contract
- `root_ticket_id`
- `decision_summary`
- `user_visible_status`
- `clarification_questions[]`
- `produced_artifact_links[]`

### Output contract
- UI message payload

## Standard status enums

## Ticket status
- `new`
- `triaged`
- `planned`
- `assigned`
- `in_progress`
- `blocked`
- `review`
- `done`
- `cancelled`

## Worker result status
- `checkpoint`
- `completed`
- `failed`
- `blocked`

## Summary quality status
- `draft`
- `trusted`
- `stale`
- `invalid`

## Error categories
- `missing_input`
- `invalid_request`
- `policy_violation`
- `tool_failure`
- `timeout`
- `dependency_unavailable`
- `low_confidence`
- `unknown`

## Idempotency expectations

The following operations should be idempotent where practical:
- request intake by correlation ID
- worker completion publish by run ID + terminal state
- summary refresh by scope + source range
- ticket assignment when repeated with same assignee and version

## Required provenance fields everywhere

Every major contract should carry:
- `correlation_id`
- `ticket_id` or `root_ticket_id`
- `actor_id`
- `timestamp`
- `schema_version`

## Implementation note

These contracts can be implemented over HTTP, queue messages, workflow engines, or internal service calls. The architecture does not depend on transport choice.
