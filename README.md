# GoSpool

[![CI](https://github.com/webappsgo/usenexus/actions/workflows/ci.yml/badge.svg)](https://github.com/webappsgo/usenexus/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/webappsgo/usenexus)](https://github.com/webappsgo/usenexus/releases)
[![License](https://img.shields.io/github/license/webappsgo/usenexus)](LICENSE.md)
[![Documentation](https://readthedocs.org/projects/webappsgo-gospool/badge/?version=latest)](https://webappsgo-gospool.readthedocs.io/)

GoSpool is a self-hosted Usenet and mailing-list bridge. NNTP/NNTPS access,
inbound email-to-group delivery, a shared web archive, subscriber accounts,
scoped user API tokens, double-opt-in subscriptions, and durable real-time
subscriber email delivery are available. Configured NNTP peers support
authenticated TLS inbound feeds, NEWNEWS-based passive pull, queued outbound
push with retry, wildmat filters, per-group peering controls, article age
cutoff, and per-connection article/byte rate holds. Peer control messages are
held for administrator review before group changes are applied; the complete
article moderation workflow remains outstanding. Prometheus reports
connected peers, peer articles in/out, and queued peer deliveries.

## Official site

[https://gospool.dev](https://gospool.dev)

## Current features

- Server-rendered public group and article archive pages.
- Public Atom/RSS feeds for the archive and each public group, with private
  group filtering and conditional HTTP caching.
- Privacy and terms pages with deployment-configured disclosures and terms text.
- NNTP reader commands for public groups and articles, with STARTTLS on the
  plain NNTP port and an optional NNTPS listener. `AUTHINFO USER/PASS` is
  accepted on encrypted NNTP sessions for verified, unsuspended regular-user
  records. Regular-user registration can create eligible accounts when the
  registration mode is `open`; posting is refused by default until such an
  account is verified and authenticated.
- Versioned JSON API, read-only GraphQL, health checks, and authenticated
  Prometheus, Grafana, and Loki monitoring endpoints.
- Built-in, database-backed scheduling with restart catch-up, run history, and
  a server-rendered administrator task page at
  `/server/{admin_path}/scheduler`; task settings and history are also exposed
  through the versioned administrator API.
- Server-rendered peer configuration at `/server/{admin_path}/peers` and
  bearer-authenticated peer management and connectivity checks at `/api/v1/peers`.
- First-run administrator setup, password login, TOTP MFA, and recovery keys.
- Regular-user registration, sign-in, profile and password settings, active
  session revocation, user API tokens, and administrator-issued invitations.
- Server-rendered subscription join and management pages, email confirmation,
  member preferences, and persistent retrying real-time article fanout.
- Inbound SMTP list posting at `list+group@domain`, with bounded messages,
  recipient and connection limits, optional STARTTLS, and optional TLS-only
  SMTP AUTH. The listener tries ports 25, 2525, then an OS-assigned port unless
  `SMTP_IN_PORT` explicitly locks the port.
- SQLite by default, with PostgreSQL support.
- User and organization custom domains with TXT ownership checks, public-page
  routing, Let's Encrypt HTTP-01/TLS-ALPN-01 certificates, and encrypted
  Cloudflare DNS-01 credentials for wildcard or firewalled domains.
- Companion CLI for API root, health, public group/article reads, and scoped API
  token management. Its responsive TUI provides a view for each current CLI
  command.

Outbound SMTP supports account verification and password reset when a working
relay is configured under `server.notifications.email.smtp`. Administrators can
configure and test the relay at `/server/{admin_path}/email` and manage embedded
email templates there. Environment overrides take precedence over saved SMTP
settings. Without a working relay, email is disabled and registration skips
verification. Only `realtime` subscriber delivery is currently implemented;
digest delivery, article moderation, and other planned mailing-list workflows
remain incomplete. Inbound and queued outbound NNTP peer feeds use configured
wildmat filters and per-group peering controls. Peer `newgroup`, `rmgroup`, and
`checkgroups` messages are held for explicit administrator review and recorded
in the audit log.
This build is not ready for production use as a public news server.

## Production deployment

The Compose files publish the HTTP, HTTPS, NNTP, NNTPS, and SMTP ports. The inbound
SMTP listener uses the configured fallback port sequence. The default HTTP,
NNTP, and SMTP ports (80, 119, and 25) are privileged on Unix-like systems.
Use a service manager or choose unprivileged ports for local use.

## Client

Build the server and CLI in the project’s Go container:

```sh
make local
```

Start the server with defaults (HTTP port 80, HTTPS port 443, NNTP port 119,
and inbound SMTP port 25 with fallback):

```sh
./binaries/gospool
```

For unprivileged local ports:

```sh
PORT=8080 HTTPS_PORT=8443 NNTP_PORT=1119 SMTP_IN_PORT=2525 ./binaries/gospool
```

On first start the server creates its configuration and prints a one-time
setup token to stderr. Open `http://127.0.0.1:8080/server/auth/setup` (or the
configured HTTP address) to create the initial administrator. The token
digest is removed after setup succeeds.

The CLI is built as `binaries/gospool-cli` and supports `api`, `status`,
`groups`, article reads, token management, and an interactive TUI when run
without arguments in a terminal. For example:

```sh
./binaries/gospool-cli --server http://127.0.0.1:8080 groups
```

## Configuration

The server creates `server.yml` on first start. See the
[configuration guide](https://webappsgo-gospool.readthedocs.io/en/latest/configuration/)
for listener, user-registration, custom-domain, and environment settings. The
HTTPS listener defaults to port 443; a trusted front proxy can terminate TLS
instead. NNTP credentials are accepted after STARTTLS or over NNTPS.
Inbound SMTP can use the configured TLS certificate for STARTTLS. Outbound SMTP
uses the configured `server.notifications.email.smtp` relay and TLS mode.

## API

- API root: `https://gospool.dev/api/v1`
- Health: `https://gospool.dev/api/v1/server/healthz`
- Unversioned health: `https://gospool.dev/api/healthz`
- OpenAPI JSON: `https://gospool.dev/api/v1/server/swagger`
- GraphQL endpoint: `https://gospool.dev/api/v1/server/graphql`
- GraphQL browser page: `https://gospool.dev/server/docs/graphql`

Only public reads are available without an administrator session. See the
[API guide](https://webappsgo-gospool.readthedocs.io/en/latest/api/) for
request and response details.

## Documentation

See the [operator guide](https://webappsgo-gospool.readthedocs.io/) for
installation, administration, security, integrations, and development.

## Development

Build in the required Go container with `make local`. The primary implementation
backlog is in `TODO.AI.md`; `AI.md` defines the required project baseline and
`SPEC.md` records project-specific overrides.

## Disclaimer

GoSpool is under active development. The implementation is still being aligned
with the complete Usenet and mailing-list behavior described in the project
specification. Review the documentation and test the software in a controlled
environment before relying on it.

## License

GoSpool is distributed under the MIT License. See [LICENSE.md](LICENSE.md).
