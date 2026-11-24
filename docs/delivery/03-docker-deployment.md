# Docker Build and Deployment Profiles

Docker produces an image for the employee web application and each backend deployable. Docker Compose is the reference for local integration, isolated CI, staging and single-node on-site or off-site primary deployments. Kubernetes consumes the same immutable images for distributed workloads. [Docker Compose in production](https://docs.docker.com/compose/how-tos/production/).

## Packaging and configuration contract

| Artifact | Required behavior |
| --- | --- |
| Employee web image | Browser assets and same-origin session/API edge. Accept employee origin and selected account mode at runtime. Read scoped session state and credentials. |
| Library, Transfer, Sharing and Audit images | Multi-stage builds, pinned supported base/runtime versions, nonroot execution, explicit ports, health checks, graceful termination, and stdout/stderr logs with redaction |
| Transfer worker image | Dedicated execution identity, bounded CPU/RAM/scratch, production scanner signatures and update policy, bounded jobs and recoverable leases |
| Migration image | Service-specific migrations, independent scoped credentials, expected environment verification, one-shot execution, and an observable status |
| Release manifest | Per-image digests, source revision, target platform architecture, configuration schema, supported schema range, signature and provenance |
| Site configuration | Compose base and environment deployment overlays, trusted certificates, site DNS, account and mail configuration, and backup settings, plus secrets passed as scoped mounted files or through the site secret manager |

Use runtime and worker images from the same candidate release set. Mount read-only runtime filesystems where supported and grant writable volumes only to explicit scratch or state. Keep persistent files in PostgreSQL and object storage. Keep Docker socket and privileged host mounts out of the application processes.

## Compose profiles

- **Development.** Bring up the complete web stack using disposable data, demo users and a mail sink. Expose reachable employee, guest and object origins with development TLS. Limit debug access to the development network.
- **CI.** Allocate a unique project per job, synthetic seed and short-lived credentials. Start services only after health prerequisites and run DB migrations as distinct one-shot steps. After success or failure, retain redacted logs and destroy only that CI project's resources.
- **Staging.** Run the same images and gateway policy planned for release. Use isolated endpoints and a dedicated PostgreSQL instance, object storage, staged accounts or configured SSO and mail permissions. Persist backups independently of the deployment host.
- **On-site primary.** Deploy approved digests into the customer site. Expose the private employee and public guest paths through the approved gateway. Keep internal services and data endpoints under the site's policy.
- **Off-site primary.** Deploy the same digests in a dedicated approved facility with its own routes, identity callback, secrets and data endpoints. Test employee access over private remote connectivity and guest access through the public edge. Protect either primary with independent backups.

Health-gated Compose startup uses dependency health checks. Services still handle retries and runtime dependency recovery. Mount only the secrets needed by each service and restrict the source files at the host. Validate the effective Compose configuration in CI using dummy secrets. [Compose startup order](https://docs.docker.com/compose/how-tos/startup-order/), [Compose secrets](https://docs.docker.com/compose/how-tos/use-secrets/).

## Deliverable layout

```text
plantdocket-web Dockerfile      Employee website and session runtime
services/*/Dockerfile             Per-service multi-stage builds in backend repo
deploy/compose/compose.yaml       Shared service and network definitions
deploy/compose/dev.yaml           Local dependencies and fixtures
deploy/compose/ci.yaml            Isolated CI job profile
deploy/compose/staging.yaml       Stage endpoints and release deployment
deploy/compose/onsite.yaml        Connected single-host profile
deploy/compose/offsite.yaml       Dedicated remote primary profile
deploy/kubernetes/                Distributed workload implementation
ops/runbooks/                     Upgrade, backup, rollback and recovery
```

## Installation and evidence

1. Verify host resources, supported Docker/runtime versions, image architecture, certificates, environment identity, object storage adapter and backup destination.
2. Validate merged configuration, check signed digests, verify runtime and migration roles, and render a redacted change plan.
3. Take the designated backup, run expand migrations once, deploy services by digest, wait for readiness, then verify all exposed paths.
4. Exercise an interrupted upload, stale worker restart, DB or storage brief unavailability, disk pressure, upgrade, compatible rollback and host replacement. Record the Compose project identity and immutable release ID in the evidence.

Use the [CI gates](02-ci-cd-and-release-gates.md) for release execution and the [Kubernetes profile](04-kubernetes-deployment.md) when capacity or availability requires more than one node.
