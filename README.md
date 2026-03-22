# Directus API — Bruno Collection

Comprehensive Bruno collection covering the entire Directus 11+ API surface: REST, GraphQL, and WebSockets.

## Setup

1. Install [Bruno](https://www.usebruno.com/downloads) if you haven't already
2. Clone or download this repository
3. In Bruno, click **Open Collection** (or `Ctrl/Cmd+O`) and select this folder
4. In the top-right environment dropdown, select an environment (see below)
5. Replace the placeholder values in the environment with your Directus instance details

## Environments

| Environment  | Purpose                                | Auth                |
| ------------ | -------------------------------------- | ------------------- |
| `local`      | Local dev instance at `localhost:8055` | Static user token   |
| `production` | Production instance                    | Static user token   |
| `admin`      | Admin-level access                     | Admin static token  |
| `public`     | Unauthenticated requests               | No auth header sent |

### Variables

| Variable       | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| `base_url`     | Directus instance URL (e.g. `http://localhost:8055`)         |
| `access_token` | Static access token (leave empty for public/unauthenticated) |

## Authentication

Auth is handled by a pre-request script at the collection root. If `access_token` is set in the active environment, requests include `Authorization: Bearer {{access_token}}`. If empty (like the `public` environment), no auth header is sent.

**Testing with different user tokens:** Override the `Authorization` header on any individual request — set auth to `bearer` and provide the user's token.

## Other Tooling

Directus serves its own OpenAPI spec at `/server/specs/oas` — use that for Postman, Insomnia, or any OpenAPI-compatible tool.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for conventions and guidelines.

## Directus Docs

https://directus.io/docs
