# Configuration Rules (PART 5, 6, 12)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Guess configuration keys, defaults, or precedence.
- Log secret values or accept invalid values silently.
- Let a mode bypass required security, validation, or lifecycle behavior.

## CRITICAL - ALWAYS DO
- Follow the exact `server.*` configuration names, types, defaults, and validation.
- Resolve configuration in the specified file, environment, and flag order.
- Keep application modes explicit and apply their documented feature matrix.
- Validate configuration at startup and report safe, actionable errors.

## Key rules summary
- Config files are YAML; secrets use the specified redaction and encryption paths.
- Defaults and migrations are deterministic and backward-compatible.
- Server configuration includes listener, trusted proxy, security, and operational settings.

For complete details, see AI.md PART 5, 6, 12.
