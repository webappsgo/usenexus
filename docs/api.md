# HTTP API

The current API base is `/api/v1`. The version segment comes from
`server.api_version`. Routes below are public unless the access column says
otherwise. Public read routes do not require a token.

Every HTTP response includes an `X-Request-ID`. A valid UUID v4 supplied in
`X-Request-ID`, `X-Correlation-ID`, or `X-Trace-ID` is propagated; otherwise
the server generates one. Rate-limited requests return HTTP 429, a
`Retry-After` header, and `{"ok":false,"error":"RATE_LIMITED","message":"Too many requests"}`.
API routes do not end with `/`; a trailing slash receives a permanent redirect
to the canonical path (for example, `/api/v1/` redirects to `/api/v1`).

Health JSON reports project/build/runtime metadata, enabled features, cached
component checks, and aggregate request statistics. `healthy`, `degraded`, and
`unhealthy` are the only overall status values. A degraded dataset leaves the
endpoint at HTTP 200; a critical database or disk failure returns HTTP 503.
`/server/healthz` negotiates HTML for browsers, formatted page text for
command-line clients, and the paired API's JSON when requested. The versioned
API health route uses flat `key: value` text. `/healthz` is an optional alias
disabled by default with `server.healthz.root.enabled`.

## Response formats

Versioned API routes return JSON by default. They return plain text when the
request path ends in `.txt`, the request explicitly accepts `text/plain`, or
the User-Agent identifies a non-interactive HTTP client. A `.txt` suffix takes
priority over `Accept: application/json`; otherwise an explicit JSON Accept
header takes priority over HTTP-client detection. Text browsers receive plain
text from API routes. Health text uses flat `key: value` lines; other JSON
success and error responses are rendered as readable text while preserving the
HTTP status.

Frontend routes return HTML by default. `Accept: application/json` delegates
to the corresponding versioned API route, `Accept: text/html` selects HTML,
and `Accept: text/plain` selects formatted text, in that order. Without an
explicit Accept preference, HTTP tools such as curl and wget receive text,
text browsers receive server-rendered HTML with executable scripts removed,
the `gospool-cli` client receives JSON, and browsers receive HTML. Pages under
`/server/{admin_path}` always remain HTML. Responses that vary by client add
`Vary: Accept, User-Agent`.

Examples:

```sh
curl -q -LSsf https://example.test/api/v1/server/healthz
curl -q -LSsf https://example.test/api/v1/server/healthz.txt
curl -q -LSsf -H 'Accept: application/json' https://example.test/server/healthz
curl -q -LSsf -H 'Accept: text/plain' https://example.test/groups
```

Frontend `.txt` paths are not required; smart client detection selects text.
The `.txt` suffix applies to versioned API routes.

| Method | Path | Access | Behavior |
|---|---|---|---|
| `GET` | `/api/v1` | Public groups anonymously; includes private groups for signed-in users or administrators | API name, version, OpenAPI document path, and the structured data rendered on the homepage. |
| `GET` | `/api/v1/server/about` | Public | Structured project description and public feature information. |
| `GET` | `/api/v1/server/help` | Public | Structured help sections for the web archive, API, NNTP, accounts, and administration. |
| `GET` | `/api/v1/server/auth/login` | Public | Login form descriptor for API clients; `POST` on the same path authenticates a regular user. |
| `GET` | `/api/v1/server/auth/register` | Public while registration is open | Registration form descriptor; unavailable when user registration is disabled or private. |
| `GET` | `/api/v1/server/healthz` | Public | Public-safe project, build, runtime, feature, component-check, and aggregate request-statistics health response. Returns 503 only when a critical database or disk check is unhealthy. |
| `GET` | `/api/healthz` | Public | Unversioned alias of the versioned health API. |
| `GET` | `/api/v1/server/privacy` | Public | Deployment privacy settings and disclosures. |
| `GET` | `/api/v1/server/terms` | Public | Terms of service content. |
| `GET` | `/api/v1/groups` | Public groups anonymously; includes private groups for signed-in users or administrators | Lists visible groups. |
| `GET` | `/api/v1/groups/{group}` | Public for public groups; signed-in user or administrator session/bearer token for private groups | Returns group settings and article watermarks. |
| `GET` | `/api/v1/groups/{group}/articles` | Public for public groups; signed-in user or administrator session/bearer token for private groups | Returns up to 100 recent article summaries. |
| `GET`, `POST`, `PATCH`, `DELETE` | `/api/v1/groups/{group}/subscriptions` | Signed-in user; group moderator for roster/removal actions | Reads own membership, requests a double-opt-in public-group subscription, updates confirmed delivery preferences, or unsubscribes/removes a member. Private-group membership is invitation-only. POST returns `202` while confirmation is pending. |
| `GET`, `POST` | `/api/v1/server/{admin_path}/groups/{group}/moderators` | Server administrator session; POST requires CSRF verification | Lists assigned group moderators or grants/revokes a moderator role with `{"username":"name","action":"grant|revoke"}`. Grants require an active, verified user. |
| `GET` | `/api/v1/articles/{messageID}` | Public for public articles; signed-in user or administrator session for private articles | Returns article metadata; the body is not included. |
| `POST` | `/api/v1/groups` | Administrator session + CSRF, or `adm_` bearer token with write access | Creates a group. |
| `PATCH` | `/api/v1/groups/{group}` | Administrator session + CSRF, or `adm_` bearer token with write access | Updates group name, description, visibility, posting policy, peer-exchange setting, and default delivery mode. |
| `DELETE` | `/api/v1/groups/{group}` | Administrator session + CSRF, or `adm_` bearer token with write access | Deletes a group and its subscriptions, queued deliveries, article links, and moderation queue entries. Archived article bodies remain available only through any other groups that reference them. |
| `GET`, `POST` | `/api/v1/peers` | `adm_` Bearer token; POST requires a write-capable token | Lists configured NNTP peers without credentials or adds a peer with encrypted authentication data. |
| `GET`, `PATCH`, `DELETE` | `/api/v1/peers/{peer}` | `adm_` Bearer token; mutations require a write-capable token | Reads, updates, or removes a configured peer. |
| `POST` | `/api/v1/peers/{peer}/test` | `adm_` Bearer token with write scope | Performs a bounded NNTP greeting, optional TLS/authentication, and capabilities handshake. |
| `GET`, `PATCH` | `/api/v1/server/{admin_path}/security/ratelimit` | `adm_` Bearer token; PATCH requires a write-capable token | Reads or updates persisted per-client HTTP rate limits. |
| `GET`, `PUT` | `/api/v1/server/{admin_path}/security/geoip` | Administrator session; PUT requires CSRF | Reads or saves GeoIP country settings and operator-authored presets. |
| `GET` | `/api/v1/server/{admin_path}/scheduler` | Administrator session | Lists persisted scheduler tasks. |
| `GET` | `/api/v1/server/{admin_path}/scheduler/{id}` | Administrator session | Reads one task and its latest 100 recorded runs. |
| `PATCH` | `/api/v1/server/{admin_path}/scheduler/{id}` | Administrator session + CSRF | Updates a task schedule and/or enabled state. Required tasks cannot be disabled. |
| `POST` | `/api/v1/server/{admin_path}/scheduler/{id}/run` | Administrator session + CSRF | Queues an enabled, registered task for immediate execution. |
| `POST` | `/api/v1/server/{admin_path}/scheduler/{id}/enable` | Administrator session + CSRF | Enables a registered task. |
| `POST` | `/api/v1/server/{admin_path}/scheduler/{id}/disable` | Administrator session + CSRF | Disables a task only when its policy permits it. |
| `GET` | `/api/v1/server/{admin_path}/scheduler/{id}/history` | Administrator session | Returns the latest 100 recorded runs for a task. |
| `GET` | `/api/v1/server/swagger` | Public | Returns the current OpenAPI JSON document. |
| `GET`, `POST` | `/api/v1/server/graphql` | Public | Read-only GraphQL endpoint. `GET` accepts `query`, optional `variables` JSON, and optional `operationName`; `POST` accepts a JSON request. |
| `POST` | `/api/v1/server/auth/login` | Public | Authenticates a verified, active regular user and returns a short-lived bearer session. |
| `GET` | `/api/v1/users/tokens` | Regular-user bearer token | Lists the caller’s tokens with secrets redacted. |
| `POST` | `/api/v1/users/tokens` | Regular-user bearer token with `read-write` or `global` scope | Creates a token; the secret is returned once. |
| `DELETE` | `/api/v1/users/tokens/{id}` | Regular-user bearer token with `read-write` or `global` scope | Revokes one of the caller’s tokens. |
| `GET`, `PATCH` | `/api/v1/users/settings` | Regular-user bearer session or token | Reads account, privacy, notification, and appearance settings; partially updates any category. |
| `GET` | `/api/v1/users?q={query}` | Public | Searches public profiles with search visibility enabled; queries require at least two characters. |
| `GET` | `/api/v1/users/{username}` | Public | Returns a public profile; private profiles return 404, and email appears only when the user opts in. |
| `GET` | `/api/v1/server/{admin_path}/email` | Administrator session | Returns outbound SMTP settings with the password redacted. |
| `PATCH` | `/api/v1/server/{admin_path}/email` | Administrator session + CSRF | Saves SMTP and sender settings, then tests the relay handshake. Environment overrides retain precedence. |
| `POST` | `/api/v1/server/{admin_path}/email/test` | Administrator session + CSRF | Sends the built-in test email to the supplied recipient. |
| `GET` | `/api/v1/server/{admin_path}/email/templates` | Administrator session | Lists embedded email templates and whether each has a custom override. |
| `GET` | `/api/v1/server/{admin_path}/email/templates/{name}` | Administrator session | Reads a template and its custom/default status. |
| `PUT` | `/api/v1/server/{admin_path}/email/templates/{name}` | Administrator session + CSRF | Saves a validated custom template. |
| `POST` | `/api/v1/server/{admin_path}/email/templates/{name}/reset` | Administrator session + CSRF | Removes the custom override and restores the embedded default. |
| `POST` | `/api/v1/server/{admin_path}/email/templates/{name}/preview` | Administrator session + CSRF | Renders a template with preview values; accepts optional JSON `{ "content": "..." }`. |
| `GET`, `POST` | `/api/v1/users/domains` | Regular-user session or bearer token | Lists or adds the caller's custom domains. |
| `GET`, `DELETE` | `/api/v1/users/domains/{domain}` | Regular-user session or bearer token | Reads or removes an owned custom domain. |
| `POST` | `/api/v1/users/domains/{domain}/verify` | Regular-user session or bearer token | Checks the `_verify.{domain}` TXT ownership record. |
| `GET` | `/api/v1/users/domains/{domain}/dns` | Regular-user session or bearer token | Returns verification and routing instructions. |
| `GET`, `POST` | `/api/v1/users/domains/{domain}/ssl` | Regular-user session or bearer token | Reads SSL state or issues a certificate; DNS-01 accepts a Cloudflare API token. |
| `POST` | `/api/v1/users/domains/{domain}/ssl/renew` | Regular-user session or bearer token | Forces renewal using the saved challenge configuration. |
| `GET`, `POST` | `/api/v1/orgs/{slug}/domains` | Organization member with profile management access | Lists or adds organization domains. |
| `GET`, `DELETE` | `/api/v1/orgs/{slug}/domains/{domain}` | Organization member with profile management access | Reads or removes an organization domain. |
| `POST` | `/api/v1/orgs/{slug}/domains/{domain}/verify` | Organization member with profile management access | Checks the ownership TXT record. |
| `GET` | `/api/v1/orgs/{slug}/domains/{domain}/dns` | Organization member with profile management access | Returns verification and routing instructions. |
| `GET`, `POST` | `/api/v1/orgs/{slug}/domains/{domain}/ssl` | Organization member with profile management access | Reads SSL state or issues a certificate. |
| `POST` | `/api/v1/orgs/{slug}/domains/{domain}/ssl/renew` | Organization member with profile management access | Forces renewal using saved challenge settings. |
| `GET` | `/api/v1/server/{admin_path}/domains` | Administrator session | Lists custom domains. |
| `GET`, `DELETE` | `/api/v1/server/{admin_path}/domains/{domain}` | Administrator session; mutations also require CSRF | Reads audit history or force-deletes a domain. |
| `POST` | `/api/v1/server/{admin_path}/domains/{domain}/suspend` | Administrator session + CSRF | Suspends a domain with a required reason. |
| `POST` | `/api/v1/server/{admin_path}/domains/{domain}/unsuspend` | Administrator session + CSRF | Re-enables a suspended domain. |
| `POST` | `/api/v1/server/{admin_path}/domains/{domain}/ssl/renew` | Administrator session + CSRF | Forces certificate renewal. |

`GET /api/v1/` redirects to `/api/v1`. Article IDs contain angle brackets;
URL-encode the Message-ID when placing it in the path. Unknown routes return
404. `/api/swagger` is a compatibility alias for the versioned OpenAPI document.

Custom-domain creation returns a verification TXT value and routing targets.
For DNS-01 certificate issuance, submit `challenge: "dns-01"`,
`provider: "cloudflare"`, and `credentials.api_token`; the token is validated
and encrypted at rest. Wildcard domains require DNS-01. See the
[custom-domain guide](custom-domains.md) for DNS and TLS setup.

## Responses

Successful JSON responses use the `{ "ok": true, "data": ... }` envelope.
Group and article collections also include `count`.
An article metadata response uses `{ "ok": true, "data": {...},
"body_available": true }`; `body_available` indicates that an archive body
exists, but the API response does not contain the body. Health returns
`status`, `database`, and `uptime_seconds` directly. JSON API errors use
`ok: false`, a machine-readable `error` code, and a safe human-readable
`message`. The GraphQL endpoint uses the GraphQL response format.

## Internal metrics

Metrics routes are internal monitoring endpoints. Every service requires its
own `Authorization: Bearer …` token from `server.metrics.auth.tokens`; query
string credentials are not accepted. The root `/metrics` alias can be disabled
with `server.metrics.root.enabled: false`.

| Service | Endpoint | Response |
| --- | --- | --- |
| Prometheus | `/server/metrics` or `/server/metrics/prometheus` | Prometheus exposition text |
| Grafana | `/server/metrics/grafana` | Importable Grafana dashboard V2 resource JSON |
| Loki | `/server/metrics/loki` | Bounded recent log entries in Loki stream JSON |

The same services are available under `/api/{api_version}/server/metrics`,
`/api/metrics`, and, when enabled, `/metrics`. Use the matching service token
for every alias.

Prometheus exports HTTP request counts, latency and body sizes; SQL query
counts, duration, errors, and SQLite busy errors; authentication outcomes and
active sessions; custom-domain cache activity; scheduler runs; host and Go
runtime resources; and public group, article, subscription, user, API-token,
and outbound-delivery totals. NNTP peer metrics include current authenticated
peer connections, inbound/outbound article counters, and queued outbound peer
deliveries. System and runtime families follow
`server.metrics.include_system` and `server.metrics.include_runtime`.

Group creation accepts JSON like:

```json
{
  "name": "comp.example",
  "description": "Example group",
  "visibility": "public",
  "posting_policy": "open",
  "peering_enabled": true,
  "default_delivery_mode": "realtime"
}
```

The server validates group names, description length, visibility (`public` or
`private`), posting policy (`open`, `subscribers_only`, or `moderated`), and
default delivery mode (`realtime`, `digest`, `nomail`, or `announcement`).
Browser requests use the administrator's `admin_session` cookie and a CSRF
token in the `X-CSRF-Token` header. API clients may use an `adm_` bearer token;
`read` tokens cannot create, update, or delete groups. The configured operator
token also has administrator access.

Group updates accept any non-empty subset of `name`, `description`,
`visibility`, `posting_policy`, `peering_enabled`, and
`default_delivery_mode`. Supplying a new `name` renames the group while
preserving its article and subscription relationships.
The peer-exchange setting controls inbound peer ingestion and outbound queueing.
For example:

```json
{
  "visibility": "private",
  "posting_policy": "subscribers_only",
  "peering_enabled": false
}
```

## OpenAPI and GraphQL

The OpenAPI JSON document is at `/api/v1/server/swagger`; the interactive
Swagger UI is at `/server/docs/swagger`. The browser GraphQL page is
`/server/docs/graphql` and submits queries to `/api/v1/server/graphql`. Its
native GET form works without JavaScript; JavaScript enhances it with a POST
request and an inline result view.

The GraphQL schema currently exposes only public `groups` and
`articles(group: String!)` queries. For example:

```graphql
{
  groups { name description articleHigh articleLow }
  articles(group: "comp.example") { messageId subject from date number }
}
```

Private group detail and article-list pages require an active user or administrator session; anonymous requests receive 404. Regular-user registration and account settings use server-rendered pages at
`/server/auth/register` and `/users/*`. Account, privacy, notification, and
appearance forms at `/users/settings`, `/users/settings/privacy`,
`/users/settings/notifications`, and `/users/settings/appearance` work without
JavaScript and require a valid user session plus CSRF token. The settings API
returns those categories under `data` and accepts partial category objects.
Public profiles are server-rendered at `/users/{username}` and use the same
visibility and email opt-in rules as the public profile API.
The server-rendered settings page at
`/server/{admin_path}/email` configures outbound SMTP. When the relay handshake
succeeds, verification and password-reset links are delivered by email and
stored as single-use hashed tokens; password-reset links expire after one hour
by default, configurable through `auth.password_reset_ttl`. Without working
SMTP, email is disabled and registration skips verification. Double-opt-in
subscription requests and server-rendered management are available at
`/groups/{group}`, `/server/auth/subscriptions/confirm/{token}`, and
`/users/subscriptions`. Confirmed `realtime` subscriptions use a durable queue
with retry scheduling after send failures. `digest` subscriptions are delivered
as `multipart/digest` emails with retry scheduling. Groups default to daily
delivery and can select rolling weekly or threshold-triggered delivery on the
group administration page. A threshold digest is sent when its article-count
or human-readable size threshold is reached. `nomail` and `announcement`
preferences can be saved. `nomail` keeps the subscription active without
outgoing email; `announcement` currently does not filter delivery to moderator
and administrator posts.
Email-command management, email-based moderation, imports, webhooks, and some
routes from the full administrator API inventory in `AI.md` are not
implemented yet. Moderated articles are held for web review at
`/groups/{group}/queue`; moderator and server administrator forms approve or
reject them with CSRF protection. Authenticated inbound peer feeds, NEWNEWS-based passive pull,
and queued outbound peer feeds are available. Inbound `newgroup`, `rmgroup`,
and `checkgroups` control messages are held for explicit administrator review
at `/server/{admin_path}/peer-controls`; approval is audited before group
changes are applied. Peer article/byte
limits are configured with `nntp.peer_articles_per_minute` and
`nntp.peer_bytes_per_minute`.
Peer configuration is available at
`/server/{admin_path}/peers` and through the peer API above.
