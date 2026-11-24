# Platform Delivery and Release Readiness PRD

## Problem and user outcomes

The team currently works against production. An isolated staging environment and delivery gate are needed to validate website/browser compatibility, schema migrations, file-processing failure and rollback before changes reach users.

| User | Required outcome | Evidence |
| --- | --- | --- |
| Developer | Run a complete Docker development profile with independent data and execute the affected feature checks | Repeatable startup, seeded test data and stable public contracts |
| QA | Test the same application artifact set intended for production in staging | Candidate manifest, spec-case results and approved exceptions tracked per release |
| Release approver | Review verified scope, migration status, rollback compatibility and known-good target before authorizing rollout | Auditable promotion record and evidence associated with the exact candidate |
| Site operator | Install and recover either primary profile and verify that credentials cannot target another environment | Profile-specific environment checks, restore timing, verification manifests and escalation ownership |

## Requirements and measures

| ID | Requirement | Acceptance measure |
| --- | --- | --- |
| PL-01 | Independent staging compute, PostgreSQL instance, object/backup state, identity and mail | Staging starts with synthetic data and distinct credentials. Provisioning, deployment and migration reject a mismatched destination in a nonproduction test. |
| PL-02 | CI for every feature and contract change | Every feature PR links requirements and passes affected case IDs. Main-branch candidates store immutable images, signed manifests and reproducible verification results. |
| PL-03 | Staging and approved production promotion | All required browser and service cases pass for the candidate. The protected deploy uses the same web and backend digests. Destination origin and identity settings have separate evidence. |
| PL-04 | Recovery and operability | Prove staged rollback, clean-install and isolated restore. Record data age, lost or missing objects and critical user journeys. |
| PL-05 | On-site and off-site primary installation | Both primary profiles use the same signed Docker images. Each offered profile passes clean install, route isolation and independent restore. Distributed Kubernetes has separate qualification. |

The first pilot is a connected on-site deployment backed by isolated staging. The off-site primary profile is independently qualified before it is offered. The post-MVP plan owns active multi-site deployment, regional automation and fleet management. The [environment runbook](../delivery/01-production-to-staging.md) provides implementation order. The [CI/CD plan](../delivery/02-ci-cd-and-release-gates.md) specifies controls and evidence.
