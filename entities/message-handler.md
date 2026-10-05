---
type: Component
title: Message Handler
description: The message handler, implemented in src/handlers/messages.ts, provides the REST ingestion endpoint for synchronous, direct notification requests across the Tidewell Mutual platform.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/entities/message-handler.md
tags:
- notifications-hub
- entities
sources:
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/handlers/messages.ts
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/README.md
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T13:10:26Z'
---

<!-- anchor: src/handlers/messages.ts:L1-L5 -->
<!-- anchor: README.md:L1-L10 -->

# Message Handler

The message handler, implemented in `src/handlers/messages.ts`, provides the REST ingestion endpoint for synchronous, direct notification requests across the Tidewell Mutual platform.

## Overview

While event-driven communications are processed through [[entities/subscription-registry|Pub/Sub subscriptions]], upstream services (such as `policy-admin`, `claims-management`, and `billing-service`) invoke the direct REST API when they require immediate, on-demand dispatch of specific notifications to customers.

The endpoint executes the [[concepts/direct-dispatch-flow|Direct Dispatch Flow]] and returns an HTTP `202 Accepted` status to decouple the ingestion request from provider delivery.

## Interface Contract

### HTTP Route
- **Method**: `POST`
- **Path**: `/v1/messages`
- **Response**: `202 Accepted` (`{ status: 202 }`)

### Function Signature

```typescript
export async function postMessage(body: {
  to_customer_id: string;
  template: string;
  data: Record<string, unknown>;
}): Promise<{ status: number }>
```

### Parameters
- `to_customer_id` (`string`): The unique customer identifier whose contact details will be retrieved.
- `template` (`string`): The template identifier registered in the [[summaries/templates-and-channels|Template Catalog]] (e.g., in `src/templates.ts`).
- `data` (`Record<string, unknown>`): Key-value payload containing contextual variables used to render the template.

## Responsibilities

1. **Request Ingestion**: Exposes `postMessage` for `POST /v1/messages` to receive structured message payloads from calling services.
2. **Contact Lookup**: Retrieves recipient customer contact details on demand using the provided `to_customer_id`.
3. **Template Resolution & Rendering**: Merges the provided dynamic `data` payload with the specified `template` defined in `src/templates.ts`.
4. **Channel Dispatch**: Routes the rendered communication to the appropriate delivery medium (Email or SMS) according to [[decisions/template-routing-strategy|Template Routing Strategy]].
5. **Asynchronous Acknowledgment**: Responds with an HTTP `202 Accepted` status code.

## Dependencies

- **Template Registry (`src/templates.ts`)**: Lookups for template definition, channel association, and default subject lines via [[summaries/templates-and-channels|Templates and Channels]].
- **Channel Delivery Providers (`src/channels/config.ts`)**: Dispatches the rendered message through email or SMS transports configured in [[entities/channel-configuration|Channel Configuration]].
- **Upstream Callers**:
  - `policy-admin`
  - `claims-management`
  - `billing-service`

## Related Documentation

- [[summaries/api-spec|API Specification]]
- [[concepts/direct-dispatch-flow|Direct Dispatch Flow]]
- [[decisions/channel-provider-integration|Channel Provider Integration]]
