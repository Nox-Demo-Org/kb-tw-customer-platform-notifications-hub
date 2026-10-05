---
type: Concept
title: Direct Dispatch Flow
description: The direct dispatch flow provides a synchronous HTTP entrypoint for core Tidewell domain services (such as policy administration, claims management, and billing services) to request on-demand customer notifications.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/concepts/direct-dispatch-flow.md
tags:
- notifications-hub
- concepts
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:34Z'
---

# Direct Dispatch Flow

The direct dispatch flow provides a synchronous HTTP entrypoint for core Tidewell domain services (such as policy administration, claims management, and billing services) to request on-demand customer notifications. 

Unlike the asynchronous Pub/Sub pipeline described in [[concepts/event-driven-notifications]], direct dispatch allows client services to trigger specific communications explicitly with customized template variables.

---

## Endpoint Contract & Ingestion

Direct notification requests enter through the REST handler defined in [[entities/message-handler]] (`src/handlers/messages.ts`).

### Request Signature
The `postMessage` function processes POST requests directed to `POST /v1/messages`:

```typescript
export async function postMessage(body: { 
  to_customer_id: string; 
  template: string; 
  data: Record<string, unknown> 
}): Promise<{ status: number }>
```

- **`to_customer_id`**: The unique identifier of the recipient customer used to resolve recipient contact information.
- **`template`**: The template key matching one of the registered definitions in `src/templates.ts` (see [[summaries/templates-and-channels]]).
- **`data`**: A key-value payload (`Record<string, unknown>`) injected dynamically into the rendered message body.

### Asynchronous Acceptance
The endpoint immediately responds with `{ status: 202 }` (HTTP 202 Accepted), decoupling the caller from the latency of downstream rendering, customer resolution, and channel provider transmission. See [[summaries/api-spec]] for API interface details.

---

## Lifecycle & Pipeline Stages

```
Client Service (HTTP POST /v1/messages)
               │
               ▼
   [1] Ingest & Accept (202)
   (src/handlers/messages.ts)
               │
               ▼
   [2] Customer Lookup On-Demand
   (Resolves email/phone for to_customer_id)
               │
               ▼
   [3] Template Resolution & Rendering
   (src/templates.ts)
               │
               ▼
   [4] Channel Dispatch
   (src/channels/config.ts: Email vs SMS)
```

### 1. Ingestion and Acceptance
`postMessage` validates the incoming JSON payload containing `to_customer_id`, `template`, and `data`, returning `{ status: 202 }`.

### 2. On-Demand Customer Lookup
The handler resolves the target customer's contact details (such as email address or mobile phone number) on demand using the provided `to_customer_id`.

### 3. Template Resolution and Rendering
The system references the template catalog in `src/templates.ts` (documented in [[decisions/template-routing-strategy]]) to determine:
- **Delivery Channel**: Whether the notification must be sent via `email` or `sms`.
- **Subject Line**: Default subject text associated with the template.
- **Body Rendering**: Merges the dynamic `data` dictionary into the template body.

Supported template definitions from `TEMPLATES`:
* `"renewal-notice"` (`email`): *"Your Tidewell policy renews soon"*
* `"claim-settled"` (`email`): *"Your claim is settled"*
* `"instalment-reminder"` (`email`): *"Your next payment"*
* `"missed-payment"` (`sms`): *"We couldn't take your payment"*
* `"payout-sent"` (`sms`): *"We've paid your claim"*

### 4. Channel Provider Dispatch
Once rendered, the message payload is dispatched to the corresponding communication provider configured in [[entities/channel-configuration]] (`src/channels/config.ts`):
- **Email Route**: Sent via `EMAIL_PROVIDER` (`host: "smtp.mail.internal"`, `from: "hello@tidewell.example"`).
- **SMS Route**: Dispatched to the SMS gateway authenticated using `SMS_API_KEY`.

See [[decisions/channel-provider-integration]] for architectural details regarding provider credential management.

---

## Related Pages

- [[entities/message-handler]] – Implementation details of `postMessage`
- [[summaries/templates-and-channels]] – Registered templates, channels, and subject lines
- [[concepts/event-driven-notifications]] – Alternative notification triggering via Pub/Sub topics
- [[entities/channel-configuration]] – Host configuration and provider credentials
