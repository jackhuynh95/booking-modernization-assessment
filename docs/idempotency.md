# Idempotency

## Overview

This document will define idempotency and eventual consistency for booking commands and event processing.

Foundation status: placeholder created for Phase 5.

## Scope For Phase 5

- Idempotency keys for client commands.
- Deduplication storage.
- Command replay behavior.
- Outbox publishing.
- Inbox/event consumer deduplication.
- Retry rules.
- Consistency guarantees.
- Reconciliation jobs and compensating actions.

## Initial Position

Booking commands should be safe to retry. A repeated command with the same idempotency key should return the original result or current stable outcome instead of creating duplicate bookings.

Distributed consistency should be eventual. The service should prefer explicit state transitions, durable events, and reconciliation over cross-service transactions.

## References

- `docs/api-contract.md`
- `docs/events.md`
- `docs/architecture.md`
