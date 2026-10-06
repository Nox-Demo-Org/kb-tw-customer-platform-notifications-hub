---
mission: NOX-5
title: 'Secure SMS credential management'
role: product
status: ai_drafted
version: 1
author: NoX
ai_drafted: true
---

# Product spec: Secure SMS credential management

## Goal
Keep existing outbound SMS notification delivery behaviour exactly as it is while migrating SMS provider credentials from static configuration to dynamic secret management.

## User stories
None: this is an internal security and credential storage update with no direct end-user facing functionality changes.

## Acceptance criteria
- **AC-1: Direct API SMS dispatch regression test**
  - **Given** an upstream system submits a valid SMS notification request to the messages API (`POST /v1/messages`) [[kb:notifications-hub/brief]]
  - **When** the notification service processes the message
  - **Then** the request returns a `202 Accepted` status and the SMS message is dispatched to the recipient exactly as before.

- **AC-2: Event-driven SMS dispatch regression test**
  - **Given** an upstream domain service publishes an event mapped to an SMS template (such as billing or policy notifications) [[kb:notifications-hub/concepts/event-driven-notifications]]
  - **When** the notification service consumes and renders the event
  - **Then** the SMS notification is successfully dispatched to the customer without disruption.

## Edge cases
- **Unavailable or invalid credentials *(from the map)***: If SMS credentials fail to resolve from the secret store, outbound SMS delivery fails gracefully and logs the operational failure without exposing secret details in responses.
- **Malformed SMS payload or recipient number *(from the map)***: Existing payload validation rules reject invalid requests as before.

## Out of scope
- Any changes to customer-facing SMS message content, templates, or subject mapping [[kb:notifications-hub/decisions/template-routing-strategy]].
- Any changes to email delivery channels or internal mail routing.
- Any modifications to the external API contract for `POST /v1/messages`.

## Success metric
- Outbound SMS delivery failure rate: 0% regression from baseline *(suggestion)*, measured via messaging operational metrics over the 24 hours following deployment.

## Priority
P2 — Important security hygiene and compliance requirement to ensure credentials can be safely rotated in secret storage without code changes, though there is no direct customer-facing impact during normal operations.

## Verification checklist
- [ ] Direct SMS notifications sent via the messages API deliver successfully without regression (AC-1).
- [ ] Event-driven SMS notifications triggered by domain events deliver successfully without regression (AC-2).
