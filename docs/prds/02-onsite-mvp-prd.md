# PlantDocket Factory Document Control MVP PRD

## Product outcome

Factory staff use one browser workspace for factory floor plans, panel and electrical diagrams, digital engineering drawings, HR policies and general office documents. Every document has version history, remarks and a multi-step approval section. The ordinary reader sees the effective approved revision. Authorized contributors and departmental reviewers can work on the next revision without replacing the current copy. The same release installs on-site or off-site, with the first pilot on-site. [MVP scope](../mvp/01-scope.md).

| Audience | Job |
| --- | --- |
| Document contributor | Upload changes, explain them in remarks, submit a specific revision for review and track returns |
| Department reviewer | Review the exact file and remarks, approve an assigned step or return with a reason |
| Factory reader | Find and use the current approved version within their permissions |
| Workspace administrator | Provision memberships and scoped roles, configure approval steps, investigate activity and revoke guest grants |
| Invited recipient | Verify the invited email and download the approved revision authorized for them |
| Site operator | Install signed releases, monitor services, back up data and keys, and demonstrate a restore |

Capacity basis is 10 employees and 20 invited recipients, three simultaneous uploads, 500 MB per file, 50 GB at launch and 10 GB monthly growth. Confirm supported file formats, department approval chains and desktop browsers with the factory. The draft access model uses named site-managed accounts. Customer SSO is an option. Hosting, mail, backup and retention are site decisions.

## Committed features

| ID | Feature | Launch result |
| --- | --- | --- |
| F01 | [Named accounts and membership](../specs/F01-employee-access.md) | Individuals and roles are traceable without requiring customer SSO |
| F02 | [Browse, folders and metadata search](../specs/F02-library.md) | Employees can find and organize authorized files |
| F03 | [Resumable upload](../specs/F03-upload.md) | Three employees can upload eligible 500 MB files concurrently and recover interruptions |
| F04 | [Scan and draft registration](../specs/F04-scan-publish.md) | Clean content becomes a reviewable draft and held content stays inaccessible |
| F05 | [Version history, restore and trash](../specs/F05-versions-trash.md) | Each change is traceable and restored content follows review |
| F06 | [Employee download](../specs/F06-download.md) | Readers get the effective approved copy and reviewers can inspect authorized drafts |
| F07 | [Approved guest download and email OTP](../specs/F07-guest-sharing.md) | A verified recipient retrieves only the explicitly invited approved revision |
| F08 | [Folder permissions](../specs/F08-folder-permissions.md) | Workspace and folder roles constrain all reads, writes and shares |
| F09 | [Audit and export](../specs/F09-audit.md) | Authorized staff can investigate file and access decisions |
| F10 | [Site installation, backup and restore](../specs/F10-site-operations.md) | Operator deploys, monitors and restores either installation profile |
| F11 | [Document remarks](../specs/F11-document-remarks.md) | Every document carries traceable document and revision remarks |
| F12 | [Multi-step review and approval](../specs/F12-revision-approval.md) | All required department decisions precede final release |

See the [post-MVP roadmap](../mvp/03-post-mvp-roadmap.md) for future clients and feature expansion.

## Product journey

**Contributor.** Sign in → upload a new or revised controlled document → wait for scan clearance → add change remarks → submit for review.

**Reviewer.** Open the assigned step → inspect exact revision and remarks → approve the step or return with a reason → continue the ordered chain.

**Reader.** Browse or search → see current approved revision and authorized history → download the effective copy. A previous effective revision stays current until final approval.

**Guest.** Open the invitation → verify the invited email → download the pinned approved revision. A guest invitation cannot expose a draft, remarks or approval history.

**Operator.** Run preflight → install and provision → validate employee and guest browser routes → monitor → simulate loss → restore the database, clean objects, permissions and audit history.

## Release gate

- Run the [feature verification matrix](../delivery/05-feature-verification.md) against an isolated staging release candidate. Use the representative corpus, three concurrent 500 MB uploads, at least two supported desktop browsers and external email/browser access. Record list/search latency, upload completion, scan duration and guest first-byte behavior.
- Stage a panel drawing change through at least two review steps. Demonstrate a return with a reason, the prior revision remaining effective during review, and a switch to the new revision after final approval. Preserve the entire version, remarks and decision trail.
- Exercise restart and resume, quota, concurrent decisions, scoped review, revoked access, scanner failure and storage outage. Prove that drafts and returned revisions cannot be shared externally.
- Demonstrate that guest ingress exposes only guest routes and that unauthorized users cannot enumerate files, cross a workspace boundary, or read staging objects. Verify browser certificate trust, session expiry, request forgery protection and object-origin controls.
- Verify clean installation and the same browser journey against representative on-site and off-site primary profiles before offering each one. Record site-specific networking and restore evidence against the same signed release.
- Restore to an isolated replacement environment from a backup independent of the primary installation. Report elapsed service interruption and age of the latest recoverable file/metadata pair. The 4-hour RTO and 1-hour RPO remain planning targets until agreed for the selected profile.
- Publish signed artifacts, upgrade and backup runbooks, supported versions, a named incident owner and a recipient support path. Production promotion requires the exact verified web and backend image digests and the [release-gate evidence](../delivery/02-ci-cd-and-release-gates.md).

The [frontend](../mvp/05-frontend-delivery.md) and [backend](../mvp/06-backend-delivery.md) plans sequence delivery. The [deployment architecture](../architecture/04-deployment-architecture.md) specifies network and operator dependencies.
