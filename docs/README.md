# PlantDocket Master Documents

**Working baseline · 23 November 2026**

The first release is factory document control for drawings, diagrams, policies and office documents. It tracks every revision, documents remarks and routes review through multiple approval steps. Final approval controls the effective revision, and guest downloads are limited to approved revisions. The product installs on-site or off-site. Isolated staging and CI/CD are the first delivery priorities.

## MVP and product requirements

- [MVP scope and launch gates](mvp/01-scope.md) owns first-release scope, sequencing and open inputs.
- [Product positioning and GitHub naming](prds/01-product-positioning.md) defines the customer description and repository names.
- [Web MVP PRD](prds/02-onsite-mvp-prd.md) owns users, workflows and the launch features.
- [Platform delivery PRD](prds/03-platform-delivery-prd.md) owns staging, release, recovery and operator outcomes.
- [Frontend delivery](mvp/05-frontend-delivery.md) and [backend delivery](mvp/06-backend-delivery.md) define implementation phases and the repository scaffold.
- [Repository scaffolds](scaffolding/README.md) define web and backend layouts, their end-to-end handoffs and the separate future mobile plan.
- [Post-MVP roadmap](mvp/03-post-mvp-roadmap.md) defines future rollout and enterprise capabilities.

## Architecture and delivery operations

- [System architecture](architecture/01-system-architecture.md) owns service boundaries, permissions, data ownership and file workflows.
- [Deployment architecture](architecture/04-deployment-architecture.md) owns local, staging, on-site, off-site and AWS topologies, including ingress, load balancing, storage and recovery.
- [Production to staging](delivery/01-production-to-staging.md) defines isolation, staging data, bootstrap and exit evidence.
- [CI/CD and release gates](delivery/02-ci-cd-and-release-gates.md) defines pull request verification, candidate build, staging, approval, promotion and rollback.
- [Docker profiles](delivery/03-docker-deployment.md) and [Kubernetes distributed deployment](delivery/04-kubernetes-deployment.md) define runtime packaging and qualification.
- [Feature verification matrix](delivery/05-feature-verification.md) maps feature case IDs to release checks.
- [Cost model](costing/07-cost-model.md) owns delivery, staging cost inputs, and site and cloud operating estimates.

## MVP feature specifications

The [shared service contracts](specs/00-service-contracts.md) define ownership, API errors, idempotency and consistency boundaries across the specs.

| ID | Specification |
| --- | --- |
| F01 | [Employee sign-in and workspace membership](specs/F01-employee-access.md) |
| F02 | [Folders, browse and metadata search](specs/F02-library.md) |
| F03 | [Resumable browser upload](specs/F03-upload.md) |
| F04 | [Quarantine, malware scan and draft registration](specs/F04-scan-publish.md) |
| F05 | [Version history, restore and trash](specs/F05-versions-trash.md) |
| F06 | [Employee download](specs/F06-download.md) |
| F07 | [Invited guest sharing and email OTP](specs/F07-guest-sharing.md) |
| F08 | [Workspace and folder permissions](specs/F08-folder-permissions.md) |
| F09 | [Audit trail and export](specs/F09-audit.md) |
| F10 | [Site installation, backup and restore](specs/F10-site-operations.md) |
| F11 | [Document remarks](specs/F11-document-remarks.md) |
| F12 | [Multi-step revision approval and release](specs/F12-revision-approval.md) |
