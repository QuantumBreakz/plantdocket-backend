# Shared Service Contracts for the MVP

**Status.** Proposed implementation baseline. The [MVP scope](../mvp/01-scope.md) owns product commitments. These contracts apply across the feature specs and define how a request, job or release is judged correct when multiple services are involved. Confirm initial settings in staging.

## Data ownership

| Authority | Records and decisions | Callers use |
| --- | --- | --- |
| Employee web | Employee browser login transactions and ephemeral sessions. Holds API credentials server-side and forwards approved routes. | A host-scoped session cookie and same-origin request controls. Business authorization stays with the owning services. |
| Library | Principals and memberships, folder policies, catalog, immutable revisions, remarks, approval templates and step decisions, effective pointers, trash and quota | Scoped APIs and commands. Final approval and effective-pointer change occur in one Library transaction. |
| Transfer | Storage upload sessions and multipart handles, immutable object observations, scan jobs and leases, clean-object preparation, and object cleanup | Versioned APIs and scoped signing or streaming interfaces. Only Library marks a revision available to users. |
| Sharing | Guest grants, OTP challenges, sessions, revocation, invitations and stream observations | Grant creation and every guest download recheck exact final approval and current Library policy. |
| Audit | Event ingestion, searchable timeline, export jobs, archive manifests and verification | Versioned event ingestion and authorized query/export. The producer remains authoritative for the business decision. |
| Site provisioning | Installation ID, environment markers, routes, selected named-account mode, resource configuration and recovery policy | Provisioning records and scoped verification. Runtime identities can read their environment markers. |

Each authority owns its database and migrations. The first site may host those databases on one PostgreSQL instance. Read another service's records through its contract. A durable locator is installation-, workspace-, and version-scoped. Logical IDs survive a restore. Recovered provider locators are verified and mapped before they become eligible for delivery.

## Request, authorization and retry rules

- Use stable opaque resource IDs, UTC timestamps and byte counts as integers. Version routes and schemas. Validate syntax, authenticate, resolve scope and authorize the specific action before returning any protected fields. Every service receives an authenticated employee, scoped service identity, or verified guest session appropriate to the operation.
- Use a stable error body with `code` and `requestId`, and safe per-field details where appropriate. A 400 covers malformed input or a bad cursor, 401 covers missing or invalid employee credentials, 403 covers a disallowed action on a visible resource, and 404 conceals an inaccessible or absent resource. Use 409 for version, state, quota or idempotency conflicts and 413 for an oversize request. Use 429 for admission limits with `Retry-After` when available. Use 503 when a required authority or dependency cannot safely complete a decision.
- Where a contract requires an idempotency key, scope it to installation, workspace, actor and operation. Store a fingerprint of the validated semantic request. Serialize competing claims for the same key. Return the committed result or documented operation status for the same input. Return `409 idempotency_conflict` for different input. Reauthorize before revealing a replayed result. Generate fresh short-lived capabilities only after current authorization. Keep ordinary mutation keys for an initial minimum of 48 hours after completion. Upload keys survive their session and reconciliation window. Consumer deduplication is based on durable business IDs.
- Order changes that share a constraint at the authority that owns it, using row locks or a versioned compare-and-set and database constraints. Choose and document consistent lock order. Couple the authoritative write and its outbox event in one transaction. External calls and object copies run outside that transaction. Persist intent before side effects, retry with stable identifiers, and reconcile ambiguous outcomes before compensating or creating replacements.
- An authorization decision begun after a relevant revocation commits must deny the affected action. A request admitted before that boundary follows the feature's in-flight policy. Library checks current grants inside authoritative mutations. Calls across services bind caller, action and exact resource. Cached decisions cannot authorize a signature, guest stream, export, draft registration or new approval step.

## Cross-service handoffs

| Handoff | Durable identity | Completion rule |
| --- | --- | --- |
| Upload reservation | Upload ID, candidate version ID and reservation owned by Library | Same upload ID recovers the reservation after an ambiguous response. Library arbitrates charge, expiry and release. |
| Storage completion to scan | Immutable staging locator and candidate version ID | Transfer commits staging observation and one scan job together. Independently verify digest and coverage before draft registration. |
| Scan to Library draft | Candidate revision ID and verified clean locator | Record one immutable draft. The effective version stays unchanged. Resolve ambiguous replies before cleanup. |
| Library approval to effective revision | Exact revision, ordered approval template and step decision IDs | Intermediate approvals advance the submission. Final approval updates the effective pointer and outbox atomically. |
| Library and Sharing to delivery | Workspace, exact revision, authorization and release state | Ordinary download selects the effective copy. Guest delivery rechecks final approval of the pinned exact version. |
| Removal and cleanup | Candidate, logical object record or trash record and purge intent ID | Lock the authoritative reference state before admitting purge. Storage cleanup targets the exact provider object/version and returns durable evidence. |
| Service to Audit | Authenticated producer, event ID and producer stream sequence | Accept at-least-once delivery. Acknowledge only after durable deduplication. Archive verification accounts for missing sequence ranges. |

## Operational evidence

Log request and job correlation IDs, component, outcome, duration and safe reason codes. Use aggregate metrics for queue age, retries, policy denials and provider availability, without personal data in metric labels. Keep access/signed tokens, OTP material, object access URLs and file contents out of logs, traces and evidence. Use synthetic cases and stage identities. Record implementation choices that change user-visible outcomes in the relevant feature spec and update the [acceptance matrix](../delivery/05-feature-verification.md).
