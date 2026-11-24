# Post-MVP Enterprise Roadmap

Move from one connected customer site to repeatable enterprise deployments. Use customer commitments and operating evidence to prioritize the capabilities from the reference architecture.

Enter expansion after the web release passes the [MVP gates](01-scope.md). Containerize throughout the first build. Qualify multi-node requirements using the [Kubernetes deployment profile](../delivery/04-kubernetes-deployment.md).

| Phase | Deliverables | Trigger and exit evidence |
| --- | --- | --- |
| E1. Repeatable fleet | Supported infrastructure matrix, signed offline bundles, storage/provider conformance suite, operator upgrades, centrally collected opt-in metrics, tenant data export/import | Two clean installs and one upgrade/rollback on each supported profile. Customer signs off on key custody, egress and backup destination. |
| E2. Site resilience | Redundant edge, app replicas, independent compute and storage failure domains, PostgreSQL and object-store HA, and off-site warm standby where contracted | Capacity holds through one defined component failure. Recovery runbook survives loss of a site with measured RTO/RPO. |
| E3. AWS productization | Customer- or vendor-owned account profiles, ALB/WAF, ECS autoscaling, multi-AZ RDS, S3 and SES, IaC, scoped IAM and tested second-region recovery | Contracts match the on-site product, data residency is approved, load test and regional drill pass, and cloud unit cost fits commercial targets. |
| E4. Enterprise content | PDF/image previews, full-text search with permissions, OCR, configurable retention and legal holds, review escalations and parallel approval policies, and advanced audit | Each capability has a customer-backed PRD, workload corpus, authorization tests, and an operating budget. |
| E5. Integrations and clients | React Native employee client with website parity, plus separate partner integration and desktop sync initiatives | Stable backend contracts, native sign-in and distribution qualification, parity checks and existing permission boundaries verified. |

## Future mobile client

React Native is a later way to use the employee product. Its functionality is limited to workflows the website already ships. Browse and search, permitted file transfers, version history, remarks, scoped approval steps, restore and approved sharing follow the same product rules, visibility and permissions. Track parity against the released web baseline and reuse the same business acceptance cases. Additional product workflows require their own scope decision and web baseline.

The native client will make remote calls to the existing versioned service APIs through an approved client entry point. It may need native sign-in and transport integration while reusing the same service-side authorization, audit and data contracts. The current MVP uses versioned HTTP APIs described in OpenAPI. Any future RPC protocol choice is a separate transport decision and carries the same business contracts.

The [future mobile repository scaffold](../scaffolding/mobile.md) records the proposed layout, kickoff criteria, native boundary and parity evidence.

## Infrastructure adoption criteria

- **ECS and EKS.** A microservice can scale independently under either scheduler. Use ECS when AWS owns the runtime. Choose EKS if a customer platform mandate, reusable Helm/GitOps operations, or a measured workload calls for Kubernetes across environments. Budget cluster operations and upgrade ownership before adopting it.
- **Kafka and Schema Registry.** Start with per-service transactional outboxes and idempotent consumers. Add a broker when independent consumers require long replay windows, shared contracts, traffic isolation, or throughput the outbox transport cannot sustain. Preserve event IDs and schema compatibility during migration.
- **Temporal, search, and processing.** Add Temporal when approvals require durable timers, escalation, and operator visibility. Add OpenSearch and isolated extractors when customer-accepted documents need content search. Recheck current permissions at query, object read, snippet, and preview boundaries.
- **Caching, policy, and service mesh.** Use Redis only for a profiled cache or similarly evidenced need. Evaluate a shared policy engine when policy ownership spans deployments or services. Use a mesh when uniform mTLS and service-level policies justify its operational cost.
- **Enterprise governance.** Map contractual retention, location, audit immutability, deletion, and administrative custody to each storage provider. Confirm WORM mode and restore behavior on the selected product. Keep independently retained backups and test recovery through credential loss or storage corruption.

## On-site to cloud migration

1. Establish the target site with compatible API and database schema versions, hostnames, trust, object-store policies, and authorized identities.
2. Export each service database plus a manifest of immutable file IDs, version locators, checksums, object sizes and audit sequence references. Seed the destination, sample read verification, and record unresolved objects.
3. Place the source into a bounded change window, complete the final delta, fence writes, and verify the destination's authorization and object consistency before updating approved browser access or DNS. Invalidate sessions and reissue guest links if hostnames, keys, grant state or rollback guarantees change.
4. Rehearse rollback and document the last point where source restoration is safe. If the new site has accepted writes, preserve and reconcile its delta before returning ownership.

See the [deployment profiles](../architecture/04-deployment-architecture.md) and [cost model](../costing/07-cost-model.md) for operating differences.
