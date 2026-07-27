# auth

Authentication endpoints for Directus. Handles login, token refresh, logout, password reset, and SSO provider flows.

On a successful `login` (or `login_with_provider`), the returned tokens are automatically saved into the active environment's `access_token` and `refresh_token` variables via a post-response script, so subsequent requests are authenticated for you.

## Operations

- Login with email/password
- Login against a specific provider (local / LDAP / etc.)
- List configured authentication providers (incl. SSO)
- Initiate an SSO redirect login (OAuth2 / OpenID / SAML)
- Refresh access token
- Logout (invalidate refresh token)
- Request password reset email
- Reset password with token

## Docs

https://directus.io/docs/api/authentication
