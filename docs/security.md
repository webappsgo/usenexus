# Security

This page describes the protections present in the current implementation.
The product plan contains additional security requirements that have not yet
been implemented.

## Administrator authentication

- Administrator passwords are hashed with Argon2id and random salts.
- Sessions use random bearer values; only their SHA-256 hashes are stored.
  The `gospool_session` cookie is HTTP-only, SameSite=Lax, and expires after
  twelve hours.
- A separate `gospool_csrf` cookie and matching request token protect forms.
  Token comparisons use constant-time checks.
- After five failed login attempts from one client IP, further attempts are
  limited for the current fifteen-minute in-memory window. This limiter is
  process-local and does not persist across restarts.
- TOTP secrets are encrypted with AES-256-GCM using the owner-only
  `installation.secret` file in the configuration directory. Enrollment
  produces ten one-time recovery keys; the database stores only their hashes.
- First-run setup prints a 128-bit random token once. Only its SHA-256 digest
  is stored in `setup_token.txt`, which is removed after setup completes.
- The built-in operator bearer token is generated on first startup, stored in
  `{config_dir}/secrets.yml` as `server.token`, and printed once. It uses the
  `adm_` prefix and is not stored in the administrator token table.

The HTTP server can terminate TLS on `server.https_port` (443 by default),
including automatic Let's Encrypt certificates for eligible custom domains.
Custom-domain DNS-provider credentials and issued certificates are encrypted
with AES-256-GCM using `installation.secret`; losing that key makes the stored
secrets unrecoverable. Session cookies receive the `Secure` attribute when the
Go process itself receives a TLS connection. Plain HTTP and plain NNTP do not
encrypt traffic. NNTPS and STARTTLS start when enabled and a valid certificate
is available. `AUTHINFO USER/PASS` is accepted only on encrypted NNTP sessions
and validates verified, unsuspended regular-user records. Regular users can
register when `users.registration.mode` is `open`; verification is required
only when outbound SMTP is available. Posting is authentication-required by
default; if that requirement is disabled, posts are limited to public groups
with open posting policy.

For custom-domain HTTPS redirects behind a TLS-terminating proxy, forwarded
HTTPS is accepted only when the immediate TCP peer is loopback, private,
link-local, or listed in `server.trusted_proxies.additional`. Public proxy
addresses must be configured explicitly. Client IPs used for login throttling
and session records are resolved from the configured proxy header only when
the immediate peer is trusted. `auto` selects the first present supported
header and walks `X-Forwarded-For` and RFC 7239 `Forwarded` chains from right to
left, stopping after 32 hops. Direct clients cannot override their address
with a forwarding header. Resolved IPv4-mapped and IPv6 addresses are stored
in canonical form.

## Public endpoints and data

These routes are public in the current implementation:

- Public group and article pages, `/groups`, and `/server/about`.
- Health at `/server/healthz`, `/api/v1/server/healthz`, and `/api/healthz`; `/healthz` is optional. Health output contains public-safe fields and status-only component checks.
- Public JSON reads at `/api/v1/groups`,
  `/api/v1/groups/{group}/articles`, and `/api/v1/articles/{messageID}`.
- Read-only GraphQL at `/api/v1/server/graphql` and its browser page at
  `/server/docs/graphql`.
- OpenAPI JSON at `/api/v1/server/swagger`, `/robots.txt`, and `/sitemap.xml`.

Article and group reads expose public data. The article JSON endpoint returns
metadata and a `body_available` flag, not the message body. The public archive
page renders the stored body. Public reads do not require a token. Regular-user
token management is available at `/api/v1/users/tokens`; those routes require a
regular-user bearer token. Administrator API routes currently use the admin
session cookie and CSRF protection where the route is browser-managed.
Administrator bearer tokens are accepted only by APIs that explicitly support
them; the operator token does not replace an administrator session.

## Internal monitoring endpoints

Metrics endpoints are authenticated and should stay behind firewall or proxy
rules that restrict access to monitoring systems. Prometheus, Grafana, and
Loki each require their own bearer token from `secrets.yml`; query-string
tokens are rejected. The root `/metrics` aliases can be disabled while keeping
the internal `/server/metrics` routes available.

The application does not currently provide security-reporting policy
endpoints, `security.txt`, a PGP key, or OAuth/OIDC/LDAP/SAML flows. Admin
token creation and revocation do not yet have a self-service management page.
Unknown routes return a server-rendered 404 page. Do not assume other planned
features exist because they appear in the product plan or a container image.

## Debug endpoints

The `/debug/*` routes exist only when `--debug` or `DEBUG=true` enables them.
Every route requires `Authorization: Bearer {server.token}`; enabling debug
does not disable authentication. The operator token is not accepted in a query
string. Routes include runtime variables, sanitized configuration, registered
routes, database and scheduler status, memory and goroutine summaries, pprof,
and `/debug/error500`, which renders the themed 500 page for E2E checks. The
pprof command-line endpoint returns only the executable name.

## Browser protections

HTTP responses include a nonce-based Content Security Policy,
`X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`,
`Referrer-Policy: strict-origin-when-cross-origin`, a restrictive
`Permissions-Policy`, and cross-origin opener/resource policies. Authentication
and group-management forms are server-rendered and use CSRF tokens.
Each HTTP request receives a UUID v4 request ID that is returned in
`X-Request-ID` and included in internal-error and rate-limit logs. Per-client
HTTP limits and persistent failed-authentication lockouts are configurable on
the administrator security page. Failed-login attempts are process-local and
reset when the server restarts.

## Remaining security gaps

Passkeys/WebAuthn, distributed cache-backed rate limiting, comprehensive audit
events, administrator token management, IP restrictions, and several planned
security controls remain outstanding. HTTP rate-limit configuration is
persisted to `server.yml`; failed-login attempts are process-local. Trusted
proxy handling is active for client IP and public URL resolution. The current
build is not production-ready for a public news service.
