# test-repo-ai

Reference architecture repository for an agent orchestration system where a primary orchestrator agent continuously accepts work from a UI, decomposes work into tickets, spawns specialized agents, and keeps global state fresh without carrying an ever-growing prompt context.

## Goal

Design an orchestration architecture where:

- the main agent is always operational
- the main agent stays context-clean
- the system knows the latest state of work without depending on a long chat history
- specialized agents can be launched for bounded tasks
- every task, decision, artifact, and status change is traceable

## Core idea

The primary agent must **not** be a long-lived conversational brain with an ever-expanding prompt. It should be a **stateless orchestrator per run**.

Each run of the orchestrator reconstructs the minimum viable context from durable system state:

- active tickets
- recent events
- summarized project memory
- unresolved blockers
- relevant artifacts
- policies and operating constraints

This keeps the orchestrator fresh while preserving full system awareness through structured state.

## Repository structure

- `docs/architecture/overview.md` — system goals, principles, and top-level design
- `docs/architecture/components.md` — logical components and responsibilities
- `docs/architecture/flows.md` — end-to-end execution flows
- `docs/architecture/context-management.md` — how fresh context is built safely
- `docs/runbooks/operations.md` — operational rules for stable execution
- `docs/adr/0001-stateless-orchestrator.md` — ADR for stateless orchestrator design
- `docs/adr/0002-event-sourced-task-state.md` — ADR for durable ticket/event history
- `docs/adr/0003-specialized-ephemeral-workers.md` — ADR for worker model

## System summary

The architecture is based on six ideas:

1. **UI is only an intent entrypoint.**
2. **Ticket is the unit of work.**
3. **Orchestrator is responsible for coordination, not execution.**
4. **Specialized workers are short-lived and task-bounded.**
5. **Persistent state is the source of truth.**
6. **Context is reconstructed on demand, never accumulated blindly.**

## High-level lifecycle

1. A user asks for something in the UI.
2. The request is normalized into a parent ticket.
3. The orchestrator evaluates the request against current state.
4. The orchestrator either:
   - resolves it directly as a control decision, or
   - decomposes it into child tickets.
5. Child tickets are assigned to specialized agents.
6. Workers emit events, artifacts, and status updates.
7. A summarization layer compresses long histories into trusted summaries.
8. The next orchestrator run starts from fresh, structured context.

## Non-goals

This repository does not define implementation code, frameworks, or model vendors. It defines the architecture, operating model, and invariants required to implement a reliable multi-agent orchestrator.

## Primary design principle

**Knowing everything does not mean loading everything into the prompt.**

The orchestrator should know the current truth through state reconstruction, not through raw history replay.
