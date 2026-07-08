# AGENTS.md

## Project Intent

This repository is for the `robot-fleet-dashboard` full-stack technical assignment. Treat it as a real-time robot fleet management dashboard with WebSocket telemetry ingestion, MongoDB persistence, live health monitoring, alerts, historical charts, clustering, and Docker Compose runtime.

## Current Phase

The project is in the specification and harness phase, with boilerplate present under `apps/`.

Do not generate backend/frontend implementation code, Dockerfiles, or CI files until the user explicitly asks for an implementation step.

## Source of Truth

Use these documents before implementation decisions:

- `docs/specs/assignment-brief.md`
- `docs/specs/functional-spec.md`
- `docs/specs/telemetry-contract.md`
- `docs/specs/alert-rules.md`
- `docs/architecture/architecture-notes.md`
- `docs/database/mongodb-design.md`
- `docs/api/api-and-websocket.md`
- `docs/guardrails/development-guardrails.md`
- `docs/development/project-structure.md`
- `docs/operations/docker-compose.md`
- `docs/roadmap/IMPLEMENTATION_ROADMAP.md`
- `docs/roadmap/EPIC.md`

## Repository Layout

- `apps/backend`: Node.js/uWebSockets.js telemetry backend.
- `apps/frontend`: React/Next.js dashboard and robot detail UI.
- `apps/backend/simulator`: sample simulator command.
- `docs`: specs, architecture, API, database, operations, roadmap, evidence.
- `docker-compose.yml`: local development runtime for frontend, backend, MongoDB, and simulator.

## Required Stack From Assignment

- Node.js.
- React/Next.js.
- MongoDB.
- uWebSockets.js.
- Docker Compose.

## Implementation Guardrails

- Before React/Next.js UI work, invoke and mention `react-best-practices` if available.
- If `react-best-practices` is unavailable, state that clearly, then continue with local frontend guardrails.
- Preserve the sample-code intent where possible; update it rather than replacing everything without reason.
- Validate all incoming robot telemetry before storing or broadcasting.
- Store raw/normalized telemetry in MongoDB with indexes for robot ID and time-window queries.
- Keep alert rules deterministic and testable.
- Keep real-time broadcast and persistence failure modes explicit.
- Document any shortcuts or production follow-ups in `docs/decisions/`.
- Follow the roadmap phases in `docs/roadmap/IMPLEMENTATION_ROADMAP.md`.

## Continuous Documentation Rule

Every implementation goal must update docs in the same pass when behavior changes. Keep these files synchronized with code:

- README setup and verification notes;
- `docs/specs/*` for functional and telemetry behavior;
- `docs/api/api-and-websocket.md` for REST/WebSocket contracts;
- `docs/database/mongodb-design.md` for collections, indexes, and retention;
- `docs/architecture/architecture-notes.md` for runtime flow and scaling;
- `docs/roadmap/*` for phase status and next work;
- `docs/evidence/manual-test-evidence.md` when manual screenshots or video proof are captured.

## Shell Rule

Follow the global shell rule from `/Users/jackhuynh/.codex/RTK.md`: run shell commands through `rtk`.
