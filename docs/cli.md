# Command-line client

Build the server and client with `make build`; the client is written to
`binaries/gospool-cli`.

```sh
gospool-cli [flags] COMMAND [arguments]
```

## Interactive terminal interface

Running `gospool-cli` without arguments in an interactive terminal opens the
TUI. `gospool-cli --tui` opens it explicitly. The first run asks for a server
URL and saves it in the protected CLI configuration. The TUI has a view for
every command listed below, including prompts for group names, Message-IDs, and
token operations. Its menu supports filtering and responsive layouts. Press
`q` to quit, `?` for help, `/` to filter, `r` to retry, and Tab or Shift+Tab to
move through the menu. `TERM=dumb` always disables interactive mode.

## Commands

| Command | Result |
|---|---|
| `api` | Prints the API root response. |
| `status` | Reads server health. |
| `groups` | Lists public groups. |
| `groups articles NAME` | Lists recent article headers for a public group. |
| `articles show MESSAGE-ID` | Reads an article by Message-ID. |
| `tokens list` | Lists the current user's API tokens. |
| `tokens create NAME [SCOPE]` | Creates a token; scope defaults to `read`. |
| `tokens revoke ID` | Revokes a token. |

Token management commands require a user token with the corresponding scope.
`login` is not available because the server does not expose a client login
endpoint.

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

The client currently exposes only the commands listed above. The broader
IDEA.md management operations depend on API routes and workflows that are not
implemented yet.
