# gospool implementation backlog

Dependency order: project skeleton → server core and persistence → user/admin web flows → NNTP and SMTP protocols → integrations and background services → client CLI → deployment and documentation parity.

## P0 — Core application

- [ ] Implement the Go server, configuration, persistence, security, health, API, and server-rendered pages required by AI.md PARTs 3–17, applying SPEC.md listener and database overrides.
- [ ] Implement the NNTP/NNTPS, inbound/outbound SMTP, group, article, subscription, moderation, and peering behavior in IDEA.md and SPEC.md.
- [ ] Implement all activated PART 34–36 account, organization, and custom-domain requirements.

## P1 — Required project surfaces

- [ ] Implement scheduler, GeoIP, metrics, backup/restore, updater, service support, Tor/I2P, and localization requirements from AI.md PARTs 18–25 and 31–33.
- [ ] Implement the required client CLI and its IDEA.md operations.
- [ ] Add complete tests and testing scripts required by AI.md PART 29.

## P2 — Delivery and documentation

- [ ] Add the Makefile, Docker images/compose variants, multi-provider CI/CD, dependency automation, security policy, and project governance files required by AI.md PARTs 26–28.
- [ ] Add complete ReadTheDocs documentation and keep README, API/OpenAPI, CLI help, server pages, and docs synchronized with the shipped behavior.
- [ ] Complete the AI.md FINAL compliance review and remove resolved task entries after their resolving commits.
