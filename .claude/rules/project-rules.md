# Project Rules (PART 2, 3, 4)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Add unlisted root files or alter license/attribution without the required authority.
- Store runtime data, credentials, or generated build/test artifacts as source files.
- Hardcode host-specific filesystem paths.

## CRITICAL - ALWAYS DO
- Follow the project structure and required file inventory in AI.md.
- Use the path resolver and OS-specific locations defined by the specification.
- Keep LICENSE.md, README attribution, and dependency licenses consistent.

## Key rules summary
- Project source and documentation stay in their prescribed locations.
- Runtime, cache, config, data, and log paths are distinct and platform-aware.
- Keep project-specific product definition in IDEA.md.

For complete details, see AI.md PART 2, 3, 4.
