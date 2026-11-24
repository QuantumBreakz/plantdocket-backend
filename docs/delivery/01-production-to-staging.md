# Production to Staging Implementation Plan

**Current state.** Production is the only application environment. An isolated staging environment, staging database, and gated CI/CD pipeline are pending. Use this runbook to establish staging alongside the live installation. The first proof is a complete staged release and rollback with independently recoverable data. The production operating path serves existing users until the new promotion path is ready.

## Target environments

| Environment | Compute and ingress | State and identity | Deployment trigger |
| --- | --- | --- | --- |
| Local | Docker Compose project on a developer machine. Use employee and guest browser origins with reachable test endpoints. | Disposable PostgreSQL, test object store, test named accounts and mail sink. Synthetic data. | Developer |
| CI | Isolated ephemeral runner and Compose project per run | Per-run databases, object namespaces, keys, test named accounts and mail sink. No route to customer production. | Pull request and merge |
| Staging | Dedicated customer-approved VM or isolated subnet. Production-like gateway and distinct employee and guest DNS names. | Separate PostgreSQL instance and service databases, storage buckets/endpoint, secrets, staged account store or selected SSO test registration, email test domain or recipient allowlist, registry read identity, and backup target. | Verified release candidate |
| Production | Existing live site. Future promotion targets the selected on-site or off-site primary profile. | Live state and keys, independently administered backups, production identity and approved guest email. | Approved promotion of verified digests with evidence for the target profile |

Use a separate staging PostgreSQL instance even if it initially shares a virtualization estate. Set explicit stage/production firewall rules and distinct service credentials on both sides. A runner assigned to staging can reach only staging deployment endpoints. Treat an isolated namespace in a future shared Kubernetes cluster as an additional control. Apply network, credential, and data separation at the service boundary as well.

## Bootstrap sequence

| Step | Proposed owner | Work | Evidence |
| --- | --- | --- | --- |
| 1. Capture the live baseline | Production operator | Inventory running releases, images or binaries, database major/extensions, schema revisions per service, object semantics, selected account mode, mail and network dependencies. Record previous known-good release and backup timestamp. | Redacted baseline, current service ownership, release history, backup location and restore access |
| 2. Establish isolation | Platform and customer IT | Allocate staging compute, DNS, TLS, network routes, deployment identity and storage. Provision the separate database, environment markers, per-service login roles, migration roles and staging backup credentials. | Stage credentials can access only staging, staging DNS and services resolve from permitted clients, and CI cannot reach production DB or object endpoints |
| 3. Build reproducible state | Backend and QA | Match supported PostgreSQL major/extensions, schema history, object behavior and gateway routes. Run historical migrations from a representative supported starting version. Seed versioned synthetic fixtures for two workspaces, scoped roles, department reviewers, effective and draft revisions, remarks and stage decisions, known checksums and guest states, and up to 10,000 catalog rows. | Repeatable seed/reset, schema and extension fingerprints, initial test run with all required paths |
| 4. Prove the release path | Platform and QA | Build a signed candidate and use the same digests for representative on-site and off-site tests. Rehearse compatible rollback and restore to an independent target. | Candidate manifest, profile-specific passing gates and rollback trace, latest recoverable pair and restore times |
| 5. Introduce production promotion | Production operator and release approver | Connect restricted deployment runner or pull path, configure protected release rules, and rehearse approval and rollback. First production change is a separately scheduled release using the verified pipeline. | Approved change record, operator access audit and post-release health evidence |

For migration edge cases, generate de-identified fixtures or use an explicitly approved sanitized export with a retention limit. Default to synthetic data. Keep names, emails, file contents, keys and customer object locators out of seed repositories. Preserve representative data shapes, large object lengths, revision churn and older schema states.

## Environment verification before writes

Maintain a provisioning-owned environment registry or marker for each service database and storage target. A deployment supplies its expected environment and immutable installation ID. The deployer, migrator and smoke-test harness compare destination DNS or account/site identity, the database marker and bucket allowlist before applying changes. Treat an absent or mismatched marker as an error. Runtime credentials can read the marker and lack permission to relabel it. Keep destinations server-side and bound to the deployment job. Use separate grants for runtime data, migrations, backups and app provisioning.

Stage deployment must prove that its migrator and test credentials fail against a designated nonproduction sentinel configured with a mismatched environment. Verify isolation by reviewing credential scope, firewall rules and provider policy. Keep test artifacts within the approved environment. Roll back or repair production from its own approved artifacts and backups.

## Exit criteria and initial cost

- A second staging build can recreate state from migrations and fixtures. Test email cannot reach real customers and staging guest links resolve to staging.
- The employee web application, four services and Transfer worker report compatible versions. The employee browsers, guest flow, scan pipeline and audit archive pass the [verification plan](05-feature-verification.md).
- One candidate survives a staging rollback and a restore drill. Compare DB records to exact restorable clean bytes and audit ranges. Measure achieved recovery time and recovery point separately.
- Assign a production operator, release approver, staging owner, and QA evidence owner. Record open hosting and customer inputs in the [MVP scope](../mvp/01-scope.md).

Start by validating an 8 vCPU, 32 GiB staging VM or equivalent reusable capacity. Test the production file-size limit and representative workload before fixing hardware minimums. Track staging DB, object store, backup, runners and browser test capacity independently. [PostgreSQL point-in-time recovery](https://www.postgresql.org/docs/current/continuous-archiving.html).
