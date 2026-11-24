# Kubernetes Distributed Deployment

Choose the customer's supported Kubernetes distribution for distributed deployments, with RKE2 as a candidate. Use EKS on AWS where the platform team chooses Kubernetes. Both connected MVP installation profiles use Docker Compose. Qualify the distributed profile in development and staging before promising customer HA.

## Deployment topology

| Area | Initial specification |
| --- | --- |
| Failure domains | Three independent failure domains as an HA planning basis. Use a supported control-plane topology, redundant ingress, labeled worker placement and capacity after losing one node. Validate the actual storage and edge failure domains independently. |
| Ingress | Internal employee load balancer routes to the web deployment. Public guest gateway/DMZ routes only guest paths to Sharing. Use a maintained Gateway API implementation or customer-standard gateway. |
| Employee web | Independently scaled Deployment with a shared, scoped session store. Web configuration supplies approved origins, selected account mode and private API routes. |
| API workloads | Stateless Deployments for Library, Transfer, Sharing and Audit. Begin with two replicas per latency-sensitive API, placement across hosts using topology spread, graceful termination and readiness probes. Separate service accounts and network policies. |
| Worker | Separate Deployment with bounded concurrency, leased jobs, Pod resource requests/limits and isolated scratch. Initial scaling signal is oldest queued scan age coupled to capacity and error rate. |
| Database and objects | Use supported external PostgreSQL and S3-compatible storage where available. Provide separately replicated data paths and a cross-store restore procedure. Use platform CSI storage only for workloads that need persistent block devices. Encrypted EBS volumes in AWS are zonal. |
| State and credentials | ConfigMaps for ordinary config, bounded access to secrets, per-service identities, and environment-specific workload credentials. Default-deny network policies require a compatible enforcing CNI. Allow only declared service-to-service paths, DNS, identity, storage, mail, and telemetry. |
| Deployment ownership | Helm release values per environment and separate permissions for registry, namespace deployment, migrations and cluster administration. Development and staging use isolated databases and storage. Production policy limits deploy routes and credentials. |

## Rollout and scale settings

- Gate traffic with readiness probes. Choose liveness checks that detect a stuck process, and use startup probes where initialization can be slow. Size termination grace against normal requests and worker lease recovery. Separate guest streaming capacity and timeout policy from control APIs.
- For a two-replica API, start with a rolling update that permits one surge replica and zero planned unavailable replicas where capacity supports it. A disruption budget with at least one available replica constrains voluntary disruptions. Test involuntary node failure separately. Spread across hostnames and, on a multi-zone cluster with capacity, across zones. [Deployment strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/), [Disruption budgets](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/), [Topology spread](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/).
- Use one bounded, named migration Job per service and release, with an explicit deadline, backoff policy, expected environment check, role and a DB-level migration lock. Treat retries and duplicate starts as possible. Proceed only once the Job and schema verification succeed. [Kubernetes Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/).
- Collect latency, error rates, CPU/memory, active guest streams, scan ages, backup resets and cluster resource headroom. Add API autoscaling only after measuring concurrency and connection limits. Add worker replicas only while scanner throughput, object storage and the queue remain healthy.

## Build and qualification phases

1. **Development cluster.** Build and load the same release images and Helm chart. Render and validate manifests. Prove namespace, gateway, secret reference, Job and service contracts against a disposable cluster.
2. **Distributed staging.** Deploy across at least three nodes or representative independent hosts. Verify real gateway, CNI policy, identity, persistent infrastructure and load. Run three simultaneous 500 MB uploads alongside guest reads, and drain a node under load.
3. **Production qualification.** Show two replicas land in separate failure domains. Test node and gateway loss, DB failover, storage unavailability and isolated restore. Record impact to open transfers and client retries. Rehearse upgrade, release rollback, chart rollback and credential rotation with named owners.
4. **Cloud adaptation.** For EKS use the AWS-supported ingress integration, scoped workload IAM, encrypted CSI-backed storage where required, and managed RDS/S3 when appropriate. Check cross-AZ failures and protected backups. Qualify regional recovery independently of node distribution. [AWS Load Balancer Controller](https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html), [EKS EBS CSI](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html).

See the [deployment architecture](../architecture/04-deployment-architecture.md), [Docker delivery](03-docker-deployment.md), and [release gates](02-ci-cd-and-release-gates.md).
