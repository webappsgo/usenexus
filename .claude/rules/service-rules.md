# Service Rules (PART 24, 25)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Change host users, groups, permissions, or service state outside the prescribed service operation.
- Run privileged operations before validating paths, ownership, and authorization.
- Assume a particular init system or service manager exists.

## CRITICAL - ALWAYS DO
- Drop privileges to the dedicated service account after privileged setup.
- Keep service install, update, restart, and removal behavior platform-specific and safe.
- Preserve user data by default and require exact confirmation for destructive removal.
- Support the specified service managers and report unsupported environments clearly.

## Key rules summary
- Service lifecycle operations are explicit, idempotent, and auditable.
- Filesystem ownership and runtime directories follow the OS-specific path rules.

For complete details, see AI.md PART 24, 25.
