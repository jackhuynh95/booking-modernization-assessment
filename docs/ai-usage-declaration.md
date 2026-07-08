# AI Usage Declaration

## Overview

This document will disclose AI assistance used while preparing the PhoenixDX `.NET Backend Engineer` booking modernization assessment.

Foundation status: placeholder created for Phase 8.

## Declaration Draft

AI assistance was used to organize documentation, structure the roadmap, identify design topics to cover, and improve clarity of the technical design. The final design decisions, assumptions, tradeoffs, and submission responsibility remain human-owned and human-reviewed.

For Phase 2, AI assistance was used to draft the booking microservice structure, including service boundary, .NET layer responsibilities, legacy database adapter, outbox processor, background workers, transition data ownership, and migration approach. After the Expedia-like product reference was identified, AI assistance was also used to align the design around travel booking orchestration rather than generic CRUD booking. These items require human review before final PDF submission.

For Phase 3, AI assistance was used to draft the API contract for the booking microservice, including ASP.NET Core-style REST endpoint framing, idempotent command headers, correlation handling, Expedia-like booking line types, request/response examples, validation rules, status code conventions, and legacy/downstream ownership assumptions. Human validation is still required to confirm the contract reflects the final assessment interpretation, avoids unsupported product assumptions, and stays concise enough for the six-page PDF.

For Phase 4, AI assistance was used to draft booking lifecycle integration events, including the common event envelope, Azure Service Bus compatibility assumptions, schema versioning, correlation and causation IDs, stable event IDs, at-least-once delivery, event-specific payload examples, outbox publishing, consumer inbox/deduplication expectations, ordering rules, retry behavior, and dead-letter handling. Human review is still required to confirm event names, payload fields, and broker assumptions match the final design direction and remain concise enough for the assessment PDF.

## Allowed Uses

- Drafting documentation structure.
- Summarizing known assessment constraints.
- Suggesting architecture topics and review checklists.
- Improving wording for concise PDF presentation.

## Not Used For

- Submitting work without human review.
- Fabricating assessment requirements not present in source material.
- Creating runnable production code for this foundation phase.

## Review Checklist

- Confirm every AI-assisted claim is technically accurate.
- Remove or label any unsupported assumption.
- Ensure final PDF clearly reflects personal engineering judgment.

## References

- `docs/assessment-brief.md`
- `docs/final-submission-outline.md`
