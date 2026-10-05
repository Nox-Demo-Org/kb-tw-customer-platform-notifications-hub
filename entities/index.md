# Components and data models

One page per significant component and core data model.

## Pages

- [Channel Configuration](/entities/channel-configuration.md) — The src/channels/config.ts module defines provider configurations, endpoints, and credentials required for outgoing message dispatch across communication channels (Email and SMS) in notifications-hub.
- [Message Handler](/entities/message-handler.md) — The message handler, implemented in src/handlers/messages.ts, provides the REST ingestion endpoint for synchronous, direct notification requests across the Tidewell Mutual platform.
- [Subscription Registry](/entities/subscription-registry.md) — The Subscription Registry module (src/subscriptions.ts) defines the static dictionary that maps incoming Google Cloud Pub/Sub domain event topics to their corresponding notification template identifiers.
