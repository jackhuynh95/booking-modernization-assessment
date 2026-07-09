# Observability

## Overview

This document defines how the proposed booking microservice is monitored, debugged, and operated after extraction from the legacy system.

The plan assumes a .NET 8 service using structured logs, OpenTelemetry-style traces and metrics, durable outbox/inbox records, and dashboards that let PhoenixDX assess whether the design is operable in production rather than only architecturally plausible.

## Logging Strategy

All service logs should be structured JSON and queryable by business identifiers, workflow identifiers, and infrastructure identifiers. Log messages should describe state transitions and decisions; they should not store card data, supplier secrets, or unnecessary traveller personal data.

| Field | Purpose |
| --- | --- |
| `bookingId` | Primary key for investigating one booking lifecycle. Present once allocated. |
| `travellerId` | Traveller reference for support and duplicate-pattern investigation. Avoid logging full profile data. |
| `correlationId` | End-to-end trace key from API request through use case, persistence, outbox, broker, legacy adapter, and consumers. |
| `causationId` | Command ID, inbound event ID, worker operation ID, or prior workflow step that caused current action. |
| `idempotencyKey` | Supports duplicate request analysis. Store/log scoped key or safe hash if operational policy requires. |
| `eventId` | Stable event/outbox/broker message ID for event publishing and consumer dedupe investigations. |
| `eventType` | Booking event type such as `BookingRequested`, `BookingConfirmed`, `BookingCancelled`, `BookingFailed`, or `BookingExpired`. |
| `aggregateVersion` | Booking version after state transition, used to diagnose stale updates and replay gaps. |
| `legacyBookingId` | Optional migration reference for legacy sync and support. Not primary identity. |

Important log points:

- command accepted, rejected, replayed, or conflicted;
- booking state transition committed;
- idempotency record created, completed, replayed, or payload-mismatched;
- outbox record inserted, published, retried, or dead-lettered;
- inbound event accepted, deduped, rejected as stale, or dead-lettered;
- legacy sync succeeded, failed, timed out, or created review task;
- payment/inventory/supplier dependency timeout or retry exhausted;
- reconciliation mismatch detected and assigned.

## Metrics

Metrics should be tagged by command type, booking status, event type, dependency, failure category, and migration ownership stage where useful. High-cardinality IDs such as `bookingId` and `travellerId` belong in logs/traces, not metric labels.

| Metric | Type | Purpose |
| --- | --- | --- |
| `booking_creation_rate` | Counter/rate | Tracks accepted create-booking commands. |
| `booking_confirmation_rate` | Counter/rate | Tracks successful transition to `Confirmed`. |
| `booking_cancellation_rate` | Counter/rate | Tracks successful transition to `Cancelled`. |
| `failed_booking_count` | Counter | Counts terminal `Failed` bookings by failure category, for example payment, inventory, supplier, validation, or legacy sync. |
| `outbox_backlog` | Gauge | Unpublished or retrying outbox records. |
| `outbox_publish_latency` | Histogram | Time from outbox insert to broker acknowledgement. |
| `legacy_sync_failures` | Counter | Failed or timed-out sync attempts to legacy DB. |
| `payment_inventory_timeout_count` | Counter | Timeout count for payment, inventory, and supplier calls. |
| `reconciliation_mismatch_count` | Counter | Mismatches between booking store, legacy DB, payment, inventory, and broker/outbox state. |

Supporting metrics:

- API request count, latency, and error rate by endpoint/status code;
- idempotency replay count and payload mismatch count;
- inbox duplicate count and stale aggregate version count;
- dead-letter message count by queue/topic and event type;
- worker processing duration, retry count, and last successful run time;
- projection lag for search/reporting/trip views.

## Tracing

Every API command and worker operation should create a trace that links synchronous work with asynchronous records by `correlationId`, `causationId`, `eventId`, and `bookingId`.

| Span | Captures |
| --- | --- |
| API request span | HTTP method/route/status, caller scope, generated or supplied `correlationId`, idempotency requirement, validation result. |
| Application use case span | Use case name, command ID, idempotency lookup result, state transition, aggregate version, response outcome. |
| Legacy DB adapter span | Legacy operation name, `legacyBookingId`, timeout/retry status, migration status, SQL/stored procedure category without sensitive query payload. |
| Payment/inventory call span | Dependency name, operation, timeout/retry result, downstream reference, failure category. |
| Outbox publish span | `eventId`, `eventType`, aggregate version, broker destination, publish attempt, acknowledgement, latency, retry/dead-letter state. |

Trace continuity rules:

- API accepts `X-Correlation-Id` or generates one if missing.
- Commands create a `commandId`; emitted events use that value as `causationId`.
- Outbox records persist `correlationId`, `causationId`, and `eventId` so asynchronous publisher traces can link back to original command.
- Consumers persist inbound `eventId` in inbox records and start child traces with original correlation when possible.

## Alerts

Alerts should focus on user impact and recovery risk, not raw noise. Every alert should link to a dashboard, expected owner, and first investigation query.

| Alert | Trigger direction | First response |
| --- | --- | --- |
| High booking failure rate | `failed_booking_count` or failed/created ratio above threshold. | Break down by failure category, recent deploy/config changes, and dependency health. |
| Outbox backlog growing | `outbox_backlog` increases for sustained window or oldest unpublished record exceeds target delay. | Check broker availability, publisher errors, poison events, and DB locks. |
| Dead-letter messages | Any sustained DLQ growth or high-severity event dead-letter. | Inspect event payload/schema version, consumer error, retry count, and replay safety. |
| Legacy sync failure spike | `legacy_sync_failures` rises or `sync_failed` bookings accumulate. | Check legacy DB latency, stored procedure errors, mapping changes, and feature flag scope. |
| Reconciliation mismatch spike | `reconciliation_mismatch_count` rises across booking/payment/inventory/legacy/broker. | Group by mismatch type, validate canonical source, assign repair path. |
| Payment/inventory dependency degradation | Timeout/error rate or latency breaches target for payment, inventory, or supplier adapters. | Confirm dependency status, evaluate circuit breaker, pause risky traffic if needed. |

## Dashboards

| Dashboard | Shows |
| --- | --- |
| Booking lifecycle health | Create/confirm/cancel/fail/expire rates, command latency, status distribution, failure categories, idempotency replays/conflicts. |
| Event publishing health | Outbox backlog, publish latency, publish failures, retry counts, dead-letter count, event throughput by type, consumer lag. |
| Legacy migration health | Legacy sync success/failure, `migrationStatus` distribution, legacy adapter latency, reconciliation mismatches, ownership stage metrics. |
| Dependency health | Payment, inventory, supplier, broker, database, and legacy dependency latency/error/timeout rates plus circuit-breaker state. |

## SLOs And Operational Targets

Final production SLOs should be set from real traffic baselines. For this assessment, these targets show the intended operating posture:

| Target | Proposed objective |
| --- | --- |
| API availability | 99.9% monthly availability for booking command/read endpoints, excluding planned maintenance and documented dependency outage windows. |
| Booking command latency | 95% of create/confirm/cancel API command acknowledgements under 500 ms when downstream work is asynchronous; 99% under 2 seconds. |
| Event publish delay | 95% of outbox events published within 30 seconds of commit; 99% within 2 minutes. |
| Reconciliation freshness | Critical booking/payment/inventory/legacy mismatches detected within 15 minutes; full legacy migration reconciliation completes at least hourly during cutover. |

Error budget discussions should include business impact: failed booking rate, payment-authorised-without-booking cases, stale legacy views, and delayed customer communication.

## Incident Investigation Flow

Use this flow when support, alerts, or reconciliation identify a suspect booking:

1. Trace by `correlationId` from API gateway, service logs, traces, outbox publisher, broker, consumer, and legacy adapter.
2. Inspect booking state by `bookingId`: status, aggregate version, line statuses, payment/inventory refs, `legacyReference.migrationStatus`, and latest state transition.
3. Inspect idempotency record: key scope, request hash, command type, status, response snapshot, caller, conflict/replay history, and expiry.
4. Inspect outbox/inbox records: `eventId`, event type, aggregate version, publish status, retry count, dead-letter state, consumer inbox result, and stale/duplicate handling.
5. Inspect legacy sync status: legacy reference, last sync attempt, adapter error, retry schedule, reconciliation mismatch, and repair task owner.

Outcome should be one of: no issue/expected pending state, retry safe operation, replay event, rerun legacy sync, trigger compensating workflow, or create manual review task. Direct database mutation is not a normal recovery path.

## Phase 7 Decisions

- Structured logs carry `bookingId`, `travellerId`, `correlationId`, `causationId`, `idempotencyKey`, `eventId`, and event metadata needed for production support.
- Metrics cover booking lifecycle rates, failures, outbox health, legacy sync, dependency timeouts, and reconciliation mismatches.
- Tracing spans cover API, application use case, legacy adapter, payment/inventory calls, and outbox publishing.
- Alerts and dashboards are organized around user impact, event delivery, migration safety, and dependency health.
- Operational targets define API availability, command latency, event publish delay, and reconciliation freshness.
- Incident response follows correlation ID to booking state, idempotency, outbox/inbox, and legacy sync records before repair.

## References

- `docs/architecture.md`
- `docs/api-contract.md`
- `docs/events.md`
- `docs/idempotency.md`
