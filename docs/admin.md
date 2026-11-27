# Administrator access

## First-run setup

On first run, open `/server/auth/setup` and use the one-time token printed to
stderr. The form creates the primary administrator and starts a session. The
token digest is stored in `{config_dir}/setup_token.txt`; that file is removed
after setup succeeds. The installation encryption key is stored in
`{config_dir}/installation.secret` with owner-only permissions.

The setup form requires a username, email, and matching password fields. The
server does not send a welcome email. Keep the one-time setup token private and
complete setup before exposing the service.

## Sign-in and MFA

- Sign in at `/server/auth/login?admin=1`. Sign out from the administration
  page. The administrator sign-in route is intentionally not advertised on
  public pages.
- The `admin_session` cookie expires after 30 days absolutely and after 24
  hours of inactivity. Requests extend inactivity expiry but never the
  absolute lifetime.
- An administrator can enroll an authenticator at `/server/auth/totp`.
  Enrollment requires a valid six-digit TOTP code and the pending setup expires
  after ten minutes.
- Enrollment displays ten one-time recovery keys. Save them outside the server;
  the server stores only their hashes.
- Once TOTP is enabled, login accepts an authenticator code or an unused
  recovery key.

## Current administration page

The configured path is `/server/{server.admin_path}` (default:
`/server/administration`). The page lists groups and HTTP, NNTP, NNTPS, and
inbound SMTP listener addresses, links to health, scheduler tasks, rate-limit
and GeoIP controls, email settings, and the OpenAPI JSON document, and includes
a sign-out form. It also shows cached health status, process uptime, HTTP
request count, HTTP 5xx count for the previous 24 hours, and the five nearest
upcoming scheduled tasks. NNTPS and SMTP show as disabled when their listeners
are not available from the active configuration.

The administrator-only server information page at
`/server/{server.admin_path}/info` shows the running version, build metadata,
Go version, operating system, architecture, server mode, and database driver.

The group manager at `/server/{server.admin_path}/groups` lists all public and
private groups and provides standard HTML forms to create, rename, update, and
delete them. Mutations use the administrator session and CSRF token; no
JavaScript is required. Group forms expose name, description, visibility,
posting policy, peer-exchange setting, and the default delivery mode for new
subscribers (`realtime`, `digest`, `nomail`, or `announcement`).
Digest groups can deliver daily, weekly, or when either a configured article
count or total message-size threshold is reached. Message sizes accept
human-readable units such as `1MB`; leaving a threshold at zero disables it.
Peer ingestion is accepted only from enabled inbound peers over TLS. Peer
wildmat filters and each group's peer-exchange setting control which groups
receive incoming articles and which matching articles enter the durable
outbound push queue. Failed pushes retry through the `peer_feed_retry`
scheduler task. Enabled inbound peers are also pulled with NEWNEWS and ARTICLE.
The `nntp.peer_articles_per_minute` and `nntp.peer_bytes_per_minute` settings
place a temporary hold on a peer connection when its feed exceeds either
limit. Inbound `newgroup`, `rmgroup`, and `checkgroups` messages are held at
`/server/{server.admin_path}/peer-controls` for explicit administrator
approval, rejection, or dismissal. Approved `newgroup` and `rmgroup` commands
apply the requested group lifecycle operation; `checkgroups` adds valid
missing groups without deleting local groups. Every review is recorded in the
audit log. Posts to moderated groups are held at
`/groups/{group}/queue`; moderators or server administrators can approve or
reject them through CSRF-protected forms. Approval publishes the article and
starts subscriber fanout. Administrators can promote and demote active,
verified users through the no-JavaScript forms on the Groups page or the
versioned moderator-management API. Email-based review, rejection notifications,
and per-origin trust rules are not available yet.

The scheduler page at `/server/{server.admin_path}/scheduler` lists registered
tasks and their last and next run state. Select a task to view its recent run
history and edit its schedule. Run, enable, disable, and schedule changes use
CSRF-protected forms. The versioned administrator scheduler API also supports
schedule and enabled-state changes. `digest_delivery` checks every 15 minutes
for due batches and retries failed SMTP sends. Groups default to daily digests
after 24 hours; administrators can select a rolling weekly window or a
threshold trigger on the group administration page. A threshold digest is sent
when its configured article-count or human-readable size threshold is reached.

Regular-user invitations and account suspension are available at
`/server/{server.admin_path}/users`. Invite links are single-use and shown once
to the administrator for secure delivery. The administrator cannot set a
user’s password. The `POST /api/v1/groups` operation requires the administrator
session cookie and CSRF token. The authenticated API also supports listing,
previewing, updating, and resetting embedded account email templates at
`/api/v1/server/{server.admin_path}/email/templates`. Outbound SMTP settings
and test-email controls are available on the browser page at
`/server/{server.admin_path}/email` and through the corresponding email API
routes. SMTP changes are saved before the relay handshake; a failed handshake
keeps the settings and disables email delivery until a later successful check.
Environment variables in the `SMTP_*` family take precedence and are not
written into `server.yml` by the settings page.
HTTP rate limits are configurable at
`/server/{server.admin_path}/security/ratelimit` and through the
`/api/v1/server/{server.admin_path}/security/ratelimit` API using an `adm_`
Bearer token. Changes apply immediately and are persisted to `server.yml`.
Organization management, user invitations, and account suspension are
available in the current server UI and API. Backup settings, encrypted password
storage, scheduled backup controls, manual backup creation, listing, download,
and deletion are available at `/server/{server.admin_path}/backup` and through
the matching administrator backup API. Restores require the server to be
stopped and are not available through the live UI/API yet. Service-management
controls are not available yet.

GeoIP country settings and operator-authored presets are managed at
`/server/{server.admin_path}/security/geoip` and through the matching
`/api/v1/server/{server.admin_path}/security/geoip` API. Applying a preset only
copies its country codes into the form; the operator reviews and saves the
country policy separately. GeoIP settings and presets are persisted in
`server.yml`.

Regular-user self-registration is available when `users.registration.mode` is
`open`; `private` disables the public registration page. When an outbound SMTP
relay is configured and passes the startup handshake, registration email is
verified through a one-time link and password resets are delivered by email.
Reset links expire after one hour by default; `auth.password_reset_ttl`
configures their lifetime.
Without a working relay, email features are disabled and registration skips
email verification.
NNTP `AUTHINFO` over STARTTLS or NNTPS accepts verified, active regular-user
credentials.

## Custom-domain administration

Review custom domains, status, SSL errors, and audit history at
`/server/{admin_path}/domains`. An administrator can suspend a domain for abuse,
terms, billing, security, or an administrative request; unsuspend it; force
delete it while retaining the audit record; and force certificate renewal.
These actions are also available through the administrator domain API and
require the active administrator session plus CSRF protection for mutations.
See the [custom-domain guide](custom-domains.md) for user and organization setup.
