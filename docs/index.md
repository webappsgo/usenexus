# GoSpool

GoSpool is a self-hosted Usenet and mailing-list bridge. Each hosted newsgroup
has one canonical article archive that is available through NNTP, email
delivery, and the web interface. Operators can run private communities or
exchange selected groups with other news servers.

## Available features

- Server-rendered public group and article archive pages, with public Atom and
  RSS feeds and private-group filtering.
- NNTP/NNTPS read access, STARTTLS, and authenticated posting controls for
  verified regular-user accounts.
- TLS-authenticated inbound and queued outbound peer feeds with per-peer
  wildmat filters and per-group peer-exchange controls.
- Subscriber registration, double-opt-in group subscriptions, account settings,
  and durable real-time article delivery by email.
- Administrator setup and login, TOTP MFA, user and organization management,
  custom domains, and TLS certificate management.
- A versioned REST API, read-only GraphQL, OpenAPI/Swagger, authenticated
  monitoring endpoints, and a companion CLI.
- Inbound list email, outbound SMTP settings, scheduler controls, GeoIP and
  rate-limit administration, and optional Tor and I2P overlays.

## Current limitations

Web review of held articles, subscriber trust bypass, per-group non-member
policies and sender overrides, emergency moderation, and daily, weekly, and
threshold subscriber digests are available. Email-based moderation and parts
of the complete RFC 3977/4644 NNTP command set are not yet available.
See the [development guide](development.md) and [CLI guide](cli.md) for the
implemented surface and its limits. The README also summarizes the current
release state.

The server renders pages without requiring JavaScript. Browser scripts enhance
forms and navigation where available. Protocol, administration, and deployment
details are documented in the guides below.

## Guides

- [Installation](installation.md)
- [Configuration](configuration.md)
- [Custom domains](custom-domains.md)
- [Public feeds](feeds.md)
- [HTTP API](api.md)
- [Command-line client](cli.md)
- [Administrator access](admin.md)
- [Security](security.md)
- [GeoIP country controls](geoip.md)
- [Integrations](integrations.md)
- [Development](development.md)
- [Translating](translating.md)

## Links

- [Project repository](https://github.com/webappsgo/usenexus)
- [Project site](https://gospool.dev)
- [OpenAPI document](https://gospool.dev/api/v1/server/swagger)
- [GraphQL explorer](https://gospool.dev/server/docs/graphql)
- [MIT License](https://github.com/webappsgo/usenexus/blob/main/LICENSE.md)
