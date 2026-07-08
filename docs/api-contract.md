# API Contract

## Overview

This document defines the external HTTP contract for the proposed booking microservice in the PhoenixDX `.NET Backend Engineer` assessment.

The API is framed for an Expedia-like travel platform. A booking can contain one or more travel booking lines, such as stay, flight, car, package, or activity. The service owns booking lifecycle state and orchestration; payment, inventory, supplier catalog, traveller profile, and communications remain external domains.

## API Design Principles

| Principle | Contract decision |
| --- | --- |
| .NET 8 / ASP.NET Core style | Use resource-oriented routes under `/api/bookings`, JSON request/response bodies, route parameters as GUIDs, query parameters for filters, and Problem Details-compatible validation errors. |
| Use-case oriented endpoints | Commands map to business use cases: create, confirm, cancel, and expire. The API does not expose generic table update endpoints. |
| Idempotent command handling | Mutating endpoints require `Idempotency-Key`. Replayed commands return the original accepted/result response where possible. |
| Eventual consistency friendly responses | Commands may return `202 Accepted` when payment, inventory, supplier, legacy sync, or outbox work continues asynchronously. Clients inspect booking status through read endpoints. |
| No direct legacy table exposure | Public IDs, fields, statuses, and relationships are service-owned concepts. Legacy schema, status codes, and table keys stay behind an adapter. |
| Explicit observability | All requests accept or generate `X-Correlation-Id`, which flows into logs, traces, outbox records, events, and legacy adapter calls. |

## Required Headers

| Header | Required | Applies to | Purpose |
| --- | --- | --- | --- |
| `Idempotency-Key` | Yes | `POST /api/bookings`, `POST /api/bookings/{bookingId}/confirm`, `POST /api/bookings/{bookingId}/cancel`, `POST /api/bookings/{bookingId}/expire` | Stable client-generated key for retry-safe command handling. Scope is caller + endpoint + booking where applicable. |
| `X-Correlation-Id` | Recommended; generated if missing | All endpoints | End-to-end trace key across API, application use case, persistence, outbox, downstream calls, and legacy adapter. |

Header examples:

```http
Idempotency-Key: 9f528d6a-7f1e-4e80-9d31-2ebd7cf8ec4f
X-Correlation-Id: web-20260708-00001234
```

## Booking Line Model

`bookingLines` is required on create and must contain at least one line. Each line uses a common envelope plus type-specific details.

| Field | Required | Notes |
| --- | --- | --- |
| `lineId` | Optional on create | Client reference for request-level validation and error targeting. Service returns stable line IDs in responses. |
| `type` | Yes | One of `stay`, `flight`, `car`, `package`, `activity`. |
| `supplierRef` | Conditional | External supplier/catalog reference when known. Booking stores reference, not supplier ownership. |
| `quoteRef` | Conditional | Required where price or availability was quoted before booking. |
| `priceSnapshot` | Conditional | Required where booking relies on quoted price, currency, tax, fee, or cancellation terms. |
| `startDate` / `endDate` | Conditional | Required for dated products. Date range must be valid for line type. |
| `details` | Yes | Type-specific object. |

Line types:

| Type | Example details | Ownership boundary |
| --- | --- | --- |
| `stay` | Hotel/property ref, room type, rate plan, check-in/check-out, guest count. | Supplier/catalog owns property and room metadata; inventory owns availability. |
| `flight` | Origin, destination, flight segment refs, cabin, passenger refs. | Airline/supplier and inventory systems own flight availability and ticketing details. |
| `car` | Pickup/dropoff location, pickup/dropoff time, vehicle class, driver ref. | Car supplier owns vehicle inventory and rental conditions. |
| `package` | Component line refs or bundled offer ref, package quote, package cancellation terms. | Package pricing/catalog owns bundle composition and pricing rules. |
| `activity` | Activity/product ref, session time, participant refs. | Activity supplier/catalog owns session availability and content. |

Example line:

```json
{
  "lineId": "client-line-1",
  "type": "stay",
  "supplierRef": "hotel-supplier-456",
  "quoteRef": "quote-stay-789",
  "startDate": "2026-09-10",
  "endDate": "2026-09-14",
  "priceSnapshot": {
    "currency": "AUD",
    "totalAmount": 1280.50,
    "taxAmount": 116.40,
    "feeAmount": 24.00,
    "capturedAt": "2026-07-08T04:30:00Z"
  },
  "details": {
    "propertyRef": "property-123",
    "roomTypeRef": "room-deluxe-king",
    "ratePlanRef": "rate-refundable-breakfast",
    "guestCount": 2
  }
}
```

## Core Endpoints

| Endpoint | Purpose | Typical success |
| --- | --- | --- |
| `POST /api/bookings` | Create booking from traveller and booking lines. | `201 Created` or `202 Accepted` |
| `GET /api/bookings/{bookingId}` | Read booking detail and current lifecycle state. | `200 OK` |
| `GET /api/bookings?travellerId=&status=&from=&to=` | Search bookings by traveller, status, and date range. | `200 OK` |
| `POST /api/bookings/{bookingId}/confirm` | Confirm booking after required checks or downstream outcomes. | `202 Accepted` or `200 OK` |
| `POST /api/bookings/{bookingId}/cancel` | Cancel booking by traveller, operator, or system flow. | `202 Accepted` or `200 OK` |
| `POST /api/bookings/{bookingId}/expire` | Expire stale pending booking or quote/hold. | `202 Accepted` or `200 OK` |

### POST /api/bookings

Creates a booking request. The service validates traveller reference, line shape, date ranges, quote/price snapshots, and idempotency before creating a service-owned booking ID.

Required headers:

- `Idempotency-Key`
- `X-Correlation-Id` recommended

Request:

```json
{
  "travellerId": "traveller-123",
  "channel": "web",
  "currency": "AUD",
  "bookingLines": [
    {
      "lineId": "client-line-1",
      "type": "flight",
      "supplierRef": "air-supplier-10",
      "quoteRef": "quote-flight-555",
      "startDate": "2026-09-10",
      "endDate": "2026-09-10",
      "priceSnapshot": {
        "currency": "AUD",
        "totalAmount": 740.00,
        "taxAmount": 82.00,
        "feeAmount": 15.00,
        "capturedAt": "2026-07-08T04:30:00Z"
      },
      "details": {
        "origin": "SYD",
        "destination": "MEL",
        "segments": [
          {
            "flightRef": "QF401",
            "departureAt": "2026-09-10T06:00:00+10:00",
            "arrivalAt": "2026-09-10T07:35:00+10:00",
            "cabin": "economy"
          }
        ],
        "passengerRefs": ["traveller-123"]
      }
    }
  ],
  "contact": {
    "email": "traveller@example.com",
    "phone": "+61400000000"
  },
  "metadata": {
    "source": "expedia-like-web",
    "campaignRef": "winter-sale"
  }
}
```

Response `201 Created` when booking state is created synchronously:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "status": "Requested",
  "version": 1,
  "links": {
    "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
    "confirm": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA/confirm",
    "cancel": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA/cancel"
  },
  "correlationId": "web-20260708-00001234"
}
```

Response `202 Accepted` when downstream confirmation is still pending:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "commandId": "cmd_01JZKX9P0GZVZ3FZT7G6CF4NAE",
  "status": "PendingPayment",
  "acceptedAt": "2026-07-08T04:31:10Z",
  "message": "Booking command accepted; final confirmation is asynchronous.",
  "links": {
    "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA"
  },
  "correlationId": "web-20260708-00001234"
}
```

### GET /api/bookings/{bookingId}

Returns booking detail and current lifecycle state.

Response:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "legacyReference": {
    "legacyBookingId": "LEG-998877",
    "migrationStatus": "synced"
  },
  "travellerId": "traveller-123",
  "status": "Confirmed",
  "version": 4,
  "currency": "AUD",
  "totalAmount": 740.00,
  "createdAt": "2026-07-08T04:31:10Z",
  "updatedAt": "2026-07-08T04:35:42Z",
  "confirmedAt": "2026-07-08T04:35:42Z",
  "bookingLines": [
    {
      "bookingLineId": "line_01JZKX9M9VR1PAZN22GBB7ZJZT",
      "type": "flight",
      "status": "Confirmed",
      "supplierRef": "air-supplier-10",
      "quoteRef": "quote-flight-555",
      "startDate": "2026-09-10",
      "endDate": "2026-09-10",
      "priceSnapshot": {
        "currency": "AUD",
        "totalAmount": 740.00,
        "taxAmount": 82.00,
        "feeAmount": 15.00,
        "capturedAt": "2026-07-08T04:30:00Z"
      },
      "details": {
        "origin": "SYD",
        "destination": "MEL",
        "segments": [
          {
            "flightRef": "QF401",
            "departureAt": "2026-09-10T06:00:00+10:00",
            "arrivalAt": "2026-09-10T07:35:00+10:00",
            "cabin": "economy"
          }
        ],
        "passengerRefs": ["traveller-123"]
      }
    }
  ],
  "links": {
    "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
    "cancel": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA/cancel"
  },
  "correlationId": "web-20260708-00001234"
}
```

`legacyReference` is optional and only supports migration/reconciliation. It is not the primary identifier and must not be required by new clients.

### GET /api/bookings?travellerId=&status=&from=&to=

Searches service-owned booking records or projections. Filters are optional but should be bounded by authorization and reasonable paging defaults.

Query parameters:

| Parameter | Notes |
| --- | --- |
| `travellerId` | Stable traveller profile reference. |
| `status` | One of `Requested`, `PendingPayment`, `Confirmed`, `Cancelled`, `Failed`, `Expired`. |
| `from` / `to` | Inclusive booking date or travel date window, depending on chosen final contract. Use ISO-8601 date. |
| `pageSize` / `pageToken` | Optional pagination fields for large traveller histories. |

Response:

```json
{
  "items": [
    {
      "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
      "travellerId": "traveller-123",
      "status": "Confirmed",
      "lineTypes": ["flight"],
      "startDate": "2026-09-10",
      "endDate": "2026-09-10",
      "currency": "AUD",
      "totalAmount": 740.00,
      "updatedAt": "2026-07-08T04:35:42Z",
      "links": {
        "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA"
      }
    }
  ],
  "nextPageToken": null,
  "correlationId": "web-20260708-00001234"
}
```

### POST /api/bookings/{bookingId}/confirm

Attempts to confirm a booking. Confirmation may require final payment authorization, supplier confirmation, inventory hold conversion, legacy sync, and event publication.

Request:

```json
{
  "expectedVersion": 3,
  "paymentReference": "pay_auth_456",
  "confirmedBy": "system"
}
```

Response `202 Accepted`:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "commandId": "cmd_01JZKXBZCX6X5XATX75Q8TMQE3",
  "status": "PendingPayment",
  "acceptedAt": "2026-07-08T04:34:00Z",
  "message": "Confirm command accepted; booking will move to Confirmed after required downstream outcomes.",
  "links": {
    "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA"
  },
  "correlationId": "web-20260708-00001234"
}
```

Response `200 OK` may be used when booking is already confirmed for the same idempotency key.

### POST /api/bookings/{bookingId}/cancel

Cancels a booking when lifecycle state and cancellation policy allow it. Cancellation can be traveller, operator, or system initiated.

Request:

```json
{
  "reasonCode": "traveller_request",
  "requestedBy": "traveller-123",
  "notes": "Traveller cancelled from self-service portal.",
  "expectedVersion": 4
}
```

Response:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "commandId": "cmd_01JZKXCX4SX5FZJX7G633AHH95",
  "status": "Cancelled",
  "acceptedAt": "2026-07-08T05:10:00Z",
  "message": "Cancel command accepted.",
  "links": {
    "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA"
  },
  "correlationId": "web-20260708-00009999"
}
```

Use `202 Accepted` when refunds, supplier cancellation, communications, or legacy sync continue asynchronously. Use `200 OK` when cancellation is already complete or replayed by same idempotency key.

### POST /api/bookings/{bookingId}/expire

Expires stale bookings that remain in `Requested` or `PendingPayment` after quote, payment, or inventory hold windows close. This is usually called by a trusted internal worker, not public clients.

Request:

```json
{
  "reasonCode": "quote_expired",
  "expiredBy": "expiry-worker",
  "expectedVersion": 2
}
```

Response:

```json
{
  "bookingId": "bkg_01JZKX9M6HTR7D9F28P7S0N4CA",
  "commandId": "cmd_01JZKXDQF2JRE6BC7A6DZR0G6V",
  "status": "Expired",
  "acceptedAt": "2026-07-08T05:30:00Z",
  "message": "Expire command accepted.",
  "links": {
    "self": "/api/bookings/bkg_01JZKX9M6HTR7D9F28P7S0N4CA"
  },
  "correlationId": "expiry-20260708-000001"
}
```

## Status Code Conventions

| Status | Use |
| --- | --- |
| `201 Created` | Booking resource created synchronously. Include `Location` header pointing to `/api/bookings/{bookingId}`. |
| `202 Accepted` | Command accepted but final state depends on asynchronous payment, inventory, supplier, outbox, legacy sync, or reconciliation work. |
| `200 OK` | Read succeeded, command replay returned prior stable result, or command completed synchronously. |
| `400 Bad Request` | Malformed JSON, missing required headers, invalid query format, invalid enum syntax, or invalid route/query shape. |
| `404 Not Found` | Booking ID is unknown or not visible to caller. |
| `409 Conflict` | State/version conflict, duplicate idempotency key with different payload, or concurrency conflict. |
| `422 Unprocessable Entity` | Request shape is valid but business validation fails, such as invalid traveller reference, invalid date range, unsupported line type, missing quote snapshot, or invalid state transition. |
| `503 Service Unavailable` | Required downstream dependency is unavailable and command cannot be safely accepted. Use with `Retry-After` where appropriate. |

## Validation Rules

| Rule | Failure status | Notes |
| --- | --- | --- |
| Valid traveller reference | `422 Unprocessable Entity` | `travellerId` must reference a known traveller/profile accessible to caller. Traveller profile remains external owner. |
| At least one booking line | `422 Unprocessable Entity` | Empty `bookingLines` is invalid. |
| Valid date range | `422 Unprocessable Entity` | `startDate` must be before or equal to `endDate` where same-day products are valid. Time-zone-sensitive line details must use ISO-8601 timestamps. |
| Valid booking line type | `422 Unprocessable Entity` | Type must be `stay`, `flight`, `car`, `package`, or `activity`. |
| Quote/price snapshot required where applicable | `422 Unprocessable Entity` | Required for lines created from quoted offers, package prices, supplier holds, or auditable totals. |
| Cannot confirm invalid state transition | `409 Conflict` or `422 Unprocessable Entity` | `Cancelled`, `Failed`, and `Expired` bookings cannot be confirmed through normal flow. Version mismatch uses `409`. Business rule violation uses `422`. |
| Cannot cancel invalid state transition | `409 Conflict` or `422 Unprocessable Entity` | Already terminal bookings cannot be cancelled again unless replayed by same idempotency key or handled through an admin correction process outside this contract. |
| Idempotency key replay payload must match original | `409 Conflict` | Same key with different command body is rejected. |

## Error Response Shape

Validation and business errors should use a stable Problem Details-compatible shape.

```json
{
  "type": "https://booking.example.com/problems/validation-error",
  "title": "Validation failed",
  "status": 422,
  "detail": "One or more booking fields are invalid.",
  "correlationId": "web-20260708-00001234",
  "errors": [
    {
      "field": "bookingLines[0].quoteRef",
      "code": "quote_required",
      "message": "Quote reference is required for a quoted flight booking line."
    },
    {
      "field": "bookingLines[0].priceSnapshot",
      "code": "price_snapshot_required",
      "message": "Price snapshot is required to preserve quoted booking terms."
    }
  ]
}
```

## Legacy And Downstream Assumptions

- API exposes service-owned `bookingId` as primary ID.
- Legacy booking IDs are not public ownership keys. They may appear only as optional `legacyReference` for migration support, reconciliation, support tooling, or read compatibility.
- Legacy DB access is internal to `LegacyBookingGateway` / adapter. Legacy table names, column names, stored procedure names, and status codes must not leak into the public contract.
- Payment remains external. Booking stores payment references and lifecycle state, not card details or settlement ownership.
- Inventory/availability remains external. Booking stores holds, supplier refs, snapshots, and outcomes needed for lifecycle decisions.
- Supplier/catalog domains remain external owners of hotel, flight, car, package, and activity metadata.
- Traveller profile remains external. Booking stores stable `travellerId`, passenger references, and contact snapshot needed for booking audit.
- Communications remains external. Booking publishes events; communications sends email/SMS/push notifications.
- Eventual consistency is expected. API responses must make pending states visible instead of pretending payment, supplier, inventory, legacy sync, and communications finish atomically.

## References

- `docs/architecture.md`
- `docs/events.md`
- `docs/idempotency.md`
- `docs/observability.md`
