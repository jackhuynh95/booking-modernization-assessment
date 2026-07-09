# AI Usage Declaration

## Overview

This document discloses AI assistance used while preparing the PhoenixDX `.NET Backend Engineer` booking modernization assessment.

AI was used as a drafting and review aid. The design remains human-owned: assumptions, tradeoffs, technical accuracy, PDF suitability, and final submission responsibility require human validation before delivery.

PhoenixDX can assess AI-first mindset responsibly by looking for transparent disclosure, source-backed assumptions, coherent engineering judgment, and evidence that generated material was reviewed rather than pasted uncritically.

## AI Tools Used

| Tool | Use |
| --- | --- |
| OpenAI Codex / ChatGPT-style assistant | Drafted and refined documentation structure, wording, checklists, and consistency across architecture, API, events, reliability, observability, and AI disclosure. |
| Web/reference lookup | Used only for lightweight domain framing where noted in `docs/assessment-brief.md`, such as validating Expedia-like travel capability orientation. |

No AI tool was used to generate runnable backend implementation code for this foundation phase.

## Sections AI-Assisted

| Section | AI-assisted contribution |
| --- | --- |
| Roadmap | Organized phases, deliverables, exit criteria, and final packaging path. |
| Architecture | Drafted service boundary, .NET 8 layering, legacy DB adapter, outbox processor, background workers, migration stages, and tradeoffs. |
| API contract | Drafted ASP.NET Core-style REST endpoints, idempotency/correlation headers, travel booking line model, response examples, validation posture, and error behavior. |
| Events | Drafted lifecycle events, event envelope, event IDs, schema versioning, correlation/causation IDs, consumer expectations, ordering rules, retry/dead-letter behavior, and Azure Service Bus compatibility assumptions. |
| Reliability design | Drafted idempotency records, duplicate request behavior, outbox/inbox strategy, no-distributed-transaction rationale, reconciliation, and edge-case handling. |
| Observability wording | Drafted structured logging fields, metrics, traces, alerts, dashboards, SLO targets, and incident investigation flow. |

## Manually Validated

The following items require and received manual review against repository context and assessment intent:

- assessment constraints: PhoenixDX `.NET Backend Engineer`, PDF-only design submission, no runnable implementation in this phase;
- Expedia-like domain framing: booking as travel lifecycle orchestration across stays, flights, cars, packages, activities, travellers, payment, inventory, and communications;
- .NET 8 service boundary: API/application/domain/infrastructure separation and legacy adapter boundaries;
- event-driven consistency: booking state as canonical for migrated flows with asynchronous downstream convergence;
- idempotency and outbox strategy: retry-safe commands, response snapshots, transactional outbox, consumer inbox/dedupe, and reconciliation;
- PDF page-limit suitability: source docs are detailed, but final PDF must compress content into reviewer-readable diagrams, tables, and concise bullets.

## Preventing Blind AI Usage In A Backend Team

Responsible AI-assisted backend delivery should use AI as an accelerator, not an authority. Guardrails:

- human design review before decisions become team standards;
- source-backed assumptions, with uncertain product or platform claims labelled clearly;
- Architecture Decision Records for major tradeoffs such as service boundary, outbox, idempotency, broker choice, and legacy migration path;
- tests for generated code if implementation starts, including unit, integration, contract, and failure-mode tests;
- threat/risk review for security, privacy, data leakage, payment handling, and abuse cases;
- production-readiness checklist covering logging, metrics, tracing, alerts, SLOs, rollback, migration ownership, and incident recovery;
- no unreviewed AI output merged into production branches, docs, runbooks, or customer-facing behavior.

## Reviewer-Safe Statement

AI helped with structure, wording, and checklist completeness. It did not replace engineering review. Final assessment claims should be validated against source constraints, .NET backend practices, distributed-systems tradeoffs, and the six-page PDF limit before submission.

The responsible AI-first mindset shown here is transparency plus verification: AI drafts options quickly, while humans own correctness, risk, simplification, and final decisions.

## Phase 8 Decisions

- AI usage is disclosed plainly and non-defensively.
- AI-assisted sections are named: roadmap, architecture, API contract, events, reliability design, and observability wording.
- Manual validation covers assessment constraints, Expedia-like domain framing, .NET 8 boundary, event-driven consistency, idempotency/outbox design, and PDF suitability.
- Backend team safeguards prevent blind AI usage through human review, source-backed assumptions, ADRs, tests, risk review, production-readiness checks, and no unreviewed AI output merged.

## References

- `docs/assessment-brief.md`
- `docs/roadmap.md`
- `docs/architecture.md`
- `docs/api-contract.md`
- `docs/events.md`
- `docs/idempotency.md`
- `docs/observability.md`
- `docs/final-submission-outline.md`
