---
name: Link a Google Analytics account to a GMP organization
description: >-
  Authenticate against the Google Marketing Platform Admin API, find the right
  GMP organization, check whether an Analytics account is already linked, and
  create the link safely on an API that has no idempotency key.
api: openapi/google-marketing-platform-v1alpha-api-openapi.yml
operations:
  - listOrganizations
  - listAnalyticsAccountLinks
  - createAnalyticsAccountLink
generated: '2026-08-13'
method: generated
source: >-
  Grounded in operationIds verified verbatim in
  openapi/google-marketing-platform-v1alpha-api-openapi.yml, plus
  conventions/, errors/ and rate-limits/ in this repo.
---

# Link a Google Analytics account to a GMP organization

## Before you start

The API is at **ALPHA** launch stage. Shapes can change without the one-year
notice Google's API Terms give stable APIs.

## 1. Authenticate

OAuth 2.0 authorization code only — there are no API keys.

Scopes for this flow:

- `https://www.googleapis.com/auth/marketingplatformadmin.analytics.update` — required, this flow writes.

Local development uses Application Default Credentials:

```
gcloud auth application-default login \
  --scopes="https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/marketingplatformadmin.analytics.update"
```

**Set a quota project.** This API requires one and it is not set by default.
A credential that is otherwise valid will still fail without it.

Send the token as `Authorization: Bearer <access_token>` against
`https://marketingplatformadmin.googleapis.com`.

## 2. Find the organization — `listOrganizations`

```
GET /v1alpha/organizations
```

Optional `pageSize` and `pageToken`. Read `nextPageToken` from the response and
pass it back as `pageToken` to page; keep every other parameter identical
across the paged calls.

Take `name` from the organization you want. It is a full relative resource
name — `organizations/12345` — not a bare id. Every later call uses it whole.

## 3. Check for an existing link FIRST — `listAnalyticsAccountLinks`

```
GET /v1alpha/{parent}/analyticsAccountLinks
```

`parent` is the organization resource name from step 2.

This step is not optional. **The API publishes no idempotency key and
`createAnalyticsAccountLink` has no replay protection**, so a retry after an
ambiguous failure can produce a duplicate link. Listing first is the only
guard available.

Compare each returned `analyticsAccount` field against the account you intend
to link. That field is a cross-service reference into the Google Analytics
Admin API in the form `accounts/{account}`. If it is already present, stop —
there is nothing to create. Also read `linkVerificationState`: a link can exist
in `LINK_VERIFICATION_STATE_NOT_VERIFIED`, which is a real link that has not
been confirmed, not an absent one.

## 4. Create the link — `createAnalyticsAccountLink`

```
POST /v1alpha/{parent}/analyticsAccountLinks
```

Body carries an `AnalyticsAccountLink` whose `analyticsAccount` is the
`accounts/{account}` resource name. `name` and `displayName` are assigned by
the service — do not send them.

Requires the `...analytics.update` scope. Consumes both a request quota and a
write quota.

## 5. Handle errors

Errors are the **Google JSON envelope**, not RFC 9457 problem+json:

```json
{"error": {"code": 403, "message": "...", "status": "RESOURCE_EXHAUSTED",
           "errors": [{"reason": "rateLimitExceeded", "domain": "usageLimits"}]}}
```

Branch on `error.errors[].reason` or `error.status`. Never branch on
`error.message`.

- **403 with reason `rateLimitExceeded`** — quota exhausted. This API returns
  **403, not 429**. Do not treat it as a permanent authorization failure. Wait
  one minute and retry; quotas refill at the start of each 60-second interval.
  Limits are 1,200 requests/min and 300 writes/min per project, 600 and 120 per
  user. There are no rate-limit response headers to read.
- **403 with any other reason** — missing `...analytics.update` scope, no
  access to the organization, the API not enabled on the project, or no quota
  project set. Not retryable.
- **401** — expired or missing token. Refresh, do not retry blindly.
- **409 / ALREADY_EXISTS** — the link exists. Go back to step 3 rather than
  retrying the create.
- **400 / INVALID_ARGUMENT** — usually a bare id where a full resource name
  belongs, or a missing required field.
- **503 UNAVAILABLE / 500 UNKNOWN** — retry with exponential backoff. These are
  the only two codes in Google's own generated-client retry policy: 5 attempts,
  1s initial backoff, 60s max, multiplier 1.3, 60s timeout.

## 6. Confirm

Re-run step 3 and confirm the new `analyticsAccount` is present. There is no
event, webhook or callback on this API — polling the list is the only
confirmation channel.
