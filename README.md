# GoSpool

[![CI](https://github.com/webappsgo/usenexus/actions/workflows/ci.yml/badge.svg)](https://github.com/webappsgo/usenexus/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/webappsgo/usenexus)](https://github.com/webappsgo/usenexus/releases)
[![License](https://img.shields.io/github/license/webappsgo/usenexus)](LICENSE.md)
[![Documentation](https://readthedocs.org/projects/webappsgo-gospool/badge/?version=latest)](https://webappsgo-gospool.readthedocs.io/)

GoSpool is a self-hosted Usenet and mailing-list bridge. NNTP/NNTPS access,
inbound email-to-group delivery, a shared web archive, subscriber accounts,
and scoped user API tokens are available. Mailing-list fanout, moderation,
and peering remain in the product backlog.

## Official site

[https://gospool.dev](https://gospool.dev)

## Current features

- Server-rendered public group and article archive pages.
- Privacy and terms pages with deployment-configured disclosures and terms text.
- NNTP reader commands for public groups and articles, with STARTTLS on the
  plain NNTP port and an optional NNTPS listener. `AUTHINFO USER/PASS` is
  accepted on encrypted NNTP sessions for verified, unsuspended regular-user
  records. Regular-user registration can create eligible accounts when the
  registration mode is `open`; posting is refused by default until such an
  account is verified and authenticated.
- Versioned JSON API, read-only GraphQL, health checks, and aggregate public
  Prometheus metrics.
- First-run administrator setup, password login, TOTP MFA, and recovery keys.
- Regular-user registration, sign-in, profile and password settings, active
  session revocation, user API tokens, and administrator-issued invitations.
- Inbound SMTP list posting at `list+group@domain`, with bounded messages,
  recipient and connection limits, optional STARTTLS, and optional TLS-only
  SMTP AUTH. The listener tries ports 25, 2525, then an OS-assigned port unless
  `SMTP_IN_PORT` explicitly locks the port.
- SQLite by default, with PostgreSQL support.
- Companion CLI for API root, health, public group, and article reads.

Outbound SMTP supports account verification and password reset when a working
relay is configured under `server.notifications.email.smtp`. Administrators can
configure and test the relay at `/server/{admin_path}/email` and manage embedded
email templates there. Environment overrides take precedence over saved SMTP
settings. Without a working relay, email is disabled and registration skips verification. Peer feeds, subscription fanout,
moderation, and most of the planned mailing-list workflows are not implemented.
This build is not ready for production use as a public news server.

## Production deployment

The Compose files publish the HTTP, NNTP, NNTPS, and SMTP ports. The inbound
SMTP listener uses the configured fallback port sequence. The default HTTP,
NNTP, and SMTP ports (80, 119, and 25) are privileged on Unix-like systems.
Use a service manager or choose unprivileged ports for local use.

## Client

Build the server and CLI in the project’s Go container:

```sh
make local
```

Start the server with defaults (HTTP port 80, NNTP port 119, and inbound SMTP
port 25 with fallback):

```sh
./binaries/gospool
```

For unprivileged local ports:

```sh
PORT=8080 NNTP_PORT=1119 SMTP_IN_PORT=2525 ./binaries/gospool
```

On first start the server creates its configuration and prints a one-time
setup token to stderr. Open `http://127.0.0.1:8080/server/auth/setup` (or the
configured HTTP address) to create the initial administrator. The token
digest is removed after setup succeeds.

The CLI is built as `binaries/gospool-cli` and supports `api`, `status`,
`groups`, and `groups articles NAME`. For example:

```sh
./binaries/gospool-cli --server http://127.0.0.1:8080 groups
```

## Configuration

The server creates `server.yml` on first start. See the
[configuration guide](https://webappsgo-gospool.readthedocs.io/en/latest/configuration/)
for listener, user-registration, and environment settings. HTTP TLS termination
is not implemented. NNTP credentials are accepted after STARTTLS or over NNTPS.
Inbound SMTP can use the configured TLS certificate for STARTTLS. Outbound SMTP
uses the configured `server.notifications.email.smtp` relay and TLS mode.

## API

- API root: `https://gospool.dev/api/v1`
- Health: `https://gospool.dev/api/v1/server/healthz`
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

GoSpool is under active development. The current implementation has not
implemented the complete Usenet and mailing-list behavior described in the
product plan. Review the documentation and test the software in a controlled
environment before relying on it.

## License

GoSpool is distributed under the MIT License. See [LICENSE.md](LICENSE.md).
