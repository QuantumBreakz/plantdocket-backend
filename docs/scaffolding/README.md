# PlantDocket Repository Scaffolds

These guides define the proposed directory layouts, ownership boundaries, local workflows and CI handoffs for implementation. The MVP includes the web workspace and its backend. The customer chooses an on-site or off-site primary using the same signed application release.

| Guide | Repository ownership | Delivery |
| --- | --- | --- |
| [Frontend scaffold](frontend.md) | `QuantumBreakz/plantdocket-web` | MVP employee website |
| [Backend scaffold](backend.md) | `QuantumBreakz/plantdocket-backend` | MVP services, guest download origin and deployment |
| [Future mobile scaffold](mobile.md) | Roadmap document in `plantdocket-backend`. Proposed future repository is `plantdocket-mobile`. | Post-MVP React Native, limited to already released web workflows |

The [MVP scope](../mvp/01-scope.md) and [PRD](../prds/02-onsite-mvp-prd.md) govern what ships. The [shared service contracts](../specs/00-service-contracts.md) govern request semantics and data ownership. The [verification matrix](../delivery/05-feature-verification.md) maps the acceptance cases.

## First end-to-end implementation slice

1. Run the employee website, auth entry, Library, Transfer API and worker against synthetic local state. Prove reliable uploads and a clean immutable draft.
2. Open the draft in the web workspace, add a remark, and submit it through two department approval steps. Both sites use this product flow. The employee reader keeps the previous effective revision until the final approval commits.
3. Add approved-version delivery, email-verified guest download and audit evidence. Verify identity and access checks using at least two workspaces and both browser origins.
4. Recreate the same journey in isolated CI and staging. Verify the combined release manifest and independently test clean install, rollback and restore for each offered primary profile.

Each repository defines its own build and developer entry points. The backend publishes the versioned OpenAPI contract. The web pins and verifies that contract. The backend deployment process assembles a candidate with the exact web and service image digests before staging acceptance and promotion.
