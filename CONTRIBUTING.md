# Contributing

## Collection Structure

| Folder       | Description                                 |
| ------------ | ------------------------------------------- |
| `rest`       | REST API endpoints, one folder per resource |
| `graphql`    | GraphQL queries and mutations               |
| `websockets` | Real-time WebSocket subscriptions           |

Each resource folder under `rest/` contains:

- `folder.bru` — folder metadata
- `README.md` — brief description, list of operations, docs link
- Individual `.bru` request files

## Naming Conventions

### Folders

- Lowercase, underscore-separated: `content_versions/`, `custom_translations/`

### Files

- Pattern: `{action}_{scope}_{resource}.bru`
- Actions: `query`, `create`, `update`, `delete`
- Scope: omitted for single, `_bulk` suffix for multiple
- Examples:
  - `query_items.bru` — list/filter items
  - `query_item_by_id.bru` — single item by ID
  - `create_item.bru` — create one
  - `create_items_bulk.bru` — create many
  - `delete_items_bulk.bru` — delete many
- Special operations use descriptive names: `trigger_flow_post.bru`, `promote_version.bru`, `get_current_user.bru`

### Meta name field

- Must match the filename without `.bru` extension
- Lowercase, underscore-separated: `name: query_items`

## Request File Structure

### Required blocks

Every REST/GraphQL `.bru` file must have:

1. **meta** — name, type, seq
2. **method block** — get/post/patch/delete with url, body type, `auth: inherit`
3. **docs** — one-line description + Directus docs link

### Optional blocks

- **params:path** — required when URL contains `:param` placeholders
- **params:query** — required on all GET requests (see query params below)
- **body:json** / **body:graphql** / **body:multipart-form** — required on POST/PATCH requests

### Sequence ordering

Within each folder, `seq` values follow this order:

1. query (list)
2. query by id
3. create
4. create bulk
5. update
6. update bulk
7. delete
8. delete bulk
9. special operations (9+)

## Query Parameters

All GET requests must include these disabled query params:

```
params:query {
  ~fields: id,title,date_created
  ~filter: {"status":{"_eq":"published"}}
  ~sort: -date_created
  ~limit: 10
  ~offset: 0
  ~search: keyword
  ~meta: total_count,filter_count
}
```

This applies to both list queries and single-item queries. The `~` prefix keeps them visible but not sent by default.

## Request Bodies

- Use realistic example values, not lorem ipsum
- Single-item operations use a plain object: `{"title": "Example"}`
- Bulk create uses an array: `[{"title": "Item 1"}, {"title": "Item 2"}]`
- Bulk update uses keys + data: `{"keys": ["id-1", "id-2"], "data": {"status": "published"}}`
- Bulk delete uses an array of IDs: `["id-1", "id-2"]`

## Docs Block

Every request must include a `docs {}` block with:

- One-line description of what the request does
- Link to the relevant Directus docs page

```
docs {
  Retrieve a list of items from the specified collection.
  https://directus.io/docs/api/items
}
```

## Path Parameters

- Use `:param` syntax in URLs: `{{base_url}}/items/:collection/:id`
- Define default values in `params:path {}`:

```
params:path {
  collection: my_collection
  id: item-id
}
```

## Adding a New Resource

1. Create a folder under `rest/` with the resource name (lowercase, underscore-separated)
2. Add `folder.bru` with meta name matching the folder
3. Add `README.md` with a brief description and link to Directus docs
4. Add request files following the naming and structure conventions above
5. Ensure `seq` values are ordered correctly
6. Include full query params on all GET requests
7. Include `docs {}` block on every request

## WebSocket Requests

- Use `type: ws` in meta block
- URL format: `{{base_url}}/websocket`
- Message payloads go in `body:ws {}` blocks with `type: json`
- No test blocks on WebSocket requests

## GraphQL Requests

- Use `type: graphql` with POST method and `body: graphql`
- User data queries: `{{base_url}}/graphql`
- System data queries: `{{base_url}}/graphql/system`
- Use `body:graphql:vars {}` for query variables

## Known Limitations

- Bruno v3 does not support `test {}` blocks without causing parse failures — do not add test blocks
- The `.bru` format is Bruno-specific; for other API tooling, Directus provides an OpenAPI spec at `/server/specs/oas`
