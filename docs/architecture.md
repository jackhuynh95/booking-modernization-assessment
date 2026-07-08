# Architecture

## Overview

This document defines the Phase 2 booking microservice structure for the PhoenixDX `.NET Backend Engineer` assessment. The proposal extracts booking lifecycle ownership from the monolith into a .NET 8 service while the legacy database remains part of the transition path.

The design favors a clear service boundary, explicit adapters, idempotent commands, durable outbox publishing, and staged migration. It does not assume a big-bang rewrite or direct shared ownership of legacy tables.

The assessment describes Product A as Expedia-like. Therefore, the booking service is treated as a travel booking lifecycle orchestrator rather than a generic CRUD service. It coordinates booking intent, traveller references, date ranges, booking lines, availability/payment outcomes, confirmation, cancellation, and legacy synchronization while leaving specialized ownership to surrounding bounded contexts.

## Service Boundary

The booking microservice owns the booking lifecycle from request through confirmation, cancellation, failure, and read-back of booking status.

| Area | Booking service owns | Remains outside during transition |
| --- | --- | --- |
| Booking commands | Create, confirm, cancel, expire, fail booking workflows. | Legacy UI screens may still initiate commands through API facade or strangler route. |
| Booking state | Canonical booking aggregate state after cutover of each flow, including travel booking lines such as stay, flight, car, package, or activity references. | Historical legacy rows not yet migrated. |
| Booking rules | Status transitions, duplicate prevention, cancellation windows, retry-safe command handling, quote/hold expiry, and mixed-item booking lifecycle. | Pricing, payment settlement, inventory source of truth, traveller profile ownership, supplier catalog ownership. |
| Integration | REST API, integration events, legacy DB adapter, outbox processor. | Legacy monolith internal modules and existing reporting jobs until replaced. |
| Operations | Booking-specific logs, metrics, traces, alerts, reconciliation workers. | Platform-wide monitoring and legacy application operations. |

### Boundary Principles

- Booking is extracted as a bounded context, not as a table wrapper around the legacy database.
- Booking coordinates travel lifecycle state; it does not absorb every Expedia-like capability into one oversized service.
- The service exposes use-case APIs instead of allowing callers to mutate booking tables directly.
- The legacy database is treated as an integration dependency through an adapter, not as shared domain storage.
- Cross-system consistency is eventual, with explicit state transitions and reconciliation.
- Idempotency is required for externally retried booking commands and internally retried background work.

## Proposed .NET 8 Internal Structure

The service is organized around clean boundaries rather than technical convenience. Names below are conceptual folders/projects for the design document, not an implementation mandate for this repository.

```text
BookingService/
  Api/
    Controllers or MinimalApiEndpoints
    RequestResponseModels
    ApiValidation
    ErrorMapping
  Application/
    UseCases/
      CreateBooking
      ConfirmBooking
      CancelBooking
      ExpirePendingBooking
      ReconcileBooking
    Ports/
      BookingRepository
      LegacyBookingGateway
      PaymentGateway
      InventoryGateway
      EventPublisher
      IdempotencyStore
      Clock
    Behaviors/
      Idempotency
      Validation
      TransactionBoundary
      LoggingTelemetry
  Domain/
    BookingAggregate
    BookingStatus
    BookingLine
    BookingPeriod
    DomainEvents
    BusinessRules
  Infrastructure/
    Persistence/
      BookingDbContext
      BookingRepository
      IdempotencyRecords
      OutboxRecords
    Legacy/
      LegacyBookingDbAdapter
      LegacyBookingMapper
      LegacyWriteGuard
    Messaging/
      OutboxEventPublisher
      MessageBrokerClient
      EventSerializers
    Workers/
      OutboxProcessor
      LegacySyncWorker
      ReconciliationWorker
      ExpiryWorker
    Observability/
      Logging
      Metrics
      Tracing
```

## Layer Responsibilities

### API Layer

The API layer accepts HTTP requests, authenticates/authorizes callers, validates transport-level shape, maps errors to stable HTTP responses, and forwards commands to application use cases.

It should not contain booking business rules. Examples:

- `POST /bookings` maps to `CreateBooking`.
- `POST /bookings/{bookingId}/confirm` maps to `ConfirmBooking`.
- `POST /bookings/{bookingId}/cancel` maps to `CancelBooking`.
- `GET /bookings/{bookingId}` reads booking projection/state.

Command endpoints require an idempotency key so client retries do not create duplicate bookings or duplicate side effects.

### Application / Use-Case Layer

The application layer coordinates one business use case at a time. It owns orchestration, transaction boundaries, idempotency checks, domain aggregate loading, repository persistence, outbox writes, and calls through ports.

Typical `CreateBooking` flow:

1. Validate command and idempotency key.
2. Check idempotency store for prior result.
3. Load required availability/pricing/customer references through ports.
4. Create booking aggregate in `Pending` or `Requested` state.
5. Persist booking state and outbox event in the same transaction.
6. Store idempotency result.
7. Return accepted or created response.

Use cases depend on interfaces, not infrastructure implementations.

### Domain Layer

The domain layer contains booking concepts and rules that must remain true regardless of API, database, broker, or legacy integration details.

Core concepts:

- `Booking` aggregate root with identity, customer reference, booking lines, status, version, timestamps, and cancellation metadata.
- `BookingLine` captures a travel item reference such as stay, flight, car, package component, or activity reservation without owning that external inventory domain.
- `BookingStatus`: `Requested`, `PendingPayment`, `Confirmed`, `Cancelled`, `Failed`, `Expired`.
- State transition rules, for example: `Confirmed` cannot return to `PendingPayment`; `Cancelled` is terminal except for administrative correction workflows.
- Domain events such as `BookingRequested`, `BookingConfirmed`, `BookingCancelled`, and `BookingFailed`.

The domain does not query the legacy database, publish broker messages, or know HTTP status codes.

### Infrastructure Layer

The infrastructure layer implements persistence, messaging, legacy integration, background workers, telemetry, and external service clients.

Key components:

- Booking repository for the new service-owned booking store.
- Idempotency store for command deduplication and replayed response lookup.
- Outbox table written in the same transaction as booking state changes.
- Legacy booking adapter for transitional reads/writes.
- Message broker publisher used only by the outbox processor.
- Background workers for outbox delivery, legacy synchronization, expiry, and reconciliation.

## Legacy DB Adapter

The legacy DB adapter is the anti-corruption layer between the new service model and the old schema.

Responsibilities:

- Translate legacy booking rows into service-facing models during migration.
- Encapsulate legacy stored procedures or SQL access behind `LegacyBookingGateway`.
- Prevent domain logic from depending on legacy column names, status codes, or side effects.
- Enforce write guardrails so only approved transition flows write back to legacy tables.
- Capture correlation IDs and idempotency references when bridging commands into legacy paths.

The adapter should be temporary. Every dependency on it must be visible in the migration plan so it can be removed after ownership moves fully to the service.

## Event Publisher And Outbox Processor

The booking service writes integration events to an outbox table in the same database transaction as booking state. A separate outbox processor publishes those events to the broker.

This avoids losing events when a request succeeds but broker publishing fails.

Outbox processor behavior:

- Poll unpublished events by creation time.
- Publish with event ID, aggregate ID, aggregate version, event type, schema version, occurred time, and correlation ID.
- Mark published only after broker acknowledgement.
- Retry transient failures with backoff.
- Move repeatedly failing records to an operational review state or dead-letter path.
- Preserve per-booking ordering by aggregate version where required.

Initial published events are listed in `docs/events.md`; Phase 4 will define schemas.

## Background Workers

| Worker | Purpose | Safety controls |
| --- | --- | --- |
| `OutboxProcessor` | Publishes durable booking events to broker. | Event ID deduplication, retry backoff, publish status, ordering by aggregate/version. |
| `LegacySyncWorker` | Keeps legacy read paths updated while consumers still depend on legacy tables. | Write guard, mapping tests, reconciliation checks, feature flags. |
| `ReconciliationWorker` | Detects mismatch across booking store, legacy DB, payment, inventory, and broker state. | Read-only compare by default, emits repair task rather than blind mutation. |
| `ExpiryWorker` | Expires stale pending bookings and releases held capacity. | Idempotent state transition, command lock/version check, emits `BookingFailed` or `BookingExpired` if later added. |

Workers must use the same idempotency and concurrency rules as request handling. They should be observable as first-class production processes, not hidden scripts.

## Booking Data Ownership During Transition

Data ownership changes in stages.

| Stage | Canonical booking owner | Legacy DB role | Service DB role |
| --- | --- | --- | --- |
| 1. Observe | Legacy monolith | Source of truth for all booking writes. | Optional shadow read model for comparison. |
| 2. Strangler facade | Legacy monolith for existing flows; booking service owns new API contract. | Still stores canonical records. | Stores idempotency, outbox, and mapped command audit. |
| 3. Flow-by-flow ownership | Booking service for migrated create/confirm/cancel flows. | Transitional read compatibility and selected sync writes. | Canonical state for migrated bookings. |
| 4. Service-owned | Booking service | Historical archive/reporting input only. | Canonical booking state and event history. |
| 5. Legacy retirement | Booking service | Removed from operational path. | Full ownership plus projections/reporting feeds. |

During mixed ownership, every booking record needs an ownership marker such as `owner_system`, `source_booking_id`, and `migration_status`. This prevents the monolith and service from both accepting writes for the same booking.

## Migration Approach

The migration uses a strangler pattern.

1. Put the booking service behind a route/API facade while the monolith still serves existing flows.
2. Introduce read-only legacy adapter and compare service mapping against legacy booking records.
3. Add idempotency store and outbox infrastructure before moving writes.
4. Route low-risk create-booking traffic to the service under feature flags.
5. Sync required fields back to legacy tables for unchanged consumers.
6. Move confirm/cancel flows after create flow proves stable.
7. Reconcile service state against legacy, payment, inventory, and event broker.
8. Cut over read paths to service-owned API/projections.
9. Freeze legacy writes for migrated bookings.
10. Retire sync-back and legacy adapter once no consumers require legacy booking tables.

Rollback strategy should route traffic back to the monolith for not-yet-finalized commands. Already accepted service-owned bookings remain service-owned and are reconciled forward; ownership should not bounce between systems for the same booking.

## Dependencies And Consistency

The booking service should avoid distributed transactions with payment, inventory, or legacy systems.

| Dependency | Interaction | Consistency model |
| --- | --- | --- |
| Legacy DB | Adapter for transitional reads/sync writes. | Eventually consistent during migration. |
| Payment service/gateway | Authorization and payment outcome events. | Pending state until payment outcome known. |
| Inventory/availability | Check and reserve/hold capacity. | Reservation timeout plus reconciliation. |
| Traveller/customer service | Traveller profile and contact references. | Referenced by stable IDs; profile updates are separate from booking lifecycle. |
| Communications service | Sends confirmation, cancellation, and operational notices. | Event-driven downstream consumer. |
| Supplier/catalog services | Hotel, flight, car, package, and activity metadata. | Booking stores references/snapshots needed for audit; catalog remains external owner. |
| Message broker | Integration event publishing from outbox. | At-least-once delivery with consumer dedupe. |
| Reporting/search | Projections fed by events or sync. | Eventually consistent read models. |

The service should expose booking status clearly so clients can handle pending and retryable states without assuming immediate completion.

## Risks And Tradeoffs

| Risk | Mitigation |
| --- | --- |
| Dual writes between service DB and legacy DB cause drift. | Prefer service DB + outbox as canonical for migrated flows; use reconciliation and explicit sync status. |
| Legacy schema leaks into new domain model. | Keep all legacy access behind adapter/mappers. |
| Duplicate booking from client retry. | Require idempotency key and store command result. |
| Event published twice. | Use stable event IDs and consumer dedupe; outbox provides at-least-once semantics. |
| Payment/inventory outcome arrives after cancellation or expiry. | Model status transitions explicitly and reconcile late events. |
| Cutover affects active bookings. | Use ownership markers, feature flags, phased flow migration, and rollback rules. |

## Phase 2 Decisions

- The booking service owns booking lifecycle use cases and canonical state for migrated bookings.
- Expedia-like travel scope is handled through booking orchestration and item references, not by merging payment, inventory, traveller profile, communication, or supplier catalog ownership into the booking service.
- The legacy database is accessed only through a legacy adapter during transition.
- Application use cases coordinate domain rules, persistence, idempotency, and outbox writes.
- Integration events are published through a durable outbox processor, not inline request publishing.
- Background workers are part of the service boundary and must follow idempotent, observable behavior.
- Migration proceeds by strangler pattern with flow-by-flow ownership and explicit ownership markers.

## References

- Expedia Australia: https://www.expedia.com.au/
- `docs/assessment-brief.md`
- `docs/api-contract.md`
- `docs/events.md`
- `docs/idempotency.md`
- `docs/observability.md`
