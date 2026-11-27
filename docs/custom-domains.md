# Custom domains

GoSpool lets public user profiles and public organization pages use an owner-managed
hostname. The account owner adds a domain, proves control with DNS, and points
traffic at the GoSpool server. Domains remain unavailable until ownership is
verified. Suspended domains return a status page; deleting a domain keeps its
audit history.

## Add and verify a domain

Open **Custom domains** in the user or organization settings, or use the API.
Supported values are apex domains, subdomains, and optionally wildcards when
`server.features.custom_domains.allow_wildcard` is enabled. The defaults allow
apex and subdomain names, reserve local/test/example names, and disallow
wildcards. Reserved names and configured blocked patterns cannot be registered.

Publish the TXT record shown on the domain page:

- Type: `TXT`
- Name: `_verify.example.org` (for `example.org`)
- Value: the per-domain verification token

Choose **Check DNS verification** after DNS publication. GoSpool also retries
pending verification every 15 minutes. Unverified domains are removed after the
configured verification TTL, which defaults to 24 hours.

After verification, point the hostname at the server. Use a CNAME for a
subdomain or A/AAAA records for an apex domain. The DNS page lists the detected
public IP addresses when available. `/server/healthz` exposes the same public IP
information. A custom hostname serves its owner's public profile or public
organization page; private profiles and organizations are not exposed through
custom hostnames.

## TLS certificates

The HTTPS listener defaults to port 443 (`server.https_port`). Verified,
active custom domains can request an automatic Let's Encrypt certificate. The
autocert manager serves HTTP-01 challenges on the HTTP listener and TLS-ALPN-01
challenges on the HTTPS listener. Forward port 80 and/or 443 to GoSpool when
using those challenges.

For wildcard domains or deployments that cannot receive those challenges,
choose **DNS-01 (Cloudflare)** and provide an active Cloudflare API token with
zone read and DNS record edit access. GoSpool validates the token, encrypts it
with the installation key, and stores only the encrypted value in the database.
The token is reused for renewal and is never returned by domain APIs. Certificate
and private-key material are encrypted at rest as well. Keep
`{config_dir}/installation.secret` backed up and private; certificates and DNS
credentials cannot be decrypted without it.

The default custom-domain SSL renewal scan runs daily at 04:00 in the configured
scheduler timezone. Its renewal window is controlled by
`server.features.custom_domains.ssl_renewal_days` (7 days by default). Pending
verification retry runs every 15 minutes, and expired unverified-domain cleanup
runs daily at 05:00.

## Configuration

```yaml
server:
  https_port: 443 # HTTPS_PORT overrides this value.
  features:
    custom_domains:
      max_domains_per_user: 5
      max_domains_per_org: 20
      require_ssl: true
      allow_apex: true
      allow_subdomain: true
      allow_wildcard: false
      verification_ttl: 24h
      ssl_renewal_days: 7
      reserved: [localhost, "*.local", "*.test", "*.example", "*.invalid"]
      blocked_patterns:
        - ".*\\.(gov|mil|edu)$"
```

Set either limit to `0` for no per-owner limit. The environment overrides are
`GOSPOOL_CUSTOM_DOMAINS_MAX_PER_USER` and
`GOSPOOL_CUSTOM_DOMAINS_MAX_PER_ORG` (the longer
`MAX_DOMAINS_PER_USER` and `MAX_DOMAINS_PER_ORG` names are also accepted).
`require_ssl` redirects custom-domain HTTP requests to HTTPS. Set
`https_port: 0` only when TLS is terminated by a trusted front-end proxy.
For a public-address proxy, add its IP, CIDR, or DNS name to
`server.trusted_proxies.additional`; private and loopback peers are trusted by
default. DNS entries refresh every five minutes.

## User API

User and organization endpoints require an authenticated regular-user session
or bearer token. Organization operations also require an organization role that
can manage its profile. Domain names in path segments must be URL-encoded; for
wildcards encode `*` as `%2A`.

| Method | Path | Purpose |
|---|---|---|
| `GET`, `POST` | `/api/v1/users/domains` | List or add the caller's domains. |
| `GET`, `DELETE` | `/api/v1/users/domains/{domain}` | Read or remove a domain. |
| `POST` | `/api/v1/users/domains/{domain}/verify` | Check the ownership TXT record. |
| `GET` | `/api/v1/users/domains/{domain}/dns` | Read ownership and routing instructions. |
| `GET`, `POST` | `/api/v1/users/domains/{domain}/ssl` | Read SSL state or request certificate issuance. |
| `POST` | `/api/v1/users/domains/{domain}/ssl/renew` | Force renewal using the saved challenge configuration. |

Replace `/users` with `/orgs/{slug}` for organization-owned domains. A
DNS-01 request uses this JSON shape:

```json
{
  "challenge": "dns-01",
  "provider": "cloudflare",
  "credentials": {"api_token": "<token>"}
}
```

An automatic challenge request uses `{"challenge":"auto"}`. Successful domain
creation returns the domain and verification/routing instructions. API errors
use the standard `{ "ok": false, "error": ..., "message": ... }` envelope;
invalid configuration is a 400, an existing domain is a 409, and unavailable
certificate issuance is a 502.

## Administrator controls

Administrators can review all domains at
`/server/{admin_path}/domains`, inspect ownership and SSL state, review the
per-domain audit log, suspend or unsuspend a domain with a recorded reason,
force-delete it, and trigger SSL renewal. The matching API collection is
`/api/v1/server/{admin_path}/domains`; mutations require the administrator
session and CSRF token.
