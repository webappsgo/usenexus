# SPEC.md — gospool rule overrides

SPEC.md wins over AI.md which wins over global CLAUDE.md.
Only genuine contradictions with AI.md live here. Additions that do not
contradict the template belong in IDEA.md.

---

## Multi-listener architecture

AI.md models a single HTTP listener on port 80. gospool runs three concurrent
protocol listeners that all start and stop together as one binary:

| Listener | Default port | Controlled by |
|----------|-------------|---------------|
| HTTP (web / API / admin) | 80 / 443 | `--port` / `PORT` env var |
| NNTP plain | 119 | `--nntp-port` / `NNTP_PORT` env var |
| NNTPS | 563 | `--nntps-port` / `NNTPS_PORT` env var |
| SMTP inbound (optional) | 2525 (high-port default) | `--smtp-port` / `SMTP_IN_PORT` env var |

The server does not exit when a non-HTTP listener fails to bind — it logs
the error, disables that protocol path, and continues. The HTTP listener
failure is fatal (same as AI.md).

NNTP operating mode (`reader` / `feeder` / `both`) controls which protocol
listeners activate, but the HTTP management listener (admin UI, `/server/healthz`,
`/metrics`) is **always** bound regardless of operating mode. A `feeder`-only
deployment still exposes the HTTP management port; the web group interface and
subscriber-facing routes simply return 404.

## Container port model

AI.md mandates a single internal port 80 with one Docker port mapping. gospool
overrides this: all four listener ports must be declared in the Dockerfile with
`EXPOSE` and mapped in `docker-compose.yml`. The `PORT` / `ADDRESS` env-var
convention from AI.md applies only to the HTTP listener; the remaining protocols
use their own env vars listed above.

**Interface binding and port mapping differ by listener type.** AI.md PART 27
only models port 80 mapped as `172.17.0.1:{randomport}:80` so it sits behind a
reverse proxy. That model does not apply to protocol ports:

- The `172.17.0.1:` bridge-only prefix breaks direct client connections —
  NNTP and SMTP clients connect to the server without a proxy.
- The `{randomport}` host-side mapping breaks protocol clients that expect a
  fixed well-known port — newsreaders hard-code 119/563; MTA relays hard-code 25.

Protocol ports use their fixed standard port on both sides, bound to all
interfaces:

| Listener | docker-compose mapping | Reason |
|----------|----------------------|--------|
| HTTP | `172.17.0.1:{randomport}:80` | Behind reverse proxy — bridge only (AI.md default) |
| NNTP plain | `119:119` | Fixed well-known port, direct client access |
| NNTPS | `563:563` | Fixed well-known port, direct client access |
| SMTP inbound | `25:25` (see fallback below) | Fixed well-known port, direct MTA access |

The Dockerfile must `EXPOSE 80 119 563 25 2525` (2525 for the SMTP fallback
port so compose and tooling can see it even when the container binds the
fallback).

## SMTP inbound port fallback

On startup, if the configured inbound SMTP port is already in use, gospool tries
the following sequence before giving up:

```
configured port (default 25) → 2525 → random OS-assigned port
```

The resolved port is logged at INFO level and written to the admin dashboard.
If all three attempts fail the inbound SMTP listener is disabled (non-fatal),
the same way a failed NNTP bind is handled. Operators can lock the port with
`SMTP_IN_PORT` to suppress fallback when an exact port is required.

## SMTP inbound / outbound isolation

AI.md PART 18 covers SMTP as an outbound notification channel. gospool adds a
full inbound SMTP listener for email-to-NNTP injection. These two SMTP paths
are independent and must never interfere:

- PART 18 outbound auto-detection (probing loopback ports 25, 465, 587) MUST
  exclude the port gospool itself is bound to as the inbound listener; detecting
  its own inbound port as the outbound relay would cause a mail loop.
- The inbound SMTP listener uses a separate handler stack and is never fed
  through the PART 18 outbound retry queue.
- `SMTP_*` env vars (HOST, PORT, USERNAME, PASSWORD, FROM_NAME, FROM_EMAIL, TLS)
  control outbound relay only, exactly as PART 18 specifies; they have no effect
  on the inbound listener.

## TLS cert reload scope

AI.md's `ssl_renewal` scheduler job renews the cert and hot-reloads it into the
HTTP/HTTPS listener. gospool extends this: after a successful renewal the renewed
cert must also be hot-reloaded into the NNTPS listener without dropping existing
NNTP connections. New TLS handshakes use the new cert; in-progress sessions
continue with the old cert until they close naturally.

## Graceful shutdown drain scope

AI.md's shutdown sequence step 4 ("Wait for in-flight requests") is HTTP-only.
gospool extends this step to drain all three protocol handler pools concurrently
under the same 30 s timeout:

- HTTP in-flight requests (as per AI.md)
- NNTP command handlers — finish the current command; do not start a new one
- SMTP inbound sessions — finish any DATA phase in progress; reject new MAIL FROM

Peer feed connections (active IHAVE/TAKETHIS streams) are included in the NNTP
drain. The outbound email retry queue is flushed to the database before the
database connection closes (step 5), matching IDEA.md's graceful-shutdown
guarantee.

## NNTP listener configuration

AI.md has no concept of an NNTP listener. The following config areas are
gospool-specific additions to the PART 5 / PART 12 config model:

| Area | Description |
|------|-------------|
| Listen address | Address for plain NNTP listener; default `0.0.0.0` |
| Plain port | Default 119; `NNTP_PORT` env var overrides |
| TLS port | Default 563 for NNTPS; `NNTPS_PORT` env var overrides |
| TLS cert / key | Paths to cert and key for NNTPS; shared with HTTPS by default (same domain); independently configurable for split-domain setups |
| Auth required for read | Whether unauthenticated clients may read public groups; default false (operator opt-in per IDEA.md) |
| Auth required for post | Whether posting requires authentication; default true |
| Max connections | Total concurrent NNTP connections across plain + TLS; default 256 |
| Max connections per IP | Per-source-IP NNTP connection limit; default 10 |
| Read timeout | Idle read deadline per connection; default 5 min |
| Write timeout | Write deadline per command response; default 30 s |
| Max article size | Maximum bytes accepted for a single article via POST or IHAVE; default 1 MiB; 0 = unlimited |
| Operating mode | `reader` / `feeder` / `both`; default `both` |
| SASL mechanisms | Enabled SASL mechanism list; `PLAIN` allowed only when TLS is active |
| AUTHINFO | Whether legacy `AUTHINFO USER/PASS` is accepted alongside SASL; default true |

## Rate limiting scope

AI.md's `rate_limit.*` config block and the per-IP / per-identifier rate
limiter are HTTP middleware — they apply only to the HTTP listener. They
do not wrap NNTP or SMTP connections. Connection-level limits for those
protocols are defined in the NNTP listener configuration and SMTP inbound
listener configuration sections above.

## GeoIP country blocking scope

AI.md's GeoIP enforcement runs inside the HTTP middleware pipeline. gospool
extends country blocking to the NNTP and SMTP inbound listeners: the GeoIP
check must run at accept time (immediately after the TCP handshake, before
the NNTP/SMTP greeting is sent). A connecting IP that would be blocked on
the HTTP side is also blocked on NNTP and SMTP. Allowlisted IPs bypass
country blocking on all three listeners, matching AI.md PART 20 behavior.
Private/internal IPs (RFC 1918) are never country-blocked on any listener.

## Tor hidden service scope

AI.md PART 32 maps `.onion:80` → `localhost:{server_port}` (the HTTP port).
gospool does not extend the hidden service to NNTP or SMTP. The virtual port
remains 80; only the HTTP listener is reachable via the .onion address. NNTP
and SMTP over Tor (e.g., SOCKS proxy to port 119) are not supported and are
not in scope.

## Protocol access logging

AI.md's `access.log` with Apache/nginx/JSON format covers HTTP requests only.
gospool adds two protocol-specific log files alongside the existing set:

| Log | Purpose | Default format |
|-----|---------|----------------|
| `nntp.log` | NNTP session events | `text` (`text`, `json`) |
| `smtp_in.log` | Inbound SMTP session events | `text` (`text`, `json`) |

**NNTP log fields (text format — one line per command response):**

```
{datetime} {remote_ip} {command} {newsgroup_or_msgid} {response_code} {bytes}
```

Example:
```
2026-07-05T14:00:00Z 203.0.113.4 ARTICLE <abc@news.example.com> 220 4096
2026-07-05T14:00:01Z 203.0.113.4 POST - 240 1234
```

**SMTP inbound log fields (text format — one line per completed transaction):**

```
{datetime} {remote_ip} {helo_hostname} {mail_from} {rcpt_count} {size_bytes} {result}
```

`result` is one of: `accepted`, `rejected`, `deferred`, `spam_hold`.

Example:
```
2026-07-05T14:00:05Z 198.51.100.7 mail.sender.example sender@example.com 1 8192 accepted
```

**JSON format** wraps the same fields as a flat object with snake_case keys.

NNTP and SMTP log rotation follows the same schedule as `access.log` (PART 11
→ "Log Files" — monthly rotation, no compression by default).

## Article spool container path

AI.md PART 27 container paths define `/data/{project_name}/` for app data but
have no entry for the NNTP article body store. gospool adds:

| Path | Purpose |
|------|---------|
| `/data/{project_name}/spool/` | NNTP article body store (filesystem spool) |

The spool directory falls under `/data/` which is already mounted as
`./volumes/data:/data:z` per AI.md PART 27 — no separate volume is needed.
Operators who want to mount the spool on a separate disk can bind-mount
`./volumes/data/{project_name}/spool/` individually in a custom compose
override; the default compose file does not do this. Article metadata
(headers, cross-post references, expiry) lives in the database; article
bodies live in the spool.

## SMTP inbound listener configuration

AI.md PART 18 covers outbound SMTP only. The following config areas are
gospool-specific for the inbound listener:

| Area | Description |
|------|-------------|
| Listen address | Address for inbound SMTP listener; default `0.0.0.0` |
| Port | Default 25; fallback sequence 25 → 2525 → random; `SMTP_IN_PORT` env var locks the port |
| HELO/EHLO hostname | Hostname announced in SMTP greeting; defaults to server FQDN |
| TLS cert / key | Paths for STARTTLS on the inbound listener; shared with HTTPS cert by default |
| Require STARTTLS | Whether plaintext SMTP sessions are accepted after EHLO; default false (log warning if not used) |
| Max message size | Maximum bytes per inbound message; default 10 MiB |
| Max recipients | Maximum `RCPT TO` addresses per transaction; default 50 |
| Auth | SMTP AUTH on inbound; disabled by default (MX-style, no auth from remote MTAs); can be enabled for submission-port deployments |
| SPF / DKIM / DMARC check | Whether to validate sender domain authentication on inbound; default true |
| RBL checks | Whether to query configured DNS blocklists on the connecting IP; default true |
