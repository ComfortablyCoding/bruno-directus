# Directus API — Bruno Collection

A comprehensive [Bruno](https://usebruno.com) collection for the [Directus](https://directus.com) API, covering the REST API, GraphQL, and the real-time WebSocket API.

> **Compatibility:** these requests are aligned to **Directus v12.x**.

## Setup

1. Install [Bruno](https://www.usebruno.com/downloads) if you haven't already.
2. Clone or download this repository.
3. In Bruno, click **Open Collection** (or `Ctrl/Cmd+O`) and select this folder.
4. In the top-right environment dropdown, pick an environment (see [Environments](#environments)).
5. Set that environment's variables for your Directus instance:
   - `base_url` — your Directus URL (e.g. `http://localhost:8055`), used by every request — including
     WebSockets, which Bruno connects over `ws://`/`wss://`.
   - `access_token` — leave empty and log in, or set a static token (see below).
   - `refresh_token` — leave empty; login captures it as a runtime variable.
6. Run **rest → auth → login_user** with valid credentials to authenticate.

## Authentication

The collection uses **Bearer auth** with `{{access_token}}`, and every request that needs a token inherits it. Requests that are public by design — login, refresh, logout, password reset, registration, invite acceptance, share access, first-run setup — set no auth. Directus rejects any request that carries an expired token, so sending one to those routes would break logout and password reset exactly when you need them.

The REST login and refresh requests (and the GraphQL `auth_login` / `auth_refresh` mutations) are picked up by a collection-level after-response script that stores the returned `access_token` and `refresh_token` as **runtime variables**. Runtime variables last for the session, are never written to disk, and take precedence over environment variables — so logging in does not leave live credentials in the tracked `environments/*.yml` files. The normal flow is:

1. Run `auth → login_user` (or `login_with_provider`).
2. Every other request is now authenticated automatically.
3. When the token expires, run `auth → refresh_token` to refresh.
4. Run `auth → logout_user` to invalidate the refresh token and clear the captured tokens.

Captured tokens are **bound to the `base_url` that issued them**: switch to an environment with a different `base_url` and they are dropped before the next request, so a token is never sent to a server that didn't issue it. Because they are session-scoped, you log in again after restarting Bruno.

**Using a static token instead:** set `access_token` in the environment and skip the login step. If your instance sets `ADMIN_TOKEN`, use that value. The environment files are committed to the repository, so never commit a real token.

**Testing as another user or as the public:** change that request's auth — `bearer` with the other user's token, or no auth to make an unauthenticated request.

## Environments

| Name         | base_url                             | Intended use             |
| ------------ | ------------------------------------ | ------------------------ |
| `local`      | `http://localhost:8055`              | A local Directus         |
| `production` | `https://your-directus-instance.com` | Point at a real instance |

Both start with an empty `access_token`, which Directus treats as unauthenticated. Add an environment per instance you work with; the schema migration requests in `rest/schema` move a schema from one environment's instance to another's.

### Variables

| Variable        | Description                                                                             |
| --------------- | --------------------------------------------------------------------------------------- |
| `base_url`      | Directus instance URL for REST, GraphQL, and WebSockets (e.g. `http://localhost:8055`). |
| `access_token`  | Static fallback token. Empty = no auth. Login overrides it with a runtime variable.     |
| `refresh_token` | Leave blank — captured on login as a runtime variable and rotated on refresh.           |

Some requests also capture runtime variables for the next step of a workflow: `compare_version` → `promote_version` (`main_hash`), `get_schema_snapshot` → `diff_schema` → `apply_schema` (`schema_snapshot`, `schema_diff`), and `create_tus_upload` → `upload_tus_chunk` (`tus_upload_id`).

## Structure

- **rest/** — the REST API, one folder per resource (items, files, users, access, collections, fields, flows, deployments, …). Most resources expose query / query-by-id / create / update / delete plus bulk variants.
- **graphql/** — GraphQL queries, mutations, auth, and introspection (user data via `/graphql`, system data via `/graphql/system`).
- **websockets/** — the real-time API: collection subscriptions, item actions, GraphQL subscriptions, and the logs stream. Each request is one connection with its messages listed in order.

Each folder's documentation (its **Docs** tab in Bruno) describes its requests and any setup they need. The examples use an `articles` collection, which **rest → collections → create_collection** creates.

Many requests include extra query parameters that are **disabled by default** — enable them in Bruno to shape the response. The set is endpoint-specific: list endpoints accept the query system (`fields`, `filter`, `sort`, `limit`, `offset`, `search`, `meta`, and on items also `deep`, `aggregate`, `groupBy`, `alias`, `page`, `backlink`, `export`), while single-item reads only accept what the API actually honors. Each request's params reflect what its endpoint truly supports, and every example value names fields that exist on that collection.

## Other tooling

Directus serves its own OpenAPI spec at `/server/specs/oas` (see **rest → server → get_oas_spec**) — use that for Postman, Insomnia, or any OpenAPI-compatible tool.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for conventions and guidelines.

## Docs

Full API reference: https://directus.com/docs/api
