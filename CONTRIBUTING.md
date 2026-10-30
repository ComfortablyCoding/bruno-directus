# Contributing

## Format

This collection uses the [OpenCollection](https://spec.opencollection.com) `1.0.0` format — YAML
files, one per request, with `opencollection.yml` at the root. The older Bruno-specific `.bru`
format is no longer used.

The authoritative schema is
[`opencollection.schema.json`](https://github.com/usebruno/opencollection/blob/main/packages/oc-schema/src/opencollection.schema.json).
Most object types set `additionalProperties: false`, so an unexpected or misplaced key makes the
file invalid rather than being ignored. When in doubt, check the schema.

A file can be schema-valid and still break in Bruno — for example, the schema allows a multipart
file part's `value` to be a string, but Bruno crashes on one. Follow the rules below even where
the schema is looser.

## Collection Structure

| Folder       | Description                                 |
| ------------ | ------------------------------------------- |
| `rest`       | REST API endpoints, one folder per resource |
| `graphql`    | GraphQL queries, mutations, and auth        |
| `websockets` | Real-time WebSocket connections             |

Each folder contains:

- `folder.yml` — folder metadata **and its documentation**, under `docs`: a brief description,
  the list of operations, any setup the requests need, and a Directus docs link. This is what
  Bruno shows in the folder's Docs tab.
- Individual `.yml` request files

Do not add `README.md` files to folders — Bruno never reads or shows them. The root
`README.md` is for GitHub.

```yaml
info:
  name: roles
  type: folder
  seq: 7

docs:
  content: |-
    # roles

    One-paragraph description.

    ## Operations

    - Query all roles
    - …

    ## Docs

    https://directus.com/docs/api/roles
  type: text/markdown
```

## Naming Conventions

### Folders

- Lowercase, underscore-separated, mirroring the URL segment: `/utils` → `utils/`

### Files

- Pattern: `{action}_{resource}[_bulk].yml`
- `query_<plural>` — list endpoints that accept the query system (`query_items.yml`)
- `query_<singular>_by_id` — a single read by primary key (`query_item_by_id.yml`)
- `create_*`, `update_*`, `delete_*` — with a `_bulk` suffix for the multi-item variant
  (`create_items_bulk.yml`, `delete_items_bulk.yml`)
- `get_<thing>` — any other read: singletons, reads keyed by something other than a primary key,
  and special reads (`get_settings.yml`, `get_collection.yml`, `get_current_user.yml`,
  `get_my_roles.yml`)
- Descriptive verbs for actions: `trigger_manual_flow.yml`, `promote_version.yml`,
  `deploy_project.yml`. Name a request for what it does, not its HTTP method.

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
- `auth: inherit` on every request that needs a token, so it gets the collection's Bearer auth.
  **Omit `auth`** on requests that are public by design — login, refresh, logout, password
  reset, registration, invite acceptance, share access, first-run setup. Directus rejects any
  request carrying an expired token, even on public routes, so inheriting the token would break
  them exactly when it has expired. (OpenCollection has no `auth: none`; a missing `auth` key is
  no auth.)
- `settings: { encodeUrl: true }` on every HTTP and GraphQL request. Only deviate with a
  reason — `rest/auth/sso_login_redirect.yml` sets `followRedirects: false` because the point
  of the request is to inspect the redirect. Prefer not to restate defaults such as
  `timeout: 0`, `followRedirects: true`, or `maxRedirects: 5`; editing a request in Bruno's GUI
  may write them back, so drop them again before committing to keep the files uniform.
- `docs` is required: a short description plus the Directus docs link. Use
  `https://directus.com/docs/...` — `directus.io` redirects. Check the link resolves.

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
`permissions`, `access`), then data structure (`collections`, `fields`, `relations`), then
per-item features, then automation and instance-level operations (`flows`, `operations`,
`deployments`, `extensions`, `settings`, `license`, `schema`, `server`, `metrics`), ending with
`utils`. Add new folders where they fit and renumber the rest.

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
- Name a primary-key path param `id`, even where Directus calls it `pk`; give other path params
  descriptive names (`collection`, `field`, `provider`)
- Query params carry `disabled: true` so they show up in Bruno without being sent
- Values are strings — quote numbers (`value: "10"`)

### Which query params to include

Include only the params the endpoint actually honors — do not paste the full set everywhere.

- **List endpoints** take the query system: `fields`, `filter`, `sort`, `limit`, `offset`,
  `search`, `meta`. `query_items` additionally carries `deep`, `aggregate`, `groupBy`, `alias`,
  `page`, `backlink`, and `export` as the reference for the full set.
- **Single-item reads** take `fields`, plus `deep` and `version` for items
- Endpoints that accept neither (`/collections`, `/fields`, `/relations`) get no `params` block

**Example values must name fields that exist on that collection.** Directus rejects a query that
references an unknown field — even for admins — so a copied `sort=-date_created` turns into an
error the moment someone enables it on `directus_users`. Check the system collection's fields
(for example in `packages/system-data/src/fields/` in the Directus repository).

## Request Bodies

- Use realistic example values, not lorem ipsum
- The example user collection is `articles`, with the fields `create_collection` defines (`id`,
  `status`, `title`, `body`, `author`, `date_created`). Keep examples consistent with it.
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
(`text` or `file`), and `value`. A file part's `value` is an **array** of paths, resolved from the
collection root. Directus reads fields only if they come before the file, so put text parts
first:

```yaml
body:
  type: multipart-form
  data:
    - name: title
      type: text
      value: Example image
    - name: file
      type: file
      value:
        - path/to/file.jpg
```

## Scripts

Script placement differs by level, and the schema will reject the wrong key:

- **Collection level** (`opencollection.yml`) — under `request.scripts`
- **Folder level** (`folder.yml`) — under `request.scripts`. Folders accept only `info`,
  `items`, `request`, and `docs`; there is no `runtime` key on a folder.
- **Request level** — under `runtime.scripts`

Valid script types are `before-request`, `after-response`, `tests`, and `hooks`.

Token handling lives at the collection level: an after-response script captures tokens from
login and refresh (REST and GraphQL) and clears them on logout, and a before-request script drops
them when `base_url` changes. Requests don't need their own token scripts.

Use runtime variables (`bru.setVar()`) to pass a value from one request to the next — see
`compare_version` → `promote_version` or the `rest/schema` chain — and name the variable in both
requests' docs. Prefer `bru.setVar()` over `bru.setEnvVar()`: `setEnvVar` writes back into the
tracked `environments/*.yml` files, which puts live tokens and other response data into version
control; `setVar` sets a session-only runtime variable that `{{...}}` still resolves.

Never mutate a shared variable like `base_url` from a script. It persists across requests and
will silently break every other request in the collection. Add a purpose-specific variable to
the environments instead.

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

## Adding a New Resource

1. Create a folder under `rest/` with the resource name (lowercase, underscore-separated)
2. Add `folder.yml` with `info.name` matching the folder, `info.type: folder`, and a `docs`
   block with a brief description, the list of operations, and a Directus docs link
3. Add request files following the naming and structure conventions above
4. Ensure `seq` values are contiguous and correctly ordered
5. Include the endpoint-appropriate query params on GET requests, with real field names
6. Include a `docs` block on every request
7. Run `pnpm format` and verify the files validate against the OpenCollection schema

## WebSocket Requests

- `info.type: websocket`, with the URL and messages under `websocket`
- URL format: `{{base_url}}/websocket` — Bruno's WebSocket client connects to an `http://`/`https://`
  URL over `ws://`/`wss://`, so there is no separate WebSocket URL variable
- **Bruno opens one connection per request file**, so a message sent from one file never reaches
  a connection opened by another. Model each file as a complete session: `websocket.message` as
  an array of titled messages in the order they're sent, starting with authentication.

  ```yaml
  websocket:
    url: "{{base_url}}/websocket"
    message:
      - title: 1. auth
        selected: true
        message:
          type: json
          data: |-
            {"type": "auth", "access_token": "{{access_token}}"}
      - title: 2. subscribe
        message:
          type: json
          data: |-
            {"type": "subscribe", "collection": "articles"}
  ```

- WebSocket requests take no `settings.encodeUrl`; `WebSocketRequestSettings` allows only
  `timeout` and `keepAliveInterval`
- Set no `auth`: authentication is a message (or a URL token for the strict-auth logs endpoint),
  and collection scripts don't run for WebSocket requests

## GraphQL Requests

- `info.type: graphql`, `graphql.method: POST`, with `graphql` keys in the order `method`, `url`,
  `auth`, `body`
- Query text goes in `graphql.body.query`; variables go in `graphql.body.variables` as a
  JSON **string**
- User data: `{{base_url}}/graphql` — system data: `{{base_url}}/graphql/system`

## Formatting

`pnpm format` writes, `pnpm format:check` verifies; the `check` workflow runs the check on every
pull request and push to `main`.

## Other Tooling

The OpenCollection format is not Bruno-specific, but for unrelated API tooling Directus serves
its own OpenAPI spec at `/server/specs/oas` (see **rest → server → get_oas_spec**).
