---
name: Authenticate with the Cobee Public API
description: Exchange client credentials for a bearer JWT and confirm access by listing companies.
api: openapi/cobee-public-api-openapi-original.json
operations: [postOauthToken, getCompanies]
---

# Authenticate with the Cobee Public API

Cobee's Public API is server-to-server. It is available to Spanish customers and
credentials (`clientId`/`clientSecret`) are issued by your Cobee Customer Success
Manager for the staging and/or production environment.

## Environments

- Staging (sandbox): `https://pre-public-api.cobee.io/api/v3`
- Production: `https://public-api.cobee.io/api/v3`

## Steps

1. **Get a token** — `POST /oauth/token` (`postOauthToken`) with a JSON body
   containing `clientId` and `clientSecret`. This is an OAuth 2.0 Client
   Credentials exchange (Auth0); the response is a JWT (RS256).
2. **Send the token** — put it on every subsequent call as
   `Authorization: Bearer <jwt>`. The JWT carries `companyId` (and optional
   `corporationId`) claims that scope your access.
3. **Confirm access** — `GET /companies` (`getCompanies`) returns the companies
   your credentials can act on. Capture each `company.id` for later calls.

## Rules

- The JWT expires; re-run step 1 to refresh, do not assume a long-lived token.
- Treat `clientId`, `clientSecret`, and the JWT as secrets (env vars / secret
  manager) — never log or commit them.
- Errors come back as `{ "error": <int>, "message": <string> }` (see
  `errors/cobee-problem-types.yml`); `401` means the token is missing/expired.
