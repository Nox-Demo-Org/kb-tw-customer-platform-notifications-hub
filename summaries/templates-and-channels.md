---
type: Interface Reference
title: Template and Channel Catalog
description: notifications-hub maintains a centralized registry of notification templates, their static delivery channels, and default subject lines in src/templates.ts.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/summaries/templates-and-channels.md
tags:
- notifications-hub
- summaries
sources:
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/templates.ts
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/channels/config.ts
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/README.md
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T13:10:26Z'
---

<!-- anchor: src/templates.ts:L1-L7 -->
<!-- anchor: src/channels/config.ts:L1-L4 -->
<!-- anchor: README.md:L1-L10 -->

# Template and Channel Catalog

`notifications-hub` maintains a centralized registry of notification templates, their static delivery channels, and default subject lines in `src/templates.ts`. It pairs this catalog with channel provider credentials defined in `src/channels/config.ts`.

---

## Template Catalog

The `TEMPLATES` constant in `src/templates.ts` defines the mapping between template identifiers, their delivery channels (`email` or `sms`), and their subject lines.

| Template Key | Channel | Subject Line | Typical Trigger / Source |
| :--- | :--- | :--- | :--- |
| `renewal-notice` | `email` | `"Your Tidewell policy renews soon"` | `policy.renewal.due` event / [[entities/subscription-registry]] |
| `claim-settled` | `email` | `"Your claim is settled"` | `claims.claim.settled` event / [[entities/subscription-registry]] |
| `instalment-reminder` | `email` | `"Your next payment"` | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing.instalment.due]] event / [[entities/subscription-registry]] |
| `missed-payment` | `sms` | `"We couldn't take your payment"` | [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing.payment.missed]] event / [[entities/subscription-registry]] |
| `payout-sent` | `sms` | `"We've paid your claim"` | `payments.payout.sent` event / [[entities/subscription-registry]] |

Templates can also be invoked on demand via direct REST requests using `POST /v1/messages` (see [[entities/message-handler]] and [[concepts/direct-dispatch-flow]]).

---

## Delivery Channels and Provider Configuration

Channel connection parameters and gateway configuration are defined in `src/channels/config.ts` (see [[entities/channel-configuration]] and [[decisions/channel-provider-integration]]).

### Email Configuration
Email templates (`renewal-notice`, `claim-settled`, `instalment-reminder`) are routed through the internal SMTP relay configured via `EMAIL_PROVIDER`:
- **Host**: `smtp.mail.internal`
- **Sender (`from`)**: `hello@tidewell.example`

### SMS Configuration
SMS templates (`missed-payment`, `payout-sent`) are delivered using the SMS provider API credentials:
- **API Key**: `SMS_API_KEY` (`"[NoX Shield withheld a secret]"`)
  - *Note*: In production environments, channel provider secrets are retrieved from Secret Manager.

---

## Related Documentation
- [[summaries/api-spec]] - Complete REST API and Pub/Sub event specification.
- [[decisions/template-routing-strategy]] - Architectural decision regarding static channel-to-template mapping and subject lines.
- [[concepts/event-driven-notifications]] - Lifecycle and workflow for automated domain event messaging.
