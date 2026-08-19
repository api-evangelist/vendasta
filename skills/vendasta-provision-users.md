---
name: Provision platform users and grant them business locations
description: >-
  Create Vendasta users, grant and revoke their access to business locations, manage user custom
  fields, and send the welcome email — using the Platform REST API rather than SCIM.
api: openapi/vendasta-platform-openapi.yml
base_url: https://prod.apigateway.co/platform
operations:
  - post-users
  - get-users
  - get-users-id
  - delete-users-id
  - post-users-id-relationships-businessLocations
  - get-users-id-relationships-businessLocations
  - patch-users-id-relationships-businessLocations
  - delete-users-id-relationships-businessLocations
  - get-sendWelcomeEmail-by-userid
  - get-userCustomFields-by-userid
  - patch-userCustomFields-by-id
scopes:
  - user.admin
  - user.list
  - user.profile:read
  - user.contact:read
  - user.permission
  - user.permission:read
generated: '2026-08-13'
method: generated
source: openapi/vendasta-platform-openapi.yml
---

# Provision platform users and grant them business locations

In Vendasta, a **user** is separate from the **business locations** they can see. Creating the user
does not grant access to anything — access is a relationship you add afterwards. Get that ordering
wrong and the user logs in to an empty platform.

## Before you start

- Bearer token from `https://sso-api-prod.apigateway.co/oauth2/token`, sent as
  `Authorization: Bearer <access_token>`.
- Almost everything here needs the `user.admin` scope. Reads can use narrower scopes:
  `user.profile:read`, `user.contact:read`, `user.permission:read`. Searching by filter rather than
  by exact id additionally needs `user.list`.
- JSON:API bodies (`application/vnd.api+json`); page with `page[limit]` and follow `links.next`.
- If you need a standards-based provisioning path instead, Vendasta operates a SCIM 2.0 endpoint at
  `https://prod.apigateway.co/scim` — see `openapi/vendasta-scim-openapi.yml`. Use SCIM when your
  IdP drives the lifecycle; use this skill when your own application does.

## Step 1 — Check whether the user already exists

Call `get-users` (`GET /users`, scopes `user.admin` + `user.list`) with `filter[email]` or another
supported filter. **No idempotency key exists on this API**, so a blind `POST` on retry creates a
duplicate person.

To read one you already have an id for, call `get-users-id` (`GET /users/{id}`).

## Step 2 — Create the user

Call `post-users` (`POST /users`, scope `user.admin`). Keep the returned `id`.

Address fields were renamed during the trustedTester phase: use `line1` and `line2`, **not** the
deprecated `streetAddress` / `additionalAddress`, which carry `x-lifecycle: deprecated` with a
proposed removal date of 2021-11-04.

## Step 3 — Grant access to business locations

Call `post-users-id-relationships-businessLocations`
(`POST /users/{id}/relationships/businessLocations`, scope `user.admin`) with the location ids.

- `get-users-id-relationships-businessLocations` reads the current grants.
- `patch-users-id-relationships-businessLocations` **replaces** the whole set — read first, or you
  will silently revoke locations you meant to keep.
- `delete-users-id-relationships-businessLocations` revokes specific locations.

If you are integrating a Marketplace app, note that Vendasta also pushes a **User Permission
webhook** on `permission-granted` / `permission-revoked` so you can keep a local cache in step with
these calls — see `asyncapi/vendasta-webhooks.yml`.

## Step 4 — Send the welcome email

Call `get-sendWelcomeEmail-by-userid` (`GET /users/{id}/actions/sendWelcomeEmail`, scope
`user.admin`). Do this **after** step 3 — a user who follows the invite before being granted a
location lands in an empty Business App.

Note the shape: this action is modelled as a `GET`, so it is not safe to treat GETs on this API as
side-effect free. Do not let an agent "warm up" by calling every GET it can see.

## Step 5 — Custom fields

`get-userCustomFields-by-userid` (`GET /users/{id}/customFields`) reads the values;
`patch-userCustomFields-by-id` (`PATCH /userCustomFields/{id}`) writes one. Custom fields accept an
`externalId` you configure in Partner Center, which you can use instead of the system-generated
`fieldId`.

## Step 6 — Removal

`delete-users-id` (`DELETE /users/{id}`, scope `user.admin`) removes the user. Prefer revoking
location relationships first if you only need to cut off access.

## Errors and maturity

Errors return the JSON:API `errors[]` envelope with a machine-readable `code` and a `links.about`
pointing at `https://prod.apigateway.co/docs/errorTypes/{code}`
(`errors/vendasta-error-codes.yml`). `patch-users-id` is `x-lifecycle: proposed` — mock server only.
Every other operation here is `trustedTester` and may change without a major version, because this
API deliberately has no versions. See `lifecycle/vendasta-lifecycle.yml`.
