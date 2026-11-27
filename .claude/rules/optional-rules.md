# Optional Feature Rules (PART 34, 35, 36)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Activate an optional PART without its required declaration in SPEC.md.
- Partially implement an activated multi-user, organization, or custom-domain feature.
- Treat client-supplied identity, role, domain, or ownership claims as authoritative.

## CRITICAL - ALWAYS DO
- Implement every activated optional PART completely and follow its security model.
- Enforce scoped ownership, roles, authorization, and audit requirements on every route.
- Validate custom domains and certificate operations using the prescribed controls.
- Keep optional-feature route, API, and documentation behavior synchronized.

## Key rules summary
- PARTs 34–36 are optional only before activation; their rules are mandatory once enabled.
- Unactivated optional features must not leave dead routes or placeholder UI.

For complete details, see AI.md PART 34, 35, 36.
