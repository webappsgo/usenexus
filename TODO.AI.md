# gospool remaining implementation work

## P0 — Moderation and delivery behavior

- [ ] Add signed, expiring one-click moderation links to moderator hold emails; record every decision in the audit log and notify rejected senders with the configured reason.
- [ ] Implement sender trust rules for authenticated subscribers and configured allowlisted addresses, plus the emergency setting that holds every post regardless of trust.
- [ ] Support subscription-request comments and expose the comment to moderators during approval or rejection.

## P1 — Product features and baseline parity

- [ ] Finish the IDEA.md client commands for subscriber management, moderation, and peer configuration; expose the corresponding server operations instead of returning unsupported-command errors.
- [ ] Complete i18n parity: serve embedded locales at `/locales/{lang}.json`, move server-page, API, and email strings to translation keys, render the selected language and RTL direction, and verify all six locales across the server and client.
- [ ] Bring the shared CSS files and every page into AI.md's layered token/component system; remove the extra page stylesheet, add the required Stylelint checks to the Makefile gates, and verify responsive states in the browser.
- [ ] Complete the AI.md PART-by-PART final compliance review and implement each remaining mandatory baseline item, including the required server-page and configuration behavior.
- [ ] Maintain at least 60% overall test coverage and add regression coverage for each remaining feature and security boundary.

## P2 — Documentation and release parity

- [ ] Keep API/OpenAPI, IDEA route inventory, administration pages, README, installation, integrations, and ReadTheDocs content synchronized as features ship.
- [ ] Run and resolve required Makefile test, lint, build, container, CI workflow, and documentation checks; then complete the prescribed commit and CI verification workflow.
