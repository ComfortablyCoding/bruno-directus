# Contributing

## Format

This collection uses the [OpenCollection](https://spec.opencollection.com) `1.0.0` format — YAML
files, one per request, with `opencollection.yml` at the root. The older Bruno-specific `.bru`
format is no longer used.

The authoritative schema is
[`opencollection.schema.json`](https://github.com/usebruno/opencollection/blob/main/packages/oc-schema/src/opencollection.schema.json).
Most object types set `additionalProperties: false`, so an unexpected or misplaced key makes the
file invalid rather than being ignored. When in doubt, check the schema.

## Collection Structure

| Folder       | Description                                 |
| ------------ | ------------------------------------------- |
| `rest`       | REST API endpoints, one folder per resource |
| `graphql`    | GraphQL queries and mutations               |
| `websockets` | Real-time WebSocket subscriptions           |

Each resource folder under `rest/` contains:

- `folder.yml` — folder metadata
- `README.md` — brief description, list of operations, docs link
- Individual `.yml` request files

## Naming Conventions

### Folders

- Lowercase, underscore-separated: `content_versions/`, `custom_translations/`

### Files

- Pattern: `{action}_{scope}_{resource}.yml`
- Actions: `query`, `create`, `update`, `delete`
- Scope: omitted for single, `_bulk` suffix for multiple
- Examples:
  - `query_items.yml` — list/filter items
  - `query_item_by_id.yml` — single item by ID
  - `create_item.yml` — create one
  - `create_items_bulk.yml` — create many
  - `delete_items_bulk.yml` — delete many
- Special operations use descriptive names: `trigger_flow_post.yml`, `promote_version.yml`, `get_current_user.yml`

### The `info.name` field

- Must match the filename without the `.yml` extension
- Lowercase, underscore-separated: `name: query_items`

## Request File Structure

An HTTP request has four top-level keys, in this order: `info`, `http`, `settings`, `docs` —
plus `runtime` between `http` and `settings` when the request needs a script.

```yaml
info:
  name: query_items
  type: http
  seq: 1

http:
  method: GET
  url: "{{base_url}}/items/:collection"
  params:
    - name: fields
      value: "id,status,date_created"
      type: query
      disabled: true
    - name: collection
      value: articles
      type: path
  auth: inherit

settings:
  encodeUrl: true

docs: |-
  Retrieve a list of items from the specified collection.
  https://directus.com/docs/api/items
```

- `info.type` is `http`, `graphql`, `websocket`, or `folder`
- `auth: inherit` on every request, so the collection-level script supplies the token
- `settings: { encodeUrl: true }` on every HTTP and GraphQL request. Only deviate with a
  reason — `rest/auth/sso_login_redirect.yml` sets `followRedirects: false` because the point
  of the request is to inspect the redirect. Prefer not to restate defaults such as
  `timeout: 0`, `followRedirects: true`, or `maxRedirects: 5`; editing a request in Bruno's GUI
  may write them back, so drop them again before committing to keep the files uniform.
- `docs` is required: a one-line description plus the Directus docs link. Use
  `https://directus.com/docs/...` — `directus.io` redirects.

### Sequence ordering

Within each folder, `seq` values are contiguous starting at 1, in this order:

1. query (list)
2. query by id
3. create
4. create bulk
5. update
6. update bulk
7. delete
8. delete bulk
9. special operations (9+)

When you insert a request, renumber the rest of the folder so there are no gaps or duplicates.

Folders carry a `seq` too — without one, Bruno orders the sidebar arbitrarily. The top level is
`rest` (1), `graphql` (2), `websockets` (3). Resource folders under `rest/` are ordered by how
you actually reach for them rather than alphabetically: `auth` first, then content
(`items`, `files`, `folders`, `assets`), then people and access (`users`, `roles`, `policies`,
`permissions`), then data structure (`collections`, `fields`, `relations`), then per-item
features, then automation and instance-level operations, ending with `utilities`. Add new
folders where they fit and renumber the rest.

## Parameters

Path and query parameters share one `params` array, distinguished by `type`:

```yaml
params:
  - name: limit
    value: "10"
    type: query
    disabled: true
  - name: collection
    value: articles
    type: path
```

- Path params are **required** for every `:param` in the URL, and are never `disabled`
- Query params carry `disabled: true` so they show up in Bruno without being sent
- Values are strings — quote numbers (`value: "10"`)

### Which query params to include

Include only the params the endpoint actually honors — do not paste the full set everywhere.

- **List endpoints** take the full query system: `fields`, `filter`, `sort`, `limit`, `offset`,
  `search`, `meta`
- **Single-item reads** take `fields`, plus `version` for items
- Endpoints that accept neither get no `params` block at all

## Request Bodies

- Use realistic example values, not lorem ipsum
- Single-item operations use a plain object: `{"title": "Example"}`
- Bulk create uses an array: `[{"title": "Item 1"}, {"title": "Item 2"}]`
- Bulk update uses keys + data: `{"keys": ["id-1", "id-2"], "data": {"status": "published"}}`
  - **Exception:** `/collections` and `/fields` bulk updates take an array of objects that each
    identify their own target, since the key is the collection or field name. See
    `rest/collections/update_collections_bulk.yml`.
- Bulk delete uses an array of IDs: `["id-1", "id-2"]`

JSON bodies are a `type` + `data` string:

```yaml
body:
  type: json
  data: |-
    {
      "title": "Example Item"
    }
```

Multipart bodies are an array of parts, and each part requires `name`, `type`
(`text` or `file`), and `value`. A file part takes a bare path — there is no `@file()` wrapper
in this format:

```yaml
body:
  type: multipart-form
  data:
    - name: file
      type: file
      value: path/to/file.jpg
```

## Scripts

Script placement differs by level, and the schema will reject the wrong key:

- **Collection level** (`opencollection.yml`) — under `request.scripts`
- **Folder level** (`folder.yml`) — under `request.scripts`. Folders accept only `info`,
  `items`, `request`, and `docs`; there is no `runtime` key on a folder.
- **Request level** — under `runtime.scripts`

Valid script types are `before-request`, `after-response`, `tests`, and `hooks`.

Prefer `bru.setVar()` over `bru.setEnvVar()`. `setEnvVar` writes back into the tracked
`environments/*.yml` files, which puts live tokens and other response data into version
control; `setVar` sets a session-only runtime variable that `{{...}}` still resolves.

Never mutate a shared variable like `base_url` from a script. It persists across requests and
will silently break every other request in the collection. Add a purpose-specific variable to
the environments instead — that is why `ws_url` exists alongside `base_url`.

## Assertions and Tests

Requests may carry declarative assertions under `runtime.assertions`, and Chai-style tests as a
`runtime.scripts` entry with `type: tests`:

```yaml
runtime:
  assertions:
    - expression: res.status
      operator: eq
      value: "200"
    - expression: res.body.data
      operator: isArray
  scripts:
    - type: tests
      code: |-
        test("returns a list of items", () => {
          expect(res.status).to.equal(200);
          expect(res.getBody().data).to.be.an("array");
        });
```

Operators include `eq`, `neq`, `gt`, `gte`, `lt`, `lte`, `contains`, `isNumber`, `isString`,
`isBoolean`, `isArray`, and `isJson`.

Most requests here have none, because a meaningful assertion needs a live instance with seeded
data — asserting `res.status eq 200` on `delete_item` or `promote_version` would fail against an
empty project. If you add them, target requests that are safe against any instance (for example
`rest/server/ping_server.yml` or `get_server_health.yml`), and use the expected status for that
specific endpoint rather than a blanket 200.

## Path Parameters

- Use `:param` syntax in URLs: `{{base_url}}/items/:collection/:id`
- Every `:param` needs a matching entry with `type: path` and a realistic default value

## Adding a New Resource

1. Create a folder under `rest/` with the resource name (lowercase, underscore-separated)
2. Add `folder.yml` with `info.name` matching the folder and `info.type: folder`
3. Add `README.md` with a brief description, the list of operations, and a Directus docs link
4. Add request files following the naming and structure conventions above
5. Ensure `seq` values are contiguous and correctly ordered
6. Include the endpoint-appropriate query params on GET requests
7. Include a `docs` block on every request
8. Run `pnpm format` and verify the files validate against the OpenCollection schema

## WebSocket Requests

- `info.type: websocket`, with the payload under `websocket.message` (`type` + `data`)
- URL format: `{{ws_url}}/websocket` — `ws_url` is `ws://`/`wss://`, unlike `base_url`
- WebSocket requests take no `settings.encodeUrl`; `WebSocketRequestSettings` allows only
  `timeout` and `keepAliveInterval`
- Authentication is the `auth` message, not a header

## GraphQL Requests

- `info.type: graphql`, `graphql.method: POST`
- Query text goes in `graphql.body.query`; variables go in `graphql.body.variables` as a
  JSON **string**
- User data: `{{base_url}}/graphql` — system data: `{{base_url}}/graphql/system`

## Formatting

`pnpm format` writes, `pnpm format:check` verifies; CI runs the check on pull requests.

## Other Tooling

The OpenCollection format is not Bruno-specific, but for unrelated API tooling Directus serves
its own OpenAPI spec at `/server/specs/oas` (see **rest → server → get_oas_spec**).
