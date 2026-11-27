# Configuration

The server starts with `server.yml`, then recursively merges regular `.yml` and
`.yaml` files in `yaml.d/` in filename order. A later scalar or list replaces
the earlier value; maps merge by key. Hidden files, backups, unsupported
extensions, and symlinks that resolve outside `yaml.d/` are ignored. For
example, maintenance settings belong in `yaml.d/maintenance.yml`:

```yaml
server:
  restore:
    max_archive_size: 20GB
    max_expanded_size: 50GB
    max_files: 1000000
    max_path_length: 4096
```

On first start the server writes these defaults to `server.yml`:

```yaml
server:
  address: "[::]"
  port: 80
  https_port: 443
  trusted_proxies:
    additional: []
    client_ip_header: auto
  security:
    allowlist: []
  healthz:
    root:
      enabled: false
  geoip:
    enabled: true
    deny_countries: []
    allow_countries: []
    presets: {}
  admin_path: administration
  api_version: v1
  mode: production
  rate_limit:
    read: {requests: 120, window: 60}
    write: {requests: 10, window: 60}
    health: {requests: 120, window: 60}
    strict: {requests: 10, window: 900}
    auth_failures: {requests: 5, window: 900}
    global_burst: 240
    auth:
      login: {requests: 5, window: 900}
      password_reset: {requests: 3, window: 3600}
      registration: {requests: 5, window: 3600}
      file_upload: {requests: 10, window: 3600}
  seo:
    feed:
      max_items: 50
      title: ""
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
      blocked_patterns: [".*\\.(gov|mil|edu)$"]
  ssl:
    enabled: false
    cert: ""
    key: ""
    min_version: TLS1.2
  privacy:
    data:
      sold: false
      stored_on_server: true
      sharing:
        - condition: email
          when: SMTP configured for sending emails
          data: Email address and message content
    retention:
      period: Account data is retained while the account is active. Account deletion requests are completed within 30 days.
      export_available: true
      deletion_available: true
    cookies:
      essential:
        enabled: true
        description: Required for sessions, CSRF, and authentication.
      preferences:
        enabled: true
        description: Visitor-selected display and language preferences.
      analytics:
        enabled: false
        description: Analytics cookies.
      advertising:
        enabled: false
        description: Advertising cookies.
    third_party:
      services: []
  branding:
    content:
      privacy: ""
      terms: ""
  notifications:
    email:
      smtp:
        host: ""
        port: 587
        username: ""
        password: ""
        tls: auto
      from:
        name: ""
        email: ""
  scheduler:
    timezone: America/New_York
    catch_up_window: 1h
    tasks:
      ssl_renewal: {schedule: "0 3 * * *", enabled: true}
      geoip_update: {schedule: "0 3 * * 0", enabled: true}
      blocklist_update: {schedule: "0 4 * * *", enabled: true, retry_on_fail: true, retry_delay: 1h}
      security_cleanup: {schedule: "0 3 * * *", enabled: true}
      keyserver_retry: {schedule: "@every 15m", enabled: true}
      peer_feed_retry: {schedule: "@every 1m", enabled: true, retry_on_fail: true, retry_delay: 1m}
      digest_delivery: {schedule: "@every 15m", enabled: true, retry_on_fail: true, retry_delay: 15m}
      cve_update: {schedule: "0 5 * * *", enabled: true, retry_on_fail: true, retry_delay: 1h}
      update_check: {schedule: "0 6 * * *", enabled: true}
      session_cleanup: {schedule: "@every 15m", enabled: true}
      token_cleanup: {schedule: "@every 15m", enabled: true}
      log_rotation: {schedule: "0 0 * * *", enabled: true}
      backup_daily: {schedule: "0 2 * * *", enabled: true}
      backup_hourly: {schedule: "@hourly", enabled: false}
      healthcheck_self: {schedule: "@every 5m", enabled: true}
      tor_health: {schedule: "@every 10m", enabled: true}
      i2p_health: {schedule: "@every 10m", enabled: true}
      cluster_heartbeat: {schedule: "@every 30s", enabled: true}
      custom_domain_verification: {schedule: "*/15 * * * *", enabled: true}
      custom_domain_ssl_renewal: {schedule: "0 4 * * *", enabled: true}
      custom_domain_cleanup: {schedule: "0 5 * * *", enabled: true}
  metrics:
    enabled: true
    root:
      enabled: true
    auth:
      allow_unauthenticated: false
      tokens:
        prometheus: ""
        grafana: ""
        loki: ""
    include_system: true
    include_runtime: true
    loki:
      max_entries: 1000
      max_age: 1h
    duration_buckets: [0.001, 0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]
    size_buckets: [100, 1000, 10000, 100000, 1000000, 10000000]
nntp:
  address: 0.0.0.0
  port: 119
  tls_port: 563
  mode: both
  peer_max_age: 720h
  peer_articles_per_minute: 60
  peer_bytes_per_minute: 10485760
  max_connections: 256
  max_connections_per_ip: 10
  auth_required_for_post: true
  auth_required_for_read: false
smtp_in:
  address: 0.0.0.0
  port: 25
  require_starttls: false
  auth_enabled: false
  require_auth: false
  max_message_size: 10485760
  max_recipients: 50
  max_connections: 256
users:
  enabled: true
  registration:
    mode: open
    require_email_verification: true
    allowed_domains: []
    blocked_domains: []
    invite_expiration_days: 7
auth:
  password_reset_ttl: 1h
database:
  driver: sqlite
```

Scheduler configuration is loaded and validated. Custom-domain verification,
SSL renewal, and pending-domain cleanup run on their configured schedules.
The `digest_delivery` task checks for subscriber digest batches every 15
minutes; it sends a batch after its daily 24-hour window and retries temporary
SMTP failures. Weekly windows and per-group threshold triggers are not
available yet. Other configured scheduler tasks are documented with the
scheduler and remain subject to their implementation status in the operator
guide.

The implemented configuration fields are:

| Field | Default | Notes |
|---|---|---|
| `server.address` | `[::]` | HTTP bind address; `ADDRESS` overrides it. |
| `server.port` | `80` | HTTP port; `PORT` overrides it. |
| `server.https_port` | `443` | Direct HTTPS listener for automatic TLS certificates; `HTTPS_PORT` overrides it. Set to `0` when a trusted front-end terminates HTTPS. |
| `server.trusted_proxies.additional` | `[]` | Additional proxy IPs, CIDRs, or DNS names allowed to supply forwarding headers. Private and loopback peers are trusted by default. DNS names are refreshed every five minutes. |
| `server.trusted_proxies.client_ip_header` | `auto` | Selects `X-Forwarded-For`, `Forwarded`, `CF-Connecting-IP`, `True-Client-IP`, `X-Real-IP`, or `X-Client-IP`; pin to the header your front proxy overwrites or appends. |
| `server.security.allowlist` | `[]` | Trusted IP addresses or CIDRs that bypass IP enforcement and rate limits, but never authentication, authorization, or CSRF checks. |
| `server.healthz.root.enabled` | `false` | Mounts `/healthz` as an exact content-negotiated alias of `/server/healthz`. |
| `server.geoip.enabled` | `true` | Enables country lookups and country blocking. Missing databases and lookup errors fail open. |
| `server.geoip.deny_countries` | `[]` | Uppercase ISO 3166-1 alpha-2 country codes to block. |
| `server.geoip.allow_countries` | `[]` | If set, only these countries pass; this takes precedence over the deny list. |
| `server.geoip.presets` | `{}` | Operator-authored country code lists; presets never apply until selected and reviewed. |
| `server.rate_limit.read` | `120` requests / `60` seconds | Per-client read requests; valid authenticated tokens or sessions get an independent identity. |
| `server.rate_limit.write` | `10` requests / `60` seconds | Per-client write requests. |
| `server.rate_limit.health` | `120` requests / `60` seconds | Per-client health and status requests. |
| `server.rate_limit.global_burst` | `240` | Per-client ceiling across endpoint classes. |
| `server.rate_limit.strict` | `10` requests / `900` seconds | Strict token and restore operations. |
| `server.rate_limit.auth.*` | login `5/900`, reset `3/3600`, registration `5/3600`, upload `10/3600` | Additional limits for sensitive authentication and upload routes. |
| `server.seo.feed.max_items` | `50` | Entries in public Atom/RSS feeds; range 1–200. |
| `server.seo.feed.title` | empty | Channel title, defaulting to `GoSpool`. |
| `server.admin_path` | `administration` | One safe path segment. |
| `server.api_version` | `v1` | Version segment used by HTTP API routes. |
| `server.metrics.enabled` | `true` | Enables internal Prometheus, Grafana, and Loki endpoints. |
| `server.metrics.root.enabled` | `true` | Enables `/metrics[/{service}]` aliases; `/server/metrics` stays available when disabled. |
| `server.metrics.auth.tokens.*` | generated | Per-service bearer credentials stored in `secrets.yml`; never put live tokens in `server.yml`. |
| `server.metrics.auth.allow_unauthenticated` | `false` | Skips metrics token checks; only use behind a trusted firewall. |
| `server.metrics.include_system` / `.include_runtime` | `true` | Controls host CPU, memory, data-directory disk, process, and Go runtime metric families. |
| `server.metrics.loki.max_entries` / `.max_age` | `1000` / `1h` | Bounds recent sanitized logs exposed through the Loki service. |
| `server.metrics.duration_buckets` / `.size_buckets` | standard buckets | Histogram buckets for HTTP request duration and size metrics. |
| `server.mode` | `production` | `MODE` can override it. |
| `server.fqdn` | empty | NNTP greeting hostname and certificate hostname check; required for automatic certificate discovery. |
| `server.ssl.enabled` | `false` | Enables NNTPS and STARTTLS on NNTP and inbound SMTP when a valid certificate is available. HTTP TLS uses the separate `server.https_port` listener. |
| `server.ssl.cert` / `server.ssl.key` | empty | Optional certificate and key paths; relative paths resolve under the config directory's `ssl/` folder. Configure both or neither. |
| `server.ssl.min_version` | `TLS1.2` | Minimum TLS version for NNTPS (`TLS1.2` or `TLS1.3`). |
| `server.privacy.data.sold` | `false` | Selects the privacy page's data-use language. |
| `server.privacy.data.stored_on_server` | `true` | States whether data is stored on this server. |
| `server.privacy.data.sharing` | email relay entry | Deployment-configured sharing disclosures shown on `/server/privacy`. |
| `server.privacy.retention.period` | account retention text | Retention statement shown on `/server/privacy`. |
| `server.privacy.retention.export_available` / `.deletion_available` | `true` | States whether account export and deletion are available. |
| `server.privacy.cookies.*` | category settings | Cookie category availability and descriptions shown on `/server/privacy`. |
| `server.privacy.third_party.services` | empty | Additional services, purposes, data sent, and policy URLs shown on `/server/privacy`. |
| `server.branding.content.privacy` | empty | Additional privacy text appended to the built-in policy. |
| `server.branding.content.terms` | empty | Custom terms text replacing the default `/server/terms` content. |
| `nntp.address` | `0.0.0.0` | Plain NNTP bind address. |
| `nntp.port` | `119` | `NNTP_PORT` can override it. Set to `0` to disable plain NNTP. |
| `nntp.tls_port` | `563` | `NNTPS_PORT` can override it; requires `server.ssl.enabled: true` and a valid certificate. |
| `nntp.mode` | `both` | `NNTP_MODE` can override it. `feeder` keeps the NNTP listener available for authenticated peer feeds and rejects direct reader commands. |
| `nntp.peer_max_age` | `720h` | `NNTP_PEER_MAX_AGE` overrides it. Peer articles older than this duration are rejected. |
| `nntp.peer_articles_per_minute` | `60` | `NNTP_PEER_ARTICLES_PER_MINUTE` overrides it. Per authenticated peer connection; exceeding the limit starts a one-minute temporary hold. |
| `nntp.peer_bytes_per_minute` | `10485760` | `NNTP_PEER_BYTES_PER_MINUTE` overrides it. Maximum accepted article bytes per authenticated peer connection per minute; exceeding it starts a one-minute temporary hold. |
| `nntp.max_connections` | `256` | Enforced server-wide connection cap. |
| `nntp.max_connections_per_ip` | `10` | Per-source connection limit. |
| `nntp.auth_required_for_post` | `true` | Unauthenticated POST is refused. AUTHINFO USER/PASS works on encrypted NNTP sessions (STARTTLS or NNTPS) and checks verified, unsuspended regular-user records. Create accounts through the web registration and invitation flows. |
| `nntp.auth_required_for_read` | `false` | When enabled, NNTP content commands require authentication. Authenticate after STARTTLS or over NNTPS; plaintext NNTP cannot authenticate. |
| `smtp_in.address` | `0.0.0.0` | Inbound SMTP bind address; `SMTP_IN_ADDRESS` overrides it. |
| `smtp_in.port` | `25` | `SMTP_IN_PORT` or `--smtp-port` overrides it. Unless explicitly overridden, bind failure falls back to port 2525 and then an OS-assigned port. Set to `0` to disable. |
| `smtp_in.require_starttls` | `false` | Refuses mail transactions until STARTTLS has completed; requires `server.ssl.enabled: true`. |
| `smtp_in.auth_enabled` | `false` | Enables SMTP AUTH PLAIN for verified regular users, only after TLS. Requires `server.ssl.enabled: true`. |
| `smtp_in.require_auth` | `false` | Requires successful SMTP AUTH before accepting MAIL FROM; requires `auth_enabled`. |
| `smtp_in.max_message_size` | `10485760` | Maximum complete inbound message size in bytes (10 MiB). |
| `smtp_in.max_recipients` | `50` | Maximum accepted list recipients per SMTP transaction. |
| `smtp_in.max_connections` | `256` | Maximum concurrent inbound SMTP connections. |
| `server.notifications.email.smtp.host` | empty | Outbound relay hostname. Empty triggers first-run detection across loopback, Docker gateway, default gateway, FQDN, global IPv4, `mail.{fqdn}`, and `smtp.{fqdn}` candidates. |
| `server.notifications.email.smtp.port` | `587` | Outbound relay port. Detection checks 25, 465, then 587. Port 465 selects implicit TLS in `auto` mode. Invalid numeric/range values fall back to 587 with a warning. |
| `server.notifications.email.smtp.username` / `.password` | empty | Optional relay credentials; the SMTP password is stored in `secrets.yml`, and authentication is sent only after TLS is established. |
| `server.notifications.email.smtp.tls` | `auto` | `auto`, `starttls`, `tls`, or `none`; auto uses implicit TLS on 465 and STARTTLS when offered on other ports. Unknown values fall back to `auto` with a warning. |
| `server.notifications.email.from.name` / `.email` | app name / `no-reply@{fqdn}` | Outbound From display name and mailbox. Email enables only after a successful startup handshake. |
| `server.scheduler.timezone` | `America/New_York` | IANA time zone used for cron schedules. The scheduler embeds zone data for minimal containers. |
| `server.scheduler.catch_up_window` | `1h` | Runs a missed task at startup when it is no more than this old. Must be a positive duration. |
| `server.scheduler.tasks.{task}.schedule` | per-task schedule | Five-field cron (Sunday is `0` or `7`), `@hourly`, `@daily`, `@weekly`, `@monthly`, or `@every` seconds, minutes, or hours. |
| `server.scheduler.tasks.{task}.enabled` | per-task setting | Enables or disables a configurable task. Critical maintenance tasks cannot be disabled. |
| `server.scheduler.tasks.{task}.retry_on_fail` / `.retry_delay` | false / task-specific | Enables bounded retries for supported tasks; retry delay is a positive duration. |
| `server.features.custom_domains.*` | see defaults above | Controls owner limits, supported hostname types, verification TTL, SSL requirement, certificate renewal window, and reserved/blocked domains. See [custom domains](custom-domains.md). |
| `SMTP_HOST`, `SMTP_PORT`, `SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_TLS`, `SMTP_FROM_NAME`, `SMTP_FROM_EMAIL` | unset | Override the corresponding `server.notifications.email` values, including changes made on the administrator email settings page. Environment-provided values are not written back to `server.yml`. |
| `users.enabled` | `true` | Enables subscriber accounts and `/users/*` self-service routes. |
| `users.registration.mode` | `open` | `open` allows self-registration; `private` hides public registration while retaining admin-issued invites. |
| `users.registration.require_email_verification` | `true` | Requires one-time email verification when outbound SMTP is configured and passes its startup handshake. Without working SMTP, email is disabled and registration skips verification. |
| `users.registration.allowed_domains` | `[]` | Optional exact or parent domains allowed during public registration and admin invitations. |
| `users.registration.blocked_domains` | `[]` | Domains rejected during public registration and admin invitations. |
| `users.registration.invite_expiration_days` | `7` | Lifetime of admin-issued user invitations. |
| `auth.password_reset_ttl` | `1h` | Lifetime of one-time password reset links. |
| `database.driver` | `sqlite` | SQLite or PostgreSQL (`pgx`). |
| `database.dsn` | empty | `DATABASE_URL` overrides when this field is empty. |
| `web.base_url` | empty | Public origin used when constructing absolute account verification and password-reset links. |
| `GOSPOOL_CUSTOM_DOMAINS_MAX_PER_USER` / `_MAX_PER_ORG` | unset | Overrides per-user and per-organization custom-domain limits. The longer `MAX_DOMAINS_PER_USER` / `MAX_DOMAINS_PER_ORG` spellings are accepted too. |

For NNTPS, `server.ssl.enabled` must be `true` and `nntp.tls_port` must be
greater than zero. STARTTLS on the plain NNTP listener also requires
`server.ssl.enabled` and an available valid certificate. If `server.ssl.cert`
and `server.ssl.key` are empty, the
server searches for a certificate and key in this order:

1. `/etc/letsencrypt/live/domain/`
2. `/etc/letsencrypt/live/{server.fqdn}/`
3. `{config_dir}/ssl/letsencrypt/{server.fqdn}/`
4. `{config_dir}/ssl/local/{server.fqdn}/`

NNTPS verifies the certificate validity dates and, when configured, its
hostname. Certificates are reloaded for new TLS handshakes. The HTTP listener
does not currently provide HTTPS.

The environment overrides currently read by the server are `ADDRESS`, `PORT`,
`MODE`, `DEBUG`, `NNTP_ADDRESS`, `NNTP_PORT`, `NNTPS_PORT`, `NNTP_MODE`,
`SMTP_IN_ADDRESS`, `SMTP_IN_PORT`, `SMTP_IN_REQUIRE_STARTTLS`,
`SMTP_IN_AUTH_ENABLED`, `SMTP_IN_REQUIRE_AUTH`, `SMTP_IN_MAX_MESSAGE_SIZE`,
`SMTP_IN_MAX_RECIPIENTS`, `SMTP_IN_MAX_CONNECTIONS`, `SMTP_HOST`, `SMTP_PORT`,
`SMTP_USERNAME`, `SMTP_PASSWORD`, `SMTP_TLS`, `SMTP_FROM_NAME`,
`SMTP_FROM_EMAIL`, and `DATABASE_URL`. Directory overrides are `CONFIG_DIR`,
`DATA_DIR`, `DB_DIR`, `SPOOL_DIR`, `CACHE_DIR`, `LOG_DIR`, and `BACKUP_DIR`.
SQLite data defaults under the platform-specific database directory.

Rate limits are always on. Windows use seconds and counters are persisted in
the active database. Exceeded requests receive HTTP 429, a `Retry-After`
header, and the canonical JSON error response.

## Restore Limits

Restore limits belong in `yaml.d/maintenance.yml` under `server.restore` and apply to the complete
expanded result when a full archive and incremental archive are combined.
The server checks them before applying any restored file:

| Key | Default | Meaning |
| --- | --- | --- |
| `server.restore.max_archive_size` | `20GB` | Maximum size of one archive file. |
| `server.restore.max_expanded_size` | `50GB` | Maximum combined expanded size. |
| `server.restore.max_files` | `1000000` | Maximum number of archive entries. |
| `server.restore.max_path_length` | `4096` | Maximum archive path length in bytes. |

Byte sizes accept a bare byte count or a `KB`, `MB`, `GB`, or `TB` suffix.
Zero, negative, or malformed values fall back to their defaults with a warning.
