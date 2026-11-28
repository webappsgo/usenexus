# Command-line client

Build the server and client with `make build`; the client is written to
`binaries/gospool-cli`.

```sh
gospool-cli [flags] COMMAND [arguments]
```

## Interactive interfaces

Running `gospool-cli` without arguments automatically opens the native GUI when
the optional GUI build and a local display are available, or the TUI when an
interactive terminal is available. Configure `display.mode` as `auto` (the
default), `gui`, or `tui` in `cli.yml` to select or force a mode. Forced GUI or
TUI mode exits with an error when its display or terminal is unavailable.
Commands, `--help`, and `--version` always use the CLI. `TERM=dumb`, remote
sessions, and containers disable GUI detection. With no usable display or
terminal and no command, the client exits with a usage error.

The default static client includes the TUI but not the GUI. The optional GUI
artifact is named `gospool-cli-gui-{os}-{arch}` and is dynamically linked to
the platform graphics loader; Linux builds require the loader used by the Go
toolchain. Build it with `make build-gui`. BSD has no GUI artifact. Both
interfaces expose the supported command actions in the CLI command family
order, including subscriber email invitations and CSV import. Press `q` to
quit, `?` for help, `/` to filter, `r` to retry, and Tab or Shift+Tab to move
through the TUI menu. The GUI toolkit currently provides no platform
accessibility API; use the CLI or TUI for accessible operation. The TUI and GUI
still lack in-session token-revocation re-login and the not-yet-implemented
server API actions listed below.

The first interactive run asks for a server URL and saves it in the protected
CLI configuration.

## Commands

| Command | Result |
|---|---|
| `api` | Prints the API root response. |
| `status` | Reads protected operational status and build metadata. |
| `login` | Prompts for username/email and password, then stores the returned bearer token in protected config and token files. |
| `logout` | Removes cached bearer credentials. |
| `groups` | Lists public groups. |
| `groups show NAME` | Reads a group, including private groups with administrator access. |
| `groups create NAME [DESCRIPTION]` | Creates a group (administrator token required). |
| `groups update NAME KEY=VALUE...` | Updates group settings (administrator token required). |
| `groups delete NAME` | Deletes a group (administrator token required). |
| `groups articles NAME` | Lists recent article headers for a public group. |
| `articles show MESSAGE-ID` | Reads an article by Message-ID. |
| `articles search [KEY=VALUE...]` | Searches by `group`, `sender`, `subject`, RFC3339 `from`/`to`, and `limit`. |
| `articles cancel MESSAGE-ID` | Cancels an active article (administrator token required). |
| `tokens list` | Lists the current user's API tokens. |
| `tokens create NAME [SCOPE]` | Creates a token; scope defaults to `read`. |
| `tokens revoke ID` | Revokes a token. |
| `peers [list]` | Lists configured NNTP peers. |
| `peers show ID` | Reads a peer configuration and connection status. |
| `peers create hostname=HOST [KEY=VALUE...]` | Adds a peer; defaults are port `563`, TLS enabled, direction `both`, wildmat `*`, and enabled. |
| `peers update ID KEY=VALUE...` | Updates peer settings. |
| `peers delete ID` | Removes a peer. |
| `peers test ID` | Tests the peer's NNTP greeting, TLS, authentication, and capabilities. |
| `subscribers list GROUP` | Lists confirmed subscribers for a group (administrator or group moderator). |
| `subscribers add GROUP user_id=ID [delivery_mode=MODE]` | Adds an active, verified account; mode defaults to the group's configured default. |
| `subscribers invite GROUP email=ADDRESS [delivery_mode=MODE]` | Sends a confirmation request to an existing active, verified account. |
| `subscribers import GROUP FILE.csv [--dry-run]` | Imports CSV rows with `user_id,delivery_mode` columns; an empty mode uses the group's default. Uploads are limited to 1 MiB and 1000 rows. |
| `subscribers mode GROUP user_id=ID delivery_mode=MODE` | Changes a subscriber's delivery mode. |
| `subscribers remove GROUP user_id=ID` | Removes a subscriber. |
| `moderation list GROUP` | Lists pending articles for a group. |
| `moderation approve GROUP MESSAGE-ID` | Approves a held article. |
| `moderation reject GROUP MESSAGE-ID [REASON]` | Rejects a held article with an optional reason. |
| `moderation hold GROUP SENDER-PATTERN` | Holds matching sender posts for a group. |
| `moderation unhold GROUP SENDER-PATTERN` | Removes a sender hold. |
| `orgs list` | Lists organizations available to the authenticated account. |
| `orgs show SLUG` | Shows organization details. |
| `orgs create SLUG [KEY=VALUE...]` | Creates an organization where the configured creation policy allows it. |
| `orgs delete SLUG` | Deletes an organization when the caller is its owner. |
| `orgs members SLUG` | Lists organization members. |
| `orgs member-add SLUG username=NAME [role=ROLE]` | Adds an organization member. |
| `orgs member-role SLUG USER-ID role=ROLE` | Changes a member role. |
| `orgs member-remove SLUG USER-ID` | Removes an organization member. |
| `users list` | Lists regular accounts (administrator token required). |
| `users invite USERNAME EMAIL` | Creates a single-use user invite (administrator token required). |
| `users suspend USER-ID` / `users unsuspend USER-ID` | Changes account suspension (administrator token required). |
| `audit list|search [KEY=VALUE...]` | Queries audit records by actor, date, category, severity, IP, range, or text. |
| `audit export [format=json|csv] [KEY=VALUE...]` | Exports matching audit records. |
| `webhooks list` | Lists webhook endpoint metadata with signing secrets redacted. |
| `webhooks add HTTPS-URL events=EVENT,... secret_file=PATH` | Creates an endpoint; the URL must resolve only to public addresses, and the secret file must be regular, mode `0600`, and contain at least 32 characters. |
| `webhooks show ID` | Shows endpoint metadata and the latest 20 delivery records without payloads or secrets. |
| `webhooks logs ID [limit=1..100]` | Shows a bounded recent delivery history. |
| `webhooks enable|disable|delete ID` | Changes endpoint enabled state or removes its metadata. |
| `config reload` | Requests a server configuration reload (administrator token required; server handler is not implemented yet). |

Token commands require a user token with the corresponding scope. `login`
reads the password without echoing it and stores the returned session bearer
token with mode `0600`; `logout` removes the cached token from the active
configuration and default token file. Revoked or expired cached credentials
are deleted on the next failed request, and the CLI exits with status `77`.

Peer commands require a server administrator bearer token. Peer settings accept
`hostname`, `port`, `tls`, `direction`, `auth_user`, `wildmat`, and `enabled`.
For authenticated peers, pass `password_file=PATH` using a regular file with no
group or world permissions; password values are never accepted as command-line
arguments. An update can remove saved credentials with
`clear_password=true`.

Subscriber and moderation commands require an administrator credential or a
user account assigned as a moderator for the group. Delivery modes are
`realtime`, `digest`, `nomail`, and `announcement`. Subscriber add currently
accepts an existing verified account ID. Subscriber invitations only target
existing active, verified user accounts and use the confirmation email flow.
CSV imports require the header `user_id,delivery_mode`; use an empty mode to
select the group's configured default. `--dry-run` validates the entire file
without writing subscriptions. Webhook CRUD stores endpoint metadata and
encrypted signing secrets, but no event delivery, retry, or delivery-log
pipeline is implemented. The INN/Mailman/mbox import workflows and live config
reload still need server-side implementation. The audit API supports actor, resource type, date, category,
severity, IP, range, and text filters, with CSV and JSON export.

The CLI currently falls back to configured cluster URLs on connection failures
and server errors after retries. It does not refresh `server.cluster` from
health responses or promote a recovered node. Automatic CLI update discovery,
download, checksum verification, replacement, and re-execution are not
implemented. The TUI also lacks the in-session re-login flow required after a
revoked token. These remain PART 33 gaps.

## Configuration and output

Flags are placed before the command. `--server` overrides `server.primary` in
the CLI configuration; `GOSPOOL_SERVER_PRIMARY` is another override. Use
`--token`, `--token-file`, `GOSPOOL_TOKEN`, or the protected configured token
file for authentication. Token and configuration files must be regular files
with no group or world permissions. `--user` supplies an `@user` or `+org`
context for routes that support it.

`--output` accepts `json`, `plain`, `table`, `csv`, or `yaml`; `--json` is an
alias for JSON. `--color` accepts `auto`, `yes`, or `no`. `--version` and `-v`
print build metadata. `--help` lists the available flags and commands.
`display.mode` accepts `auto`, `gui`, or `tui`; it is a configuration setting,
not a command-line mode flag.

## Shell completions

The server binary (`gospool`) and client (`gospool-cli`) can print completion
scripts and shell initialization commands. The supported shells are Bash,
Zsh, Fish, Ksh, PowerShell, and the `pwsh` alias. For example:

```sh
gospool-cli --shell completions bash > gospool-cli-completions.bash
eval "$(gospool-cli --shell init)"
gospool --shell completions fish > gospool-completions.fish
```

When no shell is supplied, the command detects it from `SHELL`; if detection
fails, pass the shell explicitly. POSIX `sh` and `dash` do not expose a
programmable completion registration interface, so completion and init
requests for those shells return an error.

```sh
gospool-cli --server http://127.0.0.1:8080 --json groups
gospool-cli --server http://127.0.0.1:8080 articles show '<message@example.test>'
```

Group creation, updates, deletion, and peer management require an administrator
bearer token. The configured operator token has administrator access and can be
passed with `--token`. Group updates accept settings such as `visibility` and
`default_delivery_mode`.
