---
name: Manage a Vendasta account and its users
description: Authenticate, look up a partner account, and read its users, settings and activated add-ons via the Vendasta Marketplace API V1.
api: openapi/vendasta-marketplace-openapi-original.yml
operations:
- getOAuthToken
- listAccounts
- getAccount
- listAccountUsers
- getAccountSettings
- listActivatedAddons
generated: '2026-07-21'
method: generated
---

# Manage a Vendasta account and its users

Operating instructions for reading Vendasta accounts. Base URL: `https://developers.vendasta.com/api/v1`
(use `https://developers-demo.vendasta.com/api/v1` for the Demo sandbox). operationIds below are
assigned in `overlays/vendasta-marketplace-overlay.yaml` (the source spec omits them); the
underlying path/method is shown for each step.

## Auth (getOAuthToken — POST /oauth/token)

1. Exchange your client credentials at `POST /oauth/token` (JSON body).
2. The response returns `access_token`, `token_type` (Bearer) and `expires` (a Unix timestamp).
3. Send `Authorization: Bearer <access_token>` on every subsequent call. Refresh before `expires`.

## Steps

1. **List accounts** (`listAccounts` — `GET /account/`). Page with `page_size` + `cursor`; follow
   the `pagination` cursor in the response for the next page.
2. **Get one account** (`getAccount` — `GET /account/{account_id}`) using an id from the list.
3. **List its users** (`listAccountUsers` — `GET /account/{account_id}/users`).
4. **Read settings** (`getAccountSettings` — `GET /account/{account_id}/settings`).
5. **Read activated add-ons** (`listActivatedAddons` — `GET /account/{account_id}/addons`).

## Rules

- Pagination is cursor-based (`page_size` + opaque `cursor`); never fabricate cursors. See
  `conventions/vendasta-conventions.yml`.
- Handle `401` by refreshing the token, `403` as a permission/scope problem, `404` as a missing
  account. See `errors/vendasta-problem-types.yml`.
- No idempotency-key contract is documented; these are read operations so retries are safe.
