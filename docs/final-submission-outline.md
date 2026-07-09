# Final Submission Outline

## Overview

This document will guide the final six-page PDF and submission email for the PhoenixDX `.NET Backend Engineer` booking modernization assessment.

Phase 9 and Phase 10 status: final draft, PDF export, and submission email packaging complete.

Final draft source: `docs/final-submission.md`  
Submission email draft: `docs/submission-email.md`  
Final PDF target: `output/pdf/Jack_Huynh_Booking_Modernization_Assessment.pdf`

## Six-Page PDF Draft Plan

| Page | Content |
| --- | --- |
| 1 | Executive summary, assumptions, Expedia-like domain framing, outcomes. |
| 2 | Proposed booking microservice boundary, structure, legacy integration, migration approach. |
| 3 | API contract summary with endpoints, line model, validation, errors, idempotency usage. |
| 4 | Event definitions, envelope, eventual consistency flow, outbox/inbox, reconciliation. |
| 5 | Idempotency strategy, edge cases, reliability controls, failure-mode behavior. |
| 6 | Observability plan, SLO targets, AI usage declaration, closing rationale. |

## Production Readiness Space Allocation

The final PDF must reserve visible space for Phase 7 and Phase 8 so they are not reduced to footnotes.

| Topic | PDF treatment |
| --- | --- |
| Observability | Compact table covering logs, metrics, traces, alerts, dashboards, SLO targets, and incident flow. |
| AI declaration | Short, direct disclosure of tools used, AI-assisted sections, manual validation, and team safeguards against unreviewed AI output. |
| PhoenixDX assessment angle | Show AI-first mindset as transparent, reviewed, source-backed, and production-minded rather than defensive or generic. |

## Submission Email Draft Inputs

- Candidate name and role target.
- Short note that attached PDF is the technical design submission.
- One-sentence summary of design focus.
- AI usage disclosed in PDF.
- Contact details.

## Final Review Checklist

- Fits six pages.
- No unsupported implementation claim.
- Diagrams/tables readable in PDF.
- Booking/PhoenixDX/.NET identity consistent.
- AI usage declaration included.
- Edge cases and observability not treated as afterthoughts.
- Submission email concise and professional.

## References

- `docs/roadmap.md`
- `docs/assessment-brief.md`
