# Events

## Overview

This document will define integration events emitted and consumed by the proposed booking microservice.

Foundation status: placeholder created for Phase 4.

## Scope For Phase 4

- Event names.
- Event schema fields.
- Producer and consumer responsibilities.
- Event versioning.
- Ordering expectations.
- Replay and deduplication behavior.
- Failure handling and dead-letter strategy.

## Candidate Events

| Event | Direction | Purpose |
| --- | --- | --- |
| `BookingRequested` | Published | Booking create command accepted. |
| `BookingConfirmed` | Published | Booking confirmed and available to downstream systems. |
| `BookingCancelled` | Published | Booking cancelled by customer/system/operator. |
| `BookingFailed` | Published | Booking could not complete after business or integration failure. |
| `PaymentAuthorised` | Consumed | Payment dependency succeeded. |
| `PaymentFailed` | Consumed | Payment dependency failed. |

## References

- `docs/architecture.md`
- `docs/idempotency.md`
