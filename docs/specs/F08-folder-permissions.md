# F08 Workspace and Folder Permissions

**Problem and audience.** Workspace administrators need to protect folders while letting employees find and use documents allowed by policy.

**Use-case scenario.** An administrator limits a sensitive folder to selected employees. A viewer can read its clean files but cannot upload or share unless specifically granted the appropriate role.

**Product flow.** Choose folder → inspect effective access → assign a role or restrict inheritance → review resulting access and confirm.

**Use cases.** Grant, remove, and review folder access. Authorize browse, search, upload, download, restore, remark, review, approve, share, and administration. **Non-use cases.** Cross-workspace permissions, guest membership, bespoke policy languages, and per-version employee ACLs.

## Requirements

- **FR1.** Workspace membership is required. Workspace administrators manage roles. A folder inherits access by default. An administrator can restrict inheritance and apply local grants for permitted workspace members. A folder grant cannot exceed the employee's workspace action ceiling. Preserve operator recovery for an accidentally locked folder via an authorized workspace administrator.
- **FR2.** Define viewer as browse/read, contributor as read and allowed mutation and sharing in its scope, and administrator as access administration. Library resolves the effective policy from workspace, folder hierarchy and action. Transfer, Sharing and Audit use their own scoped service identity and check authoritative resource rights where applicable.
- **FR3.** Changes are atomic, validated and audited. Recheck the latest effective policy when accepting an operation, before a worker registers a draft, and before issuing new download authorization. Use the stable authorization reason internally while avoiding resource enumeration in public errors.
- **NFR1.** Prefer current policy over low latency when a dependency fails. Library remains authoritative for policy decisions. Reflect permission changes in the browser through a refresh or recoverable error.
- **NFR2.** Automated tests cover workspace boundaries, roles, inheritance, restricted subfolders, unauthorized listing/counts, and access removed during a deferred job.

**Impact and acceptance.** Compare a viewer, contributor and administrator across two workspaces and a restricted child folder. Verify that removal blocks the next protected operation and that pending jobs cannot register drafts after loss of required access. Track denied authorizations and investigate drift.

**Dependencies.** [F01](F01-employee-access.md), [F02](F02-library.md), Library policy records and [F09](F09-audit.md).

## Policy decision table

| Action | Viewer | Contributor | Workspace admin |
| --- | --- | --- | --- |
| Browse, search and retrieve | In granted scope | In granted scope | In granted scope |
| Create/rename folders, upload, restore, trash, share approved revision | Denied | In granted scope | In granted scope |
| View or change folder grants and recover locked folders | Denied | Denied | Allowed while active in the same workspace |

Folder roots use workspace defaults. A restricted folder resets inherited content grants and starts from its explicit grants. Unrestricted descendants inherit the nearest effective grants. A local grant may narrow or reinstate an action within the principal's workspace-role ceiling. Workspace-admin capability to repair grants is distinct from reading restricted file contents. Validate proposed grants against active membership and allowed roles before writing.

Review permission is a scoped assignment to eligible named people or department groups. It authorizes inspection and a decision for the current configured step when the person also holds document access. It adds neither general write rights nor authority for other departments. Contributor and administrator status alone grant no approval authority. Remark and history access follow the target document or exact revision scope.

Expose a scoped GET and conditional PUT at `/v1/workspaces/{workspaceId}/folders/{folderId}/permissions`. The update carries the expected policy revision, inheritance mode and local grants. Return 409 for stale updates. Internal authorization receives principal, workspace, resource, action and exact revision where applicable. Return a short-lived decision tied to those values. Recheck draft registration, step decisions and guest download at execution time.

### Policy evaluation

1. Resolve the resource inside the requested workspace, and require an active employee membership. Confirm that the verb is valid for the caller service and derive the exact folder and version from Library records. Resource IDs and claimed parent IDs supplied by another service are inputs to validate.
2. Set the workspace role as the action ceiling. Traverse the authoritative root-to-target folder chain. Inherited folders start with the parent grant set. A local entry replaces the inherited grant for that principal. An explicit empty entry removes it. A restricted folder starts with an empty grant set and applies its explicit entries. An accessible descendant of a concealed parent can appear in authorized search or be opened by its known ID. Omit concealed ancestry and deny browsing through the concealed parent.
3. Evaluate the required verb and the feature state. Use distinct verbs for content read, folder create/rename, draft registration, version restore, trash, remark, submit, approve-step, effective read, history read, share and policy administration. Sharing requires final approval and current content-read and share actions. Restoring to a different authorized folder requires authority on the trash item and destination. Metadata, bulk operations, both delivery paths and delayed jobs use the same evaluator.
4. Scope administrator grant inspection to the workspace. A repair or policy-inspection response exposes policy identifiers and scoped member roles. Reading file content still requires an effective content grant.

### Policy writes and decision boundary

The policy write includes folder ID, local principal grants, inheritance mode, expected local revision and change reason. Reject invalid principals, duplicate grants, inconsistent modes and verbs beyond the workspace ceiling. Serialize with Library permission-dependent writes using the same workspace lock or revision transaction. Commit the grant replacement, local policy revision and audit event together. If a parent policy changes, descendant effective decisions change without needing a separate write to every descendant.

Record action, policy revision evidence and reason on an internal decision. The caller uses it only for the exact immediate request. A cached decision is advisory; new protected delivery, draft registration or step decision obtains fresh authority. Return concealed 404 for invisible resources, 403 for a forbidden action on a visible resource, and 503 when the policy store or scope resolution is unavailable. Denial diagnostics stay internally scoped.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F08-A01 | A viewer, contributor and admin attempt reads, writes and share across a default folder and restricted child | Results match the decision table and inherited scopes; denied results expose no names or counts |
| F08-A02 | A caller substitutes a foreign workspace, folder ID or service identity | Access is denied and the foreign data and audit context remain undisclosed |
| F08-A03 | Remove upload permission after Transfer finalizes but before the worker registers the draft | Worker cannot register that draft and Library records a scoped denial reason without exposing the candidate to readers |
| F08-A04 | Two administrators submit updates against the same policy revision | One update succeeds and the other receives 409. Audit shows the committed policy change. |
| F08-A05 | An explicit grant allows one descendant under a concealed ancestor | Authorized search and direct access return the descendant, omit concealed ancestry, and accessing the ancestor itself is denied. |
| F08-A06 | A reviewer attempts a step decision during a concurrent administrator revocation | If revocation commits first, Library denies the decision. A decision committed earlier remains in history. Later unauthorized actions are denied. |
| F08-A07 | A policy admin repairs a restricted folder with no content grant | Policy repair succeeds and is audited. The admin cannot list or sign restricted content before a content grant is in effect. |
| F08-A08 | Attempt a new sign, guest stream and review decision with a stale allow decision or unavailable policy authority | Each new action reauthorizes or stops; an unavailable authority does not create a capability, stream or approval decision. |
