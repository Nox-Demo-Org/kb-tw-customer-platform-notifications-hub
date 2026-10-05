---
okf_version: '0.2'
title: notifications-hub
description: notifications-hub is a central communications service for Tidewell Mutual that handles outgoing customer notifications across multiple channels (Email and SMS).
generated:
  at: '2026-10-05T12:42:34Z'
---

# notifications-hub

`notifications-hub` is a central communications service for Tidewell Mutual that handles outgoing customer notifications across multiple channels (Email and SMS).

### Main Components
- **REST Ingestion (`src/handlers/messages.ts`)**: Exposes `POST /v1/messages` for on-demand synchronous dispatch requested by other services (such as `policy-admin`, `claims-management`, and `billing-service`).
- **Pub/Sub Subscriptions (`src/subscriptions.ts`)**: Listens to domain events across Google Cloud Pub/Sub topics to trigger notification workflows automatically.
- **Template Catalog (`src/templates.ts`)**: Defines mapping of message templates to notification channels and default subject lines.
- **Channel Configuration (`src/channels/config.ts`)**: Configures communication provider gateways (SMTP email provider and SMS API credentials).

### Data Flow
1. **Event-Driven Messages**: Pub/Sub events (e.g., `policy.renewal.due`, `claims.claim.settled`, [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-instalment-due|billing-service (billing.instalment.due)]], [[ap:kb-tw-billing-billing-service/summaries/events-spec#billing-payment-missed|billing-service (billing.payment.missed)]], `payments.payout.sent`) arrive via subscription topics. `notifications-hub` maps the topic to a corresponding notification template.
2. **Direct API Dispatch**: Other services invoke `POST /v1/messages` with `to_customer_id`, `template`, and dynamic `data` payload.
3. **Rendering & Delivery**: The service resolves customer contact details on demand, renders the template, and sends the communication through the configured channel (Email host or SMS gateway).

### Key Architectural Decisions
- **Decoupled Event Subscriptions**: Domain events trigger asynchronous notifications without tight coupling between core domain services and messaging channels.
- **Multi-Channel Dispatching**: Centralized catalog associates each notification type with its designated delivery medium (e.g., missed payments and claim payouts via SMS; renewal notices, settlement notices, and instalment reminders via Email).
- **Asynchronous Acceptance**: API messages return HTTP 202 Accepted, decoupling request ingestion from provider dispatch.

<!-- okf:contents -->

## Contents

- [Concepts and flows](/concepts/index.md) — 2 pages. Flows, lifecycles and cross-cutting mechanisms.
- [Architecture decisions](/decisions/index.md) — 2 pages. One ADR per architecture decision the code or documents make evident.
- [Components and data models](/entities/index.md) — 3 pages. One page per significant component and core data model.
- [Interfaces and references](/summaries/index.md) — 2 pages. API, event and module references for the application.
- [Change log](/log.md) — every generation and sync, newest first.

<!-- /okf:contents -->
