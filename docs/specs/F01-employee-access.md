# F01 Named Accounts and Workspace Membership

**Problem and audience.** Factory users need individual access appropriate to their job. Document history must identify who changed, reviewed or approved a revision.

**Use-case scenario.** A site administrator provisions a Maintenance reviewer. The reviewer signs in to the private website, sees assigned review work and makes a decision that is traceable to their own account.

**Product flow.** Open employee workspace → sign in → view authorized workspaces and roles → sign out when finished.

**Use cases.** Provision, disable and recover operator-managed accounts, sign in and out, and manage memberships. **Non-use cases.** Shared approval accounts, open registration and mandatory company SSO.

## Launch access model

The working MVP baseline is named site-managed accounts. If a customer already has a supported identity provider, company SSO can be selected for that installation. In both modes, the service records a stable principal ID and requires current workspace membership and scoped permissions. An account cannot obtain approval authority simply by signing in.

- **FR1.** An authorized site operator provisions named accounts and provides a one-time activation or recovery mechanism using an approved separate channel. A workspace administrator grants a membership and roles in their own workspace. An account and a workspace membership have separate lifecycle states.
- **FR2.** For local accounts, use salted, appropriately costed password hashing, rate limits on sign-in and recovery, expiring single-use activation/reset secrets, and redacted logs. Store recovery material as hashes where practical. Customer identity policy determines any additional controls. [NIST authentication guidance](https://pages.nist.gov/800-63-4/sp800-63b.html), [OWASP password storage](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html).
- **FR3.** The employee web service uses a host-only Secure HttpOnly session cookie and server-side session state. Rotate the session at login, reject cross-origin mutations and invalid CSRF proof, and invalidate active sessions on account disablement. Local logout has no effect on unrelated accounts or sessions unless policy requires it.
- **FR4.** The selected identity mode yields the same internal principal context. Bind any configured external identity to trusted issuer and subject, independent of email. The web forwards an authenticated employee context over the private API. Services verify it and Library checks current memberships and permissions.
- **NFR1.** Keep credentials and reset secrets out of audit, browser-accessible storage and diagnostics. Log safe reasons with correlation IDs.
- **NFR2.** A lost response, concurrent administrator change or customer identity outage must not create duplicate memberships, orphan administrative access or allow authorization through stale state.

## Contract and consistency

The employee web owns login and ephemeral session state. Keep local credentials in the site identity boundary under restricted roles. Library owns the principals' application lifecycle, workspaces and memberships. `GET /v1/me` returns the authorized principal and active memberships. A valid active account with no workspace membership sees an empty result. Unauthenticated or disabled sessions receive no protected data. A required authority outage returns 503.

Serialize membership changes by workspace. A scoped administrator can use `PUT /v1/workspaces/{workspaceId}/members/{principalId}` to set a membership and `DELETE` at the same path to remove it. A repeated identical mutation creates no second event. Reject removal or downgrade of the last administrator with 409 until an authorized replacement exists. Account-level disablement is audited. For the final workspace administrator, require a replacement or an audited site recovery procedure before removing access.

**Impact and acceptance.** Staff can enter their designated workspaces and every approval can be traced to an individual. A disabled user, removed member or guest cookie has no employee access.

**Dependencies.** Site provisioning, employee session boundary, Library and [F08](F08-folder-permissions.md).

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F01-A01 | Activate an operator-provisioned local account with two memberships and sign in using supported browsers | The same principal and authorized workspaces appear. Access to a third workspace is concealed. |
| F01-A02 | Reuse, expire or guess an activation or reset secret, or submit invalid credentials | The system grants no session, throttles repeated attempts and records no secret in logs. |
| F01-A03 | Remove a membership or disable an account while its browser holds a session | New protected access is denied in the relevant scope. Change and actor are audited. |
| F01-A04 | A contributor attempts to assign a role or the final admin attempts to remove themselves | Scoped denial or final-admin conflict. No membership changes. |
| F01-A05 | Two administrators race to remove or downgrade each other | A valid administrator remains and committed events match the outcome. |
| F01-A06 | Change a user's display name or email | Principal ID stays stable and existing history remains attributable. A different account inherits no permissions. |
| F01-A07 | The configured account authority becomes unavailable or the active session expires | The system requires recovery or reauthentication. No stale decision authorizes an approval. |
| F01-A08 | Lose a successful membership response and repeat the same operation | The existing membership and single audit event remain. |
| F01-A09 | Attempt a cross-origin write with a valid employee session or present a guest session to the employee site | Both requests are denied before a protected write and secrets stay out of the browser's script-readable storage. |
| F01-A10 | Reuse a pre-login session ID, then reuse an authenticated ID after logout | Session rotation and invalidation block both reuse attempts. |
