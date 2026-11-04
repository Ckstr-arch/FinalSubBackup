# Exercises · practise safely, then explain the result

[Track home](README.md) · [Lessons](lessons.md) · [Source walkthrough](source-walkthrough.md) · [Solo route](../solo-path.md)

These are learner activities, not completed test results or preassigned contributions.
Use a configured local application and synthetic content; keep real identities, cookies, tokens and recovery values out of notes/screenshots.
The inspection exercises do not require a database reset or a production change.
DevTools edits are temporary: reload to restore the server-rendered page.
If you later edit a source file, keep a pre-edit copy or a small reviewed diff and restore only your own temporary lines; never reset/clean the whole repository.

## Exercise 1 · Map a card without changing it — Understand

**Objective:** connect visible HTML, the DOM and its PHP source.
Prerequisites: the public homepage and the first lesson on semantic elements.
Sources: [views.php](../../app/views.php), [layout.php](../../app/layout.php), [app.css](../../public/assets/app.css).

1. Open Home, choose one club card and inspect its article, heading, paragraph and link.
2. Write the parent chain from `main#main` to the heading. Predict which element supplies the card background.
3. Find the same markup in the homepage branch and identify the `foreach` and two escaped values.
4. Compare View Source with Elements. Locate where PHP syntax has disappeared and actual text has appeared.
5. If the database has no clubs, inspect the empty-state branch rather than creating or deleting shared records.

**Expected:** `.grid.three` owns the layout, `.card` owns the surface, and the PHP loop provides real rows.
The heading text is not stored in CSS and no production `home.html` is involved.
**Evidence:** a small source→element→selector table and your explanation of one escaped value.
**Restore:** none needed; reload if you experimented in Elements.

<details><summary>Hint and answer direction</summary>

The loop does not create a new CSS rule per club; each article reuses the same classes.
The database name/description become HTML text through `e()`.
A class names a styling hook, not a particular database record.

</details>

## Exercise 2 · Prove a layout prediction — Understand → Modify

**Objective:** explain Grid, Flexbox and a breakpoint using observed computed values.
Prerequisites: lesson sections on box sizing and layout.
Sources: `.grid.three`, `.card`, `.workspace-header` and media queries in the stylesheet.

1. At 1440px, predict how many card tracks appear; measure the grid content width and gap in DevTools.
2. At 1051/1050px, compare the matching card and header-search rules. At 501/500px, compare again.
3. At 801/800px, inspect how the authenticated sidebar becomes a drawer; if you have no disposable signed-in account, read its source and mark this part NOT RUN.
4. Temporarily turn off `display:grid` on the card wrapper in DevTools. Predict normal block behavior before observing it.
5. Reenable it and change only the temporary column definition to `1fr`. Compare your experiment with the actual breakpoint strategy.

**Expected:** three/two/one card tracks at the relevant sizes; the header search disappears at 1050px; mobile sidebar behavior starts at 800px.
A layout can change before a literal phone width because the component needs more space.
**Evidence:** viewport, winning selector, computed track count and observed overflow; label DevTools overrides as temporary.
**Restore:** reload. No source patch or database mutation is needed.

<details><summary>Hint and answer direction</summary>

Look for an active media-query rule rather than assuming the device name selects the design.
Subtract gaps before dividing fractional tracks.
A long word can still overflow a narrow card even if the track itself shrinks; inspect wrapping and minimum width separately.

</details>

## Exercise 3 · Explain the winning color and size — Modify

**Objective:** understand theme tokens and cascade before proposing a permanent style change.
Prerequisites: light/dark toggle and the Computed panel.
Sources: root token blocks, `.card`, `.metric small`, `theme-init.js` and theme listeners.

1. Select a card, record its computed background and follow `var(--surface)` to the root value.
2. Toggle theme with the real control. Compare the root `data-theme`, background and text without modifying the database.
3. Reload and observe persistence. Do not clear all site storage, which may affect sessions.
4. On a dashboard metric, predict the label font size from the first matching rule; then inspect the final winning declaration.
5. Temporarily change one token in DevTools and list which components change. Identify an intentional hardcoded photo/banner color that does not.

**Expected:** token consumers change with the root attribute; saved preference is the `sportshub-backup-theme` value.
The late `.metric small` rule wins over earlier equal-specificity font-size rules.
The default dark palette can work before any preference has been saved.
**Evidence:** one computed property with its source, plus a prediction that was confirmed or corrected.
**Restore:** reload temporary overrides and use the toggle to return to your preferred theme.

<details><summary>Hint and answer direction</summary>

The media query is not extra specificity.
A theme token is a named value, not an overlay that automatically recolors every image or hardcoded declaration.
Storage failures should leave the current page usable; do not claim that scenario passed unless you actually tested it.

</details>

## Exercise 4 · Keyboard navigation is a user journey — Modify → Engineer

**Objective:** evaluate focus and names rather than just the screenshot.
Prerequisites: Tab, Shift+Tab, Enter and Escape; a disposable local login for portal checks.
Sources: shell, menu/theme listeners and focus/hidden/drawer CSS.

1. Start with a public page and navigate using the keyboard. Activate Skip to content and check the main target.
2. Activate theme with the keyboard and inspect its accessible name and pressed state.
3. At 390px while signed in, open the drawer, wait for visibility/focus, then Tab and Shift+Tab through its boundaries.
4. Press Escape; check that the drawer closes, scrolling returns and focus returns to the toggle.
5. Open again and resize to 801px. Check that modal attributes/body scroll lock are cleared.
6. Open the profile disclosure and close it with Escape. Distinguish native details behavior from sidebar-dialog behavior.

**Expected:** focus is perceivable, the drawer receives focus after its visible paint, and close/desktop transitions do not leave a modal trap.
The current implementation's simple focusable selector and lack of inert background remain review topics, not certification.
**Evidence:** PASS/FAIL/NOT RUN per interaction, viewport and a precise reproduction for any problem.
**Restore:** close the menu and return to your original viewport; leave other accounts/data untouched.

<details><summary>Hint and answer direction</summary>

Do not inspect visibility immediately after only the ARIA attribute changes; CSS transitions can still be settling.
Focus should be tested as `document.activeElement`, not guessed from the last clicked area.
Never dump session storage, cookies or a whole form to debug focus.

</details>

## Exercise 5 · Trace a form without submitting a mutation — Engineer

**Objective:** explain native and enhanced paths while preserving data.
Prerequisites: the form lesson and a readable native join form.
Sources: `action_button()`, `handle_action()`, `perform_action()`, `reserve()` and JS `endpoint()`/submit handler.

1. Inspect the form markup and list **field names only**: action, id and csrf. Do not copy token values.
2. Predict the native method/destination and the REST method/path from the current code.
3. Read the point where JS decides whether to call `preventDefault()`. Explain what happens for an unmapped action.
4. Follow routing-only fields being removed from JSON and CSRF being added to headers.
5. Read `response.ok`, `textContent`, recovery acknowledgment, redirect validation and `finally`.
6. Use a public GET club search to observe an actual safe network read. Do not press Apply simply to manufacture a Network screenshot.

**Expected:** GET filtering changes the URL/read result; a join mutation would still require server session/CSRF/validation and cannot be authorized by the button.
Native success uses 303 navigation; REST returns JSON and may supply redirect metadata.
A lost response does not prove a write failed, so automatic replay is unsafe.
**Evidence:** a plain-text call chain, names/method/path and the server check for one forged-input scenario; no credentials or request-body secrets.
**Restore:** no write was needed. Clear only your search through navigation if desired.
For a later end-to-end mutation test, agree a disposable account/database scenario with the domain/backend owners first.

<details><summary>Hint and answer direction</summary>

The user ID comes from the authenticated server session; hidden IDs identify targets, not trustworthy authority.
A POST form's omitted action URL submits to the current route.
Client conversion of numeric fields is convenience; the API must still reject invalid fields/types.

</details>

## Exercise 6 · Turn evidence into one bounded human contribution — Engineer

**Objective:** make a change you can personally explain, not claim the baseline as new work.
Prerequisites: completed inspection exercises, agreed scope and the [contribution workflow](../../docs/build-order-and-contributions.md).
A can investigate a Results table caption/header-association improvement; B can investigate feedback focus after failed REST submission.
These are proposed tasks, not implementation performed by this guide; verify that the need still exists on your current branch.

Write a hypothesis, exact affected branch/selectors/functions, current symptom, expected improvement and review owner before editing.
Make the smallest useful patch manually, preserve server protections and native behavior, and compare the before/after user experience.
A changes markup with B/domain review; B changes shared interactions with A and affected API/domain review.
If no issue is reproduced, record the substantive investigation and select another agreed task honestly.

**Expected evidence:** your actual diff, explanation, tests/observations, limitations, real reviewer feedback and assistance disclosure.
**Restore:** retain an agreed finished patch for review; remove only your own temporary experiment lines if abandoned.
Do not rewrite Git history, invent reviews or split one cosmetic change into fake authorship evidence.
If working alone, use a second deliberate review pass after a break and state honestly that independent peer review is still pending.

## Manual checks after a real frontend patch

Record each as PASS, FAIL or NOT RUN; this table is not prefilled execution evidence.

| Check | What would show a problem? |
|---|---|
| Public + affected role page at 1440px/390px | Missing controls, clipped text or document-wide overflow |
| 1051/1050, 801/800, 501/500 transitions | CSS and menu behavior disagree or a control remains inaccessible |
| Both themes, including hover/error/disabled states | Unreadable foreground/background, lost state distinction |
| Keyboard/skip/profile/drawer | Invisible focus, unreachable actions or focus left behind on close |
| 200% zoom and long synthetic text | Text overlaps, actions disappear or content cannot be reached |
| Reduced-motion and print preview | Unexpected drawer motion or hidden/clipped report data |
| Native GET filtering; relevant native/REST mutation only in agreed disposable scenario | Paths diverge in domain outcome, validation or permission behavior |
| Asset/console inspection | Missing local CSS/JS/image, uncaught script error or unintended third-party dependency |

For contrast, inspect actual computed color pairs in both themes with a contrast tool.
WCAG AA targets include 4.5:1 for normal text and 3:1 for large text; non-text controls/focus have separate requirements.
One passing pair or an automated audit does not prove the whole interface accessible.
Use [W3C contrast guidance](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html) and [W3C Easy Checks](https://www.w3.org/WAI/test-evaluate/preliminary/).
The application uses system-font fallbacks; do not add a remote CDN/framework just to perform a CSS lesson.

## A practical DevTools debugging order

1. **Elements:** is the expected node actually present, with correct role/page context and named controls?
2. **Styles/Computed:** which selector wins, what is inherited and which media query is active?
3. **Layout/box diagram:** what are content width, padding, gap, minimum size and overflow?
4. **Accessibility tree:** what name, role and state does the user receive?
5. **Event breakpoint:** which handler runs, and are optional nodes missing? Inspect nonsensitive selectors/state only.
6. **Network:** inspect method/path/status for a safe read first. Never share token/cookie/password headers or recovery responses.

Do not treat every issue as CSS: an absent row may be PHP/data state, a 403 may be authorization/Origin/CSRF, and a 404 asset may be document-root/routing configuration.
Keep the public webroot `public/`; do not expose `.env` or private source to make a path error disappear.

## Self-check rubric

Score yourself 0 = not yet, 1 = can explain with the guide, 2 = can demonstrate and explain independently.
This is a learning aid, not the module's grading rubric or proof of authorship.

| Area | Explain or demonstrate |
|---|---|
| HTML | Locate an actual heading/card/form and justify its semantics |
| CSS | Explain box sizing, one winning declaration and Grid versus Flex |
| Responsive/theme | Predict one breakpoint and trace a light/dark token consumer |
| JavaScript | Trace menu/theme state and explain optional chaining/events |
| PHP | Trace a loop/value through escaping into emitted HTML |
| Forms | Explain named controls, native path, optional Fetch and error handling |
| Security boundary | Name checks only the server can enforce and distinguish CSRF from roles |
| Evidence | Show your prediction, observation, actual patch/test/review and honest limitations |

If any area is 0, revisit one matching exercise instead of restarting the entire course.
If every area is 1, repeat one component without the guide and explain your reasoning aloud.
A useful final explanation is: **“This was already present; this is what I inspected or changed; this is what I observed; this is what I still need to verify.”**
