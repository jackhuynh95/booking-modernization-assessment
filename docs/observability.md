# Observability

## Overview

This document will define how the proposed booking service is monitored, debugged, and operated.

Foundation status: placeholder created for Phase 7.

## Scope For Phase 7

- Structured logs.
- Metrics.
- Distributed traces.
- Correlation IDs.
- Alerting.
- Dashboards.
- SLOs and error budgets.
- Runbook-style investigation paths.

## Initial Signals

| Signal | Examples |
| --- | --- |
| Logs | Booking command accepted/rejected, state transition, dependency failure, retry exhausted. |
| Metrics | Request latency, error rate, booking success rate, idempotency replay count, outbox lag, event publish failures. |
| Traces | API request through validation, persistence, outbox publish, downstream dependency calls. |
| Alerts | Elevated booking failure rate, outbox backlog, payment integration errors, database latency, duplicate command spike. |

## References

- `docs/architecture.md`
- `docs/idempotency.md`
