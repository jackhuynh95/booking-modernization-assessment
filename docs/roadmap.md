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
| 5. Idempotency and eventual consistency | Complete | `docs/idempotency.md` | Idempotency keys, deduplication, retries, outbox/inbox, reconciliation, and consistency tradeoffs defined. |
| 6. Edge cases | Complete | `docs/idempotency.md` | Cancellation, duplicate submission, payment timeout, inventory conflict, race conditions, partial failure, and replay behavior covered. |
| 7. Observability plan | Complete | `docs/observability.md` | Logs, metrics, traces, alerts, dashboards, SLOs, and failure investigation paths defined. |
| 8. AI usage declaration | Complete | `docs/ai-usage-declaration.md` | AI-assisted work disclosed with scope, human review, backend team safeguards, and responsibility statement. |
| 9. Final 6-page PDF draft | Complete | `docs/final-submission.md` | Page-by-page content fits six pages and references source docs. |
| 10. Review, polish, export, submission email | Complete | Final PDF and email text | Design reviewed for coherence, constraints, omissions, PDF readability, and email readiness. |

## Immediate Next Goal

Group 3: Final Submission Packaging complete:

- drafted the six-page PDF content from source docs;
- compressed architecture, API, events, reliability, observability, and AI declaration into reviewer-readable tables/diagrams;
- ensured assumptions and non-goals are explicit;
- prepared submission email wording.

## Definition Of Done For Foundation

- Roadmap exists and covers all required deliverables.
- Assessment constraints are captured.
- Repo identity is booking/PhoenixDX/PDF-only.
- Old unrelated runtime config is removed.
- No source-code implementation is started.
- Next goal can start directly on final PDF draft and submission packaging.

## References

- `docs/assessment-brief.md`
- `docs/final-submission-outline.md`
- `docs/final-submission.md`
- `docs/submission-email.md`
- `docs/final-review-checklist.md`
