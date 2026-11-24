# Web MVP Scope and Installation Gates

**Decision.** Deliver one factory document control website for floor plans, electrical and panel diagrams, digital engineering drawings, HR policies and general office documents. Every document has its revision trail, remarks, approval status and history. The web and backend release installs at the factory or at an approved off-site facility. The first pilot remains on-site. Both profiles use the same signed images under Docker Compose. See the [deployment profiles](../architecture/04-deployment-architecture.md#deployment-profiles).

| Category | MVP commitment | Later release |
| --- | --- | --- |
| Document control | Folder and metadata search, resumable upload, scan clearance, immutable revision trail, remarks, multi-step approvals and current effective revision | Rich previews, OCR/full-text search, collaborative editing and future clients in the [roadmap](03-post-mvp-roadmap.md) |
| Roles and distribution | Named accounts, scoped administrators/contributors/reviewers/readers, folder grants, effective internal copy, verified guest delivery of an explicitly chosen approved revision | Public links, guest folder grants and partner APIs |
| Operations | Audit export, signed container releases, isolated staging and database, feature-level E2E, on-site or off-site installation, and tested restore from independent backups | Multiple active sites, Kubernetes HA, warm standby, AWS rollout, and contractual immutability variants |

The operating assumptions are 10 employees, 20 recipients, three simultaneous uploads, 500 MB per file, 50 GB at launch, and 10 GB net growth per month. Verify three concurrent guest downloads as an engineering test assumption. Measure against planning targets of 4-hour recovery time and 1-hour data loss for the selected profile. Agree contractual targets with the customer. The previously agreed AWS targets cover primary Region loss when that deployment is delivered.

## Work waves

| Wave | Exit gate |
| --- | --- |
| W0 Delivery foundation | Inventory current production. Provision separate staging PostgreSQL, object storage, identity, mail, and backup. CI builds signed images, deploys staging, and records a smoke run and rollback. This is the first implementation priority. |
| W1 Core vertical slice | Upload a revised factory drawing, add a remark, approve it in two configured steps and download the newly effective copy. Verify that the earlier copy remains effective during review. |
| W2 Feature complete | Deliver the committed specs, including approval and remark history, approved-only guest sharing, trash/recovery and audit. Trace cases to the verification matrix. |
| W3 Site acceptance | Stage the same release artifacts under representative on-site and off-site primary settings. Verify browser paths, migration/rollback and independent restore for each profile before offering it. Release the first on-site pilot through its approval path. |
| W4 Distributed qualification | Prove required service availability across nodes and measure independent state recovery. Use the [post-MVP roadmap](03-post-mvp-roadmap.md) for subsequent launches. |

## Acceptance owners and open inputs

Product owns the committed workflows and safe boundaries for later scope. QA owns case execution and review. Engineering owns contract compatibility, isolation and evidence. Customer IT owns the site network, identities, approved browsers, mail and backup destination with the operator. The release approver signs the production promotion.

The documentation lives in [plantdocket-web](https://github.com/QuantumBreakz/plantdocket-web) and [plantdocket-backend](https://github.com/QuantumBreakz/plantdocket-backend). Confirm the factory's actual department approval chains, who can approve each step, reviewer separation, document categories and permitted formats. The working access baseline uses named site-managed accounts, with optional SSO. Confirm staging, storage, mail, private and guest origins, backup destination, retention and recovery commitments. For off-site hosting, also confirm data location and operator ownership.

See the [product PRD](../prds/02-onsite-mvp-prd.md), [platform delivery PRD](../prds/03-platform-delivery-prd.md), and [verification matrix](../delivery/05-feature-verification.md).
