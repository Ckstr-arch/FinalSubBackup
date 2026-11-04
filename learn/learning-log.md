# Your learning and evidence log

[Learning home](README.md) · [Solo route](solo-path.md) · [Official team contribution template](../docs/contribution-log.md)

This file is an unfilled learning template. Make your own copy or add a dated entry on your task branch. Do not put private credentials, real student records, cookies, authenticator seeds, recovery codes, raw request headers or database dumps here. Use synthetic records and sanitized results.

The established application is a Codex-assisted baseline. Entries below describe work that happens after you do it; they do not assign old module authorship. Use the team contribution template for final evidence and accepted assignments.

## Stage progress

Edit `[ ]` to `[x]` in Markdown after you can demonstrate the checkpoint. A tick is your learning record, not an automatically executed test.

| Stage | Observed existing flow | Explained without notes | Own exercise/change | Verification recorded |
|---|---|---|---|---|
| 01 — Frontend and UX | [ ] | [ ] | [ ] | [ ] |
| 02 — Student experience | [ ] | [ ] | [ ] | [ ] |
| 03 — Admin operations | [ ] | [ ] | [ ] | [ ] |
| 04 — Backend/database/security | [ ] | [ ] | [ ] | [ ] |
| 05 — Assemble/deploy | [ ] | [ ] | [ ] | [ ] |

## Session entry template

```text
Date and timezone:
Person / GitHub handle:
Stage and lesson:
Baseline / branch / actual commit (when available):

My question before reading:
My prediction:
Exact file, function or page branch inspected:
What the code actually does:
What I changed or investigated myself:
Command or manual steps actually executed:
Environment (OS, native/Docker, PHP/MySQL versions; no credentials):
Expected outcome:
Actual outcome:
Failure/negative case observed:
Evidence location (sanitized screenshot, test output, PR):
Peer reviewer and actual feedback, if any:
Assistance used and what I verified myself:
What remains uncertain:
My next bounded step:
```

## Explain-back prompts

Use one per session. Say the answer in your own words, then check the real code.

1. What does the browser receive after PHP executes, and which files remain private?
2. Where does this app save the theme, the session identity and a membership?
3. What happens after a successful native POST compared with an enhanced REST request?
4. Why does prepared SQL not replace role checks or output escaping?
5. Why is a pending club member denied private club content?
6. Why can two concurrent approvals exceed capacity without an appropriate lock/check?
7. Why does an enrolled user's password step not immediately authenticate them?
8. What breaks if an existing database is restored without the matching `APP_KEY`?
9. What does passing a local test prove, and what does it leave unverified about a host?
10. Which contribution can you show with an actual diff, observation and explanation?

## Example of honest wording

> Reviewed the existing assisted membership flow and traced `reserve()` through the pending-state insert. Implemented a scoped UI/status clarification, checked synthetic pending and approved users locally, and recorded the actual test result in the linked PR.

This is a wording example, **not a completed contribution**. Replace it with facts about your own work. If a check was not run, write `not run` and its reason. If a task is blocked, describe the exact missing capability and the next diagnostic step.
