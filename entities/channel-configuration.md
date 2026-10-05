---
type: Component
title: Channel Configuration
description: The src/channels/config.ts module defines provider configurations, endpoints, and credentials required for outgoing message dispatch across communication channels (Email and SMS) in notifications-hub.
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/entities/channel-configuration.md
tags:
- notifications-hub
- entities
sources:
- resource: https://github.com/Nox-Demo-Org/notifications-hub/blob/HEAD/src/channels/config.ts
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T13:10:26Z'
---

<!-- anchor: src/channels/config.ts:L1-L4 -->

# Channel Configuration

The `src/channels/config.ts` module defines provider configurations, endpoints, and credentials required for outgoing message dispatch across communication channels (Email and SMS) in `notifications-hub`.

## Responsibilities

- **Email Provider Configuration**: Exposes host settings and default sender address (`from`) used when dispatching email templates (e.g., `renewal-notice`, `claim-settled`, `instalment-reminder`).
- **SMS Gateway Credentials**: Stores API credentials required to authenticate with the SMS gateway when delivering text messages (e.g., `missed-payment`, `payout-sent`).
- **Secret & Transport Definitions**: Provides the central integration constants used downstream during template delivery flows initiated by [[entities/message-handler]] and [[entities/subscription-registry]].

## Configuration Constants

Defined in `src/channels/config.ts`:

| Export | Type | Properties / Value | Description |
|---|---|---|---|
| `EMAIL_PROVIDER` | `object` | `{ host: "smtp.mail.internal", from: "hello@tidewell.example" }` | Internal SMTP host hostname and outgoing sender email address. |
| `SMS_API_KEY` | `string` | `"[NoX Shield withheld a secret]"` | Secret API key used for authenticating requests to the SMS gateway API. |

### Environment and Secret Management

In production environments, credentials such as `SMS_API_KEY` are intended to be sourced from Secret Manager rather than hardcoded configuration. The static key currently declared in `src/channels/config.ts` represents a development/local testing key (flagged with a `TODO(remove)`). For architectural context on secret isolation and gateway abstraction, see [[decisions/channel-provider-integration]].

## Dependencies

- **Downstream Consumers**:
  - [[concepts/direct-dispatch-flow]]: Uses channel configurations when resolving delivery pathways for synchronous API requests (`POST /v1/messages`).
  - [[concepts/event-driven-notifications]]: Relies on provider configuration to dispatch communications triggered by domain Pub/Sub events.
  - [[summaries/templates-and-channels]]: Template channel mappings (`email` vs `sms`) route to these specific provider configurations.
- **External Services**:
  - Internal SMTP Relay (`smtp.mail.internal`): Relays outgoing email traffic under the sender identity `hello@tidewell.example`.
  - SMS Provider API: Consumes `SMS_API_KEY` for authenticating SMS dispatches.
