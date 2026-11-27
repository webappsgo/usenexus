# Binary Rules (PART 7, 8, 33)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Enable CGO or introduce runtime dependencies outside the documented exceptions.
- Change CLI flag names, exit codes, or output contracts without specification support.
- Leave a documented CLI operation unsupported.

## CRITICAL - ALWAYS DO
- Build static binaries with the required names, metadata, and linker settings.
- Implement the server CLI, maintenance operations, and shell integrations as specified.
- Keep the required client CLI in parity with supported server operations.
- Include help, version, status, and errors in their specified output formats.

## Key rules summary
- Cross-build the required platform matrix with `CGO_ENABLED=0`.
- Runtime behavior belongs in the server binary; the CLI is an API client.
- Keep completions and CLI documentation synchronized with commands.

For complete details, see AI.md PART 7, 8, 33.
