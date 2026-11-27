# Backend Rules (PART 9, 10, 11, 32)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Leak secrets, credentials, tokens, or sensitive user data in logs or errors.
- Build SQL from untrusted input or ignore database ownership and transaction rules.
- Treat cache state as authoritative data.
- Make Tor or I2P required for normal server operation.

## CRITICAL - ALWAYS DO
- Use typed errors, stable API errors, bounded caches, and documented timeouts.
- Apply migrations safely and preserve transactional consistency.
- Enforce authentication, authorization, CSRF, input validation, and secure headers.
- Hash or encrypt secrets as specified; redact them from all logs and outputs.
- Support the required Tor and I2P lifecycle and run without their daemons when unavailable.

## Key rules summary
- Database operations follow ownership boundaries, migration rules, and cluster guarantees.
- Security controls are defense-in-depth and fail according to the specified policy.
- Overlay services are supervised, bounded, and shut down gracefully.

For complete details, see AI.md PART 9, 10, 11, 32.
