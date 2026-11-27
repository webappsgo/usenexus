# Development

## Requirements

- Docker with the `casjaysdev/go:latest` image.
- Make.

Go is built and tested only in the project Go container:

```sh
make test
make build
```

`make test` runs Stylelint, `go vet`, `staticcheck`, `govulncheck`, ShellCheck,
`shfmt`, the Go tests with coverage, and the basic Docker binary checks. The
project implementation standard is `AI.md`, with project-specific overrides
in `SPEC.md`. `IDEA.md` is the product behavior reference. For documentation
changes, install `docs/requirements.txt` in a Python environment and build the
site with `mkdocs build --strict` before submitting.

The binary integration checks run the server in a temporary Docker container.
They verify that `/server/healthz` returns formatted non-HTML output for
`Accept: text/plain` and server-rendered HTML for `Accept: text/html`. The Tor
and I2P bootstrap waits default to 300 and 600 seconds; set `TOR_WAIT` or
`I2P_WAIT` to change either timeout. A network-dependent bootstrap timeout is
reported as a skip after the health contract is checked. Set
`REQUIRE_OVERLAY=1` to fail instead of skipping when either overlay is not
healthy within its wait. For example:

```sh
TOR_WAIT=300 I2P_WAIT=600 REQUIRE_OVERLAY=1 tests/docker.sh
```

Build binaries into an isolated directory and point the test runner at it:

```sh
mkdir -p "${TMPDIR:-/tmp}/webappsgo"
GOSPOOL_BUILD_DIR="$(mktemp -d "${TMPDIR:-/tmp}/webappsgo/gospool-XXXXXX")"
make BINDIR="$GOSPOOL_BUILD_DIR" build
BINDIR="$GOSPOOL_BUILD_DIR" tests/run_tests.sh
make BINDIR="$GOSPOOL_BUILD_DIR" clean
```

The build and test scripts store runtime state under
`${TMPDIR:-/tmp}/webappsgo/` and remove their
container and temporary state when it exits.

The Docker and Incus suites also verify operator-token protection on the debug
routes: missing and wrong credentials return `401`, owner tokens return `403`,
the `server.token` from the test container's `secrets.yml` authenticates, and
query-string credentials are rejected. The deterministic `/debug/error500`
route returns `500` after successful authentication. The token is not printed
by the test scripts.

## Layout

- `src/config`: YAML configuration, defaults, and protected first-run secrets.
- `src/paths`: platform and container paths.
- `src/server`: HTTP routes, templates, auth, API, and GraphQL.
- `src/server/store`: database schema and persistence.
- `src/server/service`: article ingestion and cryptographic helpers.
- `src/client`: companion HTTP API client.
- `docker`: image and Compose definitions.
