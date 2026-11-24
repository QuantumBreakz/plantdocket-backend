# F03 Resumable Browser Upload

**Problem and audience.** Contributors need to transfer files reliably when a network connection drops or the employee workspace reloads.

**Use-case scenario.** A contributor chooses a 500 MB document. The connection drops midway. After returning to the workspace and reselecting the same file as needed, the contributor continues from confirmed parts.

**Product flow.** Choose destination → select file → see transfer progress → pause, resume or cancel → see scan-pending status after completion.

**Use cases.** Initial upload, create a candidate file revision, resume after interrupted connectivity, retry failed parts, and cancel. **Non-use cases.** Automatic folder backup, continuous background uploads, content deduplication, and upload by guests.

## Requirements

- **FR1.** The API authorizes folder and workspace access, enforces a 500 MB limit (500,000,000 bytes) and reserves quota before issuing a session. Recheck reservation, authorization and actual size at finalization. Reject oversized objects before publication and release quota through expiry or cleanup.
- **FR2.** Use unique staging keys, bounded multipart part signing, and API-owned completion. Return accepted part numbers for resume. Reissue credentials scoped to that upload and reconcile lost completion responses without creating a duplicate version.
- **FR3.** Record the exact immutable staging object and verified byte count before committing the scan job. The worker independently verifies SHA-256 before release. Require a matching selected file to resume if original access is unavailable.
- **NFR1.** Support three parallel 500 MB uploads using bounded browser and API memory. Scope URLs to approved hostname, part, operation and expiry.
- **NFR2.** Cancel and expire abandoned sessions. Reconcile database/objects, limit retries, and alert on durable staging accumulation. Redact credentials and signed URLs.

**Impact and acceptance.** After interrupting each of three concurrent transfers, all can resume with accepted parts preserved. Duplicate completion creates one scan job. Oversized, unauthorized and corrupt files are rejected or held. Measure resumed success and retransmitted bytes.

**Dependencies.** [F02](F02-library.md), [F04](F04-scan-publish.md), [F08](F08-folder-permissions.md) and verified S3-compatible multipart support. [S3 multipart overview](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html).

## Contract and decisions

| Operation | Expected behavior |
| --- | --- |
| `POST /v1/workspaces/{workspaceId}/uploads` | Accept folder ID, name, byte length and streaming-calculated SHA-256. For a replacement, require existing file ID and expected effective revision. Require an idempotency key. Return 201 with session ID, expiry and part size. |
| `GET /v1/uploads/{uploadId}` | Return session state and confirmed parts only to the authorized originator. |
| `POST /v1/uploads/{uploadId}/parts:sign` | Sign one or a bounded batch of part numbers for the exact upload. Verify authorization, length policy and session status before issuing a URL. |
| `POST /v1/uploads/{uploadId}:complete` | Require an idempotency key and the ordered list of part numbers and provider receipts. Return 202 with the candidate and current finalization or scan status. Duplicate completion returns the same operation's status. |
| `DELETE /v1/uploads/{uploadId}` | Cancel a receiving session, enqueue cleanup and return 204. Return 409 when finalization already won the state change. |

Start with 16 MiB parts, 24-hour sessions, five-minute signed part URLs and two part streams per browser upload. Supported size is 1 through 500,000,000 bytes inclusive. Use opaque staging locators with a workspace, principal and version binding. Require current access before signing or finalizing. Size above the bound returns 413, malformed or expired sessions are actionable, and quota or terminal-state conflicts return 409 with a stable error code. An idempotency key repeated with a different payload returns 409. A multipart ETag is storage metadata. Use confirmed size and independently verified exact byte digest in the ingest path.

The browser uses bounded file slices and incremental SHA-256. Transfer control requests are same-origin. File parts go directly to the approved object endpoint. Allow the exact employee origin and necessary methods and headers. Expose the multipart receipt header needed by the browser, initially `ETag` where supported. CORS complements signed-operation admission. Keep the object bucket private. Capture no signed URL in browser persistence or telemetry. Persist only an opaque resume reference scoped to the employee and site, and reauthorize after page reload. [S3 CORS configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/ManageCorsUsing.html).

### Records and ownership

| Owner | Record | Integrity rule |
| --- | --- | --- |
| Library | Candidate, destination/name reservation, and byte reservation | One candidate per upload ID. Lock quota accounting and the destination claim in the reservation transaction. Store the uploader, declared size, target file and expected effective revision. |
| Transfer | Upload session and creation intent | One session per owner and idempotent request, with candidate ID, unique staging key, declared digest, expiry, state revision and provider handle after initialization |
| Transfer | Finalization evidence and scan job | Exact staging locator, observed size, expected SHA-256, completion fingerprint and one scan job per candidate. Commit the staging observation and scan job together. |

Use Library to check `chargedBytes + reservedBytes + requestedBytes` before reserving. The receiving window is 24 hours by default. A completed receiving session moves the reservation into a pending-draft-registration state. Only successful draft registration charges the bytes. A rejected, held or canceled unregistered candidate releases its reservation through an idempotent Library transition. Retained quarantine remains in the physical storage budget. Library owns release for a lost session after consulting Transfer and the deadline; a stale cleanup cannot release or delete a registered draft object.

### Lifecycle and failure handling

1. **Initiate.** Persist a Transfer creation intent before calling Library, and reuse the same upload ID through retries. After Library confirms the reservation, create and record the multipart handle. If a provider call succeeds but the response is lost, reconcile known handles for the staging key where supported. Expire any surplus incomplete handles through scoped cleanup. Expose part signing only after the session is `receiving`.
2. **Receive and resume.** Derive legal part numbers and expected lengths from declared size and fixed part size. Cap signed batches and TTL at session expiry. List and validate provider receipts and part sizes for resume. Treat client reports as hints. For a full-size upload, require all expected parts, exact total length and the correct final-part size. Pin approved signed host and operation. Require the same original byte length and digest before resuming a reselected file.
3. **Finalize.** Move `receiving → finalizing` by compare-and-set, persist the completion fingerprint and recheck scope. Finalization owns the candidate until it reaches `scan_pending` or terminal rejection. Parse the provider outcome, even if the HTTP status alone indicates success. If the call outcome is ambiguous, look for the completed object under the unique staging key, verify immutable identity and size, and confirm the corresponding upload state before retrying or rejecting. A provider error or unconfirmed object never creates a scan job. [S3 CompleteMultipartUpload](https://docs.aws.amazon.com/AmazonS3/latest/API/API_CompleteMultipartUpload.html).
4. **Cancel and expire.** Cancel or expire by winning the state transition while receiving. Deny new signatures immediately. A previously issued signed part can race with abort. Recheck the handle and staging prefix until cleanup converges, then release the Library reservation once. Track unresolved provider work and alert rather than claiming deletion from a single abort response.

Separate admission indicators from business outcome. `scan_pending` means a verified-size staged object has been handed off. F04 owns integrity verification and draft registration. F12 owns approval and release. Record stuck finalizations, reserved versus charged bytes, orphan handles, resumed transfer rate, and byte retransmission with correlation IDs. A required authority returning 503 leaves durable intent retriable.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F03-A01 | Three authenticated browser sessions upload 500,000,000-byte fixtures. Interrupt connections and reload sessions. | All resume from accepted parts after reauth and file reselection as needed, register one clean draft each, and preserve exact checksums. Record retransmission and peak memory. |
| F03-A02 | Create a session at 500,000,001 bytes, or finalize with bytes that exceed the authorized length | Initial oversize returns 413. Mismatched final size stays quarantined, registers no draft, and releases quota after reconciliation. |
| F03-A03 | Lose the completion response after storage has committed and retry with the same key | One immutable staging object and candidate, one scan job, and the existing status returned without a second upload completion |
| F03-A04 | Cancel a partial upload or advance time beyond the fixture's expiry | Further part signing is denied, pending parts are aborted, and the reservation is released once |
| F03-A05 | Try a different payload under the same idempotency key, or resume a session belonging to a different workspace | 409 idempotency conflict for the changed payload, 404 for wrong scope, and no new signed URL or reservation |
| F03-A06 | Two upload requests race for the remaining workspace quota | Only a valid reservation fits the limit. The other receives `409 quota_exceeded`. No hidden over-reservation exists. |
| F03-A07 | Cancel races finalization and a previously signed part arrives late | Exactly one state path wins. If cancellation wins, no scan or draft registration occurs and storage/reservation cleanup converges. |
| F03-A08 | Lose the Library reservation response before the Transfer session is ready | Repeating initiation recovers the same reservation and candidate. An abandoned creation intent reaches cleanup within its configured window. |
| F03-A09 | Supply missing, reordered, wrong-sized or stale part receipts | The API rejects invalid completion or returns a safe retry. No durable verified staging observation or scan job is created. |
| F03-A10 | Storage reports an uncertain or embedded completion error while no complete exact object can be verified | Status remains retriable or becomes visibly rejected after reconciliation. No downstream scan job is emitted. |
| F03-A11 | Upload a signed part from the approved employee origin and exercise the browser flow from an unapproved origin | Approved browser can send the permitted part and read its receipt. The unapproved browser receives no usable cross-origin transfer response, and unsigned access remains denied. |
