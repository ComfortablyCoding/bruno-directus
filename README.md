# Directus API — Bruno Collection

A comprehensive [Bruno](https://usebruno.com) collection for the [Directus](https://directus.com) API, covering the REST API, GraphQL, and the real-time WebSocket API.

> **Compatibility:** these requests are aligned to **Directus v12.x**.

## Getting started

1. Open this folder as a collection in Bruno.
2. Pick an environment (top-right in Bruno) and set its variables:
   - `base_url` — your Directus URL (e.g. `http://localhost:8055`).
   - `access_token` — a static token, or leave the placeholder and log in (see below).
   - `refresh_token` — populated automatically after login; leave blank initially.
3. Run **rest → auth → login_user** with valid credentials.

## Authentication

Auth is handled automatically. Every request uses `auth: inherit`, and a collection-level **pre-request script** (see `opencollection.yml`) reads `access_token` from the active environment and attaches it as a `Bearer` token — except for WebSocket connections and the `/auth/login` and `/auth/refresh` routes, which never receive it. (Requests never declare `auth: none`; that would be misleading, since the script still attaches the token.)

The **login** and **refresh** requests each run a post-response script that saves the returned `access_token` and `refresh_token` back into the active environment. So the normal flow is:

1. Run `auth → login_user` (or `login_with_provider`).
2. Every other request is now authenticated automatically.
3. When the token expires, run `auth → refresh_token`.

You can also use a static token instead: set `access_token` in the environment and skip the login step. The `admin` environment ships with `access_token: "1234"` for a typical local dev setup.

## Environments

| Name         | base_url                             | Intended use                        |
| ------------ | ------------------------------------ | ----------------------------------- |
| `local`      | `http://localhost:8055`              | Local dev; log in to get a token    |
| `admin`      | `http://localhost:8055`              | Local dev with a static admin token |
| `public`     | `http://localhost:8055`              | Unauthenticated / public requests   |
| `production` | `https://your-directus-instance.com` | Point at a real instance            |

## Structure

- **rest/** — the REST API, grouped by resource (items, files, users, collections, fields, flows, …). Most resources expose query / query-by-id / create / update / delete plus bulk variants.
- **graphql/** — GraphQL queries, mutations, and introspection (user data via `/graphql`, system data via `/graphql/system`).
- **websockets/** — the real-time API (auth handshake, subscribe, ping/pong, unsubscribe).

Many requests include extra query parameters that are **disabled by default** — enable them in Bruno to shape the response. The set is endpoint-specific: list endpoints accept the full query system (`fields`, `filter`, `sort`, `limit`, `offset`, `search`, `meta`), while single-item reads only accept what the API actually honors (`fields`, and `version` for items). Each request's params reflect what its endpoint truly supports.

## Docs

Full API reference: https://directus.com/docs/api
