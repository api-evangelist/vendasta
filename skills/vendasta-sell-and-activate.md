---
name: Sell a product to a business and confirm activation
description: >-
  Use the Vendasta Platform REST API to find what a partner can sell, create or locate the sales
  account, place an order, and confirm the resulting subscription is active on the business location.
api: openapi/vendasta-platform-openapi.yml
base_url: https://prod.apigateway.co/platform
operations:
  - get-partnerActivatableProducts
  - post-salesAccounts
  - get-salesAccounts
  - post-orders
  - get-orders-id
  - get-subscriptionAssignments
scopes:
  - partner:read
  - product
  - sales.account
  - order
  - order:read
generated: '2026-08-13'
method: generated
source: openapi/vendasta-platform-openapi.yml
---

# Sell a product to a business and confirm activation

Vendasta's commercial loop is: a **partner** resells **products** to a **sales account**, which
becomes an **order**, which activates a **subscription assignment** on a **business location**.
This skill walks that loop with real operations from the Platform REST API.

## Before you start

- Every request needs an OAuth2 bearer token from `https://sso-api-prod.apigateway.co/oauth2/token`.
  Mint it with the 2-legged service-account assertion (RS256 JWT, `aud`
  `https://iam-prod.apigateway.co`) or the 3-legged OIDC authorization-code flow. Send it as
  `Authorization: Bearer <access_token>`. See `authentication/vendasta-authentication.yml`.
- Request the scopes this flow needs **at token time** — Vendasta scopes are per-operation and you
  must re-mint the token if the scope set changes.
- Bodies are **JSON:API** (`application/vnd.api+json`). Each resource has a `type` and an `id`.
- Use the demo environment (`https://demo.apigateway.co/platform`,
  `https://sso-api-demo.apigateway.co`) while building. A demo instance is not self-serve — request
  one from support@vendasta.com.

## Step 1 — Find what the partner can actually sell

Call `get-partnerActivatableProducts` (`GET /partnerActivatableProducts`) with scopes
`partner:read` and `product`. This is the list of products the partner has enabled in their
marketplace; anything not in this list cannot be ordered. To inspect one, call
`get-partnerActivatableProducts-id` (`GET /partnerActivatableProducts/{id}`).

Page with `page[limit]` and follow `links.next` — never construct the next page URL yourself.

## Step 2 — Locate or create the sales account

Call `get-salesAccounts` (`GET /salesAccounts`, scope `sales.account`) and filter with
`filter[...]` query params to see whether the business already exists. If it does not, call
`post-salesAccounts` (`POST /salesAccounts`) to create it.

**There is no idempotency key on this API.** A retried `POST /salesAccounts` creates a second
account. Search first, and record the returned `id` before doing anything else.

## Step 3 — Place the order

Call `post-orders` (`POST /orders`, scope `order`).

- You may send only the **SKU** on line items — `amount` and `intervalCode` are filled in
  automatically from the retail prices configured for the account's market. Any amount you send,
  including `0`, is kept as-is.
- Set `statusCode` to `submitted` to route the order to partner admins for manual approval, or
  `draft` to create it without notifying anyone. Omit it for immediate activation.
- Same warning as Step 2: no idempotency. On a network timeout, call `get-orders`
  (`GET /orders`, scope `order:read`) and look for the order before retrying.

## Step 4 — Read the order back

Call `get-orders-id` (`GET /orders/{id}`, scope `order` or `order:read`). If the product has a
fulfillment form, use `get-orderFulfillmentForms` and `post-orderFulfillmentForms-submit`
(`POST /orderFulfillmentForms/{id}/actions/submit`) to complete it — activation will not proceed
until the form is submitted.

## Step 5 — Confirm the subscription is live

Call `get-subscriptionAssignments` (`GET /subscriptionAssignments`, scope `sales.account`) to see
what is actually activated for the business location. Do this before ordering a product again —
it is the only way to avoid double-activating something the account already has.

To cancel, `cancel-subscriptionAssignments-by-id`
(`POST /subscriptionAssignments/{id}/actions/requestCancellation`); to reverse a cancellation,
`restore-subscriptionAssignments-by-id`
(`POST /subscriptionAssignments/{id}/actions/undoCancellation`). Both are `x-lifecycle: proposed`,
which in Vendasta's own vocabulary means **mock server only** — do not build a production
cancellation path on them yet.

## Errors

Failures return the JSON:API `errors[]` envelope. Read `code` (the machine-readable platform error),
`source.parameter` when a query parameter is at fault, and `links.about`
(`https://prod.apigateway.co/docs/errorTypes/{code}`) for the error's own documentation page. See
`errors/vendasta-error-codes.yml`.

## Maturity warning

Every operation in this skill is `x-lifecycle: trustedTester` or `proposed`. Vendasta defines
trustedTester as "newly built and undergoing rapid iteration... breaking changes may occur". No
Platform REST operation has reached General Availability, and the Guides overview states these APIs
are "now in maintenance mode". Pin nothing to a stable-API assumption. See
`lifecycle/vendasta-lifecycle.yml`.
