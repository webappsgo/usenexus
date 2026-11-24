# GoSpool

[![License](https://img.shields.io/github/license/webappsgo/usenexus)](LICENSE.md)
[![Documentation](https://readthedocs.org/projects/webappsgo-gospool/badge/?version=latest)](https://webappsgo-gospool.readthedocs.io/)

GoSpool is a self-hosted Usenet and mailing-list bridge. Its planned product
surface includes NNTP/NNTPS, inbound and outbound email, moderation, peering,
subscriber self-service, and a web archive.

The current implementation provides an HTTP server with public group and
article archive pages, a versioned JSON API, a GraphQL read API, first-run
administrator setup, TOTP MFA with recovery keys, health checks, SQLite or
PostgreSQL persistence, and a companion CLI for status and public reads.
Protocol listeners and most mailing-list workflows are not implemented yet;
see the project backlog before deploying this build as a news server.

## Quick start

Build and run unit checks in the project’s Go container:

```sh
make test
make local
```

Start the HTTP server with the defaults:

```sh
./binaries/gospool
```

The first start creates the configuration directory and prints a one-time
setup token to stderr. Open `/server/auth/setup` on the configured HTTP
listener to create the initial administrator. The token is stored only as a
SHA-256 digest and is removed after setup completes.

## Documentation

See the [operator guide](https://webappsgo-gospool.readthedocs.io/) for current
installation, configuration, API, administration, and security details.

## License

GoSpool is distributed under the MIT License. See [LICENSE.md](LICENSE.md).
