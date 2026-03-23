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

If an `access_token` is defined in the active environment, an `Authorization: Bearer {{access_token}}` header is automatically included via a collection-level pre-request script. The header is **not** added when:

- The request's auth mode is set to `inherit` or `none`
- The request is a WebSocket connection
- The URL targets `/auth/login` or `/auth/refresh`

If the token is empty (such as in the `public` environment), no authorization header is sent.

## Other Tooling

Directus serves its own OpenAPI spec at `/server/specs/oas` — use that for Postman, Insomnia, or any OpenAPI-compatible tool.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for conventions and guidelines.

## Directus Docs

https://directus.io/docs
