---
name: Set an Analytics property service level
description: >-
  Move a Google Analytics property between the STANDARD and paid 360 service
  levels through the Google Marketing Platform Admin API, with the checks a
  commercially consequential, non-idempotent write needs.
api: openapi/google-marketing-platform-v1alpha-api-openapi.yml
operations:
  - listOrganizations
  - listAnalyticsAccountLinks
  - setPropertyServiceLevel
generated: '2026-08-13'
method: generated
source: >-
  Grounded in operationIds verified verbatim in
  openapi/google-marketing-platform-v1alpha-api-openapi.yml, enum values from
  grpc/google-marketing-platform-resources.proto, and this repo's conventions/,
  errors/ and plans/ artifacts.
---

# Set an Analytics property service level

## What this actually does

`setPropertyServiceLevel` moves a Google Analytics property between
`ANALYTICS_SERVICE_LEVEL_STANDARD` and `ANALYTICS_SERVICE_LEVEL_360`. The 360
level is the **paid** Analytics tier, sold on annual contracts through Google
sales or a reseller. Google publishes no price list for it.

Treat this as a **human-in-the-loop** operation. An agent should propose it and
get explicit confirmation, never execute it unattended.

## 1. Authenticate

OAuth 2.0 authorization code. Required scope:

- `https://www.googleapis.com/auth/marketingplatformadmin.analytics.update`

Set a Cloud quota project on the credential — this API requires one and does
not set it by default. Send `Authorization: Bearer <access_token>` to
`https://marketingplatformadmin.googleapis.com`.

## 2. Resolve the organization — `listOrganizations`

```
GET /v1alpha/organizations
```

Page with `pageSize` / `pageToken` / `nextPageToken`. Keep the `name` value
whole: `organizations/12345`.

## 3. Resolve the account link — `listAnalyticsAccountLinks`

```
GET /v1alpha/{parent}/analyticsAccountLinks
```

`parent` is the organization resource name. The link you want is identified by
its `name`:

```
organizations/{organization}/analyticsAccountLinks/{analytics_account_link}
```

Check `linkVerificationState`. A link in
`LINK_VERIFICATION_STATE_NOT_VERIFIED` has not been confirmed by the Analytics
side, and service-level changes against it should be treated as unsafe.

## 4. Confirm the current state before writing

There is no `getProperty` on this API. Read the current service level from the
usage report before changing anything:

```
POST /v1alpha/{organization}:reportPropertyUsage
```

This method is published in the provider's Discovery Document (added
2025-10-29) but is **not** in this repo's OpenAPI — call it against the
Discovery Document or a GAPIC client rather than looking for an operationId
here. It returns `PropertyUsage` rows carrying `property`, `serviceLevel`,
`propertyType`, `totalEventCount` and `billableEventCount`, plus an aggregate
`BillInfo` with `baseFee`, `eventFee`, `priceProtectionCredit` and `total`.

Present the current level and the billable event count to the human before
proposing a change. A move to 360 changes what the account is billed.

## 5. Set the level — `setPropertyServiceLevel`

```
POST /v1alpha/{analyticsAccountLink}:setPropertyServiceLevel
```

`analyticsAccountLink` is the full link resource name from step 3. The request
body carries the target property and the service level. Both fields are marked
REQUIRED in the provider's protos; omitting either returns
`400 INVALID_ARGUMENT`.

Valid service levels:

- `ANALYTICS_SERVICE_LEVEL_STANDARD`
- `ANALYTICS_SERVICE_LEVEL_360`

`ANALYTICS_SERVICE_LEVEL_UNSPECIFIED` is the proto zero value, not a
settable target.

This is a **declarative set**, so re-sending the same target level is
harmless — it is the one write on this API that is naturally safe to repeat.
It still has no idempotency key, so a retry that races a concurrent change
will silently win.

## 6. Handle errors

Google JSON error envelope, not RFC 9457. Branch on
`error.errors[].reason` or `error.status`.

- **403 `rateLimitExceeded`** — quota, not permission. This API signals
  exhaustion with 403, never 429, and returns no rate-limit headers. Writes are
  capped at 300/min per project and 120/min per user. Wait a minute, retry.
- **403 other reasons** — missing `...analytics.update` scope, no access to the
  organization, API not enabled, or no quota project. Not retryable.
- **404 NOT_FOUND** — the link resource name is wrong. Re-run step 3; a bare id
  instead of a full relative resource name is the usual cause.
- **400 INVALID_ARGUMENT** — missing required field or an unrecognized service
  level enum value.
- **503 UNAVAILABLE / 500 UNKNOWN** — retryable. Google's own generated clients
  use 5 attempts, 1s initial backoff, 60s max, 1.3 multiplier, 60s timeout, and
  retry only these two codes.

## 7. Confirm

Re-run `reportPropertyUsage` and confirm the property's `serviceLevel` is the
target value. There are no events or webhooks on this API — re-reading is the
only confirmation available.
