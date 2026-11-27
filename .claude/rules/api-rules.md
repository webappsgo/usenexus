# API Rules (PART 13, 14, 15)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Change API route prefixes, response envelopes, status codes, or version rules ad hoc.
- Expose protected operations without the required auth scope and CSRF protections.
- Disable certificate verification or weaken TLS to make a connection succeed.

## CRITICAL - ALWAYS DO
- Keep health, readiness, liveness, metrics, and version endpoints semantically distinct.
- Implement the exact versioned route and error-response contracts.
- Publish accurate OpenAPI documentation for every API operation.
- Validate TLS certificates and use the required ACME renewal and reload lifecycle.

## Key rules summary
- Admin API routes use `/api/{api_version}/server/{admin_path}/...`.
- Responses use the documented JSON envelope and stable error codes.
- TLS configuration, minimum versions, and certificate ownership follow AI.md.

For complete details, see AI.md PART 13, 14, 15.
