---
name: Audit GMP organizations and their Analytics account links
description: >-
  Read-only sweep across every Google Marketing Platform organization the
  caller can reach, enumerating linked Google Analytics accounts and their
  verification state, for governance and inventory.
api: openapi/google-marketing-platform-v1alpha-api-openapi.yml
operations:
  - listOrganizations
  - listAnalyticsAccountLinks
generated: '2026-08-13'
method: generated
source: >-
  Grounded in operationIds verified verbatim in
  openapi/google-marketing-platform-v1alpha-api-openapi.yml, plus this repo's
  conventions/, errors/ and rate-limits/ artifacts.
---

# Audit GMP organizations and their Analytics account links

A read-only inventory pass. Nothing here mutates state, so it needs only the
read scope and can run unattended.

## 1. Authenticate with the read scope only

```
gcloud auth application-default login \
  --scopes="https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/marketingplatformadmin.analytics.read"
```

Use `https://www.googleapis.com/auth/marketingplatformadmin.analytics.read`,
not the update scope. An audit that holds a write scope it does not need is a
finding in its own right.

Set a Cloud quota project on the credential — required, and not set by default.

## 2. Enumerate organizations — `listOrganizations`

```
GET /v1alpha/organizations
```

Page fully:

1. Call with `pageSize`.
2. If the response carries `nextPageToken`, call again with that value as
   `pageToken` and **every other parameter unchanged**.
3. Stop when `nextPageToken` is absent or empty.

Record each organization's `name` (`organizations/{org_id}`) and
`displayName`.

## 3. Enumerate links per organization — `listAnalyticsAccountLinks`

For each organization resource name from step 2:

```
GET /v1alpha/{parent}/analyticsAccountLinks
```

Page the same way. For each `AnalyticsAccountLink` record:

- `name` — `organizations/{organization}/analyticsAccountLinks/{link}`
- `displayName` — output only, the Analytics account's display name
- `analyticsAccount` — `accounts/{account}`, a cross-service reference into the
  Google Analytics Admin API
- `linkVerificationState` — one of
  `LINK_VERIFICATION_STATE_VERIFIED`,
  `LINK_VERIFICATION_STATE_NOT_VERIFIED`,
  `LINK_VERIFICATION_STATE_UNSPECIFIED`

## 4. What to flag

- Links sitting in `LINK_VERIFICATION_STATE_NOT_VERIFIED` — a link exists but
  the Analytics side has not confirmed it.
- The same `analyticsAccount` appearing more than once under one organization —
  this API has no idempotency key, so duplicate links are a real outcome of a
  retried `createAnalyticsAccountLink`.
- Organizations with zero links.

## 5. Stay inside the quota

Reads consume the request quotas only: 1,200/min per project and 600/min per
user, refilled every 60 seconds. A wide sweep across many organizations is the
realistic way to hit them.

On **HTTP 403 with `error.errors[].reason == "rateLimitExceeded"`**, pause 60
seconds and resume from the last `pageToken`. Do not abort — this API uses 403
rather than 429 for quota exhaustion, and it returns no rate-limit headers, so
there is nothing to read ahead on. A 403 carrying any other reason is a real
permission problem and should stop the run.

Retry `503 UNAVAILABLE` and `500 UNKNOWN` with exponential backoff (5 attempts,
1s initial, 60s max, 1.3 multiplier). Do not retry `400`, `401` or `404`.

## 6. Extend the audit beyond this repo's OpenAPI

Two further read methods are published in the provider's Discovery Document but
are absent from the OpenAPI in this repo, so they have no operationId to bind
to here. Call them via the Discovery Document or a GAPIC client:

- `organizations.reportPropertyUsage` — per-property `totalEventCount`,
  `billableEventCount`, `serviceLevel` and `propertyType`, plus aggregate
  `BillInfo`. This is where an audit gets cost exposure.
- `organizations.findSalesPartnerManagedClients` — client organizations managed
  by a sales-partner organization, with start and end dates.
