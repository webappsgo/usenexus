# AI Rules (PART 0, 1)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Modify AI.md or guess when requirements are unclear.
- Edit before reading the relevant source file and AI.md PART.
- Add unrequested behavior, placeholders, stubs, or TODO comments.
- Claim a check passed without observing its output.

## CRITICAL - ALWAYS DO
- Treat AI.md as read-only source of implementation rules; SPEC.md overrides only where explicit.
- Treat IDEA.md as product behavior and keep it synchronized with implementation.
- Search for existing code and documentation before adding new artifacts.
- Return to the original section after following a cross-reference.
- Keep user-facing documentation consistent with current behavior.

## Key rules summary
- CGO stays disabled; new password hashes use Argon2id; the server renders pages with Go templates.
- The required client CLI is `{project_name}-cli`; Dockerfile belongs at `docker/Dockerfile`.
- Premium features and external cron are prohibited; use the built-in scheduler.

For complete details, see AI.md PART 0, 1.
