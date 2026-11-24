# GoSpool

[![CI](https://github.com/webappsgo/usenexus/actions/workflows/ci.yml/badge.svg)](https://github.com/webappsgo/usenexus/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/webappsgo/usenexus)](https://github.com/webappsgo/usenexus/releases)
[![License](https://img.shields.io/github/license/webappsgo/usenexus)](LICENSE.md)
[![Documentation](https://readthedocs.org/projects/webappsgo-gospool/badge/?version=latest)](https://webappsgo-gospool.readthedocs.io/)

GoSpool is a self-hosted Usenet and mailing-list bridge. Its planned product
surface includes NNTP/NNTPS, inbound and outbound email, moderation, peering,
subscriber self-service, and a web archive.

## Official site

[https://gospool.dev](https://gospool.dev)

## Current features

- Server-rendered public group and article archive pages.
- Plain NNTP reader commands for public groups and articles. `AUTHINFO USER/PASS`
  is accepted only on encrypted NNTPS, for verified, unsuspended
  regular-user records. There is no account-creation flow, so posting is
  refused by default unless operators have provisioned eligible records.
- Optional NNTPS listener when TLS is enabled and a valid certificate is
  configured or discovered.
- Versioned JSON API, read-only GraphQL, health checks, and aggregate public
  Prometheus metrics.
- First-run administrator setup, password login, TOTP MFA, and recovery keys.
- SQLite by default, with PostgreSQL support.
- Companion CLI for API root, health, public group, and article reads.

SMTP, peer feeds, STARTTLS, subscriber management, moderation, and most of the
planned mailing-list workflows are not implemented. This build is not ready
for production use as a public news server.

## Production deployment

The shipped Compose files publish ports for planned services that the current
server does not bind. Build and run locally to explore the available behavior.
The default HTTP and NNTP ports (80 and 119) are privileged on Unix-like
systems. Use a service manager or choose unprivileged ports for local use.

## Client

Build the server and CLI in the project’s Go container:

```sh
make local
```

Start the server with defaults (HTTP port 80 and NNTP port 119):

```sh
./binaries/gospool
```

For unprivileged local ports:

```sh
PORT=8080 NNTP_PORT=1119 ./binaries/gospool
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
for implemented settings and environment overrides. HTTP TLS termination and
SMTP are unavailable. NNTP credentials are accepted over NNTPS, but the
application does not provide an account-creation flow.

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
