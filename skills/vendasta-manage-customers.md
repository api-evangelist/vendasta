---
name: Create and manage Vendasta customers
description: Authenticate and create, look up, update or delete customer records via the Vendasta Marketplace API V1.
api: openapi/vendasta-marketplace-openapi-original.yml
operations:
- getOAuthToken
- createCustomer
- listCustomers
- getCustomer
- updateCustomer
- deleteCustomer
generated: '2026-07-21'
method: generated
---

# Create and manage Vendasta customers

Operating instructions for the customer surface of the Vendasta Marketplace API V1. Base URL:
`https://developers.vendasta.com/api/v1` (Demo: `https://developers-demo.vendasta.com/api/v1`).
Note: the customer endpoints are POST-based RPC-style routes. operationIds are assigned in
`overlays/vendasta-marketplace-overlay.yaml`.

## Auth (getOAuthToken — POST /oauth/token)

Exchange client credentials at `POST /oauth/token`, then send `Authorization: Bearer <access_token>`.

## Steps

1. **Create a customer** (`createCustomer` — `POST /customer`).
2. **List customers** (`listCustomers` — `POST /customer/list-customers`) with filters in the body.
3. **Get a customer** (`getCustomer` — `POST /customer/get-customer`).
4. **Update a customer** (`updateCustomer` — `POST /customer/update-customer`).
5. **Delete a customer** (`deleteCustomer` — `POST /customer/delete-customer`).

To link identities across systems, use `POST /customer/associate-id` (associateCustomerIds).

## Rules

- Auth failures surface as `401`; permission problems as `403`; unknown customer as `404`. See
  `errors/vendasta-problem-types.yml`.
- No idempotency-key header is documented; treat `createCustomer` retries carefully to avoid
  duplicates — re-check with `listCustomers`/`getCustomer` before retrying a create.
- See `conventions/vendasta-conventions.yml` for auth, pagination and error semantics.
