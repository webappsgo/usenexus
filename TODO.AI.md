# gospool remaining implementation work

## P0 — Moderation and delivery behavior

- [ ] Add signed, expiring one-click moderation links to moderator hold emails; record every decision in the audit log and notify rejected senders with the configured reason.
- [ ] Implement sender trust rules for authenticated subscribers and configured allowlisted addresses, plus the emergency setting that holds every post regardless of trust.
- [ ] Complete the `announcement` subscription mode so subscribers receive only moderator and administrator posts; retain `nomail` subscriptions without queueing email.
- [ ] Add the email-command interface for `SUBSCRIBE`, `UNSUBSCRIBE`, `DIGEST`, and `HELP`, using the existing confirmation-token safeguards and never changing membership on an unconfirmed command.
- [ ] Support subscription-request comments and expose the comment to moderators during approval or rejection.

## P1 — Product features and baseline parity

- [ ] Implement RFC 4155 `rnews` batch injection through the server binary with the same deduplication, age, and group-filter rules as live peer feeds.
- [ ] Finish the required CLI operations in IDEA.md, including group lifecycle, imports, and article injection; expose server functionality instead of returning unsupported-command errors.
- [ ] Complete server and email localization by moving visible strings to translation keys and verifying all six embedded locales across public pages, admin pages, errors, and notifications.
- [ ] Complete the AI.md PART-by-PART final compliance review and implement each remaining mandatory baseline item, including the required server-page and configuration behavior.
- [ ] Raise package and overall test coverage to the AI.md requirements while adding regression coverage for every remaining feature.

## P2 — Documentation and release parity

- [ ] Keep API/OpenAPI, IDEA route inventory, administration pages, README, installation, integrations, and ReadTheDocs content synchronized as features ship.
- [ ] Run and resolve required Makefile test, lint, build, container, CI workflow, and documentation checks; then complete the prescribed commit and CI verification workflow.
