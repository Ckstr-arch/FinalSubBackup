# Exercises · Predict, observe, explain, improve

[Dashboard](README.md) · [Lessons](lessons.md) · [Source walkthrough](source-walkthrough.md) · [Validation policy](../validation.md)

These exercises are **future learner work**, not completed student contributions. Pick one journey, save your prediction first, then compare it to source or a controlled local observation.

## Safety and setup for every lab

- Use the [configured local application](../../README.md), actual MySQL 8+ and synthetic disposable accounts/records.
- Source-only tasks need no database. A local study installation may contain wanted work; do not treat “local” as permission to reset it.
- Reset-capable suites must target the guarded `sportshub_backup_test` only, after you prove it contains no wanted data. Run them sequentially.
- Never delete a valued club/resource to clean a lab. Read its cascade effects first; cancel/remove only your own planned synthetic records through eligible UI actions.
- Do not photograph/log passwords, `.env`, cookies, CSRF/factor values, private reports or real student records.
- Source edits require a narrow agreed E/F boundary and actual peer review. Preserve the original diff; manually undo only your temporary hunk if abandoning it. Stop and coordinate if another person changed those lines.

## Lab 1 · Understand the four CRUD contracts

**Objective:** identify inputs, rows and rejection cases before running a mutation. **Sources:** [entity descriptors](../../app/models.php), [management views](../../app/views.php), [schema](../../database/schema.sql). **Prerequisites:** PHP arrays and basic SQL; no running server is needed.

From the repository root:

```bash
rg -n 'function entities|function save_entity|registration_deadline|home_score|ON DELETE' app/models.php database/schema.sql
```

Use editor search if `rg` is unavailable. Fill a prediction table for a synthetic club, tournament, match event and equipment resource:

| Entity | Required fields | Create/replace route | One rejected input | Delete's linked effect |
|---|---|---|---|---|
| clubs | Your answer | Your answer | Your answer | Your answer |
| tournaments | Your answer | Your answer | Your answer | Your answer |
| events | Your answer | Your answer | Your answer | Your answer |
| resources | Your answer | Your answer | Your answer | Your answer |

Predict these edge cases: capacity 0; tournament end equal to start; deadline equal to start; only one score; both scores 0 on a match; scores on training; nonexistent club; tournament deletion with a linked fixture.

**Expected evidence:** source-linked prediction/correction table and a two-minute explanation of why a table-name allowlist differs from prepared values. **Undo:** no changes were made.

<details><summary>Hints and expected reasoning</summary>

Capacity 0 fails; end equality fails; deadline equality currently passes. A missing score pair fails, while a paired 0:0 match with opponent is valid. Scores on training fail. Event club must exist. Tournament deletion clears event tournament links but removes tournament registrations; it does not delete the events themselves. Use the descriptor for exact field names rather than guessing from labels.

</details>

## Lab 2 · Observe native creation, then REST approval

**Objective:** connect a browser action to persistent data and a student's access. **Sources:** [two request flows](source-walkthrough.md#4-flow-a--native-resource-creation), actions/models/API. **Prerequisites:** running disposable local setup, separate synthetic administrator/student browser profiles, knowledge of CSRF/privacy.

1. As admin, create one expendable equipment resource with JavaScript disabled. Record URL, submitted field names, 303 and destination GET; omit token/cookie values.
2. Reread the new synthetic card and audit entry. Explain which transaction groups those writes.
3. Restore JavaScript and let the synthetic student apply to a study club with spare capacity.
4. Predict what the pending student can read; test the membership gate without editing server guards.
5. As admin, use Approve and observe PATCH `/api/v1/admin/memberships/{membership_id}`. Distinguish the membership ID from the club ID.
6. Refresh the student club space. Explain why the new request receives approved access; a stale open tab was not itself permission.
7. Attempt a student GET to the admin reports API using their normal local browser session; expected 403. Use method/path/status evidence, never raw credential-bearing headers.

**Expected outcomes:** resource persists with audit; application creates pending; approval changes it to approved; student access is scoped to that club; admin refusal stays enforced.

**Questions:** Why does the review response not need to return a complete membership row? Why is the final student refresh meaningful? Which native and API CSRF statuses differ?

**Evidence:** synthetic before/after state, safe request timeline, observed status and your own call-chain explanation. **Cleanup:** remove only the isolated resource if the planned lab verified it has no wanted bookings; let the study student leave only their own membership if desired. Do not erase audit history to disguise the exercise.

<details><summary>Hints</summary>

The action result can contain a message/redirect rather than the row; a subsequent read verifies durable state. Access is rechecked each request against approved membership. Native bad CSRF is 419; REST bad CSRF is 403. The browser's menu is not the role check.

</details>

## Lab 3 · Modify future-attendance feedback

**Objective:** make a real, scoped usability improvement while preserving authoritative validation. **Sources:** Reports branch in [views](../../app/views.php), attendance action in [actions](../../app/actions.php), existing integration cases. **Level:** modify. **E/F agreement:** E edits only the agreed report action cell; F reviews time/state behavior; B reviews accessible presentation and H reviews permission/API parity.

This task is **proposed**, not implemented in these lessons. First reproduce the current mismatch: a future event with a registered participant shows Mark attended, but submitting it is rejected by the server.

1. Prepare three synthetic cases: future registered, started registered, already attended. Use intended test fixtures or reviewed admin records; do not change production clocks/data.
2. Write expected UI and server behavior for each case before coding.
3. Agree a readable label/explanation or control state; avoid using color alone or a mouse-only tooltip.
4. Manually change the narrow markup branch. Do not remove the service's start-time, role or CSRF checks.
5. Test keyboard access, dark/light themes and narrow/wide viewport layouts.
6. Send the future action through its ordinary endpoint with a valid synthetic admin session; it must still reject with 422.
7. Confirm a started registered row can become attended and already attended is represented honestly.

Suggested non-destructive checks from source root:

```bash
php -l app/views.php
php -l app/actions.php
git diff --check
git diff -- app/views.php app/actions.php
```

Use the documented Docker project prefix when PHP runs in a container. These commands are instructions, not evidence they were executed by your group. Syntax checks cannot prove the action's permission/state behavior.

**Expected evidence:** personally produced diff, synthetic before/after, three-state observations, preserved direct-request rejection and real review notes. **Undo:** manually reverse only your recorded temporary hunk after checking the current diff; do not restore/reset a whole shared file.

<details><summary>Hints</summary>

The current UI condition tests status registered without checking the event start. The backend separately joins its event and rejects starts in the future. Use the same time interpretation rather than comparing a formatted display label lexicographically. A hidden or disabled control is still not authorization.

</details>

## Lab 4 · Engineer a competing approval on paper

**Objective:** explain why lock order and read visibility jointly protect capacity. **Sources:** `review_request()` and [existing concurrency regression](../../tests/concurrency.php). **Prerequisites:** Level 1/2 understanding of transactions; this default exercise is read-only.

Draw two different pending memberships for a club with capacity 1 and no approved members. Administrator A holds the parent lock while B waits. A approves one request and commits. Predict what B must read and why its different request remains pending.

Annotate the actual order: candidate read → parent lock → request lock/reread → capacity count → status/audit/commit or rollback. Explain where READ COMMITTED matters after the wait. Then apply the reasoning to two conflicting synthetic pending bookings.

The current regression constructs conflicting pending booking rows deliberately to exercise review-time protection. Ordinary booking creation already rejects overlapping pending intervals; do not describe such a pair as the ordinary happy path.

**Expected:** one approved membership/booking, waiting conflicting request rejected, truthful audit for the accepted service operation. The test's competing transaction deliberately updates synthetic data to model a competitor; do not claim every step was an actual HTTP admin approval.

**Evidence:** interleaving diagram, invariant, source statements and one proposed additional edge case with exact expected outcome. **Undo:** paper/source inspection has no state to revert. Optional runner execution needs the explicit disposable-schema gate and is recorded separately from this exercise.

<details><summary>Hints</summary>

Locking the parent coordinates different request rows. Locking/rereading the child catches stale status on the same request. READ COMMITTED lets a later count see a competitor committed while waiting. Consistent lock order reduces avoidable conflicts but is not proof every concurrent edit path is tested.

</details>

## Lab 5 · Explain report and moderation evidence

**Objective:** avoid confusing counts, reversible decisions and deletion. **Sources:** Reports/Results and community/moderation service. **Setup:** source-only notebook dataset first; optional synthetic local observation after prediction.

Use one student with three event registrations, two attended, and one separate tournament entry. Predict session registrations, attended count and tournament roster. Then model a reported post: hide → author edit while hidden → admin restore. Explain which report status changes and what remains if the author later deletes that expendable synthetic post.

**Expected:** 3 session registration rows, 2 attended; tournament entry stays independent. Hidden edit stays hidden. Moderation resolves reports in its decision transaction. Restore does not reopen them. Post deletion erases body/children and cannot be reversed through moderation.

**Evidence:** your calculated counts, state table, safe screenshot only if actually captured, and one limitation of generic standings. **Cleanup:** restore only your reversible synthetic hide state if appropriate; do not erase actual audit/report history. Irreversible deletion is optional and only for the expressly expendable study post.

## Manual regression checklist

- [ ] Guest/student read and forged admin-write requests remain refused.
- [ ] Four management forms preserve names/types and required relationships.
- [ ] Native POST and enhanced REST reach the shared service correctly.
- [ ] Required/enum/date/capacity/paired-score failures leave intended state intact.
- [ ] Admin create/update/delete persist with their transactional audit.
- [ ] Membership/booking review rejects nonpending, full or overlapping state.
- [ ] Future attendance rejects; started participation can be marked; counts mean rows, not invented unique people.
- [ ] Moderation reasons/reporters remain private; hidden edit cannot restore; deleted cannot restore.
- [ ] Synthetic parent deletion follows the documented cascades/SET NULL behavior.
- [ ] Keyboard/mobile/dark/light/native-form behavior remains usable.

## Review rubric · use evidence, not invented scores

| Level | Observable explanation/evidence |
|---|---|
| Understand | Correct source/route/field/table map, predicted allowed and denied case, baseline acknowledged |
| Modify | Narrow personal patch, before/after synthetic evidence, focused actual checks and shared review |
| Engineer | Explain a timing/privacy/relationship edge, preserved invariant, meaningful regression design and remaining limit |

A peer records what they actually examined and what changed after feedback. This is a learning rubric, not an assessment-mark guarantee. Do not prefill names, pass counts, commits, PRs or dates.

## Write a truthful handoff

Record starting commit, exact branch/function scope, prediction, actual result, your diff or review contribution, tests performed/skipped, reviewer feedback and assistance disclosure. A read-only investigation can be genuine work without pretending you wrote its existing service.

Use [the contribution log](../../docs/contribution-log.md), [team workflow](../../docs/build-order-and-contributions.md) and [learning validation](../validation.md). End by explaining one journey in your own words: **who asks, which server rule decides, what row changes, what the next reader sees, and what remains unproven**.
