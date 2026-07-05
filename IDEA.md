## Project description

gospool is a single self-contained binary that bridges the Usenet world and modern mailing-list
workflows. It speaks both NNTP (RFC 3977) and NNTPS (RFC 4642), making it a fully standards-compliant
news server, while simultaneously treating every newsgroup it hosts as a mailing list.

When a message arrives — whether posted by an NNTP client or sent via email — it is stored once in a
canonical article store and becomes available through both protocols immediately. There is no separate
"list archive" and "newsgroup": they are the same data viewed two ways. Posts fan out as email to
subscribers; inbound emails are injected back as newsgroup articles. NNTP clients see a normal
newsgroup with complete history and threading; email subscribers get standard mailing-list delivery.

Subscribers manage their membership and delivery preferences (real-time vs. digest) the way they would
with a Mailman list — via a web interface or email commands. Moderation, cross-posting, and threading
via `References` / `In-Reply-To` work uniformly regardless of how a message entered the system.

gospool also supports standard Usenet peering, so hosted groups can propagate to and from the wider
network via server-to-server feeds — groups are not siloed unless the operator chooses isolation.

Target users are self-hosters, community operators, and organizations that want a bridge between the
classic Usenet/newsreader workflow and email-list culture, without running two separate systems that
drift out of sync.

## Project variables

project_name:     gospool
project_org:      webappsgo
# FROZEN — set once at first-time setup, never edit
internal_name:    gospool
app_name:         GoSpool
official_site:    gospool.dev
maintainer_name:  webappsgo
maintainer_email: git-admin@casjaysdev.pro

## Business logic

### Product scope & non-goals

**In scope:**
- NNTP server (plain + TLS) — full RFC 3977 / RFC 4642 compliance; full command set: `ARTICLE`, `HEAD`, `BODY`, `STAT`, `GROUP`, `LISTGROUP`, `LAST`, `NEXT`, `POST`, `IHAVE`, `OVER`, `HDR`, `LIST`, `NEWNEWS`, `NEWGROUPS`, `MODE READER`, `QUIT`, `STARTTLS`; RFC 4644 streaming extension (`STREAMING`, `CHECK`, `TAKETHIS`)
- NNTP-to-email fanout: every newsgroup post is delivered to email subscribers
- Email-to-NNTP injection: inbound list email is stored as a newsgroup article
- Single canonical article store — article body on filesystem (hash-sharded by `Message-ID`); headers, metadata, and threading data in SQLite
- Mailing-list subscriber management: join, leave, digest vs. real-time, held messages
- Mailman-style web interface: subscriber self-service (public routes) + full admin panel (at `/server/{admin_path}/`)
- Email-command interface for subscription management (`SUBSCRIBE`, `UNSUBSCRIBE`, `DIGEST`, `HELP`) with confirmation token flow
- Moderation: post approval queue (web + email-based approve/reject), moderator promotion/demotion, rejection with reason; per-origin trust bypass — known subscribers posting via authenticated NNTP or from a whitelisted address skip the hold queue even on moderated groups; unknown/unauthenticated senders always held; emergency moderation flag forces all posts to queue regardless of trust
- Cross-posting: article appears in all target groups, duplicate delivery suppressed per-subscriber
- Threading: `References` and `In-Reply-To` headers honoured and propagated from both inbound paths
- NNTP operating mode: configurable at server level — `reader` (serve NNTP clients only, no peer feeds), `feeder` (peer feed exchange only, no direct client reading), `both` (default, all connections); controlled via `nntp.mode` in YAML
- Usenet peering: active feed (IHAVE/TAKETHIS push, RFC 4644 streaming) + passive pull (NEWNEWS-based); peer-initiated group creation requires explicit admin approval (no PGP control message chain)
- Group lifecycle: create/delete/rename via admin UI, CLI, and API; high/low water marks tracked for NNTP `LIST ACTIVE` compliance
- Per-group configuration: public/private, moderated/unmoderated, announcement-only, peering on/off, reply-to rewriting, message header/footer, retention days, posting policy
- Article expiry: per-group retention rules; scheduled expiry job purges expired articles from DB and FS
- Cancel / Supersedes: handled on inbound from both NNTP and peer feeds; restricted to original sender or group moderator
- Cancel-Lock (RFC 8315): server generates a Cancel-Lock header on all locally-posted articles; cancel and supersedes requests validated against the lock before applying; prevents forged cancels from peers or unauthenticated senders
- `Path:` header management: server's path component added to all locally-posted articles; inbound articles checked for the server's path component before peer-forwarding to prevent feed loops
- `Injection-Info` header: generated on all articles posted via this server — includes injection date, obfuscated posting host, and cancel-lock reference
- Crosspost limit: configurable max number of groups in `Newsgroups:` header; articles exceeding the limit rejected at ingest with a descriptive error
- Article age cutoff: configurable max age for accepting articles from peer feeds; articles older than the cutoff rejected silently; prevents stale-article floods from misbehaving peers
- Feed filtering hooks: pre-ingest pipe hook (external process) for spam/virus scanning; replaces INN filter_innd with a clean modern interface
- Tag-based topic filtering: admin defines tags per group; subscribers opt in to specific tags via preferences UI — delivery filtered to matching articles only; replaces Mailman topics with a simpler checkbox model
- Subscriber delivery modes: `realtime`, `digest`, `nomail` (stay subscribed, receive nothing — reads via NNTP), `announcement` (receive only moderator/admin posts)
- Announcement-only list type: per-group flag; non-admin posts auto-rejected with configurable response
- Per-list message header/footer: plain text appended/prepended to every outbound email per group; configurable in admin UI
- DMARC mitigation: detect strict DMARC domains on inbound `From:`; munge to `display name via list@domain` format to prevent rejection
- Ban list: block specific addresses at group or server level; banned sender posts silently discarded or bounced (configurable)
- Double opt-in: confirmation token emailed on subscribe; address not activated until confirmed
- Mass subscribe/unsubscribe: admin bulk operation via CSV upload or paste in admin UI
- Invitation flow: admin invites an address; invitee receives confirmation link; no action until accepted
- Built-in SMTP engine: inbound listener (configurable port, high-port default) + outbound delivery with optional smarthost relay; DKIM signing per domain; VERP bounce tracking; bounce classification — hard bounce (5xx permanent) triggers immediate unsubscribe; soft bounce (4xx temporary) increments a per-address counter, unsubscribe after N consecutive soft bounces (configurable); DSN messages (RFC 3461/3464) parsed for bounce classification
- Outbound smarthost relay: optional; when configured, all outbound list email routes through it; configured via YAML or standard `SMTP_*` env vars (see PART 5); when not configured, gospool delivers directly via MX lookup; account/admin notification emails (password reset, welcome, etc.) follow PART 18 SMTP rules — disabled and hidden if no working outbound path
- DKIM key management: keys stored in DB encrypted at rest (envelope encryption, same pattern as peer credentials); managed via admin UI (upload existing PEM, generate new pair, activate/deactivate per domain); config accepts optional filesystem path as one-time import on first startup
- Digest delivery: MIME `multipart/digest` (RFC 2046); per-group configurable window (daily default, weekly option, threshold-based trigger)
- Inbound email addressed via `list+groupname@domain` subaddressing (single MX record)
- Per-group Atom/RSS feed for public groups (read-only, no auth)
- Public read-only web archive for public groups (Pipermail-style, no auth)
- Private group archive: members-only web archive for private groups; requires login; non-members see the group info page but not article content
- Per-group info page: public landing page for each group showing description, posting address, subscribe/unsubscribe form, link to archive, moderator contact, and recent activity count; serves as the canonical URL for the group
- Prometheus metrics endpoint: peers connected, articles/sec in/out, delivery queue depth, bounce rate
- Companion CLI binary (`gospool-cli`): group management, subscriber management, moderation queue, peer config, article inspection
- Built-in scheduler for: digest delivery, article expiry, email retry queue, peer feed retry, TLS cert renewal, GeoIP update, backup
- Full NNTP `LIST` command variants: `LIST ACTIVE [wildmat]`, `LIST ACTIVE.TIMES [wildmat]`, `LIST NEWSGROUPS [wildmat]`, `LIST OVERVIEW.FMT`, `LIST DISTRIBUTIONS`, `LIST DISTRIB.PATS`, `LIST MODERATORS`, `LIST SUBSCRIPTIONS` — all INN-standard variants with wildmat pattern support
- NNTP `DATE` command (RFC 3977 §7.1): returns current server time in UTC for client clock sync
- NNTP `AUTHINFO USER/PASS` (RFC 4643 legacy): supported in addition to SASL; TLS required before accepting plaintext credentials; maps to the same credential store as SASL
- Per-peer flood control: configurable inbound article rate limit per peer connection (articles/minute + bytes/minute); excess triggers a temporary hold and peer notification; protects against runaway feeders without dropping the connection
- SASL auth on NNTP listener: `PLAIN` (TLS-only) + `SCRAM-SHA-256`; see PART 11
- Built-in spam engine: Bayesian classifier trained implicitly by moderator approve/reject actions; SPF/DKIM/DMARC inbound scoring; RBL DNS blacklist checks (default: Spamhaus ZEN + SORBS, configurable); configurable header/body scoring rules; per-group score thresholds — below = deliver, above = hold, high = reject
- Built-in AV: hash-based detection against ClamAV `.hdb` signature databases; scheduler refreshes databases daily (same pattern as GeoIP); catches all known malware hashes on attachments without a daemon
- Milter hook interface for operators who want full rspamd/ClamAV daemon integration instead of or in addition to built-in engines
- RFC 2369 mailing list headers on all outbound email: `List-Id`, `List-Unsubscribe`, `List-Subscribe`, `List-Post`, `List-Archive`, `List-Help`, `List-Owner` — populated per-group; `List-Unsubscribe-Post` (RFC 8058 one-click unsubscribe) included when HTTPS public URL is configured
- Welcome email: configurable per-group plain-text message sent to new subscribers on confirmed join; template supports `{display_name}`, `{group_name}`, `{list_address}`, `{unsubscribe_url}`
- Goodbye email: configurable per-group plain-text message sent on unsubscribe; same template vars
- Autoresponse: per-group auto-reply to any posting address that is not a subscriber; configurable plain-text body, subject prefix, and enable/disable toggle; suppressed if `Precedence: bulk/list/junk` to prevent bounce loops
- Per-list message size limit: configurable maximum bytes per message (headers + body), separate from global SMTP limit; oversized messages auto-rejected with notification to sender
- Per-list MIME type policy: configurable allowed/blocked MIME content types per group; e.g. block `application/*` on announcement lists, allow `text/plain` only; rejected messages bounced with reason
- Moderator hold notification: when a message is held for moderation, an email is sent to all group moderators with a summary (From, Subject, date, short excerpt) and direct approve/reject links — no login required for one-click moderation via signed tokens
- Rejection notification to sender: when a held post is rejected by a moderator, the original sender receives an email with the rejection reason; configurable per group (enabled by default)
- Subscription request comment: when joining a moderated-membership group, subscriber can include a plain-text reason; moderator sees the comment when approving or rejecting the membership request
- Non-member posting policy: per-group, four options — `hold` (send to moderation queue), `reject` (bounce with reason to sender), `discard` (silent drop, no notification), `accept` (non-members may post freely without moderation); default is `hold`
- Periodic membership reminder: configurable scheduled email to all subscribers listing their active subscriptions with unsubscribe instructions; default interval monthly; can be disabled per group or server-wide
- Password reset flow: "Forgot password" link on login page; token emailed to registered address; token valid for 1 hour; entire flow disabled and link hidden when no outbound email path is configured — follows PART 18 SMTP rules
- Email address change: authenticated user can request a new email address; confirmation token sent to the new address; change does not take effect until confirmed; old address notified as a security alert
- Group moderator web interface: focused web queue view for group moderators — shows held articles with approve/reject controls, sender history, and spam score; no access to server admin or org management; accessible via the same web server under `/groups/{name}/queue/`; also reachable via signed token in moderator hold notification emails
- Webhooks: admin-configurable HTTP callbacks for server events; per-event subscription — new article posted, article held for moderation, article approved/rejected, subscriber joined/left, peer connection state change, TLS cert renewed/expiring; payload is a signed JSON envelope; delivery retried on failure with exponential backoff; webhook log viewable in admin UI
- Audit log: append-only log of admin and moderation actions — group create/delete/config change, subscriber add/remove/suspend, moderator assignment change, peer add/remove, user account change, server config change; stored in DB; queryable by actor, resource type, and date range via admin UI and API; never purged automatically
- Admin email alerts: server admin receives email notifications for operational events — moderation queue depth exceeds threshold, per-group bounce rate exceeds threshold, TLS cert expiring within N days, peer offline for longer than threshold, disk space below threshold; follows PART 18 template system; disabled when no outbound email path is configured
- Graceful shutdown: on SIGTERM, server stops accepting new connections, drains the in-flight SMTP/NNTP handler pool, flushes the outbound email retry queue to DB, and then exits cleanly; scheduler jobs are checkpointed so they resume on next startup without data loss; configurable drain timeout (default 30 s) after which the process exits forcibly
- Multi-user support (PART 34): subscriber accounts with login, delivery prefs, token-based API auth; registration mode configurable (open / invite / admin_only / disabled); default `open`
- Organizations (PART 35): an organization owns a set of newsgroups and has a shared moderator team; operators group related lists under a named org for delegation and billing separation
- Custom domains (PART 36): each org or the server itself can serve web/email under a custom domain; ACME cert provisioned per domain; `List-Id` and email headers use the custom domain

**Non-goals (v1):**
- Web-based newsreader UI (NNTP clients and standard email clients are the UI)
- Full Usenet hierarchy propagation at INN/Diablo scale (peering yes; backbone carrier no)
- IMAP or POP3 access
- OAuth / SSO identity providers (local accounts only in v1)
- `gospool-cli` as a newsreader/NNTP client (group/admin management only)

### RFC compliance

gospool implements the following RFC-defined protocols. Full compliance is mandatory — see PART 1.

**NNTP**
- RFC 3977 — Network News Transfer Protocol (core command set)
- RFC 4642 — Using TLS with NNTP (STARTTLS + NNTPS)
- RFC 4643 — NNTP Authentication (AUTHINFO)
- RFC 4644 — NNTP Extension for Streaming Feeds (CHECK / TAKETHIS)
- RFC 8315 — Cancel-Locks for Usenet Articles

**SMTP / email**
- RFC 5321 — Simple Mail Transfer Protocol
- RFC 5322 — Internet Message Format
- RFC 6531 — SMTP Extension for Internationalized Email
- RFC 4422 — SASL (Simple Authentication and Security Layer)
- RFC 5802 — SCRAM-SHA-256 (SASL mechanism)
- RFC 6376 — DKIM Signatures
- RFC 7208 — SPF (Sender Policy Framework)
- RFC 7489 — DMARC
- RFC 3461 — SMTP Service Extension for Delivery Status Notifications
- RFC 3462 — The Multipart/Report Content Type for Message Processing
- RFC 3464 — An Extensible Message Format for Delivery Status Notifications (DSN)

**MIME**
- RFC 2045 — MIME Part One: Format of Internet Message Bodies
- RFC 2046 — MIME Part Two: Media Types (including `multipart/digest` for digests)
- RFC 2047 — MIME Part Three: Message Header Extensions
- RFC 2049 — MIME Part Five: Conformance Criteria

**Mailing list**
- RFC 2369 — List-* header fields (`List-Id`, `List-Unsubscribe`, `List-Post`, etc.)
- RFC 2919 — `List-Id` header
- RFC 8058 — One-click `List-Unsubscribe`

**TLS / PKI**
- RFC 8446 — TLS 1.3
- RFC 8555 — ACME (Automatic Certificate Management Environment) for TLS cert renewal

**Other**
- RFC 4155 — The `application/mbox` Media Type (mbox archive format, used by migration tool)
- RFC 4287 — Atom Syndication Format (per-group Atom feeds)
- RFC 9116 — `security.txt` (see PART 15)

### Configuration areas

YAML config file (see PART 5). Configurable areas:

- **Server**: listen address, ports, TLS cert paths, admin path, privilege-drop user (default `gospool`) — see PART 24
- **NNTP**: plain/TLS ports, auth requirements, max connections, operating mode (reader / feeder / both)
- **SMTP**: inbound port, outbound smarthost relay, DKIM key import path, VERP bounce tracking
- **Peering**: per-peer hostname, direction, auth, group filter, inbound rate limits
- **Storage**: database path, article store path, per-group retention defaults — see PART 10
- **Scheduler**: digest delivery window, retry intervals, expiry scan frequency — see PART 19
- **Web**: public base URL, admin path, session secret — see PART 16/17
- **Logging**: level, format, output path — see PART 11
- **Multi-user**: enabled flag, registration mode — see PART 34
- **Organizations**: enabled flag — see PART 35
- **Custom domains**: enabled flag, per-domain TLS — see PART 36
- **Spam engine**: RBL server list, per-group score thresholds, enable flags
- **AV engine**: signature database path, update URL, enable flag
- **Env var overrides**: all runtime config values can be overridden via environment variables without editing YAML — see PART 5 for the full runtime and init-only variable table; `SMTP_*` vars (HOST, PORT, USERNAME, PASSWORD, FROM_NAME, FROM_EMAIL, TLS) override the outbound relay settings

### Roles & permissions

| Role | Scope | Description |
|------|-------|-------------|
| **Server admin** | Global | Full control — manage groups, users, orgs, peering, server config, TLS certs |
| **Org owner** | Org | Full control of their org's groups, members, and moderators; cannot change server config |
| **Org admin** | Org | Manage org's groups and members; cannot add/remove org owners |
| **Group moderator** | Group | Approve/reject held articles for assigned groups; accessible via email one-click or web queue |
| **Subscriber** | Group | Read via NNTP or email; post to subscribed or open-posting groups; manage own delivery prefs |
| **Peer server** | Global | Machine identity for NNTP feed exchange; limited to feed-in / feed-out operations |
| **Unauthenticated** | Public groups | Read-only NNTP access to public groups (if operator enables it) |

### Data model & sensitivity

**Article** *(metadata in SQLite; body bytes on filesystem)*
- `message_id` — globally unique; `<local-part@domain>` RFC 5322 format; immutable; used as shard key for body file path
- `newsgroups[]` — one or more groups this article belongs to
- `from`, `subject`, `date`, `references[]`, `in_reply_to`
- `body_path` — path to raw MIME body file on filesystem (hash-sharded, e.g. `store/ab/cd/abcd…`)
- `headers` — full original header block preserved verbatim in DB
- `origin` — `nntp` | `email` | `peer`
- `state` — `active` | `pending_moderation` | `rejected` | `cancelled`
- `expires_at` — optional expiry date from `Expires:` header or group default

**Newsgroup**
- `name` — dotted hierarchy name (`comp.lang.go`)
- `description`, `created_at`
- `org_id` — optional FK to owning Organization; null = server-global group
- `moderated` — bool; if true, unapproved posts held for moderator
- `posting_policy` — `open` | `subscribers_only` | `moderated`
- `reply_to` — override `Reply-To` header on outbound email (list address or original)
- `peering_enabled` — bool
- `retention_days` — article retention window; 0 = indefinite
- `max_message_bytes` — per-group size limit; 0 = use global default
- `allowed_mime_types[]` — empty = allow all; non-empty = allowlist
- `welcome_template`, `goodbye_template`, `autoresponse_template` — per-group email text; null = disabled
- `announcement_only` — bool; non-moderator posts auto-rejected
- `header_prefix`, `footer_text` — prepended/appended to outbound email body

**Subscriber** *(maps to `users` table — see PART 34)*
- `email`, `display_name`
- `username` — login identity for web interface
- `groups[]` — per-group: delivery mode (`realtime` | `digest` | `nomail` | `announcement`), posting allowed, held (awaiting approval)
- `password_hash` — hashed; used for web interface login (see PART 11)
- `api_token_hash` — for `gospool-cli` and API auth; hashed at rest (see PART 11)
- `created_at`, `last_active_at`
- `suspended` — bool; suspended users cannot post or log in

**Organization** *(PART 35)*
- `name` — URL-safe slug; unique server-wide
- `display_name`, `description`
- `custom_domain` — optional; null = use server domain (PART 36)
- `created_at`, `created_by`
- Members: join table `org_members(org_id, user_id, role)` — role: `owner` | `admin` | `moderator`

**Moderator assignment**
- `subscriber_id`, `group_name`, `granted_by`, `granted_at`

**PeerServer**
- `hostname`, `port`, `tls` — bool
- `direction` — `in` | `out` | `both`
- `auth_user`, `auth_pass_hash`
- `groups[]` — group filter for this peer; empty = all groups

**Sensitivity:**
- Subscriber email addresses are PII; never logged in plaintext, never exposed via NNTP `LIST` or article headers beyond what the original message contained
- All password and credential hash fields: never stored or logged as plaintext; see PART 11
- Peer credentials: stored encrypted at rest; see PART 11

### Trust boundaries & external services

| Integration | Trust level | Failure mode |
|-------------|-------------|--------------|
| Inbound SMTP (email → article) | Untrusted — all inbound email is adversarial input | Reject message; bounce with 5xx; never crash server |
| Inbound NNTP POST | Untrusted unless authenticated | Reject with 480/502; article not stored |
| Outbound SMTP (article → subscriber email) | Trusted delivery channel; remote MTA is untrusted relay | Retry queue with exponential backoff; dead-letter after N attempts |
| Outbound NNTP feed to peer | Trusted by mutual pre-configuration; peer MTA is untrusted | Feed stalls; backlog retained; retry on reconnect |
| Inbound NNTP feed from peer | Trusted to the extent of its credential; content still untrusted | Malformed articles rejected; peer not allowed to inject admin commands |
| ACME / Let's Encrypt (TLS cert renewal) | Trusted CA infrastructure | Fall back to existing cert until renewal succeeds; alert admin |
| Local filesystem (article store, DB) | Trusted | Startup failure with clear error if paths missing or permissions wrong |

### Threat model & abuse cases

**Primary assets:**
- Article store integrity — canonical, tamper-evident message history
- Subscriber PII — email addresses, delivery preferences
- Server availability — NNTP and SMTP listening surfaces
- Peer trust — credentials and feed relationships

**Trusted vs. untrusted inputs:**

| Input | Trust |
|-------|-------|
| Local admin API / web interface (authenticated admin) | Trusted |
| NNTP POST from authenticated subscriber | Semi-trusted (content untrusted; identity verified) |
| NNTP POST from unauthenticated client | Untrusted |
| Inbound SMTP from any sender | Untrusted |
| Inbound NNTP feed from credentialed peer | Semi-trusted (content untrusted; feed auth verified) |
| Web subscription management (authenticated subscriber) | Semi-trusted |
| Email management commands (SUBSCRIBE / UNSUBSCRIBE in email body) | Untrusted — must verify against known subscriber addresses |

**Abuse cases and required defenses:**

| Threat | Defense |
|--------|---------|
| Article spam via NNTP POST | Require authentication for posting; rate-limit per-credential; moderation queue for flagged content |
| Article spam via inbound email | SPF/DKIM/DMARC validation on inbound; reject unsigned or failing messages; moderation queue |
| Email address harvesting via NNTP `OVER`/`HDR` | Suppress or munge `From:` headers in NNTP responses for non-public groups; operator config option |
| Subscriber list enumeration | List membership endpoints require authentication; no unauthenticated roster dump |
| Mail loop (list → subscriber → list) | Loop detection via `X-Loop` header; `Precedence: list` header; reject messages with own `Message-ID` in `References` |
| Peer feed injection of forged cancels | Cancel messages from peers require matching `Message-ID` and sender; verify against article store before applying |
| Credential stuffing (web login) | Hashed passwords (see PART 11); per-IP login rate limiting; account lockout after N failures |
| SSRF via peer hostname | Peer hostnames resolved at config time with allowlist; no user-supplied URL fetch at runtime |
| Path traversal via `message_id` in filesystem store | Sanitize `message_id` before any filesystem path construction; use hash-based shard directory layout |
| Malicious MIME attachments | Store verbatim; no server-side execution or rendering; size limits enforced |
| Article expiry / cancel weaponisation | Cancel and supersede operations restricted to original sender or group moderator |

### Security decisions & exceptions

- **Unauthenticated read access** is a deliberate feature for public groups (Usenet tradition); operators must explicitly enable it. When enabled, `From:` header visibility is configurable.
- **Inbound SMTP on port 25** requires the binary to bind a privileged port; the recommended deployment is behind a MTA proxy (Postfix/Exim relay) that forwards to a high port. Running as root is explicitly a non-goal; document the proxy setup path.
- **Plaintext NNTP (port 119)** is supported for legacy clients and LAN deployments; operators are warned in the startup banner and docs that NNTPS is strongly preferred.
- **Email-command subscription management** (e.g. `SUBSCRIBE` in email body) is intentionally supported for compatibility with classic list workflows; verification is done against the `From:` address in the subscriber database, with a confirmation token sent back — no action taken on the first email alone.
- **Built-in Bayesian spam engine** is trained implicitly by moderator actions (approve = ham, reject = spam); operators who want a heavier-weight classifier can replace or supplement it via the milter hook without changing this binary.

### NNTP operating modes

gospool can operate as a reader, a feeder, or both (configurable):

| Mode | Accepts | Use case |
|------|---------|----------|
| `both` (default) | NNTP reader clients + peer feeds | General-purpose server |
| `reader` | NNTP reader clients only | Public-facing leaf node; no peer feeds |
| `feeder` | Peer feeds only | Backbone/relay node; no direct reader clients |

Connections for the wrong mode are refused at the protocol layer.

### Organizations (PART 35)

An **organization** is an administrative boundary that owns a set of newsgroups and has a shared moderator team. Designed for:
- A company hosting internal mailing lists under a named org (`acme-internal`)
- An open-source project managing its discussion lists (`myproject`)
- A community or club grouping related groups

**What an org provides:**
- Named namespace: all groups under an org are logically grouped in the admin UI and `gospool-cli`
- Delegated administration: org owners/admins can manage their groups without touching server-global config
- Shared moderator team: moderators assigned to an org automatically have access to all org-owned groups (in addition to per-group moderator assignments)
- Custom domain (PART 36): org can optionally serve its web interface and email under a dedicated domain
- API token scoping: a `gospool-cli` token can be scoped to a specific org for automation

**Org creation flow:**
1. Server admin creates an org (name, display name, optional description)
2. Server admin assigns one or more users as org owners
3. Org owners can manage their own groups, members, and moderator assignments from that point on
4. Server admin retains override access to all orgs

**Group ownership:**
- Groups with `org_id` set are "org groups" — managed by org owners/admins
- Groups with `org_id = null` are "server groups" — managed only by server admin
- A group cannot belong to more than one org

### Multi-user & registration (PART 34)

**Multi-user is enabled for gospool.** Subscribers are full user accounts with:
- Login via web interface and `gospool-cli`
- Delivery preference management per group
- API token for CLI/API auth (scoped to the user's own groups or org membership)
- Optional display name and timezone for web interface

**Registration mode** (configurable; see PART 34):

| Mode | Behavior |
|------|----------|
| `open` (default) | Anyone can self-register; email verification required before account is active |
| `invite` | Only admin-issued one-time invite links can create accounts |
| `admin_only` | Server admin creates accounts directly; activation link emailed to new user |
| `disabled` | No new accounts; existing users can still log in |

Self-hosted deployments default to `open`.

### Migration tool (`gospool import`)

A `gospool import` subcommand (part of the `gospool` server binary) handles one-time data import from INN and Mailman. All commands are idempotent — re-running skips already-imported `Message-ID`s and member addresses.

**All import commands share:** dry-run mode (report what would change, touch nothing), progress output, skip-errors mode (log bad records and continue), and resume safety (Message-ID and member address deduplication means re-runs are safe).

#### `gospool import inn`

Imports an INN spool:
1. Read INN `active` file → create newsgroups with high/low water marks
2. Read INN `newsgroups` file → set group descriptions
3. Walk tradSpool directory → parse each article, store body and metadata
4. Deduplicate by `Message-ID` — already-present articles skipped
5. Optionally import peer definitions from INN peer config files

Accepts: spool directory, active file, newsgroups file, optional peer config file, optional wildmat group filter.

#### `gospool import mailman2`

Imports a Mailman 2.x installation:
1. Parse each list's `config.pck` → create newsgroup preserving all settings (moderated, reply-to, footer, size limit, MIME policy, welcome/goodbye templates, announcement-only flag)
2. Import member database → create subscriber records with delivery mode preserved
3. Import mbox archive → inject articles with email→article conversion and Message-ID dedup

Accepts: Mailman lists directory, optional single-list filter, option to skip archive import.

#### `gospool import mailman3`

Imports a Mailman 3.x installation:
1. Connect to Mailman 3 database (SQLite or PostgreSQL) → read list config and member records
2. Create newsgroups and subscribers as with mailman2
3. Import mbox archive export (user exports from HyperKitty separately)

Accepts: database DSN, optional mbox file, optional single-list filter.

#### `gospool import mbox`

Generic RFC 4155 mbox import into a named group. Accepts: mbox file path, target group name, option to create the group if it does not exist.

### gospool-cli capabilities

`gospool-cli` is the companion binary (PART 33). Follows all PART 8 and PART 33 rules: binary naming, NO_COLOR, token auth, user/org context (`--user`), flag-to-config save, and token revocation handling. TUI mode auto-detected when run with no subcommand in an interactive terminal.

**Group management:** list groups, show group detail, create/delete groups, update group config; all filterable by org

**Subscriber management:** list, add, remove, and invite subscribers per group; bulk import from CSV; set delivery mode per subscriber

**Moderation:** view held-message queue, approve or reject articles with optional reason, hold or unhold a sender

**Peer management:** list, add, remove, and show status of NNTP peer connections

**Article inspection:** show article by Message-ID, cancel an article, search articles by group/sender/subject/date

**Import / migration:** import from INN spool, Mailman 2.x, Mailman 3.x, or generic mbox — all with dry-run and progress options; see Migration tool section above

**Organization management (PART 35):** list, create, and delete orgs; manage org member roles (owner / admin / moderator)

**User management (PART 34):** list users, invite by email, suspend/unsuspend accounts, list and revoke API tokens

**Webhooks:** list configured webhook endpoints, add/remove/enable/disable webhooks, view recent delivery log

**Audit log:** query the audit log by actor, resource type, date range; export to JSON or CSV

**Server / admin:** show server status and version, trigger config reload, log in / log out (stores token per PART 33)

### Email templates

gospool uses the PART 18 standard template system. Required standard templates (see PART 18): `welcome`, `password_reset`, `email_verify`, `login_alert`, `security_alert`, `mfa_reminder`, `2fa_enabled`, `2fa_disabled`, `password_changed`, `backup_complete`, `backup_failed`, `ssl_expiring`, `ssl_renewed`, `scheduler_error`, `breach_notification`, `test`.

gospool-specific additional templates (beyond the PART 18 standard set):

| Template | Trigger |
|----------|---------|
| `list_welcome` | Subscriber confirmed join to a group; uses per-group welcome text if configured, else server default |
| `list_goodbye` | Subscriber leaves a group (unsubscribe or admin removal); uses per-group goodbye text if configured |
| `moderation_hold` | Article held for moderation; sent to all group moderators with summary and approve/reject links |
| `moderation_rejection` | Held article rejected by moderator; sent to original poster with rejection reason |
| `subscription_request` | Membership request submitted to a moderated-membership group; sent to moderators for approval |
| `subscription_request_confirm` | Confirmation to the requester that their membership request was received and is pending review |
| `membership_reminder` | Periodic reminder to subscriber listing their active group subscriptions with unsubscribe instructions |
| `admin_queue_alert` | Moderation queue depth or bounce rate exceeds admin-configured threshold |
| `admin_peer_alert` | Peer server offline or TLS cert expiring; sent to server admin |
| `admin_disk_alert` | Disk space below configured threshold; sent to server admin |

All templates: plain-text + HTML variants; per-group override for list_welcome and list_goodbye; all others are server-level defaults. When no outbound email path is configured (PART 18 SMTP rules), all templates are disabled and not rendered.
