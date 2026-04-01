# Data Model

## Purpose

This document defines the minimum durable data model required to implement a reliable orchestrator-agent system.

The model is intentionally split into:
- canonical event data
- current operational views
- summary and artifact references
- policy and execution metadata

## Design rules

1. Tickets are the canonical unit of work.
2. Events are the canonical history of change.
3. Materialized views are optimized for reads, not truth ownership.
4. Summaries are derived objects and must reference sources.
5. Every artifact must be linkable to a ticket.

## Core entities

## 1. Request

Represents an inbound UI/API request before or during ticket creation.

### Fields
- `request_id`
- `correlation_id`
- `actor_id`
- `source_channel`
- `raw_input`
- `normalized_intent`
- `scope`
- `priority_hint`
- `constraints`
- `created_at`

## 2. Ticket

Represents durable work state.

### Fields
- `ticket_id`
- `parent_ticket_id`
- `root_ticket_id`
- `request_id`
- `title`
- `objective`
- `description`
- `status`
- `priority`
- `owner_type` — `orchestrator`, `worker`, `human`, `system`
- `owner_id`
- `worker_type`
- `dependencies`
- `acceptance_criteria`
- `input_artifact_ids`
- `output_artifact_ids`
- `summary_id`
- `blocked_reason`
- `risk_level`
- `retry_count`
- `due_at`
- `created_at`
- `updated_at`
- `closed_at`

### Notes
- `root_ticket_id` supports graph-level reconciliation.
- `worker_type` is nullable unless assigned to a worker.
- `summary_id` points to the latest trusted ticket summary.

## 3. Ticket Relation

Optional explicit relationship table if dependencies become richer than simple IDs.

### Fields
- `relation_id`
- `from_ticket_id`
- `to_ticket_id`
- `relation_type` — `depends_on`, `blocks`, `duplicates`, `follows`, `derived_from`
- `created_at`

## 4. Event

Immutable record of something that happened.

### Fields
- `event_id`
- `correlation_id`
- `ticket_id`
- `root_ticket_id`
- `event_type`
- `actor_type`
- `actor_id`
- `payload`
- `source_summary_ids`
- `source_artifact_ids`
- `created_at`
- `schema_version`

### Recommended event types
- `request_received`
- `intent_normalized`
- `ticket_created`
- `ticket_updated`
- `ticket_assigned`
- `ticket_reassigned`
- `dependency_added`
- `worker_started`
- `worker_checkpointed`
- `worker_completed`
- `worker_failed`
- `blocker_detected`
- `blocker_cleared`
- `artifact_published`
- `summary_refreshed`
- `ticket_reviewed`
- `ticket_closed`
- `ticket_cancelled`

## 5. Artifact

Represents any persisted output.

### Fields
- `artifact_id`
- `ticket_id`
- `artifact_type` — `doc`, `plan`, `draft`, `dataset`, `report`, `decision_record`, `log_bundle`
- `title`
- `uri`
- `content_hash`
- `format`
- `version`
- `produced_by_actor_type`
- `produced_by_actor_id`
- `created_at`
- `metadata`

## 6. Summary

Compressed, versioned representation of work history or domain state.

### Fields
- `summary_id`
- `summary_type` — `ticket`, `root_ticket`, `project`, `domain`, `artifact_set`
- `scope_id`
- `title`
- `body`
- `source_event_start`
- `source_event_end`
- `source_artifact_ids`
- `open_questions`
- `open_risks`
- `generated_by`
- `quality_status` — `draft`, `trusted`, `stale`, `invalid`
- `created_at`
- `supersedes_summary_id`

## 7. Agent Run

Represents a single orchestrator or worker execution.

### Fields
- `run_id`
- `ticket_id`
- `agent_role` — `orchestrator`, `worker`
- `worker_type`
- `trigger_type`
- `context_package_id`
- `status` — `started`, `completed`, `failed`, `cancelled`
- `started_at`
- `ended_at`
- `latency_ms`
- `token_usage`
- `tool_usage`
- `error_code`
- `error_message`

## 8. Context Package

Represents the exact bounded context assembled for a run.

### Fields
- `context_package_id`
- `ticket_id`
- `run_id`
- `context_type` — `orchestrator`, `worker`
- `included_ticket_ids`
- `included_event_range`
- `included_summary_ids`
- `included_artifact_ids`
- `included_policy_ids`
- `budget_tokens_estimate`
- `created_at`

## 9. Policy

Represents execution rules and constraints.

### Fields
- `policy_id`
- `name`
- `category` — `safety`, `routing`, `retry`, `approval`, `cost`, `latency`
- `rule_body`
- `version`
- `active`
- `created_at`
- `updated_at`

## 10. Blocker

Optional explicit blocker record to simplify operational handling.

### Fields
- `blocker_id`
- `ticket_id`
- `category`
- `description`
- `detected_by`
- `severity`
- `status` — `open`, `acknowledged`, `resolved`
- `opened_at`
- `resolved_at`

## Materialized views

## 1. Ticket Current View

Operational read model for current ticket state.

### Includes
- core ticket fields
- latest owner
- latest status
- blocker flag
- child ticket counts
- dependency counts
- latest summary pointer
- last event timestamp

## 2. Root Ticket Dashboard View

Supports parent-level orchestration.

### Includes
- root ticket ID
- progress counts by status
- blocked subtree count
- oldest open blocker
- latest worker outcome
- next recommended action

## 3. Summary Freshness View

Tracks stale summaries.

### Includes
- scope ID
- last summary timestamp
- latest source event timestamp
- freshness gap
- stale flag

## State machine guidance

### Ticket states
- `new`
- `triaged`
- `planned`
- `assigned`
- `in_progress`
- `blocked`
- `review`
- `done`
- `cancelled`

### Agent run states
- `started`
- `completed`
- `failed`
- `cancelled`

## Indexing guidance

Prioritize indexes on:
- `ticket_id`
- `root_ticket_id`
- `parent_ticket_id`
- `status`
- `owner_id`
- `event_type`
- `created_at`
- `correlation_id`
- `summary_type + scope_id`

## Minimal implementation recommendation

A first implementation only needs:
- Ticket table
- Event table
- Artifact table
- Summary table
- Ticket Current View

Everything else can be layered in later without breaking the architecture.
