# Integrations

## Available

- SQLite is the default database.
- PostgreSQL is supported by the storage package through the `pgx` driver.
- Plain NNTP listens on port 119 by default and supports STARTTLS when TLS is
  enabled with a valid certificate. NNTPS on port 563 is also available under
  those TLS conditions.
- Inbound SMTP receives posts at `list+group@{server.fqdn}` and injects them
  into matching public groups. Email commands are accepted at
  `{group}-request@{server.fqdn}`, `commands@{server.fqdn}`, and
  `list+commands@{server.fqdn}`; a recognized first-line command sent to
  `list+group@{server.fqdn}` is also accepted. `SUBSCRIBE [group]` uses the
  group's configured default delivery mode, `UNSUBSCRIBE [group]` removes the
  subscription, `DIGEST [group]` selects digest delivery, and `HELP` lists the
  command syntax. Group-specific addresses allow the group argument to be
  omitted. Each command requires a single-use confirmation link sent to the
  verified primary email of an active account. It defaults to port 25 and tries
  2525 then an OS-assigned port when the configured port cannot bind, unless
  `SMTP_IN_PORT` explicitly locks the port.
- Outbound SMTP can deliver registration verification and password-reset
  messages through `server.notifications.email.smtp`. The relay is tested at startup;
  account-email delivery is disabled if the handshake fails. The sender applies
  the configured TLS mode and builds RFC 5322 multipart messages. All 27 default
  notification templates are embedded in the server; overrides in
  `{config_dir}/template/email/email_verify.tmpl` and
  `{config_dir}/template/email/password_reset.tmpl` are read on every send.
  Each template starts with `Subject: ...`, then a `---` separator and a
  plain-text body. Account templates use `{verify_link}` or `{reset_link}` as
  appropriate, plus `{recipient_email}`, `{app_name}`, `{fqdn}`, and `{expires}`;
  the HTML alternative is generated from the rendered text.
- Confirmed `realtime` subscriptions receive accepted articles through a
  durable outbound queue. `digest` subscriptions receive RFC 2046
  `multipart/digest` batches on the group's daily (default), rolling weekly, or
  threshold schedule. Thresholds can use article count, accumulated message
  size, or either configured trigger. Queue rows are created with article
  links, duplicate Message-IDs do not enqueue a second copy, and failed sends
  retry with exponential backoff capped at 24 hours. The worker handles up to
  50 deliveries per 15-second pass. `nomail` keeps the subscription active for
  NNTP reading while suppressing email. `announcement` subscriptions are
  stored; article delivery filtering to moderator and administrator posts is
  still in progress.
- The HTTP API supports JSON and a read-only GraphQL endpoint. The OpenAPI JSON
  document is at `/api/v1/server/swagger`; interactive Swagger UI is at
  `/server/docs/swagger`; GraphQL is at `/server/docs/graphql` and
  `/api/v1/server/graphql`.
- Public discovery includes `/robots.txt` and `/sitemap.xml`. There are no
  `/.well-known/*`, OAuth/OIDC, app-association, webhook, or federation
  endpoints in the current server.

### NNTP reader

The reader supports `CAPABILITIES`, `HELP`, `QUIT`, `DATE`, `MODE READER`,
`LIST ACTIVE`, `LIST ACTIVE.TIMES`, `LIST NEWSGROUPS`, `LIST OVERVIEW.FMT`,
`GROUP`, `LISTGROUP`, `ARTICLE`, `HEAD`, `BODY`, `STAT`, `NEXT`, `LAST`,
`OVER`, `XOVER`, `HDR`, `XHDR`, `STARTTLS`, and `POST`. Only public groups are exposed.
`AUTHINFO USER/PASS` is available only on encrypted NNTP sessions, either after
STARTTLS or over NNTPS. It checks verified, unsuspended regular-user credentials.
Regular users can register (when mode is `open`), sign in, and manage their
profile, password, sessions, and API tokens. Unauthenticated `POST` receives
`480 authentication required` by default. With posting authentication disabled,
non-member posts follow each public group's policy, which defaults to `hold`;
groups can hold, reject, silently discard, or accept them. Authenticated
confirmed subscribers and verified allowlisted sender addresses can bypass the
ordinary hold, while emergency moderation always queues posts. The implemented command set includes
IHAVE, STREAMING, CHECK, TAKETHIS, NEWNEWS, NEWGROUPS, XPAT, and LIST variants.
Peer configuration is available in the server-rendered `/server/{admin_path}/peers`
page and authenticated `/api/v1/peers` endpoints. Authenticated TLS peers can
push articles with IHAVE or STREAMING; configured
inbound peers are also pulled with NEWNEWS and ARTICLE. Outbound articles are
queued and retried by the `peer_feed_retry` scheduler task. Wildmat filters and
each group's peer-exchange setting apply in both directions. The
`nntp.peer_articles_per_minute` and `nntp.peer_bytes_per_minute` settings impose
per-connection limits and a temporary hold. Peer control messages are held
for administrator review at `/server/{admin_path}/peer-controls`; approval is
required before group changes are applied. Moderated articles can be reviewed
through the server-rendered `/groups/{group}/queue` page by a group moderator or
server administrator. Server administrators can promote or demote active,
verified users through the Groups page and moderator-management API. Email-based
review and rejection notifications are not implemented yet. Prometheus exports connected peer counts,
inbound/outbound article counters, and outbound queue depth. If `nntp.auth_required_for_read` is enabled,
authenticated content reads require STARTTLS or NNTPS; plaintext NNTP cannot
authenticate.

## Planned

Webhooks, ACME certificate renewal, and Mailman/INN imports are part of the
product plan but are not currently available in the running server. Outbound
SMTP currently covers account
verification, password reset, and confirmed `realtime` subscription fanout.
All 27 embedded email templates can be listed, previewed, updated, and reset through the admin interface at
`/server/{admin_path}/email/templates`. Administrators can configure and test
the relay at `/server/{admin_path}/email`.
