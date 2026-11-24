# F10 Site Installation, Backup, and Restore

**Problem and audience.** Customer IT and support teams need to install a verified release, identify incidents quickly and restore a connected site after host loss.

**Use-case scenario.** An operator installs the website at the customer site or an approved remote primary facility. Following loss of that primary, the operator recovers service, permissions, files and audit history from an independent copy.

**Product flow.** Preflight → install signed release → provision → verify employees and guests → monitor → restore to an isolated target → validate → switch approved traffic.

**Use cases.** Install, upgrade, verify backups, restore after host loss, collect scoped diagnostics, and hand over ownership. **Non-use cases.** Automatic failover, active-active writes, a disconnected MVP with internet guests, and relying on the same host for primary and only backup.

## Requirements

- **FR1.** Provide the same signed web and service images, configuration schema, migrations, health checks and upgrade/restore steps for on-site and off-site primary profiles. Preflight checks the selected location, browser routes and trust, named account access or configured SSO callbacks, SMTP, DNS/time, keys, object origins and independent backup access.
- **FR2.** Back up each service DB and WAL, exact clean file references and objects, audit data, recovery keys/configuration, and release digests. Verify recoverability from independent credentials and track the latest complete file/database checkpoint.
- **FR3.** Restore to an isolated environment, reconcile records and objects, authorize from the correct identity and version policy, invalidate rollback-sensitive guest grants and replay eligible events. Operator verifies the latest recoverable point and signs off before directing users.
- **NFR1.** Measure outage duration and data age against planning targets of four hours and one hour until the customer accepts the site targets. Exercise actual WAN and file volumes. Alert on stale backups, scan/OTP failure, disk exhaustion and recovery gaps.
- **NFR2.** Restrict restore and archive access, protect encryption keys outside the source site, redact credentials from diagnostics, and retain an operator incident record.

**Impact and acceptance.** A named operator installs either profile and recovers it using the independent copy, bundle, keys and runbook. An employee can upload, retrieve and browse history. A newly invited guest downloads after OTP. Record the profile, measured RTO/RPO, missing-data checks and handover owner.

**Dependencies.** All MVP services and customer networking, infrastructure, the selected account authority, email and off-site storage. See the [deployment architecture](../architecture/04-deployment-architecture.md).

## Installation and recovery contract

Require a verified artifact manifest, target site and environment, allowed DB and object endpoints, independent backup access, before/after schema versions, rollback decision, and operator identity. Preflight checks connectivity, trusted certificates, correct backups, allowed guest route, image architecture, storage behavior and declared state markers before any mutation. The staging and production paths use separate runner identities and credentials. Test a mismatched target against a safe nonproduction sentinel.

The latest recovery point is the newest consistent timestamp for all service DBs, exact referenced clean objects and required audit data. During a drill record (1) outage timestamp, (2) restorable checkpoint, (3) availability timestamp, (4) integrity and permission checks, and (5) operator sign-off. Alert on a recovery point older than the configured objective. Recovery quarantines unresolved authorizations and reissues rollback-sensitive guest invitations. Only approved traffic reaches the recovered installation.

### Release and checkpoint records

| Record | Minimum contents and acceptance rule |
| --- | --- |
| Release manifest | Signed release ID, web and backend source commits, image and migration digests, compatible schema range and configuration version. Stage gate evidence references the exact web and backend artifact set. |
| Deployment record | Installation/environment IDs, primary profile and approved facility, activated release, verified destinations, migrations, actor, timestamps, health checks and rollback decision |
| Recovery checkpoint | Site, release and schema versions, consistent DB restore target, immutable object manifest with IDs/versions/checksums, audit ranges and manifests, required key references and verification state |
| Recovery activation | New deployment identity, current site epoch, approving operator, verified recovery checkpoint and source-fencing evidence retained outside the restored DB state |

Verify signed artifacts and environment before any write. Serialize installation or migration per site and service. Use separate roles for schema changes, runtime, backup writes and restore reads. Check state before routing traffic to a known-good, schema-compatible release. An incompatible migration requires forward repair or an approved restore with a recorded data-loss decision.

### Hosting profile checks

For on-site, verify employee access from the customer LAN or approved VPN and guest access through the customer DMZ. For off-site primary, prove employee access over the approved private route to the dedicated facility, reachable signed object operations, public guest-only ingress and configured account authentication and approved mail connectivity. Bind profile values, allocated data resources and operator identity to the installation. Preflight verifies the declared backup destination and accounts against approved separation from the primary. Treat unresolved facility or credential independence as a release blocker.

### Build and verify a coherent checkpoint

1. Keep PostgreSQL base backups and continuous WAL available off-site. Continuously copy eligible clean objects and retained audit batches into separate storage. Record omissions and verification failures as work to reconcile. [PostgreSQL point-in-time recovery](https://www.postgresql.org/docs/current/continuous-archiving.html).
2. For the connected MVP, use a controlled write boundary to identify a consistent point for all service DBs and handoffs on the supported single PostgreSQL instance. Briefly fence new mutations, drain or record in-flight cross-service operations, flush required commit/audit boundaries, and capture the DB restore target and referenced-object manifest. Resume writes once a durable candidate checkpoint is established. The off-site copies can finish asynchronously.
3. Verify the DB base/WAL chain can reach the captured target, all manifest objects match expected versions or approved remappings and checksums, required audit ranges are covered, and keys/configuration and release digests can be retrieved independently. Mark the checkpoint eligible only after the remote verification completes. Report age of the most recent eligible checkpoint, separate from last backup or copy time.

Choose the checkpoint interval and copy/replay headroom against the one-hour planning target. Record required event watermarks for in-flight jobs. A future deployment with multiple PostgreSQL instances needs its own coordinated checkpoint protocol and qualification. If the site cannot meet the planning target, record the effective gap and resolve the objective with the customer before calling recovery ready.

### Restore and activation

Restore into an isolated replacement installation. Verify the environment, recover keys from independent custody, and bring the business databases to the eligible checkpoint. Recreate ephemeral employee browser session state and require fresh sign-in. Remap object references only after validating exact bytes. Reconcile unfinished Transfer jobs, pending grants and producer outboxes by business ID before opening service. A missing object or permission record remains inaccessible and a critical missing dependency prevents readiness.

Demonstrate that the former writer is fenced before enabling mutations, including when it later comes back online. Routing change alone is insufficient evidence. Advance a site recovery epoch through a controlled activation record retained outside the restored databases. Sharing denies earlier grants and sessions even if a pre-revocation row reappears after restore. Reconcile employee permission changes against the current account authority and independently retained audit evidence, and keep unresolved grants denied. Verify employee file flow, a new guest OTP flow, continued backup, and operator sign-off before changing approved traffic.

Compute achieved RPO from simulated outage time minus the newest coherent restored checkpoint. Compute RTO from outage to successful required operations and service reopening. Record transfer volume and remaining gaps.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F10-A01 | Attempt a migration with stage config and a safe sentinel marked as a different environment | Preflight ends before any DB or object write. Evidence identifies the rejected mismatch. |
| F10-A02 | Install into an empty representative on-site stage from the signed release bundle | Health, named accounts, draft upload, multi-step approval, approved guest verification and independent backup complete with verified digests |
| F10-A03 | Rehearse upgrade from a supported prior schema and deploy the previous compatible image set after a simulated failure | Migration is serialized, rollback returns critical flows, and application records remain readable |
| F10-A04 | For each offered primary profile, restore from separate backup copies to a replacement environment without primary credentials | Checkpoint reconciles exact object digests, the effective approved pointer, remarks, approval steps and audit batches. Record profile and measured RTO/RPO against agreed thresholds. |
| F10-A05 | A referenced object or required key is missing during isolated recovery | Restore reports the missing dependency, withholds unverified content, and fails critical readiness until recovery scope is reconciled |
| F10-A06 | A DB checkpoint exists and file replication has missed one required clean object or audit range | The candidate does not advance the eligible recovery watermark. The system reports which dependency and age limit are affected. |
| F10-A07 | Restore to a point before guest revocation, with a pre-recovery employee web cookie and incomplete employee grant reconciliation | Older guest links and sessions fail under the new epoch. The employee signs in again and unresolved access stays denied until reconciliation. |
| F10-A08 | A prior writer reappears after replacement activation | Only the activated target can accept authorized mutations. The incident record shows verifiable source fencing and the affected deployment identity. |
| F10-A09 | Request rollback after a schema change incompatible with the known-good image | Rollback is blocked before applying the old binaries. Operator evidence records the compatible repair or explicitly authorized restore decision. |
| F10-A10 | Install the same candidate as a representative off-site primary with dedicated state and remote employee access | Employee signs in and completes transfer over the approved route. External guest verifies and downloads. Guest ingress cannot reach employee services, object administration or private state. |
| F10-A11 | Configure a primary profile with a backup destination in the same declared failure domain or using its application credentials | Preflight rejects the unapproved backup separation, and the release cannot pass the recovery gate with that configuration. |
