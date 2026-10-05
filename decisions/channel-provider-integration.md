---
type: Architecture Decision
title: Channel Provider Integration and Secret Management
description: 'Specifically, outbound delivery requires: - An internal SMTP relay host for email distribution.'
resource: https://github.com/Nox-Demo-Org/kb-tw-customer-platform-notifications-hub/blob/main/decisions/channel-provider-integration.md
tags:
- notifications-hub
- decisions
generated:
  by: nox/gemini-3.7-flash
  at: '2026-10-05T12:42:34Z'
---

# Channel Provider Integration and Secret Management

## Status
Accepted

## Context
`notifications-hub` serves as the centralized outbound communication engine for Tidewell Mutual, dispatching customer communications over Email and SMS channels. To maintain operational separation and clean delivery pipelines, the service requires standard provider configurations and secret management workflows for outbound integrations. 

Specifically, outbound delivery requires:
- An internal SMTP relay host for email distribution.
- A standardized sender identity (`from` address) for customer trust and deliverability.
- API credentials for the external or downstream SMS gateway.
- A secure mechanism to supply and rotate channel credentials across staging and production environments without modifying application routing or rendering logic.

## Decision
1. **Centralized Provider Configuration**: Isolate all channel transport parameters within `src/channels/config.ts` (see [[entities/channel-configuration]]), decouped from template rendering and message handlers ([[entities/message-handler]]).
2. **Email Channel Parameters**: Configure email routing via `EMAIL_PROVIDER`:
   - `host`: `smtp.mail.internal`
   - `from`: `hello@tidewell.example`
3. **SMS Channel Authentication**: Authenticate SMS gateway interactions using an API key (`SMS_API_KEY`).
4. **Secret Management Policy**: In production, channel credentials and API keys must be retrieved dynamically from Google Cloud Secret Manager rather than hardcoded in the codebase.
5. **Coupling with Template Routing**: Channel configurations directly back the delivery channels assigned in the template registry (see [[decisions/template-routing-strategy]] and [[summaries/templates-and-channels]]).

## Consequences
### Positive
- Provider connection parameters and sender identities are maintained in a single location, allowing straightforward updates if internal relay hosts or sender identities change.
- Transport concerns remain cleanly separated from pub/sub subscription mapping ([[entities/subscription-registry]]) and direct API dispatch flows ([[concepts/direct-dispatch-flow]]).

### Negative & Known Technical Debt
- A static fallback development key (`twsms_9f3b7c21d6e84a0b95c2f1e7a8d4c6b2`) was committed to `src/channels/config.ts` (marked with `TODO(remove)`). This static secret must be removed and fully transitioned to environment-based secret resolution via Google Cloud Secret Manager.
