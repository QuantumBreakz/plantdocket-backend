# F07 Invited Guest Sharing and Email OTP

**Problem and audience.** Employees need to send one file revision to an external person while retaining expiry and revocation control. Recipients need to retrieve it from a browser using their invited address.

**Use-case scenario.** An employee with share permission invites a contractor. The contractor enters a one-time code received at the invited email and downloads the selected revision. The sender then revokes the invitation.

**Product flow.** Select approved revision → enter recipient email → send invitation → recipient opens link → requests and enters code → downloads → employee can revoke.

**Use cases.** Single-recipient invitation, code request and verification, guest download, resending under rate limits and revocation. **Non-use cases.** Guest account enrollment, public bearer links, shared folders, guest uploads and access to later revisions.

## Requirements

- **FR1.** Require current share permission and an approved revision backed by recorded final approval. An explicitly chosen superseded approved revision also requires history access. Scope the grant to one workspace, immutable revision, invited address and Library share generation. Use a high-entropy token and store its digest. Initial invite lifetime is seven days.
- **FR2.** Send OTP only to the invited address. Bind it to the grant and challenge, retain a keyed digest, enforce ten-minute expiry and at most five verification attempts per challenge as starting policy. Rate-limit by grant and source, bound resend, and invalidate prior codes when replacing a challenge. Sessions use short-lived, secure, same-origin cookies.
- **FR3.** Recheck grant, verified session, generation, final approval state and current issuer share permission for every attachment request. Stream the exact approved revision through Sharing with bounded memory, timeouts and validated ranges. Revocation blocks subsequent requests. An active response may finish.
- **NFR1.** Public edge serves guest routes only. Do not put secrets in logs, referrers or analytics. Neutral responses limit address and grant enumeration. Protect guest forms against CSRF.
- **NFR2.** Measure OTP delivery and guest first byte. Exercise three concurrent 500 MB guest streams as a starting load assumption. Apply defined throttling during overload to preserve employee API capacity.

**Impact and acceptance.** A real invited address receives a code and downloads the expected revision. Wrong, expired, reused and superseded codes fail. A forwarded link cannot redeem without email access. Revocation, file trash, lost issuer access and range retries cannot create a new authorized response. Record deliverability, verified redemption, and guest download outcomes.

**Dependencies.** [F05](F05-versions-trash.md), [F08](F08-folder-permissions.md), [F09](F09-audit.md), approved guest domain, DMZ and SMTP relay.

## Contract and decisions

| Interface | Behavior |
| --- | --- |
| Employee `POST /v1/shares` | One approved version and one email. Verify final approval, current share permission and generation. Return 201 with grant ID and expiry. Idempotent retry queues one invitation. |
| Employee `DELETE /v1/shares/{grantId}` | Authorize scope and mark revocation once. Return 204. |
| Guest `GET /invite/{token}` | Validate the opaque secret, set a short-lived redemption context and redirect to a clean URL. Mask the invited address. Exclude raw invite paths and queries from logs. |
| Guest `POST /guest/challenges` and `POST /guest/challenges/verify` | Issue only to the invite address and verify under the bound grant/challenge. Use a 60-second resend cooldown and rate limits per grant and IP. Code consumption is atomic. |
| Guest `GET /guest/download` | Verify current grant/session, final approval, issuer share permission and Library generation before each request. Stream 200 or an eligible 206, reject invalid ranges with 416, and use private/no-store response policy. |

Employees create and revoke invitations through the private website. Public guest pages and redemption routes are served by Sharing on a separate hostname. Guest sessions authorize only the invited revision. The public origin does not forward employee routes or expose the private object endpoint.

The initial guest session lifetime is 15 minutes. One-time code expiry and attempts follow the requirements above. A forwarder who lacks access to the invited inbox cannot redeem. Audit stores an opaque recipient verification ID. Avoid exposing whether an email address has an account or membership. Newly revoked grants deny subsequent range and full-object requests. An open response may finish.

### Records and expiry

| Record | Fields and constraints |
| --- | --- |
| Share grant | Workspace, exact file/version, issuer principal, Library share generation, issuance recovery epoch, protected invited address, invitation-token digest, state, and UTC expiry |
| OTP challenge | Grant, redemption context, challenge ID, keyed code digest, creation/expiry, attempt count, replacement and consumption markers |
| Guest session | Opaque credential digest, verified grant/challenge, issued and expiry times, and site recovery epoch. Its effective expiry is capped by the grant. |
| Delivery intent | Stable message ID, restricted destination and encrypted pending email payload, provider attempt/status, and discard deadline |

Commit grant and invitation outbox once per employee request. Protect recipient addresses at rest and restrict reads to the notification operation and authorized administration. Store token and code digests separately from encrypted short-lived mail payloads. Remove pending payloads on confirmed send, cancellation or expiry. Retry delivery from the same intent. If acknowledgment is ambiguous, a repeated message carries the same invitation or active challenge, without extending validity or minting a second code.

### Redemption, revocation and download

1. Exchange a valid invitation for a scoped context, redirect to a clean browser URL, then send a challenge only to the invite address. An unverified context reveals at most a masked address. Rate-limit sends by grant and source with the 60-second cooldown. Keep responses neutral for unusable invitations. Exclude secrets and raw invite paths from proxy and application logs.
2. Under a grant/challenge lock or conditional write, verify grant state, active challenge, attempts, scope, expiry and the code digest. An incorrect eligible attempt increments the counter. A valid submission consumes the challenge and creates one session in the same transaction. A resend invalidates the preceding challenge. Use secure HTTP-only, same-origin cookies and protect state-changing forms. Verify a fresh code after session expiry.
3. Before every full or range request, read the active grant and session in Sharing. Verify the exact version and final approval, issuer read/share permissions and current share generation through Library. The invitation never follows the effective pointer to a different revision. An authority failure stops delivery with a recoverable response. Transfer serves the approved exact locator to Sharing only. Preserve the request's approved version throughout streaming, and keep storage access URLs behind the service boundary.
4. Authorize revoke in Sharing, persist state and audit together, and reject any request whose grant decision starts after revocation commits. A stream admitted earlier may complete. Trashing changes the Library generation and invalidates old grants, including after restore. Site recovery invalidates earlier guest sessions/grants as described in F10. Record admission, bytes streamed or interruption, and reason codes separately. Use bounded concurrency, timeouts and per-grant admission controls.

## Acceptance cases

| ID | Given and when | Expected result and evidence |
| --- | --- | --- |
| F07-A01 | An authorized employee invites an approved staging inbox; recipient completes verification from outside the site network | Mail reaches the invited address and the browser receives only the expected approved revision with matching SHA-256 |
| F07-A02 | Use a forwarded link without code, request resend, then try stale, expired, reused and over-attempt codes | No attachment is available and only the active valid code redeems once. Verify cooldown and attempt counters. |
| F07-A03 | Revoke a grant or trash a file during an established browser session, then open a second range request | New request is denied. Restore after trash does not revive the old link. Revocation is auditable. |
| F07-A04 | Request employee, object administration and internal paths through the public guest hostname | Guest ingress denies those routes and exposes no catalog or storage details |
| F07-A05 | Run three 500 MB guest streams while employees browse and submit transfers, then deliberately exceed the configured stream limit | All three baseline transfers finish with matching bytes and employee browse remains within the F02 target. Above the limit, new requests receive bounded throttling. Record first byte, concurrency, throughput and errors. |
| F07-A06 | Race two correct submissions and one late resend against the same challenge | At most one verified session is created for the challenge. A replaced or consumed challenge cannot verify, and rate/attempt evidence matches the committed order. |
| F07-A07 | Lose the SMTP acknowledgement and replay the delivery intent | The retry preserves the same invitation or challenge, and the durable record shows one semantic issuance without logging the secret. |
| F07-A08 | Remove the issuer's read or share permission, or make Library unreachable after the guest session starts | Every later full/range request rechecks and fails closed. No attachment response or storage URL is returned. |
| F07-A09 | Advance the site recovery epoch and retry an old link, session and range request | Pre-recovery grant and session are rejected. A newly issued grant and verified session can access only the restored authorized version. |
| F07-A10 | Try to invite and download a clean draft, returned revision or revision with only its first required step approved | Library denies sharing and download. Guest pages expose neither remarks nor approval history. An authorized exact approved version remains eligible. |
