# F02 Folders, Browse, and Metadata Search

**Problem and audience.** Employees need to locate company files quickly and organize them in a predictable workspace hierarchy.

**Use-case scenario.** A contributor creates a project folder. A viewer who has access finds a document by name and sees its effective approved revision and basic metadata.

**Product flow.** Select workspace → open folder or search → review name, revision/status, type, size and dates → open authorized actions.

**Use cases.** Create and rename folders, browse paginated children, find a file by name or selected metadata, and see the latest accessible revision. **Non-use cases.** Full-text extraction, OCR, semantic search, cross-workspace moves, and indexing guest documents for anonymous requests.

## Requirements

- **FR1.** Library stores workspace-scoped folder and file IDs. Validate parent, scope and name rules. Require appropriate rights for create and rename. Define same-parent name conflicts and return an actionable conflict.
- **FR2.** Provide cursor-based folder listing and metadata search with stable ordering. Filter authorization before response pagination, counts and details. Exclude trashed files and unapproved revisions from ordinary search results.
- **FR3.** Allow uploaders with access to view pending status, while other users see only the effective approved version. Keep stable file links through a rename.
- **NFR1.** Target p95 under 500 ms for list/search API responses on the customer LAN with a pilot fixture of up to 10,000 catalog items. Measure against the actual customer's file count.
- **NFR2.** Protect query inputs and avoid leaking forbidden names, counts, or metadata through errors and cached responses.

**Impact and acceptance.** A viewer can find a known authorized file by name. Search returns no inaccessible entries, counts or snippets. Tests cover pagination, duplicate names, nested folders and a rename during refresh. Measure successful lookup and list/search latency.

**Dependencies.** [F01](F01-employee-access.md), [F08](F08-folder-permissions.md), Library and its PostgreSQL indexes.

## Contract and decisions

| Operation | Inputs and result |
| --- | --- |
| `POST /v1/workspaces/{workspaceId}/folders` | Parent folder ID and display name, optional idempotency key. Returns 201 with stable folder ID. |
| `PATCH /v1/workspaces/{workspaceId}/folders/{folderId}` | Rename by ID and expected record version. Returns updated metadata. |
| `GET /v1/workspaces/{workspaceId}/folders/{folderId}/children` | Cursor and page size. Returns only authorized children and next cursor. |
| `GET /v1/workspaces/{workspaceId}/search` | Name query, type/date filters and cursor. Returns only authorized effective approved records and next cursor. |

Limit display names to 1–255 Unicode characters after trimming and normalization. Reject separator and control characters. Use normalized case-folded keys for same-parent name conflicts and preserve the display name. Return 409 for name or record-version conflicts. Page size defaults to 50 and caps at 100. Order by normalized name then stable ID. Bind opaque cursors to workspace, caller and query context. Return 400 for invalid cursors, and reevaluate authorization on each page. Allow names and safe metadata to appear in logs only when policy permits.

### Catalog records and namespace

Library owns a workspace root, folders, logical document records and active destination-name claims. Record a title, document category and optional site reference such as a drawing number. Parent references are workspace-scoped. Root ID and location are invariant. Treat the normalized sibling namespace as shared by folders, visible files and pending new-file claims. Trash releases its active claim. Upload and trash restore use the same claim rules. Metadata changes, where permitted, remain attributable in the document trail. Changes to classification or template assignment affect future submissions and preserve prior approval evidence.

### Query and mutation semantics

- **Write.** Resolve the parent and require F08 content-write permission. For rename, check the record revision and permission on the folder. Acquire the name claim and write the catalog row, new record revision and audit outbox event atomically. An identical rename of the same row succeeds without creating another event. A conflicting name returns `409 name_conflict`, independent of the type of entry holding the name. If the claimant itself is concealed, return only the generic name-unavailable reason.
- **Read.** For children, authorize the parent and each returnable entry. For search, scope to workspace and effective content-read permission before applying page boundaries or exposing flags. Match normalized filename as a bounded substring; type matches the recorded media type and date ranges evaluate the current effective approval time in UTC with an inclusive start and exclusive end. Folder rows carry names and IDs. File rows carry safe metadata and the effective approved revision ID. Return effective approved files in ordinary search. Authorized contributors and reviewers access drafts through scoped work and review views. This library includes floor diagrams, panel drawings, engineering designs, HR policies and office documents.
- **Page.** Use seek pagination on normalized name and stable ID, with a signed cursor carrying scope, filter fingerprint and the last sort key. Begin with a 15-minute validity setting. Reauthorize each page. Rename or final approval between pages can change results. A new first page refreshes the view. Keep inaccessible ancestor names, totals and hidden IDs out of cursors and responses.

Use indexes for workspace/parent ordering, active name claims and approved metadata access. Measure candidate filtering and query plans on the stated fixture; use bounded, parameterized filters. If authoritative policy cannot be evaluated, return 503. Responses can carry `items` and `nextCursor` without global totals. Monitor p95, slow query counts, policy time and name-conflict frequency by operation.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F02-A01 | A contributor creates and renames a folder and a viewer searches for its approved document | Folder ID is stable, viewer sees the authorized file with approved metadata, and rename is audited |
| F02-A02 | Two callers use names that collide after normalization under the same parent | One creation returns 201, the other 409. Same name in a different authorized parent succeeds. |
| F02-A03 | A viewer pages through results while losing access to the restricted folder, or reuses another workspace's cursor | Next page contains no denied records or counts. Out-of-scope cursor returns 400 or workspace authorization returns 404 before data. |
| F02-A04 | Run 100 list and 100 search requests after warm-up against a representative 10,000-item staging catalog | Each operation's p95 server response is under the 500 ms planning target on the recorded profile. Save the fixture size and samples. |
| F02-A05 | A pending new-file reservation and folder creation contend for equivalent names under one parent | Only one active name claim exists. The losing operation gets a safe name-conflict response with no hidden claimant details. |
| F02-A06 | Search a pending first upload, a pending replacement, an approved revision and one clean draft and a trashed file | Search returns the effective approved metadata where present, excludes the new pending and trashed files, and reveals no unapproved candidate. |
| F02-A07 | Replay a cursor with a modified query, after its expiry, and after a legitimate rename | Modified or expired cursor returns 400; legitimate subsequent pages respect the documented non-snapshot ordering and current access. |
