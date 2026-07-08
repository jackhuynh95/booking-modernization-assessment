# API Contract

## Overview

This document will capture the external contract for the proposed booking microservice.

Foundation status: placeholder created for Phase 3.

## Scope For Phase 3

- REST endpoint list.
- Command request shapes.
- Query response shapes.
- Validation rules.
- Error model.
- Status code conventions.
- Idempotency header/field usage.
- Pagination/filtering where needed.
- Contract assumptions for legacy and downstream integrations.

## Candidate Endpoints

| Endpoint | Purpose |
| --- | --- |
| `POST /bookings` | Create booking request. |
| `GET /bookings/{bookingId}` | Read booking state. |
| `POST /bookings/{bookingId}/confirm` | Confirm booking if business checks pass. |
| `POST /bookings/{bookingId}/cancel` | Cancel booking. |
| `GET /bookings` | Search bookings by customer, status, or date range. |

## References

- `docs/architecture.md`
- `docs/idempotency.md`
