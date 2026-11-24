# F12 Revision Review, Approval and Release

**Problem and audience.** Manufacturing, maintenance and office teams need to know which revision of a controlled document is approved for use. Contributors and reviewers need a recorded decision for each change.

**Use-case scenario.** Engineering uploads a revised panel diagram. It passes Engineering review and then goes to Maintenance under the factory's configured policy. The previous approved diagram remains effective until both steps approve the exact revision.

**Product flow.** Create revision → scan clearance → submit → review each configured step in order → approve the last required step or return with a reason → show the effective revision and authorized history.

**Use cases.** Submit an exact revision, inspect its approval status and decisions, return it with feedback, and release an approved revision. **Non-use cases.** Parallel active production revisions, electronic signatures that claim regulatory certification, and automatic approval after upload or malware scan.

## MVP approval policy

Multiple approval steps and department routing are MVP requirements. An authorized administrator assigns an ordered template to a folder or document category. The proposed precedence is the nearest configured folder template, then the category default. Every controlled document must resolve a template before submission. A step identifies the department group or designated approvers permitted to act. The proposed rule is one eligible decision per required step, with all steps required and completed in order. The approver must also be able to inspect the revision. The proposed default excludes the submitter. Confirm actual department chains, reassignment and separation rules with the site owner.

Each document has an approval section showing its effective revision, active step, completed steps and authorized decision history. A new document with no approved revision remains visible only to authorized contributors, reviewers and administrators acting within scope. Final approval has no automatic bypass.

## States and transitions

| State | Allowed actions and visibility |
| --- | --- |
| Ingesting or held | Transfer may retry or quarantine. No review or ordinary download. |
| Draft | A clean, immutable revision may be inspected by authorized contributors and reviewers. An authorized contributor submits it. |
| In review | One active submission per document. Only its current step can receive a decision. Intermediate approval advances the step. The existing effective revision stays available. |
| Returned | A return at any step ends the submission and records a required reason. Earlier decisions stay in history. Resubmission uses a new draft with a fresh review chain. |
| Approved and effective | The committed approval replaces the effective revision for ordinary readers in the same transaction. |
| Approved and superseded | A previously effective revision remains available in authorized history. It retains its approval record. |

## Contract and service requirements

- **FR1.** Library owns approval templates, review submissions, step decisions and the effective pointer. Bind a submitted revision to its exact digest, submitter, starting effective revision and a snapshot of the ordered template and its eligibility rules. Evaluate an actor's current eligibility when they decide. If a step loses all eligible reviewers, show the blocked step to administrators and allow authorized cancellation and resubmission under a corrected policy with a reason and history.
- **FR2.** `POST /v1/files/{fileId}/versions/{versionId}:submit-review` creates an idempotent submission for an eligible clean draft. `POST /v1/files/{fileId}/reviews/{reviewId}/steps/{stepId}:decide` accepts approve or return, reason or note, expected review revision and idempotency key. `GET /v1/files/{fileId}/reviews` returns scope-filtered step and decision history. Template edits are audited and affect future submissions.
- **FR3.** Serialize decisions on the document. Check the current step, live scoped permission, exact content, scan outcome, document state and starting effective revision. Commit each step decision and its audit event together. Only the last approval transaction also changes the effective pointer and supersedes the prior revision. A stale, out-of-order or conflicting decision returns 409. Replacing reviewed content requires a new revision and complete review.
- **NFR1.** The current effective revision remains stable through scan, review, return, retry and approval failure. Ordinary browse and downloads switch only after the approval commit.
- **NFR2.** Keep immutable approval evidence with reviewer and department, step and template version, reason or note, time, review and revision IDs, digest and previous/effective revision. Restore preserves all decisions. A new draft restored from earlier bytes follows approval again.

### Reviewer queue and template setup

| API | Behavior |
| --- | --- |
| `GET /v1/workspaces/{workspaceId}/review-queue` | Return a cursor page of active steps assigned to the signed-in reviewer and visible under current document permissions. Include document title and reference, revision, current step number, total required steps and submission time. Order by the time the step became active and its ID. Recheck eligibility on each page. |
| `POST /v1/workspaces/{workspaceId}/approval-templates` | Create a template and its initial version with ordered steps and eligible workspace users or department groups. |
| `POST /v1/workspaces/{workspaceId}/approval-templates/{templateId}/versions` | Create an immutable version for future submissions. Keep the version bound to any in-flight submission intact. |
| `PUT /v1/workspaces/{workspaceId}/approval-template-assignments/{scopeType}/{scopeId}` | Assign a template version to a folder or category with a conditional assignment revision. Apply the change to future submissions and record it in Audit. |

Reject templates with missing or ambiguous steps and only allow valid workspace users or department groups. Drafts without a usable template remain drafts and show an actionable configuration issue. Adding another draft does not replace an active submission. A change under review requires an explicit cancellation and new submission to change the reviewed revision. The reviewer queue shows actionable steps. The decision endpoint remains authoritative when eligibility or active state changes after the queue is read.

**Impact and acceptance.** A factory reader can reliably retrieve the currently approved drawing or policy. A reviewer can reconstruct who submitted and approved the exact revision. Audit and recovery tests verify the committed decision and effective pointer.

**Dependencies.** [F04 scan](F04-scan-publish.md), [F05 versions](F05-versions-trash.md), [F08 permissions](F08-folder-permissions.md), [F09 audit](F09-audit.md) and [F11 remarks](F11-document-remarks.md).

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F12-A01 | Upload and scan a first draft, then submit and approve it through two distinct steps | Ordinary delivery stays blocked after the first approval. Final approval makes the revision effective with both decisions and audit evidence. |
| F12-A02 | Approve the first step of a replacement drawing and return it at the second with a reason | The previous approved drawing remains effective. Both step decisions and the return reason stay bound to that submission. |
| F12-A03 | Approve a second revision after a returned first revision | Readers see only the new effective revision. Prior bytes, remarks and decisions remain available to authorized users. |
| F12-A04 | A submitter self-approves under the proposed policy, a different department decides out of order, or the reviewer loses permission | No action advances the review or changes the effective pointer. |
| F12-A05 | Two decisions contend for the same step and a response is lost | One decision commits. The duplicate returns its existing result and a conflicting decision receives 409. |
| F12-A06 | Cancel the old submission and submit changed content, or trash the document, then attempt an old decision | The stale decision cannot change the effective version or reactivate the document. |
| F12-A07 | Restore earlier bytes as a new revision and attempt ordinary download before its approval | The restored copy remains a draft and the prior effective revision stays current. |
| F12-A08 | Try to submit a held or unverifiable revision | No review is created and all ordinary delivery remains denied. |
| F12-A09 | Edit the template after a review starts and remove eligibility for the active step | The in-flight step rules are preserved and live eligibility is enforced. Blocking is visible. Authorized cancellation and resubmission retain the old decisions and apply the new template. |
| F12-A10 | Submit a draft whose folder and category have no valid template | Submission fails with a configuration error. The draft remains intact and the existing effective revision stays current. |
| F12-A11 | Engineering and Maintenance open their queues before and after Engineering approves the first step | Only eligible Engineering reviewers see the first step. Maintenance sees the second step only after it becomes active. Revoking a reviewer's access removes the item on their next queue request. |
| F12-A12 | An administrator changes the template assignment during an active review, then starts a separate review under the new assignment | The active review keeps its original template version and the later review uses the new version. An unauthorized administrator or stale assignment update cannot change routing. |
