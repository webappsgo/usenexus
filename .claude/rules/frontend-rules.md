# Frontend Rules (PART 16, 17)

⚠️ **These rules are NON-NEGOTIABLE. Violations are bugs.** ⚠️

## CRITICAL - NEVER DO
- Render user-facing pages only in JavaScript or require JavaScript for ordinary navigation/forms.
- Add inline styles, event handlers, executable inline scripts, or page-specific stylesheet files.
- Expose admin links on public routes or render public chrome inside the isolated admin tree.
- Skip accessible landmarks, labels, focus states, or mobile layout requirements.

## CRITICAL - ALWAYS DO
- Render pages on the server and make forms work without JavaScript.
- Use the shared CSS layers, semantic tokens, component catalog, and page shell.
- Give every `/server/*` page the required shared chrome, shell, heading, and cards.
- Give admin pages their isolated header, navigation, main landmark, footer, and skip links.
- Keep interactive API tools self-hosted and provide a no-JavaScript fallback.

## Key rules summary
- Public and admin page shells are distinct and consistent across their route families.
- CSS uses logical properties, accessible contrast, reduced-motion handling, and tokens.
- Admin mutations are CSRF-protected and server-rendered first.

For complete details, see AI.md PART 16, 17.
