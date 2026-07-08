# Roadmap

## Overview

This roadmap runs from assessment understanding through final PDF export and submission email. It keeps the repository docs-first and avoids source-code implementation unless a later goal explicitly changes scope.

## Phase Plan

| Phase | Status | Deliverable | Exit Criteria |
| --- | --- | --- | --- |
| 1. Understand assessment and assumptions | Complete | `docs/assessment-brief.md` | Constraints, assumptions, non-goals, unknowns, and Expedia-like domain framing captured. |
| 2. Booking microservice structure | Complete | `docs/architecture.md` | Service boundary, responsibilities, data ownership, dependencies, Expedia-like travel orchestration boundary, and migration approach proposed. |
| 3. API contract | Complete | `docs/api-contract.md` | Core REST endpoints, request/response shapes, travel booking line model, status codes, validation, and error model drafted. |
| 4. Event definitions | Complete | `docs/events.md` | Booking event list, travel booking payload references, schemas, producers/consumers, ordering, and versioning strategy drafted. |
| 5. Idempotency and eventual consistency | Not started | `docs/idempotency.md` | Idempotency keys, deduplication, retries, outbox/inbox, reconciliation, and consistency tradeoffs defined. |
| 6. Edge cases | Not started | Cross-doc sections | Cancellation, duplicate submission, payment timeout, inventory conflict, race conditions, partial failure, and replay behavior covered. |
| 7. Observability plan | Not started | `docs/observability.md` | Logs, metrics, traces, alerts, dashboards, SLOs, and failure investigation paths defined. |
| 8. AI usage declaration | Not started | `docs/ai-usage-declaration.md` | AI-assisted work disclosed with scope, human review, and responsibility statement. |
| 9. Final 6-page PDF draft | Not started | `docs/final-submission-outline.md` | Page-by-page content plan fits six pages and references source docs. |
| 10. Review, polish, export, submission email | Not started | Final PDF and email text | Design reviewed for coherence, constraints, omissions, PDF readability, and email readiness. |

## Immediate Next Goal

Start Phase 5 by drafting idempotency and eventual consistency:

- idempotency key scope and storage;
- command replay behavior and payload mismatch handling;
- outbox publishing and inbox/event consumer deduplication;
- retry, timeout, and dead-letter rules;
- eventual consistency guarantees across booking, payment, inventory, supplier, legacy, and communications services;
- reconciliation and compensating action strategy;
- alignment with API command responses and lifecycle events.

## Definition Of Done For Foundation

- Roadmap exists and covers all required deliverables.
- Assessment constraints are captured.
- Repo identity is booking/PhoenixDX/PDF-only.
- Old unrelated runtime config is removed.
- No source-code implementation is started.
- Next goal can start directly on idempotency and eventual consistency.

## References

- `docs/assessment-brief.md`
- `docs/final-submission-outline.md`
