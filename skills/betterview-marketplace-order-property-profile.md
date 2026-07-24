---
name: Order and retrieve a Betterview property profile
description: >-
  Authenticate to the Betterview Property Intelligence API, order a property
  profile for one or more addresses, poll the order status, and retrieve the
  scored profile (Roof Spotlight Index, peril risks, flags, Partner Connect
  data).
api: https://dev.betterview.com/reference
method: generated
source: https://betterview.readme.io/llms.txt
operations:
  - getproductsrequest-1
  - addprofilerequest
  - get_api-properties-bulk-status-id
  - get_api-properties-recent
  - pullfullprofilerecord
  - get_api-properties-quota
---

# Order and retrieve a Betterview property profile

Operating instructions for an agent using the Betterview Property Intelligence
API. Every operation id below is verbatim from the published Betterview API
reference (`https://dev.betterview.com/reference`). No OpenAPI bundle is
published, so treat request/response shapes as documented on each reference
page, not as fabricated here.

## 1. Authenticate
Follow Betterview's authentication workflow to obtain a **bearer token**
(`https://betterview.readme.io/docs/authentication`). Send it in the
`Authorization` header on every request. See
`authentication/betterview-marketplace-authentication.yml`.

## 2. Confirm available products (optional)
Call **`getproductsrequest-1`** (List Products) to get the array of product
names that can be applied to an order, so you request products the account is
entitled to.

## 3. Check quota (optional)
Call **`get_api-properties-quota`** to read remaining quota credits (or
`unlimited`) before submitting a large order.

## 4. Order profiles
Call **`addprofilerequest`** (Run Profile(s)) with an array of addresses to
score. Prefer the **asynchronous callback pattern**: supply a callback path and
let Betterview push final results when scoring completes — this is the
provider-recommended flow (see
`conventions/betterview-marketplace-conventions.yml`). Properties may include
optional `policy` and `account` objects to tie insights to policy data.

## 5. Track the order
Poll **`get_api-properties-bulk-status-id`** (Retrieve order status) for the
order id. The response reports the total property count, how many are done, and
which are still processing — the array fills in as scoring completes. You can
also list recently generated profiles with **`get_api-properties-recent`**
(default past 30 days; `start`/`end` widen the window up to 90 days).

## 6. Retrieve the profile
When results are ready (or if not using callbacks), call
**`pullfullprofilerecord`** (Retrieve Profile full) for a property to read the
complete scored profile, including the Roof Spotlight Index (RSI), peril risks,
flags, and Partner Connect third-party data. Fields that are not yet available
may be `null`.

## Conventions & limits
- Respect the documented rate limits (`https://betterview.readme.io/docs/ratelimiting`).
- No idempotency-key contract is documented; avoid blind retries of
  `addprofilerequest`.
- Betterview discourages on-demand pulls in favor of callbacks; use pulls only
  for backwards compatibility.
