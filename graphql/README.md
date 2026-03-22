# graphql

GraphQL API requests for Directus.

## Endpoints

- **User data**: `POST {{base_url}}/graphql` — access collections defined by your data model
- **System data**: `POST {{base_url}}/graphql/system` — access system collections (users, roles, files, etc.)

## Structure

- `queries/` — read operations (query)
- `mutations/` — write operations (create, update, delete)

## Authentication

All requests inherit the collection-level Bearer token via `auth: inherit`.

## Docs

https://directus.io/docs/api/graphql
