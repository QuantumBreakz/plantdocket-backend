# MVP Backend Delivery Phases

The `plantdocket-backend` repository holds the independent Go deployables for Library, Transfer API, Transfer Worker, Sharing and Audit. The `plantdocket-web` repository owns the employee website and session edge. The backend publishes the versioned OpenAPI contract. Each repo builds independently, and the release manifest pairs the tested web and backend digests. The [architecture](../architecture/01-system-architecture.md) assigns ownership and the [MVP PRD](../prds/02-onsite-mvp-prd.md) defines acceptance.

The [backend repository scaffold](../scaffolding/backend.md) owns the precise layout, local entry points, service databases and cross-repository release handoff. Each service owns its migrations, health checks, tests and image.

## Delivery sequence

| Phase | Deliverable | Acceptance |
| --- | --- | --- |
| B0. Runtime and contracts | Local Docker stack, separate service roles, web API handoff, test accounts, SMTP sink, S3 adapter, staging database and CI | Deploy approved web and backend artifacts to staging and show rollback. Employee browser reaches an authenticated API route. Verify signed multipart, origin policy, exact object verification and ranges. |
| B1. Authority and catalogs | Named accounts, workspace membership and folder permissions, item list/search, audit outbox. | Caller and service identities are scoped. Negative tests cover workspace ID substitution, forbidden writes, permission changes and stale sessions. |
| B2. Transfer and revision | Upload reservation, resume/complete, scan jobs, quarantine, draft registration, controlled download, history/restore and trash. | Use 500 MB fixtures, concurrent revisions, duplicate requests, lost completion responses, worker crashes, and object/database failure ordering. |
| B3. Document review | Remarks, multi-step department templates, exact-revision submission and step decisions | A clean draft is unavailable to ordinary readers before final approval. Prior effective copy stays available, and history survives rejected and concurrent decisions. |
| B4. Guest and audit | Sharing and OTP service, edge-isolated browser workflow, authorized attachment stream, revoke and audit export. | Real mail delivery, expired/replayed codes, inaccessible private routes, revoked range requests, retained archive verification and denied exports. |
| B5. Productize both installation profiles | Shared signed bundle, profile-specific routes and secrets, migrations, backup, monitoring and recovery | Clean install and compatible rollback on representative on-site and off-site targets. Independent restore, measured RTO/RPO and operator handover for each offered profile. |

## First contracts

Routes are proposed contract surfaces for `/v1`. The implementation OpenAPI defines request and response types, schemas and error behavior.

| Owner | Public API | Internal responsibility |
| --- | --- | --- |
| Library | Workspaces and memberships, folders and children, authorized search, version listing, remarks, approval templates and decisions, restore, trash, folder grants | Authorize a specific action/version, reserve a version and quota, register the exact clean draft idempotently, and release stale reservations. Own effective-version release after final approval. |
| Transfer | Create upload session, get accepted parts, sign part, complete, abort, poll status, authorize an employee download | Read the clean object authorized by Library for Sharing, pin scan input, run leased jobs, reconcile object operations and draft registration state |
| Sharing | Create/list/revoke shares on employee ingress. Redeem invite, send/verify code, and retrieve attachment on guest ingress. | Guard session/grant expiry, verify current authorization, issue scoped read requests, revoke and audit |
| Audit | Privileged event listing and export on employee ingress | Idempotent ingest of producer outbox events, audit archive writer, batch manifests and export custody |

Use stable UUIDs, cursors, UTC timestamps, bounded input and a structured error with `code` and `requestId`. Use 401 for missing or invalid credentials, 403 for denied actions on visible resources, and 404 for concealed resources. Use 409 with a stable code for state and idempotency conflicts. Scope idempotency keys to actor, workspace, and operation. A retry with the same payload returns the original operation result or documented current status. Clients distinguish dependency failures, permission loss, quota failure, and invalid version state before retrying.

## Consistency and failure handling

- Use Library's version ID as the idempotency key for draft registration. Transfer owns reconciliation of S3 multipart state. Record enough state to recover the final object and checksum if completion succeeded but its response was lost. Keep a stable reservation TTL and periodic reconciliation in Library.
- Transfer commits upload completion and its worker job together. Lease jobs atomically with a bounded expiration, retry budget and operator-visible terminal states. A resumed worker verifies the pinned staging object before scanning, and an immutable clean key makes repeated promotion observable.
- Library records immutable drafts without changing the effective revision. Step decisions validate the current department and exact revision. Final approval advances the effective pointer and commits the audit event in the same transaction. A revoked uploader cannot register a new draft.
- Write one outbox record per committed security-relevant decision. Audit deduplicates by source and event ID. Explicitly document the retry and retention policies while Audit is unavailable. Treat audit delivery lag as an operational alert.
- Render guest attachment metadata only after verification. Require recorded final approval at invitation and recheck it before each request. Range validation, timeouts, rate limits and bounded concurrent streams protect site resources. If the authoritative permission dependency is unavailable, deny the operation and surface a recoverable guest error.
- Use expand/migrate/contract schema changes. Back up before upgrades, verify a healthy former version reads compatible data, and plan database restoration for migrations that cannot be rolled back safely.

## Test and operations matrix

| Layer | Required evidence |
| --- | --- |
| Unit and contract | Permission matrix, grant/OTP state transitions, validation, provider adapter and OpenAPI consumer contracts |
| Integration | Real PostgreSQL and S3-compatible multipart, retry idempotency, outbox commit/replay, mail delivery, audit export and manifest verification |
| End to end | Employee and guest browser journeys on supported browsers, isolated sessions, failed uploads, scanner holds, unauthorized actions, three 500 MB uploads, remarks, multi-step review, and exact-version download retries |
| Site | DMZ route allowlist, secrets and network configuration, disk pressure, backup/WAL freshness, signed bundle, clean restore and rollback |
| Cloud later | Multi-AZ task placement and DB failover, regional object/DB consistency, measured standby recovery, load and cost instrumentation |

Use the [shared service contracts](../specs/00-service-contracts.md) when implementing service ownership, idempotency, authorization and cross-service handoffs. The [MVP feature specs](../prds/02-onsite-mvp-prd.md#committed-features) describe feature-specific domain behavior. The [staging implementation plan](../delivery/01-production-to-staging.md) and [verification matrix](../delivery/05-feature-verification.md) define how to exercise it.
