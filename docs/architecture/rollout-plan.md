# Rollout Plan

## Purpose

This document describes how to introduce the orchestrator-agent platform safely in stages.

The system should not jump from design directly to full autonomy. It should mature through controlled phases.

## Rollout principles

1. Start with narrow task types.
2. Keep human visibility high in early phases.
3. Expand autonomy only after recovery and observability are proven.
4. Prefer safe degradation over aggressive automation.

## Phase 0: Architecture and schema readiness

### Objective
Prepare the platform skeleton.

### Deliverables
- request intake contract
- ticket schema
- event schema
- artifact schema
- summary schema
- orchestrator run record
- worker contract v1

### Exit criteria
- schemas reviewed
- key invariants agreed
- recovery and policy approach documented

## Phase 1: Assisted orchestration

### Objective
Use the orchestrator mainly as a controlled planner with limited worker execution.

### Scope
- one UI entry channel
- root tickets
- child tickets
- one or two worker types
- strong operator visibility
- manual review for sensitive outcomes

### Recommended autonomy level
Low.
The system can route and draft, but humans stay close to decisions.

### Exit criteria
- stable ticket lifecycle
- no hidden state drift
- worker failures handled safely
- observability dashboards useful in practice

## Phase 2: Controlled execution

### Objective
Allow more worker automation for bounded tasks.

### Scope
- more worker types
- automatic retries for safe categories
- summary refresh jobs
- periodic reconciliation jobs
- stronger concurrency controls

### Recommended autonomy level
Medium.
The system may complete routine sub-work autonomously, but risky paths still require review.

### Exit criteria
- retry behavior proven safe
- recovery paths exercised successfully
- stale summary handling validated
- blocked ticket escalation working reliably

## Phase 3: Operational scale-up

### Objective
Scale across higher ticket volumes and more domains.

### Scope
- worker pools by specialization
- stronger policy layering
- more robust routing strategies
- broader dashboarding and alerts
- selective human override tooling

### Recommended autonomy level
Medium to high, but gated by risk class.

### Exit criteria
- acceptable latency and failure rates under load
- clear performance limits known
- policy enforcement remains reliable at scale

## Phase 4: Advanced autonomy

### Objective
Support broader autonomous execution in carefully approved domains.

### Scope
- deeper decomposition where justified
- richer memory retrieval
- more sophisticated policy engine
- risk-based approval automation

### Warning
Do not enter this phase if the platform still relies on implicit prompt memory or weak auditability.

## Safe rollout controls

## 1. Feature flags
Use flags for:
- new worker types
- autonomous retries
- auto-closure of tickets
- summary-driven retrieval behaviors
- sensitive tool access

## 2. Domain allowlists
Start only with low-risk domains and tasks.

## 3. Approval gates
Require review for:
- destructive actions
- external publication
- critical task closure
- policy exceptions

## 4. Shadow mode
Before full automation, consider shadow-running the orchestrator and comparing decisions with human operators.

## Rollback strategy

If instability appears:
- disable autonomous worker dispatch for affected task types
- revert to manual approval for terminal actions
- keep request intake and state persistence running
- preserve all events for postmortem analysis

## Readiness checklist for each phase

- ticket state consistency validated
- event audit trail intact
- recovery jobs operational
- alerts tested
- policy controls enforced
- worker contracts versioned
- summary freshness under control

## Key invariant

Autonomy should expand only when the platform proves it can remain fresh, observable, recoverable, and policy-bounded.
