---
type: Architecture Decision
title: Static Channel-to-Template Mapping and Subject Lines
description: To ensure consistent customer communication, upstream services (such as policy-admin, claims-management, and billing-service) do not specify delivery channels, transport mechanics, or notification subject lines directly.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/decisions/template-routing-strategy.md
tags:
- notifications-hub
- decisions
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:34Z'
---

# Static Channel-to-Template Mapping and Subject Lines

## Status
Accepted

## Context
The `notifications-hub` service acts as the centralized outbound communications dispatcher across Tidewell Mutual's distributed architecture. It receives triggers from two distinct entry points:
1. Asynchronous domain events consumed from Google Cloud Pub/Sub topics via [[entities/subscription-registry|src/subscriptions.ts]] (see [[concepts/event-driven-notifications]]).
2. Synchronous REST ingestion requests through `POST /v1/messages` in [[entities/message-handler|src/handlers/messages.ts]] (see [[concepts/direct-dispatch-flow]]).

To ensure consistent customer communication, upstream services (such as `policy-admin`, `claims-management`, and [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service]]) do not specify delivery channels, transport mechanics, or notification subject lines directly. Instead, incoming events or API requests supply domain payloads and reference abstract template identifiers or domain event topics.

The architecture required a deterministic, lightweight strategy to resolve:
- Which communication channel (`email` vs. `sms`) handles each notification.
- The standard subject line to apply to the outgoing communication.
- The mapping between ingested domain event topics and target templates.

## Decision
We define a centralized, static template registry in `src/templates.ts` using TypeScript's `as const` assertion, paired with an explicit topic-to-template map in `src/subscriptions.ts`.

### 1. Static Template Registry (`src/templates.ts`)
Each template identifier is mapped statically to a designated delivery channel and standard subject text:

```typescript
export const TEMPLATES = {
  "renewal-notice": { channel: "email", subject: "Your Tidewell policy renews soon" },
  "claim-settled": { channel: "email", subject: "Your claim is settled" },
  "instalment-reminder": { channel: "email", subject: "Your next payment" },
  "missed-payment": { channel: "sms", subject: "We couldn't take your payment" },
  "payout-sent": { channel: "sms", subject: "We've paid your claim" },
} as const;
```

For a comprehensive catalog of these templates, see [[summaries/templates-and-channels]].

### 2. Pub/Sub Topic Mapping (`src/subscriptions.ts`)
Pub/Sub event topics map directly to template keys via the `SUBSCRIPTIONS` dictionary:

| Pub/Sub Event Topic | Template Key | Channel | Default Subject Line |
| :--- | :--- | :--- | :--- |
| `policy.renewal.due` | `renewal-notice` | `email` | `"Your Tidewell policy renews soon"` |
| `claims.claim.settled` | `claim-settled` | `email` | `"Your claim is settled"` |
| [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]] (via [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service]]) | `instalment-reminder` | `email` | `"Your next payment"` |
| [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]] (via [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service]]) | `missed-payment` | `sms` | `"We couldn't take your payment"` |
| `payments.payout.sent` | `payout-sent` | `sms` | `"We've paid your claim"` |

### 3. Channel Selection Logic
Urgent financial notifications (`missed-payment`, `payout-sent`) are assigned exclusively to `sms`. Informational and documentation-heavy updates (`renewal-notice`, `claim-settled`, `instalment-reminder`) are assigned exclusively to `email`. Channel transport configuration and credentials are managed separately in [[entities/channel-configuration|src/channels/config.ts]] (see [[decisions/channel-provider-integration]]).

## Consequences

### Positive
- **Type Safety and Autocompletion**: Using `as const` provides strict compile-time validation for template identifiers, channels, and subject headers across [[entities/message-handler|message ingestion]] and [[entities/subscription-registry|subscription handling]].
- **Decoupled Upstream Services**: Domain microservices emit semantic events without needing awareness of communication channels, delivery routes, or copy management.
- **Zero Runtime Lookup Overhead**: Template definitions and routing paths are bundled in code, eliminating database or external CMS query latency during message dispatch.

### Negative / Trade-offs
- **Deployment Requirement for Copy Changes**: Updating a subject line, adding a template, or changing a delivery channel from SMS to Email requires a code update and service deployment.
- **Single Channel per Template**: Templates are tied to a single static channel (`email` or `sms`); dynamic channel fallback or per-recipient channel preference selection is not supported in this static catalog structure.
