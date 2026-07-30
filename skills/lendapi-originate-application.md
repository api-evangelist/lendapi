---
name: Originate a loan application with LendAPI
description: Create an applicant application against a LendAPI lending product, submit it, poll its decision status, and react to the approve / decline / manual-review outcome.
api: openapi/lendapi-openapi.json
operations:
  - create-application
  - post_page-submit
  - get-an-application
  - put_application
generated: '2026-07-19'
method: generated
source: openapi/lendapi-openapi.json
---

# Originate a loan application

Use this when you need to take an applicant from raw identity data to a credit
decision on LendAPI.

## Before you start

- Base URL is `https://app.lendapi.com/api/v1/`. There is no separate sandbox host.
- Send `AUTHORIZATION: Bearer <api_key>` on every request.
- **The key selects the environment.** A test-mode key keeps the request off live
  data; a live key does not. Confirm which key you hold before creating anything.
- You need a `product_id` for the lending product the application is opened against.
  Products are configured in Product Studio; the public API does not list them.

## Steps

1. **Create the application** — `create-application` (`POST /application/`).
   Required: `first_name`, `last_name`, `email`, `product_id`. Supply as much of
   `address1`, `address2`, `city`, `state`, `zipcode`, `mobile`, `ssn` and
   `birth_date` as the product's decision tree needs — `birth_date` is `MM/DD/YYYY`,
   `state` is the two-letter code, `zipcode` is 5 digits, `mobile` is digits only.
   Put product-specific inputs in `data` as a JSON key/value bag of LendAPI Variables,
   e.g. `{"loan_amount": 2000, "loan_term": 12}`. Keep the returned application id.

2. **Submit the pages** — `post_page-submit` (`POST /page-submit/`).
   Only needed when you are building your own frontend instead of using the Product
   Studio built-in UI. Skip it if Product Studio is rendering the flow.

3. **Amend before decisioning if needed** — `put_application` (`PUT /application`).
   Use this to correct or complete applicant fields.

4. **Read the decision** — `get-an-application` (`GET /application/{id}`).
   Returns the application's basic info and status. The rules-engine and
   pricing-engine results come back under the `engine` node of the response — that is
   where you find whether the application was approved or declined and on what terms.

5. **Pull the credit report if enabled** — `get-an-application-1`
   (`GET /get_app_credit_report/{id}`). Returns the bureau credit report PDF as an S3
   link. **The link expires in 30 minutes** — download it immediately rather than
   storing the URL.

## Prefer webhooks over polling

Do not loop on `get-an-application`. LendAPI pushes the whole application lifecycle as
webhook events: `application.review`, `application.approved`, `application.declined`,
`application.request_docs`, `application.processing`, `application.booked`,
`application.funded`, `application.doc_ai_completed`. The envelope is
`{api_version, type, live_mode, data}` — branch on `type`, and check `live_mode` to
keep test traffic out of production ledgers. See `asyncapi/lendapi-webhooks.yml`.

## Rules and cautions

- **There is no idempotency key.** `POST /application/` is not documented as
  retry-safe. If a create times out, look the application up before retrying or you
  risk originating a duplicate.
- **Error bodies are empty.** Every operation documents only `200` and a bare `400`
  with an empty object schema. There are no documented 401/403/404/429/5xx contracts,
  so branch on the HTTP status code alone and never parse an error `code` field.
- **A decline is not an error.** `application.declined` arrives as a `200` plus a
  webhook, not as a 4xx. Treat decision outcomes and transport errors separately.
- You are handling SSNs and full consumer identity. Never log request bodies, and
  never echo the credit-report link into a shared channel.
