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

The `{{base_url}}` environment variable uses `http://` or `https://`. For WebSocket requests you must use `ws://` or `wss://` manually (e.g. `ws://localhost:8055/websocket`).

## Docs

https://directus.io/docs/guides/realtime/subscriptions
