# Delivery and Deployment Cost Model

**Planning basis · USD · 23 November 2026**

This separates one-time product engineering, per-customer installation, and monthly site or cloud operations. Quoted customer requirements, local vendor quotes and regional AWS prices should replace the illustrative inputs before procurement. Sales margin, tax, contracted third-party licensing and customer-specific compliance work get separate line items.

## Workload and storage

Use 10 employees, 20 invited recipients, three concurrent 500 MB uploads, 50 GB of initial active data, and 10 GB net growth per month. After 12 months, active data reaches 170 GB. A provisional 1.4× factor for history plus 10 GB of staging gives 248 GB stored on the primary. The proposed 355 GB of usable fixed object capacity leaves about 30% free. Add independent backup retention and raw-to-usable storage overhead. Model actual guest egress by download count × bytes, including repeats and range retries.

At an assumed 10 Mbps per upload, three uploads need 30 Mbps inbound and a 500 MB file takes about seven minutes before overhead. Backups and guest downloads share capacity unless isolated. Replicating a 250 GB seed over 100 Mbps takes about 5.6 hours at the line rate, so prepare and load the first recovery copy ahead of the drill and measure sustained throughput.

## Product engineering estimate

| Workstream | Illustrative hours | Main effort |
| --- | ---: | --- |
| Product and acceptance | 100 | PRD, customer requirements, journey validation and handover |
| Employee website and guest experience | 240 | Web workspace, sign-in/session edge, browser transfers, guest flow and web release packaging |
| Backend services | 440 | Service boundaries, identity/permissions, transfer pipeline, sharing and audit |
| Platform and reliability | 200 | Local stack, shared release packaging, first on-site qualification, observability, backup and restore |
| Verification and security review | 160 | Browser and traffic tests, negative authorization checks, operational and recovery drills |
| Total | 1,140 | Baseline estimate for first on-site pilot |

At a blended planning rate of $85/hour, the example totals **$96,900** before contingency or **$116,280** with 20%. The 240-hour web workstream is a planning placeholder to re-estimate against the approved website. These baseline hours predate the corrected remarks and multi-step approval scope. Re-estimate web, Library and acceptance effort before commercial use. Staff mix and parallel delivery determine calendar duration. Track AWS productionization and later features as separate delivery programs.

## Site unit economics

| Cost | Reuse customer infrastructure | Buy an MVP host and storage |
| --- | ---: | ---: |
| Infrastructure purchase assumption | $0 incremental | $3,500 |
| One-time install, 20 hours × $75 | $1,500 | $1,500 |
| Monthly ops, 6 hours × $75 | $450 | $450 |
| Monthly facility or VM allocation | $40 | $40 |
| Monthly off-site backup budget | $25 | $25 |
| Monthly support allowance | $85 | $85 |
| First year before project engineering | **$8,700** | **$12,200** |

The local profile uses existing developer workstations where available. Budget for browser test capacity and CI. For the on-site customer, price any additional DMZ appliance, commercial object-store support, identity and certificate operations, connectivity and coverage beyond business hours. HA adds a second edge path, independent hosts, resilient storage and operations capacity. A warm recovery site adds reserved compute, transfer and operator drills. Quote those additions against the purchased resilience target.

Add the [staging foundation](../delivery/01-production-to-staging.md) to the delivery budget. Price a separate VM or allocation, staging PostgreSQL and object/backup storage, registry, runners, browser coverage and operator time. The site and AWS figures model production and recovery. Staging and shared engineering are separate costs.

## Off-site primary cost inputs

For an off-site primary, replace the on-site host and facility charges with the approved facility or dedicated compute and storage quote. Add setup for private employee connectivity and guest ingress, outbound transfers and mail, independent backup, and the hosting operator's coverage. Budget separate qualification time for remote routing and independent restore. The first on-site estimate above uses the shared product build. An off-site primary and the AWS scale example are separate deployment estimates.

## AWS scale example

This example uses US East (N. Virginia) public rate examples where indicated, 730 hours/month, Linux/x86 Fargate, regular on-demand capacity, two Availability Zones, small traffic and no credits. It illustrates a costing method. The primary and regional recovery locations remain subject to measured latency, residency, and service eligibility.

| Component | Example quantity or formula | Monthly USD |
| --- | --- | ---: |
| ECS Fargate | Employee web and four services × 2 replicas × 0.5 vCPU and 1 GiB, plus two workers at 2 vCPU and 4 GiB each. 9 vCPU and 18 GiB total × 730 hours. | $324 |
| RDS PostgreSQL Multi-AZ | One instance deployment, preliminary $0.14/h compute budget, plus $8 storage/backup allowance | $110 |
| ALB | Public guest and private employee ALB, 2 × 730 × ($0.0225 + 0.5 LCU × $0.008) | $39 |
| NAT and processing | Two zonal NAT gateways × 730 × $0.045, plus $3 processing allowance | $69 |
| S3 storage, requests and lifecycle | 248 GB year-end example plus requests and lifecycle/backup allowance | $13 |
| WAF, logs, DNS, KMS, SES | Operating allowance, replace with usage and service-specific rates | $45 |
| Public IPv4 and incidentals | Operating allowance | $9 |
| Primary subtotal | Rounded | **$609** |
| Recovery Region | Pilot-light edge, replicated DB backup and objects, delivery/monitoring allowance. Size continuously running resources separately. | **$110** |
| Total | Primary + pilot-light allowance | **$719** |
| Total with planning reserve | 15% on both profiles | **$827/month** |

The resulting steady-state illustration is about **$9,922/year** with the reserve. It covers modeled infrastructure only. Add support labor, nonproduction environments, incident-time recovery compute, and actual internet egress, storage replication, log and endpoint usage beyond the allowances. Budget for commercial support separately. An EKS standard-support control plane adds $0.10/hour per cluster as a distinct charge. Recalculate the ECS compute row for the selected EKS workers and storage. [AWS Fargate pricing](https://aws.amazon.com/fargate/pricing/), [Elastic Load Balancing pricing](https://aws.amazon.com/elasticloadbalancing/pricing/), [RDS PostgreSQL pricing](https://aws.amazon.com/rds/postgresql/pricing/), [Amazon VPC pricing](https://aws.amazon.com/vpc/pricing/), [S3 pricing](https://aws.amazon.com/s3/pricing/), [EKS pricing](https://aws.amazon.com/eks/pricing/).

At 1 TB of active content, the same history factor alone implies about 1.4 TB before staging and backups. At 10× guest downloads, model ALB processing, compute concurrency and internet egress from observed bytes. Logs, egress, worker CPU and retention can grow at different rates. Price the selected deployment options in both regions, include backup retention and ISP constraints, and capture unit cost per site and per active workspace. [AWS Pricing Calculator](https://calculator.aws/).
