---
name: Run a LendAPI decision tree and pricing engine
description: Discover a tenant's versioned decision trees and pricing engines, read the parameters each version requires, execute them, and interpret the outcomes.
api: openapi/lendapi-openapi.json
operations:
  - list-all-decision-trees
  - list-decision-versions
  - list-decision-parameters
  - run-decision
  - list-outcomes-by-action
  - list-all-pricing-engines
  - list-pricing-engine-versions
  - list-pricing-engine-parameters
  - run-pricing-engine
  - create-a-variable
generated: '2026-07-19'
method: generated
source: openapi/lendapi-openapi.json
---

# Run a decision tree and a pricing engine

LendAPI's decision engine and pricing engine are both **versioned** and both
**parameter-driven**. Never hardcode an input set — always read the parameter list for
the specific version you are about to run.

## Before you start

- Base URL `https://app.lendapi.com/api/v1/`, header `AUTHORIZATION: Bearer <api_key>`.
- Everything is scoped to the Tenant that owns the key.

## Decisioning

1. **List the decision trees** — `list-all-decision-trees` (`GET /decisions/`).
   Paginate with `start` (offset) and `size` (page size). No cursor is offered and no
   total count is documented, so keep requesting until a short page comes back.

2. **List the versions** — `list-decision-versions`
   (`GET /decision/{id}/versions/`). Pick the version deliberately; running "latest"
   implicitly makes your results non-reproducible.

3. **Read the required inputs** — `list-decision-parameters`
   (`GET /decision/{id}/{decision_version}/params/`). This returns every parameter
   required for that decision *and that version*. Build your payload from this
   response, not from a previous run.

4. **Execute** — `run-decision` (`POST /decision/{id}/run/`).

5. **Resolve what an outcome means** — `list-outcomes-by-action`
   (`GET /outcomes/{id}/`) returns all Outcomes under the Tenant tied to a specific
   Action, which is how you map an engine result onto your own workflow branches.

## Pricing

The pricing engine mirrors the decision engine exactly:

1. `list-all-pricing-engines` (`GET /pricing_engines`) — same `start` / `size` paging.
2. `list-pricing-engine-versions` (`GET /pricing_engine/{id}/versions/`).
3. `list-pricing-engine-parameters`
   (`GET /pricing_engine/{id}/{engine_version}/params/`).
4. `run-pricing-engine` (`POST /pricing_engine/{id}/run/`).

## Feeding the engines

Inputs are LendAPI **Variables**. Create a new one with `create-a-variable`
(`POST /variable/`) when a decision or pricing model needs a data point the tenant has
not defined yet. Variables are what travel in an application's `data` bag, so defining
the Variable is what makes the value addressable by an engine.

## Rules and cautions

- **Version everything you log.** Record the decision id, the decision version, the
  pricing engine id and the engine version alongside every result. These are
  underwriting decisions — you will need to reconstruct exactly which logic ran.
- **Run operations are POSTs with no idempotency key.** A retried run may execute
  twice. Make your own call sites idempotent if a double execution would double-write
  to your ledger.
- Only `200` and a bare `400` are documented. Do not expect a structured error body.
- When you run a decision as part of an application, prefer reading the result from
  the `engine` node of `get-an-application` so the decision stays bound to the
  application record.
