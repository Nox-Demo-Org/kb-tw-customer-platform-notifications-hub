---
type: Interface Reference
title: API and Event Contract Reference
description: notifications-hub exposes a REST endpoint for synchronous on-demand notification requests and subscribes to asynchronous Google Cloud Pub/Sub events emitted by upstream Tidewell services.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/summaries/api-spec.md
tags:
- notifications-hub
- summaries
sources:
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/handlers/messages.ts
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/subscriptions.ts
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/README.md
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T13:10:26Z'
---

<!-- anchor: src/handlers/messages.ts:L1-L5 -->
<!-- anchor: src/subscriptions.ts:L1-L8 -->
<!-- anchor: README.md:L1-L10 -->

# API and Event Contract Reference

`notifications-hub` exposes a REST endpoint for synchronous on-demand notification requests and subscribes to asynchronous Google Cloud Pub/Sub events emitted by upstream Tidewell services.

---

## Inbound REST API

Direct message dispatch requests are ingested via the handler in `src/handlers/messages.ts`.

### `POST /v1/messages`

Invoked directly by core domain services—including `policy-admin`, `claims-management`, and `billing-service`—to dispatch notifications on demand.

- **Handler Function**: `postMessage` (`src/handlers/messages.ts`)
- **Processing**: The handler receives the request, resolves customer contact details on demand, renders the template, and delegates delivery to the target channel (see [[concepts/direct-dispatch-flow]] and [[entities/message-handler]]).
- **Response Status**: `202 Accepted`

#### Request Body Schema

```typescript
{
  to_customer_id: string;
  template: string;
  data: Record<string, unknown>;
}
```

| Field | Type | Description |
| --- | --- | --- |
| `to_customer_id` | `string` | Unique identifier of the target customer whose contact details are retrieved. |
| `template` | `string` | Template identifier defined in the template catalog (see [[summaries/templates-and-channels]]). |
| `data` | `Record<string, unknown>` | Key-value dictionary containing template rendering variables. |

#### Response Format

```json
{
  "status": 202
}
```

---

## Consumed Pub/Sub Events

`notifications-hub` subscribes to domain events across Google Cloud Pub/Sub topics. Inbound events are mapped to specific notification templates via the registry in `src/subscriptions.ts` (see [[entities/subscription-registry]] and [[concepts/event-driven-notifications]]).

### Event-to-Template Mapping

| Pub/Sub Topic | Publisher Service | Triggered Template | Channel Type | External Reference |
| --- | --- | --- | --- | --- |
| `policy.renewal.due` | `policy-admin` | `renewal-notice` | Email | — |
| `claims.claim.settled` | `claims-management` | `claim-settled` | Email | — |
| [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] | `billing-service` | `instalment-reminder` | Email | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] |
| [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] | `billing-service` | `missed-payment` | SMS | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] |
| `payments.payout.sent` | `payments-gateway` | `payout-sent` | SMS | — |

*Note on channel assignment and subject headers: Refer to [[summaries/templates-and-channels]] and [[decisions/template-routing-strategy]].*

---

## Pub/Sub Conventions

All asynchronous events follow Tidewell's enterprise Pub/Sub standards:
- **Topic Naming**: `<domain>.<entity>.<past-tense-verb>` (e.g., [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]], `claims.claim.settled`).
- **Subscription Naming**: `<consumer>-<topic-short-name>` (e.g., `notifications-hub` consumer subscriptions).
- **Payload Format**: JSON formatted with `snake_case` field names, entity identifiers as `string`, and currency amounts in pence as integer values.

### Explicitly Excluded Events
- **`claims.handler.assigned`**: Published by `claims-management` when a claim handler is assigned, but **not** consumed by `notifications-hub`. Customers do not receive an automated notification when a handler picks up their claim.

---

## Related Documentation
- [[entities/message-handler]] — Implementation details of `POST /v1/messages`.
- [[entities/subscription-registry]] — Pub/Sub subscription dictionary (`SUBSCRIPTIONS`).
- [[summaries/templates-and-channels]] — Template metadata, subject lines, and channel assignments.
- [[concepts/event-driven-notifications]] — End-to-end event subscription lifecycle.
- [[concepts/direct-dispatch-flow]] — End-to-end direct API dispatch lifecycle.
