# F09 Audit Trail and Export

**Problem and audience.** Workspace administrators, security staff and operators need to reconstruct activity and prove which access decisions the system observed.

**Use-case scenario.** An administrator investigates a recipient report and retrieves the linked invite, OTP verification, authorization, stream observation and revocation facts.

**Product flow.** Select time window and subject → inspect matching events → export the approved dataset → verify batch integrity.

**Use cases.** Review changes and access events, export by scope and time, detect gaps and verify retained batches. **Non-use cases.** General user analytics, public exposure of audit records, proof that a human consumed a file, and regulatory WORM certification at MVP launch.

## Requirements

- **FR1.** Capture membership and permissions, draft registration and scan outcomes, remarks, review submissions, each department decision, final effective revision changes, restore/trash, approved guest distribution and operator recovery actions. Define actor type, object IDs, action, outcome, source, UTC time, sequence/request correlation and workspace scope.
- **FR2.** Store security-relevant committed events transactionally in each producer's outbox. Deduplicate Audit ingestion on producer and event ID. Preserve observation timestamps separately from delivery time and alert on backlog age.
- **FR3.** Restrict audit queries and export to authorized roles. Stream safe exports and protect them with access controls. Export batches and verification manifests to separately administered storage on the approved schedule and retention policy.
- **NFR1.** Exclude raw credentials, codes, signed URLs and file contents. Restrict recipient email visibility by purpose and scope. Handle archive downtime, bounded replay and intact alert signals.
- **NFR2.** Record and investigate producer lag. Verify archive integrity periodically. Enable immutability settings when the signed customer retention policy and provider support allow them.

**Impact and acceptance.** A reviewer reconstructs every change from prior effective revision to final multi-step approval. Duplicate events ingest once, unauthorized export is denied, and altered or missing batches are detected. Check that an Audit outage delays ingestion while producer events remain durable and recoverable.

**Dependencies.** All event-producing services, access controls, off-site audit storage and [F10](F10-site-operations.md).

## Event and export contract

Use a versioned event envelope with `schemaVersion`, `eventId`, `source`, `occurredAt`, `workspaceId`, `actorType`, `actorId`, `action`, `decision`, `requestId` and relevant entity/version IDs. Capture download authorization separately from observed transfer state. For failed authorization recorded outside a committed mutation, use the scoped security-event transport and associated request ID. Order query results by occurred time and event ID while preserving ingestion time and sequence evidence for reconciliation.

`GET /v1/audit/events` accepts workspace, UTC time range, event action and opaque cursor. `POST /v1/audit/exports` requires a privileged scoped identity, time range and purpose. Return 202 with an export ID. Record requester, filter digest, permission decision and corresponding archive reference or ingestion bound. Emit UTF-8 CSV with formula-safe fields or newline-delimited JSON under controlled storage access. Starting lag alert is five minutes. Retention and immutable archive policy are customer-configured.

### Ingestion and event integrity

For critical changes, allocate an ordered sequence in a documented producer stream as part of the same commit as the business state and outbox record. Use a transactional counter or another mechanism that distinguishes rolled-back allocations from missing committed events. Send through a supervised dispatcher with retries and stable event IDs. Ingestion binds the transport identity to permitted producer and event types, validates required schema and scope, and persists once by producer/event ID. An identical replay returns success. A changed body under the same identity is quarantined and reported, preserving the first event. Acknowledge only after the event is durable.

Accept out-of-order delivery. Track the contiguous committed high-water mark per producer stream and alert on missing ranges, retry age or failed verification. Treat denied requests and transfer observations as separate observations associated with correlation IDs. Preserve source event time and ingestion time. Report time discrepancies without reassigning an event's occurrence. Use opaque recipient identifiers unless an approved scoped export specifically requires another field.

### Query, export and archive

| Stage | Behavior |
| --- | --- |
| Query | Validate workspace, privileged role and bounded UTC range. Order by occurrence time and stable event identity. Cursor binds caller, workspace and filters. Report the ingestion completeness marker when available. |
| Export selection | Create a durable job with validated filter, purpose, actor, resolved scope and ingestion upper bound. Retry from that snapshot. Persist success/failure and a checksum independently of the temporary delivery capability. |
| Export retrieval | Use `GET /v1/audit/exports/{exportId}` for status and `GET /v1/audit/exports/{exportId}/content` for retrieval. Recheck current rights against the actor and workspace. A new download after privilege removal is denied, even when a file was generated earlier. Expire temporary artifacts according to the configured policy. |
| Retained archive | Write append-only batches under separate credentials. Record producer ranges, counts, content digest, schema versions and previous manifest reference. Sign or authenticate manifests with controlled keys. Verify content and continuity against an independently retained checkpoint. Mark gaps explicitly. |

Track outbox age, resend counts, accepted and quarantined events, missing sequence ranges, retained checkpoint age, export failures and dropped security observations. Keep support and QA evidence limited to public-safe fields. The customer sets retention and whether the selected provider enforces immutable storage.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F09-A01 | Create, verify, download and revoke a guest share | A reviewer reconstructs grant creation, verification, authorization and revocation. The separate transfer observation can be correlated without treating it as proof of receipt. |
| F09-A02 | Stop Audit ingestion, create events and start it again | Backlog alert fires, publisher records remain durable, replay succeeds, and retries do not duplicate source event IDs |
| F09-A03 | An outsider requests exports and a privileged reviewer exports a fixture that includes user-controlled names | Outsider is denied. Authorized export is scoped, includes no secrets, and handles spreadsheet formulas safely. |
| F09-A04 | Delete or alter a copy of a nonproduction retained batch or manifest and verify the archive | Verification reports a missing artifact or digest mismatch and preserves a diagnostic audit record. |
| F09-A05 | Deliver events out of order, replay an identical body and submit a changed body under the same event ID | Identical replay deduplicates, changed body is quarantined, and the completeness marker advances only after the missing sequence is received. |
| F09-A06 | Remove export privileges after a file is generated and then request retrieval | Source events remain intact. New retrieval is denied and the denial is recorded in the request's audit scope. |
| F09-A07 | Roll back a source business transaction and compare archive sequence evidence | No business event is emitted for the rolled-back change, and archive verification distinguishes the rollback from a missing committed event. |
| F09-A08 | Upload a revision, add remarks, approve two required steps and retrieve the effective copy | The audit trail joins exact revision, remark IDs, reviewers, departments, timestamps, final release and download authorization. Remarks remain readable only through their authorized feature scope. |
