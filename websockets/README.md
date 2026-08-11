# websockets

Real-time WebSocket API for subscribing to Directus collection events.

## Connection Flow

1. **Connect** to `ws://your-host/websocket` (or `wss://` for TLS)
2. **Authenticate** — send an `auth` message with your access token
3. **Subscribe** — send a `subscribe` message specifying the collection and event types
4. **Receive events** — the server pushes `create`, `update`, or `delete` payloads
5. **Keep alive** — respond to server `ping` messages with a `pong`
6. **Unsubscribe** — send an `unsubscribe` message with the subscription UID when done

## URL Note

WebSocket requests use `{{ws_url}}`, not `{{base_url}}` — the WebSocket protocol needs a `ws://`
or `wss://` scheme. Each environment defines both: `base_url` for REST and GraphQL, `ws_url` for
these requests. If you point `base_url` at a different instance, update `ws_url` to match.

Authentication is the `auth` handshake message below, not an `Authorization` header, so these
requests intentionally do not receive the collection's Bearer token.

## Docs

https://directus.com/docs/guides/realtime/subscriptions
