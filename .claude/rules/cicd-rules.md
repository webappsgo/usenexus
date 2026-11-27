# CI/CD Rules (PART 28)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Publish from untrusted pull requests or bypass required checks.
- Store credentials in workflow files or print secret values.
- Use mutable third-party actions without the specified pinning.

## CRITICAL - ALWAYS DO
- Maintain the required provider workflows and their consistent stages.
- Pin actions and external tools to verified immutable revisions.
- Run tests, linters, security scans, builds, and release jobs in isolated containers.
- Restrict release and publish jobs to the prescribed trusted events and branches.

## Key rules summary
- CI workflows are provider-native and do not add Makefile targets.
- Preserve workflow parity and keep documented checks runnable locally.

For complete details, see AI.md PART 28.
