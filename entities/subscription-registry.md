---
type: Component
title: Subscription Registry
description: The Subscription Registry module (src/subscriptions.ts) defines the static dictionary that maps incoming Google Cloud Pub/Sub domain event topics to their corresponding notification template identifiers.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/entities/subscription-registry.md
tags:
- notifications-hub
- entities
sources:
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/subscriptions.ts
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/README.md
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/package.json
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T13:10:26Z'
---

<!-- anchor: src/subscriptions.ts:L1-L8 -->
<!-- anchor: README.md:L1-L10 -->
<!-- anchor: package.json:L1-L1 -->

# Subscription Registry

The `Subscription Registry` module (`src/subscriptions.ts`) defines the static dictionary that maps incoming Google Cloud Pub/Sub domain event topics to their corresponding notification template identifiers. It serves as the primary routing table for the [[concepts/event-driven-notifications|event-driven notification flow]].

## Subscription Mapping

The subscription registry exports the `SUBSCRIPTIONS` map, which pairs domain event topics published by upstream Tidewell services with template names defined in the [[summaries/templates-and-channels|Template Catalog]]:

```typescript
export const SUBSCRIPTIONS: Record<string, string> = {
  "policy.renewal.due": "renewal-notice",
  "claims.claim.settled": "claim-settled",
  "billing.instalment.due": "instalment-reminder",
  "billing.payment.missed": "missed-payment",
  "payments.payout.sent": "payout-sent",
};
```

### Event-to-Template Translation

| Pub/Sub Event Topic | Template Identifier | Upstream Domain / Service Context | Delivery Strategy |
| :--- | :--- | :--- | :--- |
| `policy.renewal.due` | `renewal-notice` | Policy Administration | Email notice via [[summaries/templates-and-channels|Template Catalog]] |
| `claims.claim.settled` | `claim-settled` | Claims Management | Email notice via [[summaries/templates-and-channels|Template Catalog]] |
| [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] | `instalment-reminder` | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] | Email reminder via [[summaries/templates-and-channels|Template Catalog]] |
| [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] | `missed-payment` | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] | SMS alert via [[summaries/templates-and-channels|Template Catalog]] |
| `payments.payout.sent` | `payout-sent` | Payments / Claims Disbursement | SMS alert via [[summaries/templates-and-channels|Template Catalog]] |

> **Note on Ignored Events**: `notifications-hub` explicitly does not subscribe to or process `claims.handler.assigned`. Consequently, customers receive no notification message when a claim handler is assigned.

## Responsibilities

- **Topic Registration**: Define the exact set of Google Cloud Pub/Sub event topics that `notifications-hub` is configured to ingest.
- **Event Translation**: Map consumed event topic names directly to template identifiers before routing to the rendering and dispatch pipeline.
- **Routing Governance**: Support the [[decisions/template-routing-strategy|template routing strategy]] by maintaining a declarative link between domain events and user-facing communications.

## Dependencies

- **`@google-cloud/pubsub`**: Underlying client library (`^4.4.0`) used to consume messages from Google Cloud Pub/Sub topics.
- **[[summaries/templates-and-channels|Template Catalog (`src/templates.ts`)]]**: Resolves template keys mapped in `SUBSCRIPTIONS` to specific communication channels (Email or SMS) and subject lines.
- **[[summaries/api-spec|API and Event Specifications]]**: Documented contract for Pub/Sub subscriptions consumed by `notifications-hub`.
