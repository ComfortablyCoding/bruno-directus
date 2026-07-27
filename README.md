# Directus API — Bruno Collection

A comprehensive [Bruno](https://usebruno.com) collection for the [Directus](https://directus.com) API, covering the REST API, GraphQL, and the real-time WebSocket API.

> **Compatibility:** these requests are aligned to **Directus v12.x**.

## Setup

1. Install [Bruno](https://www.usebruno.com/downloads) if you haven't already.
2. Clone or download this repository.
3. In Bruno, click **Open Collection** (or `Ctrl/Cmd+O`) and select this folder.
4. In the top-right environment dropdown, pick an environment (see [Environments](#environments)).
5. Set that environment's variables for your Directus instance:
   - `base_url` — your Directus URL (e.g. `http://localhost:8055`).
   - `access_token` — a static token, or leave the placeholder and log in (see below).
   - `refresh_token` — populated automatically after login; leave blank initially.
6. Run **rest → auth → login_user** with valid credentials to authenticate.

## Authentication

Auth is handled for you. A collection-level **pre-request script** (see `opencollection.yml`) reads `access_token` from the active environment and attaches it as an `Authorization: Bearer` header on every request — except WebSocket connections and the `/auth/login` and `/auth/refresh` routes, which never receive it. If `access_token` is empty (like the `public` environment), no header is sent.

The **login** and **refresh** requests each run a post-response script that saves the returned `access_token` and `refresh_token` back into the active environment. So the normal flow is:

1. Run `auth → login_user` (or `login_with_provider`).
2. Every other request is now authenticated automatically.
3. When the token expires, run `auth → refresh_token` to refresh.

**Using a static token instead:** set `access_token` in the environment and skip the login step. The `admin` environment ships with `access_token: "1234"` for a typical local dev setup.

**Testing with a different user's token:** override the auth on any individual request — set its auth to `bearer` and supply that user's token. A request that sets its own `Authorization` header takes precedence over the collection script.

## Environments

| Name         | base_url                             | Intended use                        |
| ------------ | ------------------------------------ | ----------------------------------- |
| `local`      | `http://localhost:8055`              | Local dev; log in to get a token    |
| `admin`      | `http://localhost:8055`              | Local dev with a static admin token |
| `public`     | `http://localhost:8055`              | Unauthenticated / public requests   |
| `production` | `https://your-directus-instance.com` | Point at a real instance            |

### Variables

| Variable        | Description                                                        |
| --------------- | ------------------------------------------------------------------ |
| `base_url`      | Directus instance URL (e.g. `http://localhost:8055`).              |
| `access_token`  | Static token, or captured automatically on login. Empty = no auth. |
| `refresh_token` | Captured on login; used by `refresh_token` to rotate the pair.     |

## Structure

- **rest/** — the REST API, grouped by resource (items, files, users, collections, fields, flows, …). Most resources expose query / query-by-id / create / update / delete plus bulk variants.
- **graphql/** — GraphQL queries, mutations, and introspection (user data via `/graphql`, system data via `/graphql/system`). See [`graphql/README.md`](graphql/README.md).
- **websockets/** — the real-time API (auth handshake, subscribe, ping/pong, unsubscribe). See [`websockets/README.md`](websockets/README.md).

Many requests include extra query parameters that are **disabled by default** — enable them in Bruno to shape the response. The set is endpoint-specific: list endpoints accept the full query system (`fields`, `filter`, `sort`, `limit`, `offset`, `search`, `meta`), while single-item reads only accept what the API actually honors (`fields`, and `version` for items). Each request's params reflect what its endpoint truly supports.

## Other tooling

Directus serves its own OpenAPI spec at `/server/specs/oas` (see **rest → server → get_oas_spec**) — use that for Postman, Insomnia, or any OpenAPI-compatible tool.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for conventions and guidelines.

## Docs

Full API reference: https://directus.com/docs/api
