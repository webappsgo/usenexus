# Feature Rules (PART 18, 19, 20, 21, 22, 23)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Send notifications outside the configured provider, retry, and privacy rules.
- Use an external cron service instead of the built-in scheduler.
- Trust GeoIP data as a security boundary when lookup is unavailable.
- Expose metrics without the required access controls or labels.
- Restore unverified archives or update from untrusted release artifacts.

## CRITICAL - ALWAYS DO
- Render and deliver every configured notification through the standard channel pipeline.
- Run scheduled tasks with persistence, locking, bounded retries, and history.
- Apply GeoIP allowlists, database freshness, and documented fail-open behavior.
- Keep metrics bounded, low-cardinality, and safe to expose.
- Verify backup manifests, checksums, paths, encryption, and restore preconditions.
- Verify signed update metadata and checksums before replacing a binary.

## Key rules summary
- Email templates use the documented variable syntax and embedded defaults.
- Backup and update operations are atomic, auditable, and recoverable.

For complete details, see AI.md PART 18, 19, 20, 21, 22, 23.
