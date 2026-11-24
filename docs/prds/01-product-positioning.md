# PlantDocket Product Positioning

**Working repository name.** PlantDocket.

**Product category.** Factory document control

**Customer description.** PlantDocket gives factory teams one place to manage floor plans, electrical panel diagrams, digital designs, HR policies and office documents. It preserves each revision and its remarks, routes changes through the required departments, and shows staff which revision is approved for use. Organizations can install it on-site or at an approved off-site facility. Authorized users can share an approved version with a verified guest.

| Buyer or user | Job the first release addresses |
| --- | --- |
| IT and security | Deploy inside an approved customer environment, control access, audit events, and restore after an outage |
| Engineering, maintenance, operations and HR | Submit changes, review remarks, approve required steps and locate the effective revision |
| Factory readers and invited recipients | Retrieve the current effective copy internally or download a specifically shared approved revision |

**MVP demonstration.** An engineer uploads a revised panel diagram and adds a change remark. Engineering approves its step and Maintenance completes the final step. The old diagram stays effective until that decision commits. The approval section shows both decisions. A verified guest downloads only a specifically shared approved revision. The operator demonstrates recovery of the effective diagram and its full history.

Use `plantdocket` for repository identifiers. Public branding and commercial naming remain open. The two browser entry points are **Employee workspace** and **Guest download**.

## Proposed GitHub naming

| GitHub object | Name | Description |
| --- | --- | --- |
| Organization Project | `PlantDocket` | Delivery across the web and backend repositories |
| Private repository | `plantdocket-web` | PlantDocket browser workspace for factory drawings and policies, revision history, remarks and departmental approvals. |
| Private repository | `plantdocket-backend` | PlantDocket services for document revisions, remarks, multi-step approvals, approved guest access, audit and on-site or off-site installation. |

The backend repository owns the Master Documents, versioned OpenAPI contracts and installation tooling. The web repository consumes the published contract. Sharing serves the guest website on its own hostname. The [MVP scope](../mvp/01-scope.md) and [web MVP PRD](02-onsite-mvp-prd.md) own launch requirements.
