# PlantDocket Factory Document Control

The [Master Documents index](docs/README.md) links the current architecture, scope, product requirements, deployment plans, feature specs, and cost model.

The first release is a browser-based factory document control system for floor plans, panel and electrical drawings, digital designs, HR policies and ordinary office documents. Each document has a revision trail, remarks and a multi-step approval section. A revision takes effect after its final required approval. Guests can receive approved revisions only. The same release installs on-site or at an approved off-site facility, with an on-site first pilot and independent backups. [Deployment architecture](docs/architecture/04-deployment-architecture.md).

[Product positioning and GitHub names](docs/prds/01-product-positioning.md) describe the working name, target buyers and repositories.

Production is currently the only live environment. Start delivery work with [staging and database isolation](docs/delivery/01-production-to-staging.md), [CI/CD and release gates](docs/delivery/02-ci-cd-and-release-gates.md), and the [MVP scope and launch gates](docs/mvp/01-scope.md).
