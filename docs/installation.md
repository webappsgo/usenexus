# Installation

## Build locally

Go compilation uses the project’s Go Docker image:

```sh
make build
```

The server binary is written to `binaries/gospool`. `make test` runs the
Stylelint, Go, and shell checks, coverage-enforced Go tests, and basic Docker
binary checks.

## Run the server

```sh
./binaries/gospool --config /path/to/config --data /path/to/data
```

On first start, GoSpool creates `server.yml`, generates an installation
encryption key, and prints a one-time setup token to stderr. Visit
`/server/auth/setup` at the configured HTTP address to create the first
administrator. Protect the token output and configuration directory.

The server starts HTTP and NNTP listeners and, unless disabled, an inbound SMTP
listener. The NNTP reader supports core group, article, date, posting, and
RFC streaming commands. Posting requires authentication by default.
`AUTHINFO USER/PASS` works on encrypted NNTP sessions and accepts verified,
unsuspended regular-user records. Regular users can register in `open` mode,
sign in, manage their profile and sessions, and create scoped API tokens.
Email verification is enabled by default. When a working outbound SMTP relay
is configured, registration verification and password-reset messages are sent
through it. Without a working relay, email is disabled and registration skips
verification. If NNTP posting authentication is disabled, posts to public
groups follow each group's non-member posting policy, defaulting to moderation.
NNTPS and
STARTTLS start when `server.ssl.enabled` is true and a valid certificate is
available. Inbound SMTP can use that certificate for STARTTLS and accepts
`list+group@{server.fqdn}` recipients for public groups.

The default HTTP, plain NNTP, and inbound SMTP ports are 80, 119, and 25. On
Unix-like systems, binding ports below 1024 requires elevated privileges or an
appropriate service setup. For local development, use unprivileged ports, for
example:

```sh
PORT=8080 NNTP_PORT=1119 SMTP_IN_PORT=2525 ./binaries/gospool
```

This sets the HTTP, plain NNTP, and inbound SMTP ports. The HTTP listener is
not an HTTPS listener. NNTPS uses port 563 by default when enabled; provide a
certificate and key or configure `server.fqdn` for certificate auto-detection.
See [configuration](configuration.md) for certificate lookup paths.

## Container files

The repository includes `docker/Dockerfile` and production, development, and
test Compose files. Container publication remains under development. Inbound
SMTP is active when `smtp_in.port` is nonzero. Outbound email requires a
reachable relay configured under `server.notifications.email.smtp` and a From
mailbox under `server.notifications.email.from`; otherwise email features
remain disabled and registration skips email verification. Inbound NNTP peer
feeds require TLS and configured peer credentials. Outbound push is queued and
retried by the `peer_feed_retry` scheduler task; enabled inbound peers are
pulled with NEWNEWS and ARTICLE. Peer control messages require administrator
review. Moderated posts are held for web review by a group moderator or server
administrator. Digest emails use the `digest_delivery` scheduler task.
Groups default to daily delivery and can select rolling weekly windows or
article-count and human-readable size thresholds. Group administrators can set
non-member posting policies, sender-specific address/glob overrides, and an
emergency moderation hold. Email-based one-click moderation is not available;
moderators review held articles through the web queue.
