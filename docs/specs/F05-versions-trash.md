# F05 Version History, Restore, and Trash

**Problem and audience.** Factory teams need to trace every document change and recover prior content while retaining the approved copy needed for operations.

**Use-case scenario.** A floor layout was approved and later superseded. An engineer restores its earlier bytes as a new draft. The current approved layout remains effective until the restored draft completes its own approval chain.

**Product flow.** Open document → view authorized revision history → choose earlier bytes → create a restore draft → add remarks and submit for approval. Trash and recovery are separate document actions.

**Use cases.** Trace revisions and status, recover prior bytes, trash a document and recover it within retention. **Non-use cases.** Overwriting older versions, automatic approval on restore, editing source drawings and unrestricted permanent deletion.

## Requirements

- **FR1.** Keep a stable document ID and monotonic revision numbers. Record revision author, time, source digest, status, and restore provenance. Once recorded, content cannot be overwritten. Constituents such as remarks, review submissions and decisions link to the exact revision. A failed upload or held scan remains in authorized processing evidence.
- **FR2.** Distinguish draft, in review, returned, effective and superseded. Only the final approval transaction in [F12](F12-revision-approval.md) advances the effective pointer. Recording a clean object, restoring earlier bytes or adding remarks has no such effect.
- **FR3.** Trash hides document and delivery, ends pending review and invalidates old external grants. Restore from trash may reinstate its previously approved effective revision after scope and policy checks. Restoring earlier bytes as a new update always makes a fresh draft.
- **NFR1.** Retry safely with idempotency and conditional revision metadata. Distinguish version creation from final approval. Concurrent uploads may produce separate drafts. Approval serializes per document.
- **NFR2.** Keep decision history and required records through retention and restore. Honour references, site retention and preservation before purging an exact object version.

## Contract and consistency

| Operation | Expected behavior |
| --- | --- |
| `GET /v1/files/{fileId}/versions` | Return scoped revisions, their release state, author, time and restore provenance |
| `POST /v1/files/{fileId}/versions/{versionId}:restore` | Idempotently create a new draft referencing the earlier exact bytes and return 201. Compare the expected effective pointer and reject stale intent with 409. |
| `POST /v1/files/{fileId}:trash` | Hide the document, close active review, increment share generation and retain recoverable state |
| `POST /v1/files/{fileId}:restore-from-trash` | Check scope, destination name, retention and purge fence. Reinstate an eligible previously approved effective revision without reopening invalidated links or reviews. |

Library stores the document revision counter, effective pointer and object references. Draft recording, author provenance and Audit outbox commit together. A restore draft can share the earlier immutable locator without charging duplicate stored bytes. Conditional writes prevent a stale pointer or state from becoming effective. Retention is site-configured. Purge starts only after a serialized eligibility check against live references, preservation and restoration. Transfer reconciles each cleanup against the exact object version. Retain historical metadata and decision evidence as required by the customer.

**Impact and acceptance.** Factory staff know which revision to use and authorized operators can reconstruct and recover each change.

**Dependencies.** [F04 scan](F04-scan-publish.md), [F08 permissions](F08-folder-permissions.md), [F09 audit](F09-audit.md), [F11 remarks](F11-document-remarks.md) and [F12 approval](F12-revision-approval.md).

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F05-A01 | Two replacements finish scanning against the same effective revision | Each gets its own immutable draft and unique revision number. The current approved version stays effective. |
| F05-A02 | Restore earlier bytes, retry, then repeat with a stale effective pointer | One new draft preserves source provenance. No approval is inherited and stale restore returns 409. |
| F05-A03 | Trash an approved document with a guest grant and an active review, then restore from trash | Ordinary access is withdrawn while trashed. Eligible approved content returns, and old grants and pending reviews stay closed. |
| F05-A04 | Advance the clock past retention with one referenced object and one purge-eligible object | Only the eligible version is queued for purge. Required audit and decision evidence remain. |
| F05-A05 | Restore a trashed document after someone claims its previous name | Return 409 and keep the document recoverable at an authorized unclaimed destination. |
| F05-A06 | A purge intent and eligible restore race | One transition wins. Cleanup never takes an object referenced by a committed restore. |
| F05-A07 | Restore the same earlier bytes twice as distinct drafts | Two revisions point to retained exact bytes, with correct reference and byte accounting. |
| F05-A08 | Trash a document while replacement scanning is pending | Late completion cannot reopen it, resubmit it or alter its effective pointer. |
