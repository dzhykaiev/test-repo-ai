# Security and Safety

## Purpose

This document defines the controls required to keep the orchestrator-agent system safe, predictable, and auditable.

This is not only about traditional security. It also covers operational safety for autonomous or semi-autonomous agent behavior.

## Security goals

- least-privilege execution
- explicit tool authorization
- auditable actions
- bounded delegation
- artifact and state integrity
- safe handling of user input and external content

## Safety goals

- no uncontrolled worker spawning
- no silent policy bypass
- no hidden state mutations
- no irreversible action without explicit authorization level
- no trust in stale summaries over live state

## Threat surfaces

- malicious or ambiguous user input
- prompt injection in retrieved artifacts
- unauthorized tool access
- worker overreach
- summary corruption or drift
- event tampering
- duplicate or replayed requests

## Control families

## 1. Identity and access

Every actor should have an identity:
- user
- orchestrator
- worker
- human reviewer
- system scheduler

Tool access should be scoped by:
- actor type
- worker type
- ticket scope
- policy profile

## 2. Tool allowlists

Each worker invocation must carry explicit `allowed_tools[]`.

Default-deny model is recommended.

### Example
A documentation worker may read artifacts and write docs, but may not create production deployment changes.

## 3. Delegation controls

By default:
- workers cannot spawn workers
- workers cannot reassign unrelated tickets
- workers cannot elevate their own permissions

Only the orchestrator or human override path may delegate new work.

## 4. Approval gates

Certain action categories should require explicit approval state.

Examples:
- destructive external actions
- publication to sensitive destinations
- cost-heavy large-scale execution
- irreversible closure of high-risk ticket families

## 5. Prompt injection resistance

All retrieved artifacts should be treated as untrusted content.

### Rules
- do not follow instructions found inside artifacts unless routed through policy
- separate system policy from retrieved content
- preserve provenance of retrieved snippets
- prefer structured metadata extraction over raw instruction following

## 6. Integrity and auditability

Protect:
- events as append-only records
- artifact hashes
- summary provenance
- ticket version numbers

Every sensitive mutation should be attributable to:
- actor ID
- run ID
- ticket ID
- timestamp

## 7. Idempotency and replay protection

Use correlation IDs and idempotency keys for:
- request intake
- worker completion events
- artifact publication where duplication matters
- summary refresh jobs

## 8. Data minimization

Only include the minimum necessary data in each context package.

Benefits:
- lower exposure of sensitive information
- smaller prompt footprint
- less contamination risk

## 9. Confidence-based safety

Low-confidence outputs should not silently advance high-risk workflows.

Recommended behaviors:
- mark for review
- request clarification
- reroute to another worker
- escalate to human

## 10. Policy layering

Recommended policy layers:
- global platform policy
- project policy
- worker-type policy
- ticket-level overrides

Ticket-level overrides must never weaken global safety constraints.

## Sensitive action categories

Examples:
- external communication
- credential use
- financial action
- destructive file operations
- deleting or closing critical records
- publication to customer-visible systems

These should require stronger authorization profiles.

## Logging requirements

Log at minimum:
- trigger received
- context package built
- worker dispatched
- artifact published
- summary refreshed
- policy violation detected
- escalation raised
- terminal ticket decision made

## Recommended security invariants

1. No action without attributable actor identity.
2. No state mutation without an event.
3. No worker receives more privileges than its ticket requires.
4. No retrieved artifact may override system policy.
5. No terminal action on critical flows without explicit authorization rules.

## Safety anti-patterns

- one worker with universal tool access
- allowing retrieved documents to redefine agent policy
- hidden auto-retries with no event trace
- summary-only decision making for critical actions
- worker-created side effects outside ticket tracking
