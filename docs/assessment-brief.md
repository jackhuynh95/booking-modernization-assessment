# Assessment Brief

## Overview

Repository: `booking-modernization-assessment`

Assessment: PhoenixDX `.NET Backend Engineer` technical design submission.

Deliverable: PDF-only technical design for modernizing a booking capability by proposing a booking microservice boundary, API contract, event model, idempotency strategy, eventual consistency approach, observability plan, edge-case coverage, AI usage declaration, and final submission outline.

No runnable source-code implementation is part of this foundation phase.

## Source Material Read

- User-provided goal: "Assessment Foundation & Roadmap Setup".
- Existing `README.md` summary: ".NET 8 booking microservice extraction design for legacy modernization, event-driven architecture, idempotency, observability, and AI-assisted engineering declaration."
- Assessment domain framing: Product A is described as an Expedia-like travel management platform.
- External product reference checked for domain orientation: Expedia Australia shows travel capabilities such as stays, flights, cars, packages, activities, dates, travellers, room selection, trips, support, and communications.

No separate assessment PDF or external brief file exists in this repository at foundation time. This document captures all known constraints from available repo and user-provided material.

## Confirmed Constraints

| Constraint | Captured Approach |
| --- | --- |
| Repository identity | Use `booking-modernization-assessment` consistently. |
| Role/company framing | PhoenixDX `.NET Backend Engineer`. |
| Output format | Final PDF-only technical design submission. |
| Implementation status | No source-code implementation started. |
| Architecture topic | Booking microservice extraction from legacy system. |
| Domain framing | Booking belongs to an Expedia-like travel management platform, not a generic CRUD reservation table. |
| Expected design topics | Microservice structure, API contract, events, idempotency, eventual consistency, edge cases, observability, AI usage declaration. |
| Final package | 6-page PDF draft, review/polish/export, submission email. |

## Assumptions To Carry Forward

- Target backend platform is .NET 8 unless later assessment material says otherwise.
- Legacy system currently owns booking data and workflows.
- Booking is a travel lifecycle capability that may coordinate stays, flights, cars, packages, activities, travellers, dates, payments, communications, and support workflows.
- The booking service should own booking lifecycle state and orchestration, but should not own unrelated domains such as payment settlement, inventory source of truth, traveller profile, communications, or supplier catalogs.
- New booking service should avoid distributed transactions and use idempotent commands plus events/outbox where appropriate.
- API consumers may include a web/mobile client, internal services, and back-office operations.
- Final PDF should be concise enough for assessor review, with diagrams/tables preferred over long prose.

## Non-Goals

- No API server implementation.
- No database migrations.
- No Docker Compose or CI setup.
- No frontend work.
- No production deployment scripts.

## Open Questions

- Exact original assessment wording is not present in repo.
- Required page limit is known as 6 pages from user scope, but exact formatting rules are not present.
- Any PhoenixDX-specific evaluation rubric is not present.

## References

- Expedia Australia: https://www.expedia.com.au/
- `README.md`
- `docs/roadmap.md`
