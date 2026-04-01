# Context Management

## Core problem

The primary orchestrator must stay globally aware without becoming context-saturated.

A naive design keeps the orchestrator in one long-running conversation. This fails because:

- token history grows continuously
- old assumptions stay sticky
- stale information survives too long
- irrelevant information contaminates current decisions
- recovery becomes difficult after interruptions

## Design principle

The orchestrator should be **fresh on every run** and **fully informed through retrieval**, not through persistent prompt bloat.

## Context layers

### Layer 1: Control context
Always included for the orchestrator:
- system role
- operating policies
- current run purpose
- current ticket or trigger
- current global health snapshot

### Layer 2: Task graph context
Included when relevant:
- parent ticket summary
- child ticket states
- dependency graph
- open blockers

### Layer 3: Event context
A bounded slice of recent or relevant events:
- latest ticket transitions
- latest worker outcomes
- latest escalations
- latest artifact publications

### Layer 4: Summary context
Versioned compressed memory:
- project summary
- ticket summary
- domain summary
- artifact summary

### Layer 5: Artifact context
Only linked materials needed for this decision:
- previous architecture docs
- requirements
- validation output
- user-provided references

## What should never be default context

- full chat history
- every ticket in the project
- every event ever emitted
- every generated artifact
- raw logs unless debugging

## Context assembly algorithm

### For orchestrator runs
1. Read trigger type.
2. Read target ticket.
3. Read current ticket state.
4. Read parent and direct child summaries.
5. Read unresolved blockers.
6. Read a bounded recent event slice.
7. Read latest relevant summaries.
8. Read only linked artifacts needed for the decision.
9. Build compact prompt package with provenance metadata.

### For worker runs
1. Read assigned ticket.
2. Read explicit acceptance criteria.
3. Read minimal parent objective.
4. Read only the artifacts required to execute.
5. Read worker-specific constraints.
6. Exclude unrelated sibling history.

## Freshness strategies

### Rebuild every run
No invocation should rely on hidden memory from previous invocations.

### Prefer state over summary when state is available
If a current ticket state exists, do not trust a stale summary over it.

### Attach timestamps and versions
Every summary and artifact reference should carry update time and version.

### Use recency windows
For events, prefer relevant recent windows rather than long historical dumps.

## Summary discipline

Summaries must include:
- scope
- time range
- source references
- open decisions
- unresolved risks
- last refresh timestamp

Summaries must not:
- invent facts
- hide blockers
- flatten contradictory evidence without noting it

## Recommended context budgets

### Orchestrator
Keep context small enough to:
- understand trigger
- understand current work graph around target ticket
- decide next action safely

### Worker
Keep context even smaller than orchestrator context when possible.
Workers should receive only execution-relevant material.

## Failure modes and mitigations

### Failure mode: stale summary chosen over live state
Mitigation: always check live ticket state first.

### Failure mode: irrelevant context pollutes routing decision
Mitigation: context builder uses ticket- and trigger-scoped retrieval.

### Failure mode: summary drift
Mitigation: periodic refresh with source-backed regeneration.

### Failure mode: prompt inflation
Mitigation: hard caps on included artifacts, events, and sibling tickets.

## Key invariant

**The orchestrator must remember through systems, not through conversational residue.**
