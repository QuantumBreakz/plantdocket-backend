# PlantDocket Backend Repository Scaffold

**Repository.** `QuantumBreakz/plantdocket-backend`

**Scope.** The Go services, versioned API contracts, isolated test stack, installation profiles and operator runbooks. The [system architecture](../architecture/01-system-architecture.md) and [MVP scope](../mvp/01-scope.md) govern service ownership and delivery.

## Proposed code and operational layout

Use one Go module and independently build each service and worker. Each service owns its internal packages, migrations and runtime credentials. Keep shared packages limited to technical concerns such as transport, telemetry and test support.

```text
plantdocket-backend/
  README.md
  Architecture.md
  go.mod
  go.sum
  Makefile                         repository command entry points
  cmd/migrator/                    controlled service migration entry point
  Dockerfile.migrator              signed migration image input
  docs/                            scope, PRD, specs, scaffolds and verification
  contracts/
    public/openapi.yaml             web and future mobile business contract
    internal/                      scoped calls and versioned events
    fixtures/                      compatibility and sanitized contract examples
  services/
    library/
      cmd/library/                 catalog, authorization and document workflow
      internal/
      migrations/
      Dockerfile
    transfer/
      cmd/transfer-api/            sessions, receipts and signed operations
      cmd/transfer-worker/         scanner, draft registration and reconciliation
      internal/
      migrations/
      Dockerfile.api
      Dockerfile.worker
    sharing/
      cmd/sharing/                 employee share controls and separate guest origin
      internal/
      migrations/
      Dockerfile
    audit/
      cmd/audit/                   ingestion, exports and archive verification
      internal/
      migrations/
      Dockerfile
  internal/platform/               shared transport, identity and telemetry support
  deploy/
    compose/                       base plus local, CI, stage and primary profiles
    site/                          on-site/off-site configuration schemas and bundle
    kubernetes/                    later distributed node profile
    aws/                           later AWS scale profile
  ops/runbooks/                    preflight, rollout, backup, rollback and restore
  tests/
    contract/
    integration/
    journeys/
    fixtures/
  .github/workflows/               verify, build candidate, stage and promote
  .github/CODEOWNERS                service, contract and platform review ownership
  .gitignore                       generated state and local credentials
  .env.example                     configuration keys and sample values
```

The Transfer API and worker build separate images from the same Transfer source and share its owned storage and job schema. Start with two scoped runtime identities so signing, scanning and reconciliation can be operated independently.

## Services and data boundaries

| Owner | Records and entry points | Integration responsibility |
| --- | --- | --- |
| Library | Workspaces, memberships, folder permissions, immutable revisions, exact draft locators, remarks, ordered approval templates, step decisions and effective pointers | Own authorization, draft reservation and registration, reviewer eligibility and final release. Commit final approval, prior-version supersession, effective pointer and audit intent together. |
| Transfer API | Upload sessions, approved part receipts, finalization and scoped download authorization | Request Library decisions. Validate part and object identities and admit only the pinned file. Put one durable scan job beside completed staging metadata. |
| Transfer Worker | Leased scan jobs, immutable clean copy verification and reconciliation | A clean verdict registers a reviewable draft. Reconcile uncertain completion and registration by candidate ID. Bytes become effective through Library's final approval. |
| Sharing | Invited email verification, approved revision grants, guest sessions and stream observations | Serve the guest browser origin. Revalidate pinned revision approval and live access on each request and stream only the allowed bytes. |
| Audit | Deduplicated business events, authorized export and archived evidence | Accept service-owned outbox events at least once. Track producer sequence gaps and preserve restore evidence. |

Use separate PostgreSQL databases and runtime/migration roles for the owning services. A connected MVP installation may share a PostgreSQL host. Integrate services through contracts and durable handoffs. Keep staged, clean and audit storage under scoped provider identities. Start with one S3-compatible provider that passes multipart, exact-byte, signed-read and restore checks. Preserve compatibility through [shared service contracts](../specs/00-service-contracts.md).

## Dev workflow and environment entry points

| Proposed command | Effect |
| --- | --- |
| `make doctor` | Check runtime tools, local certificates, ports and required configuration without touching production. |
| `make dev-up` | Start the local site gateway, test identity, object storage, mail sink and service set under a unique Compose project. |
| `make migrate-dev` | Run service-owned migrations using local migration credentials. |
| `make seed-dev` | Prepare two synthetic workspaces, department reviewers, clean/held revisions and a test guest inbox. Print test role identifiers and keep secrets out of logs. |
| `make verify` | Run formatting, unit checks, migration validation and API/event contract compatibility. |
| `make e2e` | Run synthetic on-site-equivalent browser and API journeys against disposable state. Record case IDs, environment, evidence and failure summaries. |

These are interface names for implementation. Development and CI use disposable state and explicit environment markers. Staging gets its own database instance, object storage, identities, signing, backup and mail controls. Installation profiles set allowed endpoints before credentials are granted. See the [staging runbook](../delivery/01-production-to-staging.md) and [Docker profiles](../delivery/03-docker-deployment.md).

## Cross-repository and release handoff

1. Validate and publish the OpenAPI artifact with a version and content digest. Keep internal service routes, credential examples and real fixture data outside the public contract. Review breaking changes in both repositories.
2. Build candidate images for the web, Go services, worker and migrator. Record exact image digests, both commits, compatible schema range, deployment profile, SBOM and signature evidence. The backend repository owns the combined release manifest.
3. On staging, run version-checked migrations and then the browser/API journey. Check approval ordering, revocation, misrouted guest requests, three parallel 500 MB uploads, replay, and coherent backup and restore. Record evidence per installation profile.
4. Qualify on-site and off-site primary targets with the same signed image set. The chosen primary serves production and uses an independent recovery destination. Promote the accepted digests through the protected release gate and retain a schema-compatible rollback plan.

See [CI/CD gates](../delivery/02-ci-cd-and-release-gates.md), [feature verification](../delivery/05-feature-verification.md) and the [deployment architecture](../architecture/04-deployment-architecture.md).

## First vertical slice

Keep the [web scaffold](frontend.md) and the backend in sync. Prove a test employee can sign in, submit a clean drawing revision with remarks, complete two authorized review steps and download the newly effective copy. Then verify delivery of the exact approved copy to an email-verified guest and capture the durable audit chain. Use existing feature IDs and case IDs when adding CI evidence.
