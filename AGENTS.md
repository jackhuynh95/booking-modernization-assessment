# AGENTS.md

## Project Intent

This repository is for `booking-modernization-assessment`, a PhoenixDX `.NET Backend Engineer` PDF-only technical design submission.

The work product is a concise design document, not a runnable application. It should explain how to extract a booking microservice from a legacy system using .NET-oriented backend design, API contracts, events, idempotency, eventual consistency, observability, edge cases, and clear assessment assumptions.

## Current Phase

Foundation and roadmap setup.

Do not generate backend/frontend implementation code, Dockerfiles, CI files, or runtime harnesses unless the user explicitly starts an implementation step. Current deliverable is documentation suitable for final PDF assembly.

## Source of Truth

Use these documents before making design decisions:

- `docs/assessment-brief.md`
- `docs/roadmap.md`
- `docs/architecture.md`
- `docs/api-contract.md`
- `docs/events.md`
- `docs/idempotency.md`
- `docs/observability.md`
- `docs/ai-usage-declaration.md`
- `docs/final-submission-outline.md`

## Repository Layout

- `docs/`: evergreen assessment docs and final PDF source material.
- `README.md`: repo identity and doc index.
- No `src/`, `apps/`, Docker Compose, package manager setup, or CI should exist during the foundation phase.

## Assessment Guardrails

- Keep work docs-first and PDF-ready.
- Keep scope centered on booking modernization, not full implementation.
- Favor .NET backend terminology and pragmatic distributed-systems tradeoffs.
- Capture assumptions separately from confirmed constraints.
- Keep API, event, idempotency, consistency, and observability decisions synchronized across docs.
- Record AI assistance transparently in `docs/ai-usage-declaration.md`.
- Final submission target is a 6-page PDF plus submission email, not source code.

## Shell Rule

Follow the global shell rule from `/Users/jackhuynh/.codex/RTK.md`: run shell commands through `rtk`.
