# F04 Quarantine, Malware Scan, and Draft Registration

**Problem and audience.** Contributors and reviewers need verified draft content. Factory readers and guests need only approved releases. Operators need to explain pending or quarantined uploads.

**Use-case scenario.** A contributor uploads a replacement panel diagram. The current approved version remains effective while the new upload is scanned. A clean replacement appears as a draft for review.

**Product flow.** Upload complete → pending scan → clean draft or hold → submit a clean draft for review.

**Use cases.** Verify clean files, hold infected or unreadable objects, retry transient scanner errors and investigate held jobs. **Non-use cases.** Content disarm, DLP, arbitrary archive support, and publication overrides for unsafe scan results.

## Requirements

- **FR1.** Transfer pins the exact staging version and checksum. Grant staging reads only to explicitly scoped processing roles. Pending, infected, oversized, encrypted/uninspectable, scanner-error and stale-signature states are unavailable for download or sharing.
- **FR2.** Use a leased job with bounded retries. After a definitive clean result, copy to an immutable locator, verify the copy and register one draft. Final departmental approval is a separate F12 action.
- **FR3.** Expose safe reason classes and operator diagnostics. Store scan time, signature age, policy version and verdict with relevant correlation data. Alert on aged queue entries or policy holds needing attention.
- **NFR1.** Bound decompress size, recursion, CPU, memory, run time and scratch. Verify valid 500 MB files and signature reload under expected concurrent load. Begin with an oldest-job warning at five minutes and tune after measurement.
- **NFR2.** Preserve file availability on the effective approved revision during worker or scanner failure. Protect held files from ordinary application roles.

**Impact and acceptance.** Verify holds, bounded 500 MB processing, crash recovery and one draft per candidate. During scanning and draft registration the effective revision stays intact. Report queue, scan and hold evidence.

**Dependencies.** [F03](F03-upload.md), [F05](F05-versions-trash.md), [F09](F09-audit.md), Transfer worker and approved scanner configuration. [ClamAV configuration](https://docs.clamav.net/manual/Usage/Configuration.html).

## State and draft registration contract

The Transfer state sequence is `queued → scanning → clean_copy_verified → draft_recorded`. Retryable failures move to `retry_wait`. Exhausted failures, infection, digest mismatch, unsupported archives and policy violations move to `held`. The job binds the workspace, candidate version, staging identity, byte count and digest to its lease and attempt.

Initial retry schedule is 30 seconds, two minutes and five minutes after the initial attempt for transient failures. Hold files when signature age exceeds the configured limit, initially 24 hours pending site policy validation. The worker records a draft through Library's scoped internal candidate endpoint using the same candidate and locator across retries. Only scoped reviewers and contributors can retrieve eligible clean drafts. Guest and ordinary delivery require F12 approval. Audit records verdict, reason and attempt metadata.

### Lease and verification rules

Claim queued work using a durable lease generation. Store attempt number, schedule, lease renewal/deadline, and next retry time. A worker that lost its lease cannot update the job. After a process crash, a replacement resumes from persisted evidence. Reconcile any extra objects created before lease loss. Draft registration accepts a single candidate and exact clean locator.

Read only the staging object version captured at completion. Stream or spill within a bounded scratch allocation, count bytes, compute SHA-256 and scan the same verified input. Record scanner engine, signature generation or timestamp, start/end times, and policy version. A clean verdict is valid only with complete scan coverage, verified digest and size, and freshness policy satisfied. Treat size, recursion, encryption or scanner-limit truncation as a hold. Keep scratch and the staging bucket unavailable to ordinary employee and guest delivery roles.

### Register draft or hold

| Condition | Durable effect | Recovery |
| --- | --- | --- |
| Verified clean input | Prepare an immutable clean object for the candidate and verify exact size/digest and locator. Persist `clean_copy_verified`. | A duplicate prepare validates the recorded canonical object. Extra versions or copies remain inaccessible and are reconciled separately. |
| Transient scanner or object outage | Schedule retry after the configured delay and keep the effective approved revision current | Renew or release the job lease as appropriate. Stop after the bounded attempt budget and expose a safe hold reason. |
| Unsafe, unreadable, stale or inconsistent input | Persist hold reason and quarantine policy. Release unconsumed logical quota through Library. | Keep staged bytes restricted until site-configured expiry or handling. A released candidate requires a new authorized intake for draft registration. |
| Library request outcome is unknown | Preserve the clean object and candidate evidence | Read the authoritative candidate result, then retry the same locator if appropriate. Keep any committed objects out of orphan cleanup. |

Library validates current uploader permission, live destination and reservation inside the draft registration transaction. Commit the draft, quota charge and Audit outbox together. The effective revision pointer stays unchanged. A stale authorization or trashed destination prevents registration. Transfer completes its job only after observing Library's durable result. Track queue age, signature freshness, held bytes and pending clean copies.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F04-A01 | Upload a clean 500,000,000-byte fixture and run a scanner signature reload during the sample | One draft appears with the exact bytes. Scan time, queue age and memory are captured. The effective revision stays unchanged. |
| F04-A02 | Submit an approved inert positive fixture, a mismatched digest, an unreadable archive, and a stale-signature fixture | Each receives a distinct safe hold reason. No clean locator is authorized for any held candidate. |
| F04-A03 | Force a worker exit after clean copy, wait for lease expiry and start a competing worker | Exactly one Library draft and audit event. The recovered job verifies the same clean locator. |
| F04-A04 | Make the scanner unavailable while a replacement is queued, then restore it | Queue age alert fires. The current approved revision remains effective. Eligible retries can register one draft. |
| F04-A05 | The scanner returns clean for only a truncated prefix or reaches an archive limit | The candidate is held. No reviewable draft is registered. |
| F04-A06 | Transfer prepares a clean object, Library commits the draft and the response is lost | Reconciliation avoids a second draft, charge or event and preserves the referenced exact object. |
| F04-A07 | Worker A loses its lease while worker B takes over and they return conflicting attempt outcomes | A cannot overwrite B's durable job state. No competing physical copy becomes readable as a revision. |
| F04-A08 | Revoke the uploader before draft registration using a prior clean verdict | Library denies registration, keeps the effective revision intact and emits hold evidence. |
