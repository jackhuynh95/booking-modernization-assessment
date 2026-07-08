# Architecture

## Overview

This document will define the proposed booking microservice structure for the PhoenixDX `.NET Backend Engineer` assessment.

Foundation status: placeholder created so Phase 2 can start directly here.

## Scope For Phase 2

- Booking service boundary and responsibilities.
- Capabilities that remain in the legacy system during migration.
- Conceptual data ownership.
- Synchronous API dependencies.
- Asynchronous event flow.
- Migration strategy from legacy booking logic.
- Risks, tradeoffs, and production follow-ups.

## Draft Direction

The booking service should own booking lifecycle commands and booking state. It should integrate with legacy or adjacent systems through explicit APIs/events, avoid shared database ownership, and use idempotent commands plus durable event publishing for reliability.

## References

- `docs/assessment-brief.md`
- `docs/api-contract.md`
- `docs/events.md`
- `docs/idempotency.md`
- `docs/observability.md`
