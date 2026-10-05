# Architecture decisions

One ADR per architecture decision the code or documents make evident.

## Pages

- [Channel Provider Integration and Secret Management](/decisions/channel-provider-integration.md) — Specifically, outbound delivery requires: - An internal SMTP relay host for email distribution.
- [Static Channel-to-Template Mapping and Subject Lines](/decisions/template-routing-strategy.md) — To ensure consistent customer communication, upstream services (such as policy-admin, claims-management, and billing-service) do not specify delivery channels, transport mechanics, or notification subject lines directly.
