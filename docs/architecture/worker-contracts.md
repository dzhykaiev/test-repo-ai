# Worker Contracts

## Purpose

This document defines what a worker agent is allowed to receive, do, and return.

The goal is to make workers replaceable, observable, and safe.

## Core principle

A worker is a short-lived execution unit for one bounded task.

A worker is not allowed to become a hidden sub-orchestrator unless that behavior is explicitly designed and policy-approved.

## Worker types

Examples:
- research worker
- architecture writer
- documentation worker
- classification worker
- validation worker
- summarization worker
- issue-preparation worker

## Required worker input contract

Every worker invocation must include:
- `ticket_id`
- `root_ticket_id`
- `worker_type`
- `objective`
- `task_description`
- `acceptance_criteria[]`
- `context_package_id`
- `allowed_tools[]`
- `forbidden_actions[]`
- `output_schema`
- `timeout_seconds`
- `retry_policy`
- `correlation_id`

## Worker permissions model

### Allowed by default
- read assigned ticket
- read bounded context package
- use explicitly allowed tools
- publish artifacts to allowed destinations
- emit progress and terminal events

### Not allowed by default
- mutate unrelated tickets
- spawn additional workers
- close parent tickets
- override policy state
- silently discard blockers
- widen its own context scope autonomously

## Worker output contract

Every worker run must return:
- `run_id`
- `ticket_id`
- `status`
- `result_summary`
- `structured_findings`
- `artifact_refs[]`
- `recommended_next_actions[]`
- `confidence`
- `error_category` when relevant
- `error_details` when relevant
- `completed_at`

## Output status values
- `completed`
- `blocked`
- `failed`
- `checkpoint`

## Acceptance criteria rules

Each worker ticket must have acceptance criteria that are:
- observable
- outcome-focused
- small enough to verify
- not dependent on hidden worker interpretation alone

### Good example
- produce a decision document comparing three routing strategies
- include explicit trade-offs
- identify one recommended option
- link all cited source artifacts

### Bad example
- think deeply and solve it well

## Worker result normalization

Worker output should be normalized before parent reconciliation.

Normalize:
- status
- confidence
- artifacts
- open questions
- next-action suggestions
- blocker reason if present

## Progress events

Longer workers may emit checkpoints.

Checkpoint payload should include:
- `run_id`
- `ticket_id`
- `progress_message`
- `estimated_remaining_stage`
- `partial_artifact_refs[]`
- `detected_risks[]`

## Failure handling

### Worker-level failure categories
- `tool_failure`
- `timeout`
- `missing_input`
- `policy_violation`
- `low_confidence`
- `unexpected_exception`

### Rule
A worker must fail loudly and structurally, never ambiguously.

## Blocker handling

A blocked worker must return:
- blocker category
- blocker description
- evidence of why progress cannot continue safely
- suggested unblock actions

## Quality expectations by worker type

### Research worker
- source-backed findings
- source quality awareness
- unresolved uncertainties listed explicitly

### Documentation worker
- clear structure
- versionable output
- links to source tickets and decisions

### Validation worker
- explicit pass/fail criteria
- defect list when failing
- no silent assumptions

### Summarization worker
- source range provenance
- timestamps
- open questions retained
- contradictions surfaced, not flattened

## Worker isolation requirements

Workers should be isolated across:
- context
- retries
- credentials where possible
- artifact namespaces where practical

## Escalation triggers

A worker should recommend escalation when:
- acceptance criteria conflict with available inputs
- policy prevents progress
- confidence is below threshold
- repeated tool failure blocks completion
- task appears wrongly routed

## Versioning

Worker contracts should be versioned.

At minimum version:
- input schema
- output schema
- allowed tool set
- policy profile

## Key invariant

A worker is only successful when its output can be evaluated independently of the worker's hidden internal reasoning.
