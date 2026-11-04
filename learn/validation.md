# Learning-package validation and known limits

[Learning home](README.md) · [Source map](source-map.md) · [Application validation history](../docs/validation.md)

Review date: **4 November 2026 (Asia/Colombo)**. Source baseline: local `main` at `f455575` before this documentation addition. The new material is adapted to `FinalSubBackup`, not copied unchanged from the old TestENV stage package.

## What this review covers

- Five learning stages: frontend/UX, student experience, admin operations, backend/database/security and assembly/deployment.
- Explanations traced to actual PHP entrypoints, shared actions/helpers, templates, JavaScript, schema, CLI tools and tests.
- Relative links that resolve within this standalone repository, resource references and runnable-command prerequisites.
- Preservation of tracked application/schema/config/test/asset files; learning exercises remain instructions rather than silently implemented application changes.
- A solo route, A–H responsibility boundaries, proposed exercises and truthful evidence templates.

## Verification status

These checks were executed for this documentation task on the review date. Historical application checks in `docs/validation.md` remain historical; they are not presented as a fresh live-server run of these guides.

| Check | Result |
|---|---|
| Native-stack/local Markdown file-target audit | PASS — `php tests/audit.php`: 857 checks over 78 files, 16 runtime text files and 587 local Markdown links |
| Markdown fences, heading anchors and stage structure | PASS — 29 learning Markdown files, all five stage sets, 447 local targets and 12 linked heading anchors; fences paired; 14 Mermaid blocks inspected as source, not rendered |
| Tracked non-Markdown source preservation | PASS — SHA-256 comparison of all 36 baseline tracked application/config/schema/test/asset files; no changes |
| Existing security unit checks | PASS — 23 checks using an ephemeral private process key; no database access/reset |
| External official resource reachability | PASS — all 84 unique current learning-resource URLs returned HTTP 2xx after replacing one broken PHP reference; reachability is not a content/security certification |
| Hosted deployment, HTTPS acceptance and restoration | Not performed by this documentation task |

`git diff --check` and a separate scan of the new Markdown found no whitespace errors. Short PHP excerpts were compared with their canonical source during stage reviews. A cross-stage review corrected two source-location/escaping descriptions and added the fresh-database club prerequisite. A second source review before publication clarified that prerequisite in the assembly lab, PowerShell handling of the Bash backup example, and health-check connection initialization. No functional/browser/concurrency suites were rerun because runtime behavior was unchanged; no DB reset, container operation or deployment was needed.

This package was created in the local `main` working tree and reviewed again at the user's explicit request before authorized commit/push publication. It contains proposed learner tasks and blank evidence templates, not fabricated teammate work. The resulting commit and remote publication verification are reported separately after execution; this document does not imply application deployment.

## What these checks cannot prove

A Markdown file-link audit does not test a route, render a diagram, establish accessibility or verify external content. An HTTP URL check only shows a response at review time. Unit tests do not certify security or test every native/REST workflow. Old functional/browser/concurrency results are useful baseline evidence but do not verify a new machine or hosting plan.

Exercises involving real HTTP/database behavior are instructions for the learner. No exercise is marked completed on a student's behalf. Resetting suites need a confirmed disposable database and explicit flags; this task does not authorize destructive changes to valued local or live data.

## Source-specific limits to retain in explanations

1. This app requires **actual MySQL 8+**, not just a MySQL-compatible connector or MariaDB branding.
2. PHP renders the canonical app. `learn/` contains documentation, not a second runnable app or four module copies.
3. Optional authenticator 2FA, email verification/reset absence, single-instance file sessions and no institutional SSO remain current product limits.
4. The PDO connection does not currently configure verified remote-MySQL TLS options. A host requiring them needs reviewed connection configuration/code before use.
5. The local Docker recipe has `APP_ENV=development` and loopback HTTP. It needs a reviewed production configuration and HTTPS hosting, not a casual public port change.
6. Tournament registration is individual; fixtures/scores are administered manually. Generic standings use 3/1/0 points; automated brackets/team entry/sport-specific tie-breaks are not implemented.
7. Resources are single reservable units. Pending/approved bookings block overlaps; adjacent intervals are allowed.
8. Parent deletes can cascade through related data. Community hide/restore is different from deletion with irreversible body/child removal.
9. Automatic backups, verified restore, host patching/monitoring, real identities/contributions, image rights, final report/video and lecturer access are not created merely by adding these guides.

## Keeping the guides accurate after development

For each patch, ask whether it changes a route/payload, form mapping, database rule, role boundary, state transition, UI behavior or command. Update its primary stage explanation and the canonical contract/runbook as appropriate. Record actual validation under the reviewed commit. Leave unknowns explicit rather than inventing a host result, author, deadline or completed test.
