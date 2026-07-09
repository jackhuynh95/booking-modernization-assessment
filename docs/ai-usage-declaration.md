# AI Usage Declaration

## Overview

This document discloses AI assistance used while preparing the PhoenixDX `.NET Backend Engineer` booking modernization assessment.

AI was used as a drafting and review aid. The design remains human-owned: assumptions, tradeoffs, technical accuracy, PDF suitability, and final submission responsibility require human validation before delivery.

PhoenixDX can assess AI-first mindset responsibly by looking for transparent disclosure, source-backed assumptions, coherent engineering judgment, and evidence that generated material was reviewed rather than pasted uncritically.

## PhoenixDX Required Disclosure

AI Tools Used:

OpenAI Codex / ChatGPT-style assistant was used during this exercise. I also used project-scoped .NET agent skills installed under `.codex/skills`, including .NET architecture, modern C#, testing, coverage, and project setup related skills, to guide review checklists and terminology.

Sections AI-Assisted:

Roadmap, architecture structure, API contract, event model, idempotency and reliability design, observability plan, AI usage wording, final PDF compression, and completeness checks.

Manually Validated:

I manually reviewed the assessment constraints, Expedia-like travel domain framing, .NET 8 microservice boundary, API and event consistency, idempotency/outbox strategy, edge cases, observability practicality, PDF page limit, and final submission quality.

Preventing Blind AI Usage:

In a backend team, I would require human design review, source-backed assumptions, architecture decision records for major decisions, tests for generated implementation code, security/privacy review, production-readiness checks, and a rule that no unreviewed AI output is merged into production work.

No AI tool was used to generate runnable backend implementation code for this foundation phase.

## Reviewer-Safe Statement

AI helped with structure, wording, and checklist completeness. It did not replace engineering review. Final assessment claims should be validated against source constraints, .NET backend practices, distributed-systems tradeoffs, and the six-page PDF limit before submission.

The responsible AI-first mindset shown here is transparency plus verification: AI drafts options quickly, while humans own correctness, risk, simplification, and final decisions.

## Phase 8 Decisions

- AI declaration maps exactly to PhoenixDX requirements: AI Tools Used, Sections AI-Assisted, Manually Validated, and Preventing Blind AI Usage.
- AI tools include OpenAI Codex / ChatGPT-style assistant and project-scoped .NET agent skills under `.codex/skills`.
- AI-assisted sections include roadmap, architecture, API, events, reliability, observability, AI wording, PDF compression, and completeness checks.
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
