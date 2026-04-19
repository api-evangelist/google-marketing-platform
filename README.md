# Google Marketing Platform Admin (google-marketing-platform)
The Google Marketing Platform Admin API provides programmatic access to manage links between Google Marketing Platform organizations and Google Analytics accounts. It enables creating, updating, deleting, and listing organization links and managing service levels for integrated marketing analytics.

**URL:** [Visit APIs.json URL](https://raw.githubusercontent.com/api-evangelist/google-marketing-platform/refs/heads/main/apis.yml)

**Run:** [Capabilities Using Naftiko](https://github.com/naftiko/fleet?utm_source=api-evangelist&utm_medium=readme&utm_campaign=company-api-evangelist&utm_content=repo)

## Tags:

 - Analytics, Google Marketing Platform, Marketing, Organization Management, Platform Administration

## Timestamps

- **Created:** 2026-03-13
- **Modified:** 2026-04-18

## APIs

### Google Marketing Platform Admin API
The Marketing Platform Admin API enables programmatic management of organization-level settings including links to Google Analytics accounts, service level configuration, and organization administration.

**Human URL:** [https://developers.google.com/marketing-platform/devguides/api/admin/v1/rest](https://developers.google.com/marketing-platform/devguides/api/admin/v1/rest)

#### Tags:

 - Admin, Analytics, Organizations

#### Properties

- [Documentation](https://developers.google.com/marketing-platform/devguides/api/admin/v1/rest)
- [OpenAPI](openapi/openapi.yml)

## Common Properties

- [Portal](https://marketingplatform.google.com)
- [GettingStarted](https://developers.google.com/marketing-platform/devguides/api/admin/v1/rest)
- [Documentation](https://developers.google.com/marketing-platform)
- [Authentication](https://developers.google.com/docs/api/how-tos/authorizing)
- [Pricing](https://marketingplatform.google.com/about/)
- [TermsOfService](https://developers.google.com/terms)
- [PrivacyPolicy](https://policies.google.com/privacy)
- [StatusPage](https://status.cloud.google.com/)
- [Support](https://developers.google.com/marketing-platform/support)

## Features

| Name | Description |
|------|-------------|
| Organization Management | List and manage Google Marketing Platform organizations with programmatic access to organization settings. |
| Analytics Account Linking | Create, list, and delete links between Marketing Platform organizations and Google Analytics accounts. |
| Service Level Configuration | Set and manage Analytics property service levels including standard and 360 tier assignments. |
| Multi-Organization Access | Access and manage multiple Marketing Platform organizations from a single authenticated session. |

## Use Cases

| Name | Description |
|------|-------------|
| Enterprise Analytics Setup | Programmatically link Google Analytics accounts to Marketing Platform organizations for enterprise-scale deployments. |
| Service Tier Management | Automate the assignment of Analytics 360 service levels to properties across large organizations. |
| Organization Auditing | List and audit all Marketing Platform organizations and their linked Analytics accounts for governance. |

## Integrations

| Name | Description |
|------|-------------|
| Google Analytics | Direct linking and service level management for Google Analytics accounts within Marketing Platform organizations. |
| Google Tag Manager | Part of the Google Marketing Platform suite for tag management and measurement integration. |
| Display and Video 360 | Integrated advertising platform within Google Marketing Platform for programmatic media buying. |
| Search Ads 360 | Search campaign management platform integrated with Marketing Platform for cross-channel analytics. |

## Artifacts

Machine-readable API specifications organized by format.

### OpenAPI

- [Google Marketing Platform Admin API](openapi/openapi.yml)

### JSON-LD

- [Google Marketing Platform Context](json-ld/json-ld.yml)

## Capabilities

Naftiko capabilities organized as shared per-API definitions composed into customer-facing workflows.

### Shared Per-API Definitions

- [Google Marketing Platform Admin API](capabilities/shared/admin-api.yaml) -- 5 operations for organization and analytics management

### Workflow Capabilities

| Workflow | APIs Combined | Tools | Persona |
|----------|--------------|-------|---------|
| [Marketing Analytics Administration](capabilities/marketing-analytics.yaml) | Admin API | 5 | Marketing Platform Admin |

## Maintainers

**FN:** Kin Lane

**Email:** kin@apievangelist.com
