# Contribution log · actual work and learning evidence

Use this register alongside the [ordered build and contribution guide](build-order-and-contributions.md) and the [eight-member team plan](eight-member-plan.md). H (API/authentication/security) is the requester’s chosen responsibility. **Activity evidence remains unfilled: no implementation, test result, contribution commit or review is claimed as completed by this document.** Fill activity rows only after real work happens.

## Baseline and assistance

This repository contains a **Codex-assisted existing prototype**. It was exported from `Ckstr-arch/TestENV`'s `backup/` at source commit `b2df1bd73415aa4a62e13b3a2bf903baeddc09a9`; its underlying runtime baseline was `f81eab54ebe7179e1763f364d6b8d4f564c36541`. See [guidelines/provenance](guidelines-audit.md), [architecture](architecture.md) and [report guidance](report-and-video.md).

Reusing the baseline can preserve a working application, but does not make a teammate its original author. Copying an existing module unchanged into the same tracked path creates **no new Git diff**. Copying it to another folder creates duplicate files, not evidence of new implementation; keep one canonical runtime. Do not manufacture cosmetic commits or claim old modules as independently developed.

Real review, debugging, testing, design or documentation can be a contribution if actually performed, evidenced and accepted by the lecturer's policy. Record your own observations and reasoning; copying a generated explanation and signing it is not evidence of understanding. Disclose existing code and AI/peer assistance under the module's originality policy. Ask the lecturer about permitted reuse if unclear; this log cannot guarantee assessment credit.

## Team responsibility agreement · humans fill this

The actual team has **eight students**. The **four learning tracks describe areas of work, not the number of students**. Two people per stage is a proposed way to share implementation and review. H is explicitly chosen by the requester; the other pairings/accepted roles remain to be agreed and filled.

| Person | Own GitHub handle | Accepted stage/task boundary | Shared-file reviewer |
|---|---|---|---|
| Member A — TBD | TODO | TODO | TODO |
| Member B — TBD | TODO | TODO | TODO |
| Member C — TBD | TODO | TODO | TODO |
| Member D — TBD | TODO | TODO | TODO |
| Member E — TBD | TODO | TODO | TODO |
| Member F — TBD | TODO | TODO | TODO |
| Member G — TBD | TODO | TODO | TODO |
| Member H — requester; name TODO | TODO | H — API/authentication/security (chosen); actual task/evidence TODO | TODO |

The eight slots do not identify real names or GitHub handles. H records only the requester’s stated role choice; A–G and pairings remain proposed. Keep identity, reviewer and work evidence TODO until confirmed; a chosen role does not establish completed work or baseline authorship.

## Shared activity register · add one row per real task

| Actual date | Student / own GitHub | Task / reason | Paths + functions or page branches | Work type | Actual commit / PR | Check actually run + result | Real reviewer / feedback | Assistance disclosed | Own explanation / evidence |
|---|---|---|---|---|---|---|---|---|---|
| TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO | TODO |

Work types include **implementation, debugging, review, testing, documentation and design**; use the accurate one or combination. An existing function being inspected belongs in “paths/functions,” not automatically “implementation.” Link only real commits/PRs after they exist. For a review-only activity, use the actual reviewed commit/PR and say explicitly that no source patch was made. Record failures and skipped checks with reasons; do not copy a historic pass count as your own execution.

## Per-person task record · copy this blank template

| Field | Actual evidence to enter |
|---|---|
| Person, own handle, actual date, accepted task | TODO |
| Starting commit and existing code acknowledged | TODO |
| Problem/question and prediction before work | TODO |
| Paths/functions read; exact changes personally made | TODO |
| Work type and reasoning/trade-offs in own words | TODO |
| Own scoped branch and actual commit/PR links | TODO |
| Commands/manual steps personally run, environment and actual outcomes | TODO |
| Failure/skip, diagnosis and next step | TODO |
| Redacted before/after evidence or concrete review notes | TODO |
| Actual reviewer, feedback and response | TODO |
| AI/peer help: what was supplied and what personally checked | TODO |
| One-minute explanation and eventual video timestamp | TODO |

## Illustrative task idea · NOT a completed log entry

A learner could inspect `app/actions.php`, `perform_action()`'s booking branch, and explain why `existing.starts_at < requested.end` plus `existing.ends_at > requested.start` allows adjacent intervals while blocking overlaps. Predict two synthetic local booking outcomes, then verify them safely and record what actually happened. A genuine outcome could be a test observation or an original reviewed explanation; it need not pretend the learner invented the existing locking algorithm. **Person/date/commit/PR/result/reviewer: TODO until real work occurs.**

## GitHub evidence and review gate

- Use your own authenticated account and truthful verified Git identity. Choose a short-lived feature branch for the actual task; follow the [manual workflow](build-order-and-contributions.md).
- Before editing shared code, agree on exact functions/branches and a reviewer. Preserve existing user changes, private settings, database state and security checks.
- Inspect `git status --short` and `git diff` to distinguish actual edits from unchanged copied code. Stage only personally reviewed intended paths when performing the authorized human workflow.
- In the real PR, state the problem, changed/read paths, work type, native/REST impact, tests actually run and your own explanation. Include redacted screenshots only when they prove something relevant.
- A reviewer must inspect the actual diff or evidence and provide genuine feedback. No fabricated approval or retroactive coauthor identity; verify individual commit attribution after integration.
- Never include passwords, `.env`, cookies, private reports, authenticator keys, recovery codes or identity-bearing database exports. Use synthetic records and [the isolated test rules](local-setup.md).

An honest assistance statement can follow this shape, filled only with facts: “Existing source used: TODO. Assistance received: TODO. My actual work: TODO. I personally checked: TODO. Remaining uncertainty: TODO.” Generated guides are reference material; their publication is not automatically each student's contribution.

Before the report/video, compare every personal claim with its real diff, review/test record and explanation. Use [validation](validation.md) for the project's actual recorded checks, while keeping your own executions separate. Leave unfinished work visibly TODO.
