# Roadmap

## Overview

This roadmap runs from assessment understanding through final PDF export and submission email. It keeps the repository docs-first and avoids source-code implementation unless a later goal explicitly changes scope.

## Phase Plan

| Phase | Status | Deliverable | Exit Criteria |
| --- | --- | --- | --- |
| 1. Understand assessment and assumptions | In progress | `docs/assessment-brief.md` | Constraints, assumptions, non-goals, and unknowns captured. |
| 2. Booking microservice structure | Not started | `docs/architecture.md` | Service boundary, responsibilities, data ownership, dependencies, and migration approach proposed. |
| 3. API contract | Not started | `docs/api-contract.md` | Core REST endpoints, request/response shapes, status codes, validation, and error model drafted. |
| 4. Event definitions | Not started | `docs/events.md` | Booking event list, schemas, producers/consumers, ordering, and versioning strategy drafted. |
| 5. Idempotency and eventual consistency | Not started | `docs/idempotency.md` | Idempotency keys, deduplication, retries, outbox/inbox, reconciliation, and consistency tradeoffs defined. |
| 6. Edge cases | Not started | Cross-doc sections | Cancellation, duplicate submission, payment timeout, inventory conflict, race conditions, partial failure, and replay behavior covered. |
| 7. Observability plan | Not started | `docs/observability.md` | Logs, metrics, traces, alerts, dashboards, SLOs, and failure investigation paths defined. |
| 8. AI usage declaration | Not started | `docs/ai-usage-declaration.md` | AI-assisted work disclosed with scope, human review, and responsibility statement. |
| 9. Final 6-page PDF draft | Not started | `docs/final-submission-outline.md` | Page-by-page content plan fits six pages and references source docs. |
| 10. Review, polish, export, submission email | Not started | Final PDF and email text | Design reviewed for coherence, constraints, omissions, PDF readability, and email readiness. |

## Immediate Next Goal

Start Phase 2 by proposing the booking microservice structure:

- bounded context and service responsibilities;
- owned data model at conceptual level;
- legacy integration points;
- dependency flow;
- migration approach;
- risks and tradeoffs.

## Definition Of Done For Foundation

- Roadmap exists and covers all required deliverables.
- Assessment constraints are captured.
- Repo identity is booking/PhoenixDX/PDF-only.
- Old unrelated runtime config is removed.
- No source-code implementation is started.
- Next goal can start directly on the microservice structure.

## References

- `docs/assessment-brief.md`
- `docs/final-submission-outline.md`
