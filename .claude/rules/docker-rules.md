# Docker Rules (PART 27)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Put the Dockerfile at the repository root or ship build tooling in the runtime image.
- Run as root in the application container.
- Bake secrets, runtime data, or local build artifacts into the image.
- Publish images from a local Makefile command.

## CRITICAL - ALWAYS DO
- Keep the Dockerfile at `docker/Dockerfile` and use the specified multi-stage build.
- Use a minimal runtime image, static binary, non-root user, and exact health check.
- Keep `.dockerignore` synchronized with the build context and embedded assets.
- Expose and configure only the ports required by AI.md and SPEC.md overrides.

## Key rules summary
- Build reproducibly with metadata and platform support.
- CI/CD publishes images; local Docker targets build only.

For complete details, see AI.md PART 27.
