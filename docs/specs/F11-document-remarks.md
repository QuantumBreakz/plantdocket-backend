# F11 Document Remarks

**Problem and audience.** Factory contributors and reviewers need to explain changes to floor layouts, panel diagrams, drawings and office policies without losing context when the document is revised.

**Use-case scenario.** A reviewer opens panel drawing revision 3, records a remark about a missing terminal label, and returns the revision. Revision 4 becomes effective after review. The earlier remark remains attached to revision 3.

**Product flow.** Open document → choose the relevant revision or document-wide discussion → read authorized remarks → add a remark → follow the thread through history.

**Use cases.** Record revision feedback, provide document-wide operational context, and trace earlier discussions. **Non-use cases.** Live chat, CAD annotations, collaborative editing and approval by comment.

## Product and service requirements

- **FR1.** Every document has a remarks section. A remark references the document and optionally an exact revision. The API verifies that the revision belongs to that document and workspace. Revision remarks stay attached to their original revision after revision changes, restore and approval.
- **FR2.** Persist author ID, server UTC timestamp, text, scope and optional correction reference. Remarks are append-only in the MVP. A correction is an additional remark linked to the earlier record. An approval or return is a separate structured decision and is recorded through F12.
- **FR3.** Apply the owning document or revision's access policy before returning or accepting remarks. Access to the effective revision alone reveals neither draft remarks nor review reasons. Document-wide remarks are visible to readers of that document and are explicitly identified as document-wide when posted.
- **NFR1.** Render remarks as plain text with a proposed 4,000-character limit. Use stable ordering and pagination. Do not put text in logs, metrics or low-privilege audit views.
- **NFR2.** Scope idempotency to the author, document and action. Commit the remark and its event together. Preserve authorized remark history during backup, restore and ordinary revision supersession.

## Contract

`GET /v1/files/{fileId}/remarks` accepts a documented scope and an optional authorized revision ID, and returns a cursor page ordered by UTC time and ID. `POST /v1/files/{fileId}/remarks` accepts text, scope, optional revision ID and optional corrected remark ID. The Library rechecks access and parent scope in the transaction and returns 201 with the durable record. Cross-document revision references fail without revealing the foreign document.

Audit records the remark ID, author, document and revision identifiers, action and time. The remark content remains in Library. Document-wide and revision remarks appear under clear scope labels. A linked correction preserves the earlier record. An approval reason remains on the actual decision even when someone posts a follow-up remark.

**Impact and acceptance.** A reviewer can recover the reasoning behind a previous revision without seeing comments attached to inaccessible drafts. Product acceptance uses the panel drawing journey and verifies author, scope and time.

**Dependencies.** [F05 versions](F05-versions-trash.md), [F08 permissions](F08-folder-permissions.md), [F09 audit](F09-audit.md) and [F12 approval](F12-revision-approval.md).

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F11-A01 | Add a document-wide remark and one on a particular draft revision | Both display with their correct scope and author. An ordinary reader sees only the authorized document-wide remark and effective-revision remarks. |
| F11-A02 | Approve a later revision or restore earlier bytes as a new draft | The original remark stays attached to its original revision and remains visible to authorized history readers. |
| F11-A03 | Attempt to comment on an inaccessible document or attach a revision from another document | No remark is created and the response reveals no restricted metadata. |
| F11-A04 | Lose the response and submit the same remark with its idempotency key | Exactly one remark and one business event are committed. |
| F11-A05 | Correct a remark and enter text that looks like markup | New correction links to the prior remark. Both are preserved and displayed as text. |
| F11-A06 | Write a remark that says the document is approved | The remarks trail records the text, and the authoritative approval state and effective pointer stay unchanged. |
