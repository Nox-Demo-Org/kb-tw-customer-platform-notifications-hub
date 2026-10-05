---
type: Concept
title: Event-Driven Notifications
description: notifications-hub operates as an asynchronous, event-driven communication processor for Tidewell Mutual.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/concepts/event-driven-notifications.md
tags:
- notifications-hub
- concepts
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:34Z'
---

# Event-Driven Notifications

`notifications-hub` operates as an asynchronous, event-driven communication processor for Tidewell Mutual. By subscribing to domain events published across Google Cloud Pub/Sub, the hub triggers customer communications automatically in response to core insurance lifecycle changes without tight coupling between domain services and messaging channels.

For synchronous, on-demand dispatch requested directly over HTTP, see [[concepts/direct-dispatch-flow]] and [[entities/message-handler]].

---

## Pub/Sub Architecture & Conventions

All asynchronous inter-service communications at Tidewell Mutual use Google Cloud Pub/Sub. Messages adhere to standard enterprise formatting:
- **Topic Naming**: Formatted as `<domain>.<entity>.<past-tense-verb>` (e.g., `claims.claim.settled`, `policy.renewal.due`).
- **Subscription Naming**: Formatted as `<consumer>-<topic-short-name>`.
- **Payload Standards**: JSON formatted with `snake_case` keys, string identifiers, and currency represented as integer amounts in pence.

When an event arrives from a subscribed topic, `notifications-hub` matches the topic name against its [[entities/subscription-registry|subscription registry]] (`src/subscriptions.ts`) to determine the target notification template.

---

## Subscribed Topics & Template Translation

The mapping between incoming Google Cloud Pub/Sub topics and template identifiers is defined statically in `src/subscriptions.ts`:

| Domain / Publisher | Topic Name | Triggered Template | Default Channel |
| --- | --- | --- | --- |
| **Policy** (`policy-admin`) | `policy.renewal.due` | `renewal-notice` | Email |
| **Claims** (`claims-management`) | `claims.claim.settled` | `claim-settled` | Email |
| **Billing** (`billing-service`) | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] | `instalment-reminder` | Email |
| **Billing** (`billing-service`) | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] | `missed-payment` | SMS |
| **Payments** (`payments-gateway`) | `payments.payout.sent` | `payout-sent` | SMS |

For external billing event definitions, see [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] and [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]]. Template configurations and channel assignments are detailed in [[summaries/templates-and-channels]] and [[decisions/template-routing-strategy]].

---

## End-to-End Lifecycle

```
[ Domain Service ]
       │
       │ (Publish Event)
       ▼
[ Google Cloud Pub/Sub Topic ]
       │
       │ (Pull / Push Subscription)
       ▼
[ notifications-hub: src/subscriptions.ts ]
       │
       │ (Translate Topic to Template ID)
       ▼
[ Template & Channel Resolution: src/templates.ts ]
       │
       │ (Fetch Recipient & Render Body)
       ▼
[ Channel Delivery Gateway: src/channels/config.ts ]
       │
       ├──► Email (SMTP Host)
       └──► SMS (SMS API Gateway)
```

1. **Domain Event Publication**: An upstream domain service completes a business process and publishes a domain event to Google Cloud Pub/Sub:
   - `policy-admin` emits `policy.renewal.due` 21 days before a policy's renewal date.
   - `billing-service` emits [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] prior to scheduled collections, or [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] if collection fails.
   - `claims-management` emits `claims.claim.settled` when a claim settlement completes.
   - `payments-gateway` emits `payments.payout.sent` after executing a claim settlement payout.
2. **Event Consumption**: `notifications-hub` receives the message via its configured subscription.
3. **Template Resolution**: The hub inspects the topic name against the `SUBSCRIPTIONS` map in `src/subscriptions.ts` to select the associated template ID (see [[entities/subscription-registry]]).
4. **Channel Routing**: The template ID resolves to its delivery channel (`email` or `sms`) and subject headers as defined in [[summaries/templates-and-channels]].
5. **Dispatch Execution**: The notification is formatted with payload data and dispatched using the provider credentials managed in [[entities/channel-configuration]] and [[decisions/channel-provider-integration]].

---

## Omitted Domain Events

Certain domain events are intentionally not consumed by `notifications-hub`:
- **`claims.handler.assigned`**: Published by `claims-management` when a claims handler picks up a claim. `notifications-hub` **does not** listen to `claims.handler.assigned`, meaning customers receive no automated SMS or email notification at this stage in the claims lifecycle.

---

## Related Documentation
- [[entities/subscription-registry]] – Code-level reference for `SUBSCRIPTIONS` in `src/subscriptions.ts`
- [[summaries/api-spec]] – Pub/Sub event interfaces and REST contracts
- [[summaries/templates-and-channels]] – Channel mappings, template keys, and default subject lines
- [[decisions/template-routing-strategy]] – Architectural decision regarding static topic-to-template mappings
