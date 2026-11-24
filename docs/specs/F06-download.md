# F06 Employee Download

**Problem and audience.** Employees need the exact authorized document revision and a reliable way to retry an interrupted browser download.

**Use-case scenario.** A viewer opens the effective approved revision, receives time-limited access, and saves it through the browser's download flow.

**Product flow.** Open an authorized file → choose the effective copy or a specifically authorized history or review revision → request download → save or open under site browser policy.

**Use cases.** Download an authorized revision, renew an expired authorization after rechecking permission, and retry an interrupted transfer. **Non-use cases.** Offline synchronized libraries, public URLs, access to quarantined content, and control over copies exported outside the application.

## Requirements

- **FR1.** Transfer checks current Library permission and exact clean object identity before signing a time-limited read. Resolve the effective approved revision once and pin it across retries. Serve an appropriate safe filename and attachment disposition.
- **FR2.** Deliver the attachment using the browser download mechanism. A retry that needs a fresh URL reauthorizes the same revision. Use supported browser range behavior or restart the exact revision.
- **FR3.** Capture download authorization and detect observable transfer failures. Provide simple recovery for expiry, revoked access and unavailable storage.
- **NFR1.** Starting presigned URL lifetime is 60 seconds. Document that a transfer already started can continue and that a valid bearer URL can be reused within its scope/lifetime.
- **NFR2.** Support 500 MB downloads without buffering the whole object and ensure exact byte integrity. Avoid caching sensitive responses in the application edge.

**Impact and acceptance.** A viewer retrieves the expected bytes, retries a stalled download and cannot obtain a new URL after losing access. Expired URL, trashed file and pending revision cases remain denied. Track successful downloads and retry rates separately from download authorizations.

**Dependencies.** [F04](F04-scan-publish.md), [F05](F05-versions-trash.md), [F08](F08-folder-permissions.md), Transfer and a reachable object endpoint. [Presigned URL behavior](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html).

## Contract and decisions

`POST /v1/files/{fileId}/downloads` takes an optional `versionId`. Return 200 with a short-lived URL, expiry, immutable version ID, safe filename, byte count and SHA-256. Resolve the current version once when the client omits it. A retry includes the resolved ID. Return 404 for concealed resources or unavailable versions and 503 for storage signing dependencies that prevent safe authorization.

Use a browser attachment response so the web page does not buffer the entire file. The browser controls temporary downloads. After a failed transfer or URL expiry, the workspace offers a fresh authorized download for the same exact version. A direct browser save does not require page-level access to the saved file. Verify byte count and SHA-256 in staging by comparing the actual downloaded artifact. Site policy governs exported copies. Revocation blocks new authorizations.

### Capability issuance

1. Resolve the requested file inside its workspace and validate content-read scope through Library. Resolve an omitted version to the file's effective approved revision. Reject trash, unresolved recovery records, unscanned or held content and concealed resources. Approved historical copies require scoped history access. A draft or returned version requires a separate scoped review or contributor decision; ordinary readers receive the effective approved copy. Library returns the exact logical object ID, approved locator, byte count and checksum for the authorized version.
2. Validate that the object can be addressed through the approved employee storage endpoint. Issue a method- and exact-object-scoped URL with the shorter of the configured 60-second lifetime and credential validity. Encode a safe attachment name without control characters. Keep the signing response private and noncacheable. Exclude storage access material from error bodies, telemetry and archives.
3. Record an authorization with workspace, principal, file/version, request ID and authorization outcome before returning the capability. If storage sign or required audit-intent persistence fails, return 503 without a usable URL. A retried authorization generates fresh credentials only after a current policy decision. Authorization telemetry records the capability issuance without claiming byte delivery.

### Renewal and consistency

Renew against the exact version selected earlier. A change to the file's current pointer does not alter an existing transfer. An absent or mismatched stored version is a data integrity fault. Return a recoverable error, alert with an opaque locator ID, and never fall back to another version. Test provider behavior for initial read, supported range requests, signed URL expiry and pinned immutable identity.

Start all fresh employee downloads and renewals with authorization. An issued bearer URL may still be used within its TTL, and a request already underway may finish after expiry or revocation. Guest streaming follows the stricter F07 request boundary. [Presigned URL behavior](https://docs.aws.amazon.com/AmazonS3/latest/userguide/using-presigned-url.html).

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F06-A01 | An eligible viewer downloads the current approved version on the supported browsers | Captured artifacts match expected byte count and SHA-256. Each download authorization records the resolved version ID. |
| F06-A02 | Interrupt a large browser download, approve a new effective revision and retry | Browser resumes or the workspace reissues the original authorized revision. Captured output never combines parts across revisions. |
| F06-A03 | Advance past URL expiry or revoke the viewer's folder access before renewal | Expired query requires reauthorization. Revoked access cannot obtain a new signature. |
| F06-A04 | Attempt to download staged or trashed content, try signing a clean draft as an ordinary reader, and force a malformed range | Revision availability is denied and no clean locator is disclosed. Range failure produces a safe retry. |
| F06-A05 | Object storage loses the exact resolved version while a newer version exists | Signing or retrieval reports the approved recoverable error, alerts on the missing object and never serves the newer object in its place. |
| F06-A06 | Revoke a viewer after issuing a download URL, then request a second signed URL | New signing is denied. The first URL follows its bounded provider lifetime, and audit records issuance without asserting delivery. |
| F06-A07 | Make the signing dependency or required audit-intent write fail | A 503 response contains no usable URL or storage key; retry succeeds only after a fresh authorization. |
