---
name: Manage LendAPI sub-tenants and loan math
description: Provision login-bearing or headless sub-tenants under a parent tenant, and use the amortization and credit-risk scoring endpoints.
api: openapi/lendapi-openapi.json
operations:
  - list-all-sub-tenants
  - get-sub-tenant
  - create-sub-tenant
  - lendapi-amortization
  - integrated-function
generated: '2026-07-19'
method: generated
source: openapi/lendapi-openapi.json
---

# Manage sub-tenants, amortization, and risk scoring

Use this when you are running LendAPI as a platform — one parent Tenant with many
downstream brands, dealers, merchants, or bank partners underneath it.

## Sub-tenants

1. **List** — `list-all-sub-tenants` (`GET /sub_tenants`), paginated with `start` and
   `size`. Returns every Sub Tenant under the parent Tenant.

2. **Read one** — `get-sub-tenant` (`GET /sub_tenant/{tenant_id}`).

3. **Create** — `create-sub-tenant` (`POST /sub_tenant/`). Two shapes:
   - **With login** — supply `first_name`, `last_name`, `email` and `password` to
     create a sub-tenant whose operator can sign in.
   - **Without login** — omit all four. Use this for a headless sub-tenant that only
     ever exists as an API-side segmentation boundary.
   Pass `client_id` (optional) to carry *your* external identifier for the
   sub-tenant, so you can reconcile without maintaining a mapping table.

## Loan math

- **Amortization** — `lendapi-amortization` (`POST /amortization/`). Returns the
  payment schedule with each payment split into principal and interest across the life
  of the loan, plus the computed `APR` and `periodic_payment`. This is a pure
  calculation — nothing is persisted, so it is safe to call for quotes and previews.

- **Credit risk scoring** — `integrated-function`
  (`POST /credit_risk/integrated_function_2/`). A machine-learning endpoint returning
  subprime risk scores. Treat the score as one input to a decision tree, not as a
  decision.

## Rules and cautions

- **`create-sub-tenant` accepts a password.** Never log the request body, and never
  round-trip that password through an intermediate store.
- No idempotency key exists on `POST /sub_tenant/`. Check `list-all-sub-tenants` (or
  your own `client_id`) before retrying a create that timed out.
- Scores from `integrated-function` are consumer-impacting. Persist the inputs and the
  returned score together with a timestamp so an adverse-action decision can be
  explained later.
- Only `200` and an untyped `400` are documented on these operations.
