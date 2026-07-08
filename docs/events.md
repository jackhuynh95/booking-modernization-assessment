# Events

## Overview

This document defines integration events for the proposed booking microservice. The events support an Expedia-like travel booking lifecycle where one booking may contain stay, flight, car, package, or activity lines.

Events are used for cross-service communication, projections, communications, reporting, legacy synchronization, and operational reconciliation. They are not a dump of internal domain objects and do not expose legacy table structure.

## Event Design Principles

| Principle | Decision |
| --- | --- |
| Integration events, not domain objects | Publish stable facts useful to other services. Keep internal aggregate rules and private fields inside the booking service. |
| Azure Service Bus compatible | Use JSON body, message ID from `eventId`, subject/type from `eventType`, correlation ID, content type `application/json`, and application properties for schema version and aggregate version. |
| Schema versioning | Every event has `schemaVersion`. Additive changes create a minor-compatible version in documentation; breaking changes use a new major version or new event type. |
| Correlation and causation IDs | `correlationId` follows the original API request or worker command. `causationId` points to the command ID, inbound event ID, or workflow step that caused the event. |
| Stable event IDs | `eventId` is created once when the outbox record is inserted. Retries republish the same ID so consumers can dedupe. |
| At-least-once delivery | Broker publishing and consumers assume duplicate delivery is possible. Consumers must be idempotent. |
| Aggregate-scoped ordering | Order matters per booking aggregate. `aggregateId` and `aggregateVersion` let consumers detect gaps, duplicates, and late events. |
| Privacy-conscious payloads | Events carry traveller references and contact snapshot only when needed. They do not carry card data or supplier secrets. |

## Event Envelope

All booking events use this envelope:

```json
{
  "eventId": "evt_01JZKY2H3RE3FZSQ6BKZHMD7X8",
  "eventType": "BookingConfirmed",
  "schemaVersion": "1.0",
  "occurredAt": "2026-07-08T04:35:42Z",
  "correlationId": "web-20260708-00001234",
  "causationId": "cmd_01JZKXBZCX6X5XATX75Q8TMQE3",
  "aggregateId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "aggregateVersion": 4,
  "payload": {}
}
```

| Field | Purpose |
| --- | --- |
| `eventId` | Stable unique event identifier. Also used as broker message ID and consumer dedupe key. |
| `eventType` | Event name, for example `BookingRequested`. |
| `schemaVersion` | Payload contract version. Start at `1.0`. |
| `occurredAt` | UTC timestamp when the booking state change happened, not when broker publish succeeded. |
| `correlationId` | End-to-end trace key from API, worker, legacy adapter, outbox, and downstream consumers. |
| `causationId` | Command ID, inbound event ID, or worker operation that caused this event. |
| `aggregateId` | Booking service-owned booking ID. |
| `aggregateVersion` | Monotonic version of the booking aggregate after the state change. |
| `payload` | Event-specific data needed by consumers. |

## Shared Payload Shape

Lifecycle events reuse this booking reference model where relevant:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "Confirmed",
  "travellerRef": {
    "travellerId": "traveller-123",
    "primaryContactSnapshot": {
      "email": "traveller@example.com",
      "phone": "+61400000000"
    }
  },
  "legacyReference": {
    "legacyBookingId": "LEG-998877",
    "migrationStatus": "synced"
  },
  "quoteSnapshotRef": "quote-flight-555",
  "currency": "AUD",
  "totalAmount": 740.00,
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "type": "flight",
      "status": "Confirmed",
      "supplierRef": "air-supplier-10",
      "quoteRef": "quote-flight-555",
      "priceSnapshotRef": "price-snap_01JZKX91RHTG49BBG7VF35G4VC",
      "startDate": "2026-09-10",
      "endDate": "2026-09-10"
    }
  ]
}
```

`bookingLines.type` supports `stay`, `flight`, `car`, `package`, and `activity`. The booking service stores line references, quote/price snapshot references, and audit totals; supplier catalog, inventory, traveller profile, and payment services remain external owners.

`legacyReference` is optional and exists only for migration, support, reconciliation, and legacy read compatibility. New consumers must use `bookingId` as the primary identifier.

## Published Lifecycle Events

| Event | Purpose | Emitted when |
| --- | --- | --- |
| `BookingRequested` | New booking request accepted and persisted. | `POST /api/bookings` creates a booking in `Requested` or `PendingPayment`. |
| `BookingConfirmed` | Booking is confirmed and downstream consumers may act on the final booking. | Payment, inventory/supplier confirmation, and state transition rules have completed. |
| `BookingCancelled` | Booking is cancelled by traveller, operator, or system workflow. | Cancellation command succeeds and booking enters `Cancelled`. |
| `BookingFailed` | Booking cannot complete because a required business or integration outcome failed. | Payment, inventory, supplier, validation-after-acceptance, or legacy sync failure makes booking terminal `Failed`. |
| `BookingExpired` | Pending booking or quote/hold expired before confirmation. | Expiry worker moves stale `Requested` or `PendingPayment` booking to `Expired`. |

## BookingRequested

| Item | Detail |
| --- | --- |
| Purpose | Tell downstream systems a booking request exists and may need payment, inventory hold conversion, legacy sync, projection, or customer communication. |
| Producer | Booking service application use case through outbox. |
| Likely consumers | Payment orchestration, inventory/availability, legacy sync worker, customer trip projection, reporting/search, fraud/risk, observability pipeline. |
| Emitted when | Create booking command is accepted and booking state plus outbox record are committed. |
| Idempotency notes | `eventId` is stable across outbox publish retries. Consumers dedupe by `eventId`; consumers that create downstream work also key work by `bookingId` plus `aggregateVersion` or command reference. |

Payload:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "PendingPayment",
  "channel": "web",
  "travellerRef": {
    "travellerId": "traveller-123",
    "primaryContactSnapshot": {
      "email": "traveller@example.com",
      "phone": "+61400000000"
    }
  },
  "legacyReference": null,
  "currency": "AUD",
  "totalAmount": 740.00,
  "quoteSnapshotRef": "quote-flight-555",
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "clientLineId": "client-line-1",
      "type": "flight",
      "status": "Requested",
      "supplierRef": "air-supplier-10",
      "quoteRef": "quote-flight-555",
      "priceSnapshotRef": "price-snap_01JZKX91RHTG49BBG7VF35G4VC",
      "startDate": "2026-09-10",
      "endDate": "2026-09-10"
    }
  ],
  "requestedAt": "2026-07-08T04:31:10Z"
}
```

## BookingConfirmed

| Item | Detail |
| --- | --- |
| Purpose | Tell communications, trip views, reporting, support, and legacy sync consumers that booking is confirmed. |
| Producer | Booking service confirm use case or workflow after required downstream outcomes. |
| Likely consumers | Communications service, traveller trip projection, support tooling, reporting/search, legacy sync worker, reconciliation worker. |
| Emitted when | Booking transitions to `Confirmed` and committed aggregate version is stored. |
| Idempotency notes | Consumers should ignore duplicate `eventId`. If a late lower `aggregateVersion` arrives after a later cancellation, consumers must not overwrite newer state blindly. |

Payload:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "Confirmed",
  "confirmedAt": "2026-07-08T04:35:42Z",
  "confirmedBy": "system",
  "travellerRef": {
    "travellerId": "traveller-123",
    "primaryContactSnapshot": {
      "email": "traveller@example.com",
      "phone": "+61400000000"
    }
  },
  "legacyReference": {
    "legacyBookingId": "LEG-998877",
    "migrationStatus": "synced"
  },
  "currency": "AUD",
  "totalAmount": 740.00,
  "paymentReference": "pay_auth_456",
  "quoteSnapshotRef": "quote-flight-555",
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "type": "flight",
      "status": "Confirmed",
      "supplierRef": "air-supplier-10",
      "supplierConfirmationRef": "air-confirm-ABC123",
      "quoteRef": "quote-flight-555",
      "priceSnapshotRef": "price-snap_01JZKX91RHTG49BBG7VF35G4VC",
      "startDate": "2026-09-10",
      "endDate": "2026-09-10"
    }
  ]
}
```

## BookingCancelled

| Item | Detail |
| --- | --- |
| Purpose | Tell consumers to update trip views, stop confirmation workflows, start refund/supplier cancellation work where applicable, and notify traveller/support. |
| Producer | Booking service cancel use case or system cancellation workflow. |
| Likely consumers | Payment/refund workflow, inventory release, supplier cancellation adapter, communications service, trip projection, legacy sync worker, reporting/search. |
| Emitted when | Booking transitions to `Cancelled`. Follow-up refund or supplier events may arrive later from other services. |
| Idempotency notes | Replayed cancel command must not emit a second logical cancellation event unless aggregate state actually changes. Broker retries reuse same `eventId`. |

Payload:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "Cancelled",
  "cancelledAt": "2026-07-08T05:10:00Z",
  "cancelledBy": "traveller-123",
  "reasonCode": "traveller_request",
  "travellerRef": {
    "travellerId": "traveller-123"
  },
  "legacyReference": {
    "legacyBookingId": "LEG-998877",
    "migrationStatus": "sync_pending"
  },
  "refundExpected": true,
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "type": "stay",
      "status": "CancellationPending",
      "supplierRef": "hotel-supplier-456",
      "quoteRef": "quote-stay-789",
      "priceSnapshotRef": "price-snap_01JZKX91RHTG49BBG7VF35G4VC",
      "startDate": "2026-09-10",
      "endDate": "2026-09-14"
    }
  ]
}
```

## BookingFailed

| Item | Detail |
| --- | --- |
| Purpose | Tell consumers the booking reached a terminal failure and any pending downstream work should stop or compensate. |
| Producer | Booking service workflow, reconciliation worker, or command handler after a non-recoverable business/integration failure. |
| Likely consumers | Communications service, trip projection, payment/inventory compensation workflows, legacy sync worker, support operations, reporting/search. |
| Emitted when | Booking transitions to `Failed` because payment, inventory, supplier, validation-after-acceptance, or migration sync cannot complete safely. |
| Idempotency notes | Failure cause is part of the terminal state. Repeated detection of the same failure should not create new failure events unless the aggregate version changes through an explicit correction workflow. |

Payload:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "Failed",
  "failedAt": "2026-07-08T04:36:30Z",
  "failureCode": "inventory_unavailable",
  "failureCategory": "business",
  "retryable": false,
  "travellerRef": {
    "travellerId": "traveller-123"
  },
  "legacyReference": null,
  "quoteSnapshotRef": "quote-package-999",
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "type": "package",
      "status": "Failed",
      "supplierRef": "package-supplier-22",
      "quoteRef": "quote-package-999",
      "priceSnapshotRef": "price-snap_01JZKX91RHTG49BBG7VF35G4VC",
      "startDate": "2026-09-10",
      "endDate": "2026-09-14"
    }
  ]
}
```

## BookingExpired

| Item | Detail |
| --- | --- |
| Purpose | Tell consumers a pending booking or quote/hold window expired before confirmation. |
| Producer | Booking service expiry worker. |
| Likely consumers | Inventory release, payment workflow cleanup, trip projection, communications service, legacy sync worker, reporting/search. |
| Emitted when | Stale `Requested` or `PendingPayment` booking transitions to `Expired`. |
| Idempotency notes | Expiry worker must use optimistic concurrency or aggregate version checks so two worker attempts do not emit duplicate logical expiry events. |

Payload:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "Expired",
  "expiredAt": "2026-07-08T05:30:00Z",
  "expiryReason": "quote_expired",
  "travellerRef": {
    "travellerId": "traveller-123"
  },
  "legacyReference": null,
  "quoteSnapshotRef": "quote-activity-321",
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "type": "activity",
      "status": "Expired",
      "supplierRef": "activity-supplier-77",
      "quoteRef": "quote-activity-321",
      "priceSnapshotRef": "price-snap_01JZKX91RHTG49BBG7VF35G4VC",
      "startDate": "2026-09-12",
      "endDate": "2026-09-12"
    }
  ]
}
```

## Consumed Events

The booking service may also consume external events during orchestration. These are dependencies, not events owned by this document.

| Event | Producer | Booking service behavior |
| --- | --- | --- |
| `PaymentAuthorised` | Payment service/gateway | Advance eligible booking toward confirmation. Store payment reference, correlation ID, and causation ID. |
| `PaymentFailed` | Payment service/gateway | Transition eligible booking to `Failed` or keep retryable pending state depending on failure type. |
| `InventoryHeld` | Inventory/availability service | Record hold reference and continue confirmation workflow. |
| `InventoryUnavailable` | Inventory/availability service | Transition to `Failed` or request alternate line handling. |
| `SupplierConfirmed` | Supplier adapter/service | Record supplier confirmation reference and line status. |
| `SupplierRejected` | Supplier adapter/service | Fail or compensate affected booking/line based on booking state. |

Inbound event handlers must use an inbox/dedupe table keyed by inbound `eventId` and must check booking `aggregateVersion` or current state before applying transitions.

## Ordering Rules

Order matters within a booking aggregate, not globally across all bookings.

- The outbox stores events with `aggregateId` and `aggregateVersion`.
- The publisher should send events for the same booking in version order where broker configuration allows it.
- Azure Service Bus sessions can use `aggregateId` as session ID when strict per-booking ordering is required.
- Consumers must still tolerate duplicates, late events, and gaps because redelivery, retries, and replay can happen.
- Consumers that maintain projections should store last processed aggregate version per booking and reject stale updates unless a replay mode explicitly rebuilds state.
- Cross-aggregate ordering is not guaranteed. Reporting and search projections should converge eventually.

## Outbox Publishing

The booking service uses the transactional outbox pattern:

1. Command handler validates idempotency, loads aggregate, applies state transition, and writes booking state.
2. Same database transaction inserts one outbox record with final event envelope, payload, and stable `eventId`.
3. Outbox worker polls unpublished records, usually by creation time and aggregate/version.
4. Worker publishes JSON to the broker with `eventId` as message ID and `correlationId` as broker correlation ID.
5. Worker marks record as published only after broker acknowledgement.
6. Transient publish failures retry with backoff and retry count.
7. Records that repeatedly fail move to operational review or dead-letter handling with alerting.

The API request must not publish directly to the broker before the database transaction commits. That would risk consumers seeing an event for a booking state that was rolled back.

## Consumer Expectations

Consumers must assume at-least-once delivery:

- Keep an inbox/dedupe table keyed by `eventId`.
- Make handlers idempotent for external side effects such as email sends, refunds, supplier calls, and projection updates.
- Store `correlationId`, `causationId`, and processed time for audit.
- Validate `schemaVersion` before processing.
- Treat unknown additive fields as safe to ignore for compatible versions.
- Do not rely on distributed transactions between services.
- Use compensating actions, reconciliation, or manual review for partial failures.
- Dead-letter poison messages after bounded retries and alert the owning team.

## Retry And Dead-Letter Handling

| Failure | Expected handling |
| --- | --- |
| Broker temporarily unavailable | Outbox worker retries with exponential backoff. Booking state remains committed. |
| Broker accepts but acknowledgement lost | Worker may republish same `eventId`. Consumer dedupe handles duplicate. |
| Consumer transient failure | Broker redelivery and consumer retry. Handler remains idempotent. |
| Consumer poison message | Move to dead-letter queue with event metadata, correlation ID, schema version, and failure reason. |
| Consumer observes missing aggregate version | Pause or mark projection stale, then replay from event store/outbox projection where available or query booking API for current state. |
| Legacy sync failure | Keep booking service state canonical for migrated flow; emit alert/reconciliation task rather than rolling back confirmed booking blindly. |

## Phase 4 Decisions

- Booking events are integration contracts, not serialized internal domain objects.
- The event envelope includes event ID, type, schema version, timestamps, correlation, causation, aggregate ID, aggregate version, and payload.
- Azure Service Bus is a compatible broker target, with at-least-once delivery and optional sessions for per-booking ordering.
- `BookingRequested`, `BookingConfirmed`, `BookingCancelled`, `BookingFailed`, and `BookingExpired` cover the booking lifecycle for the assessment.
- All published events include Expedia-like travel references: booking lines, traveller reference, quote/price snapshot reference, and optional legacy reference.
- Outbox and inbox/dedupe patterns are required because there is no distributed transaction across booking, payment, inventory, supplier, legacy, and communications services.

## References

- `docs/architecture.md`
- `docs/api-contract.md`
- `docs/idempotency.md`
- `docs/observability.md`
