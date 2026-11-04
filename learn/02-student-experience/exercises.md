# Exercises · predict, observe, explain and preserve the working app

[Stage dashboard](README.md) · [Lessons](lessons.md) · [Source walkthrough](source-walkthrough.md)

These are tasks for the learner, not claims that this guide's author ran them. Use a development
installation and synthetic accounts/data only. Never reset `sportshub_backup` or the original app,
remove volumes, print private settings, or bypass role/CSRF checks. The optional PHP integration and
concurrency runners rebuild `sportshub_backup_test`; **do not run them merely to complete these labs**.
Read [the isolation gate](../../docs/local-setup.md) first if a separately agreed test task needs them.

Before a temporary code edit, inspect `git status --short` and save your own small patch privately.
Undo only your own lines in the editor from that patch; do not restore/reset a whole shared file over
someone else's changes. A lab record needs method/status/state, not passwords, cookies, CSRF values,
private report text, setup keys, recovery codes or raw identity-bearing SQL.

## Lab 1 · Predict before clicking

**Level 1 / objective:** explain Apply without a database write. **Prerequisite:** open source; setup may
still be blocked. Read `action_button()` in [views](../../app/views.php), the join map in
[app.js](../../public/assets/app.js), and `reserve()` in [models](../../app/models.php).

1. Write the native field names and distinguish club ID from membership ID.
2. Write the enhanced method/path/body and identity source. Which native field becomes a URL segment?
3. Name the table/default state and the branch that rejects duplicate pending/approved applications.
4. Explain what an approved relation in a different club should do for this club's private feed.

**Expected/evidence:** a five-layer trace and prediction, explicitly INSPECTED rather than executed.
**Cleanup:** none; no data or source should change. Check your answer against the source, not another
student's copied worksheet. A correct source-only first session is useful progress.

## Lab 2 · Membership decision, reapply and leaving

**Level 2 / objective:** connect UI state to a real relation. **Prerequisites:** own synthetic student,
a spare-capacity development club and a lab administrator in a separate browser profile. If studying
alone, use your own local admin provisioned through the private setup guide; never promote a student by
forging fields. If admin access is unavailable, predict the decision steps and mark them NOT RUN.

1. Apply once. Record method/status and pending badge; inspect the community gate.
2. Have the lab admin reject only this synthetic application. Reapply from club overview; predict whether
   the relation gets a second row/ID or returns to pending through its unique pair.
3. Approve the pending request and refresh Community/Members. Explain why the same account now qualifies.
4. Cancel only your own membership through Activity. Reload the private tab and explain the new gate.

**Expected/evidence:** pending→rejected→pending→approved→no membership. Record actual IDs only where useful
for synthetic evidence and explain the unique pair. A rejected relation is not a deleted one.
**Cleanup:** own membership cancellation is already the last step; no database reset. Leaving does not
promise erasure of all old posts. Do not create private content until the later lab requires it.

## Lab 3 · Compare native and enhanced submission

**Level 2 / objective:** prove two transports share a rule. **Prerequisites:** the lab 2 account and two
available synthetic clubs, or own cancellation between attempts. Do not spam duplicate submissions.

1. With JavaScript enabled, make one membership request. Record REST method/path, object-body field
   names and success state; omit token/cookie values from saved evidence.
2. Disable JavaScript in developer tools, reload the other club and submit its native form.
3. Compare JSON+redirect metadata with POST 303→GET. Restore JavaScript before further browser work.
4. Follow both requests to `perform_action('join')` and `reserve('memberships',...)` in source.

**Expected/evidence:** both create pending state with authenticated user identity, while response formats
and CSRF transport differ. A PHP-development API 404 may mean the router argument is missing, not that
native forms require a new backend. **Cleanup:** cancel only these own lab memberships; restore browser
settings. No source edit or service reset is required.

## Lab 4 · Booking boundary clinic

**Level 2→3 / objective:** reason about strict overlap and ownership. **Prerequisites:** a dedicated
synthetic local resource, one future date and otherwise free intervals. Ask E/F or use your own lab
admin to prepare it; do not choose a resource with wanted team reservations.

Predict before trying: A 10:00–11:00, B 11:00–12:00, C 10:30–11:30 on that future day, same resource.

1. Submit A, then B, then C. Keep each interval within four hours. Explain why pending A already matters.
2. Inspect own history. Approve A through the lab admin, then cancel it as its owner before start time.
3. Read the service's cross-owner selection/update conditions; do not experiment on another real user's row.
4. Draw what two simultaneous unlocked “check then insert” requests could both observe. Locate the actual
   resource lock that prevents that interleaving. Ask G/H/F to review a proposed concurrency change.

**Expected/evidence:** A/B can coexist if no other conflict; C overlaps and is refused. A cancellation
retains a cancelled history record, not deletion. Save interval drawing, actual status and source call.
**Cleanup:** cancel only your remaining future lab bookings. Retain audit/history, and never clear a
shared table. If your observation differs, inspect existing intervals/timezone before blaming the code.

## Lab 5 · Private community and a stale composer

**Level 3 / objective:** test the next-request authorization boundary. **Prerequisites:** approved exact-club
synthetic membership, one own harmless post and administrator access for cleanup. Use plain invented
text, not real people/contact information. Do not screenshot private report reasons.

1. Add a post, comment and reaction. Press the reaction control to remove/add and explain desired state.
2. Submit one harmless synthetic private report. Compare member feed information with the administrator's
   moderation view; record only whether boundaries hold, not the private values.
3. Have the lab admin hide then restore that post. Confirm hidden content leaves the ordinary feed.
   Read the hidden-edit API test/source: author correction changes body but cannot restore status.
4. Open two student tabs. Keep a composer visible in one; leave the club in the other. Predict the stale
   tab's next harmless write, try once, then reload to see the gate.

**Expected/evidence:** approved access is club-specific; post ownership matters; hide is reversible by
admin; author edit is not restoration; a stale DOM grants no new right after leaving.
**Cleanup:** your lab administrator may remove only the synthetic content, or reapply/approve your own
membership before using own controls. Deleting your post erases children and cannot be restored; do
not use Delete as a reversible hide experiment. Keep actual observations separate from source predictions.

## Lab 6 · One scoped student improvement

**Level 3 / objective:** make a personally understood change instead of copying the baseline.
**Prerequisites:** C/D agree one real need, a scoped feature branch and A/B/H/G reviewers as appropriate.
Candidate: a Clubs Clear filters control, if the current branch lacks it and the team agrees it helps.
A candidate is not an asserted bug or a completed contribution; recheck current source first.

1. Reproduce the confusing state; write current and expected behavior. Identify the exact page branch.
2. Implement the smallest manual change. Preserve `page=clubs`, GET behavior, escaping, field names,
   native fallback, roles/CSRF and existing theme/navigation. Do not reformat entire shared files.
3. If that PHP view changed, run `php -l app/views.php` from repository root. Then check q+sport reset,
   empty/nonempty results, keyboard, mobile, both themes and JavaScript disabled as applicable.
4. Explain what changed and what did not require service/schema edits. Ask the named reviewer for genuine
   feedback, then record actual diff/observations in the [contribution log](../../docs/contribution-log.md).

**Expected/evidence:** a useful reviewed difference or a documented decision that no code change was
needed. Include failed/skipped checks honestly. **Restore:** if this was a temporary experiment, undo
only your own lines from your saved patch; do not discard another person's work. Follow the human
workflow before committing/publishing; this guide does not perform those operations for you.

## Hints and answer checks · open after predicting

<details>
<summary>Labs 1–3 · membership and transport</summary>

Native carries action/id/csrf; REST carries club ID in URL, token in header and an empty JSON object.
User identity comes from session, not a submitted hidden user. The upsert reuses a rejected pair;
pending/approved duplicates are refused. Administrator review must recheck capacity/current state.
Native 303 is not a JSON 201 response, even though both may reach the same write service.

</details>

<details>
<summary>Labs 4–5 · interval and access boundaries</summary>

Read bound interval parameters in resource/end/start order. Strict comparisons permit adjacency;
pending/approved rows block overlap. A resource is one unit, and different resources are independent.
Community access is rechecked per request. Hidden reads are unavailable; hidden author edit is an API/
service permission rather than a feed edit control. Deleted tombstones have no restore path. Leaving
is membership removal, not account logout or guaranteed historical content erasure.

</details>

<details>
<summary>Lab 6 · patch scope</summary>

A clear-search action can target the existing unfiltered clubs route; it should not need a new SQL
schema or role change. Check the actual current UI first. Test q and sport together, not only an empty
text field. A source explanation in your own words and real review is evidence; copying this paragraph
and claiming independent implementation is not. Module policy determines acceptable contribution credit.

</details>

## Done rubric

| Level | Demonstrate before advancing |
|---|---|
| Understand | Name real fields/route/service/table and distinguish auth, membership and UI state |
| Modify | Reproduce a useful small need, make a scoped change and explain actual observed outcomes |
| Engineer | Reason about stale state/concurrency/privacy, preserve invariants and get affected-owner review |

- [ ] Native and REST trace is correct, including identity/CSRF and response differences.
- [ ] Rejected/reapply/cancel behavior and exact-club private access are explained from source.
- [ ] Tournament entry, event registration, attended history and generic standings are distinguished.
- [ ] Booking adjacency/duration/status/ownership predictions match actual code or documented observations.
- [ ] Community author/admin permissions, hidden/deleted state and private report boundary are explained.
- [ ] No reset suite, secret export, duplicate runtime or fabricated authorship was used to finish a lab.
- [ ] My own work/checks and any NOT RUN reasons are recorded separately from historic project validation.

If solo, complete the same rubric one journey at a time using [solo path](../solo-path.md); you do not
need to simulate eight people's authorship. Before moving to admin/backend, explain one complete request
in your own words and identify the H/G/E/F boundary requiring review. More reading:
[MDN form validation](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Form_validation)
and [MySQL transactions](https://dev.mysql.com/doc/refman/8.4/en/commit.html).
