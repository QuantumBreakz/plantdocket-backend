# MVP Feature Verification and Release Evidence

Every acceptance-case ID belongs to a feature specification under [MVP scope](../mvp/01-scope.md). Attach test results to the release manifest. Mark each required case as passed, failed or blocked with the test timestamp, commit, digest, environment, test version and link to evidence. A skipped or blocked mandatory case cannot serve as a pass.

| Feature | Required acceptance cases | Execution in staging |
| --- | --- | --- |
| [F01 Named access](../specs/F01-employee-access.md) | F01-A01–A10 | Named accounts, supported desktop browsers, activation, host-only sessions, CSRF and revocation |
| [F02 Library](../specs/F02-library.md) | F02-A01–A07 | API journey, shared name claims, authorized pagination, and 10,000-item performance fixture |
| [F03 Upload](../specs/F03-upload.md) | F03-A01–A11 | Exact-boundary browser transfers, reload, object origin policy, quota race, ambiguous completion, cancel and orphan reconciliation |
| [F04 Scan and register draft](../specs/F04-scan-publish.md) | F04-A01–A08 | Scanner fixtures, lease loss, partial scans, ambiguous draft registration and unchanged effective revision |
| [F05 Versions and trash](../specs/F05-versions-trash.md) | F05-A01–A08 | Concurrent draft creation, unchanged effective version, trash conflicts, shared references and restore versus purge |
| [F06 Download](../specs/F06-download.md) | F06-A01–A07 | Downloaded-file and range tests in supported browsers, expiry/revocation and signing failures |
| [F07 Approved guest access](../specs/F07-guest-sharing.md) | F07-A01–A10 | Approved-only sharing, guest browser and mail, OTP concurrency, revocation, epoch rollover and load |
| [F08 Permissions](../specs/F08-folder-permissions.md) | F08-A01–A08 | Scope, inheritance, concealed ancestors, repair and revoke versus review decision |
| [F09 Audit](../specs/F09-audit.md) | F09-A01–A08 | Revision and approval evidence, durable replay, protected retrieval and manifest verification |
| [F10 Site operations](../specs/F10-site-operations.md) | F10-A01–A11 | On-site and off-site primary install, private remote access, backup separation, restore, rollback, source fencing and revocation |
| [F11 Remarks](../specs/F11-document-remarks.md) | F11-A01–A06 | Document and revision context, draft privacy, corrections, audit and retries |
| [F12 Multi-step approval](../specs/F12-revision-approval.md) | F12-A01–A10 | Ordered department decisions, template validation, returns, conflicts, final release and immutable review history |

## Test lanes

- **Pull request (G1).** For every feature, automate the relevant contract/validation cases at the earliest reproducible layer. Run a representative end-to-end path for each changed feature in an isolated environment with a real test DB and object adapter. Run the core cross-service journey even when only a shared component changes. A test-only defect follows the same traceability.
- **Staging (G4).** Run employee and guest workflows on the paired web and backend digests. Qualify each offered primary profile with representative routes and storage. Exercise at least two supported desktop browsers, separate employee/guest origins, 500 MB paths and mail delivery. Capture manual environment evidence where automation is impractical.
- **Recovery and migration.** Rehearse all affected migration and restore paths prior to the first release and after database-major, schema, object-store, deployment or backup changes where those paths are affected. Reuse a recent restore drill on otherwise compatible candidates only when its artifact, schema range, provider and backup configuration remain applicable and the release owner records that evidence.
- **Production (G6).** Verify version and readiness, TLS, monitored latency, backup freshness, restore point and approved isolated canary signals. Run destructive, concurrency, failure injection and broad fixture suites exclusively in nonproduction environments.

## Canonical fixtures and evidence

Maintain synthetic floor plans, panel drawings, engineering diagrams, HR policies and office files. Seed two workspaces, administrators, contributors, readers, reviewers from two departments and an outsider. Include an effective revision, clean draft, partly approved revision, returned revision, remarks, held input, known checksums, 500 MB transfer fixtures, OTP and trash cases, and a guest test inbox. Keep customer content and secrets out of results.

For concurrency and crash cases, identify the commit or side-effect boundary in the fixture. Use a deterministic test pause, affected dependency response or controlled clock. Capture before/after state through scoped reads, including quota, object identity, audit count, event ordering or recovery admission as applicable. Repeat with reversed commit order where both outcomes are valid.

Example case result fields are `case_id`, `feature_id`, `environment_id`, `installation_profile`, `manifest_digest`, `suite_version`, `runner`, `started_at`, `result`, `evidence_ref` and `failure_summary`. For browser evidence, capture browser/version, origin and network profile. For performance, capture bytes, concurrency, p50/p95 and measurement scope. For recovery, record interruption time, latest coherent restored checkpoint, route restoration time and integrity checks.

Pair this matrix with the [release gates](02-ci-cd-and-release-gates.md) and the [staging runbook](01-production-to-staging.md).
