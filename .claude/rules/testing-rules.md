# Testing Rules (PART 29, 30, 31)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Put test databases, logs, runtime files, or generated outputs in the source tree.
- Run host-affecting tests directly on the host.
- Claim coverage, accessibility, or documentation checks without running them.
- Ship untranslated or inaccessible user-facing text and controls.

## CRITICAL - ALWAYS DO
- Run tests and direct lint commands through the required gate wrapper and project Makefile.
- Use project-scoped OS temporary directories and clean up task-owned artifacts.
- Test authentication, authorization, CSRF, no-JavaScript behavior, and critical failure paths.
- Build and verify ReadTheDocs content and all supported locales.
- Meet the specified coverage threshold and report actual command results.

## Key rules summary
- Use containers or VMs for tests that affect host services, networking, or privileges.
- Internationalization and accessibility are implementation requirements, not polish.

For complete details, see AI.md PART 29, 30, 31.
