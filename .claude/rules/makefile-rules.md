# Makefile Rules (PART 26)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Add Makefile targets beyond the six specified targets.
- Make a local Makefile target publish, tag, or push releases.
- Put CI-only jobs behind new Makefile targets.

## CRITICAL - ALWAYS DO
- Keep `help` as the default target; support `build`, `test`, `release`, `docker`, and `clean`.
- Build static binaries for all required platforms in the specified output directories.
- Make `test` run lint, vulnerability scanning, unit tests, coverage, and binary checks.
- Keep release artifacts local; CI/CD performs publishing.

## Key rules summary
- Build/test runtime artifacts use project-scoped OS temp directories where applicable.
- `clean` removes only the specified build output directories.

For complete details, see AI.md PART 26.
