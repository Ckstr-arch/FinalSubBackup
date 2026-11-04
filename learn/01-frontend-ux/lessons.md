# Lessons · build understanding one layer at a time

[Track home](README.md) · [Source walkthrough](source-walkthrough.md) · [Exercises](exercises.md)

Start with one club card or the navigation.
You are learning and improving an existing application, so the order below does not mean deleting working code and recreating it.
At each step ask: **what is the input, what transforms it, and what should a user observe?**

## 1. Understand the browser's document

HTML describes structure and meaning.
The browser parses HTML into the **Document Object Model (DOM)**: objects representing elements, text and their parent/child relationships.
PHP executes on the server before that HTML arrives; PHP source does not become a browser language.
View Source shows the response HTML, while DevTools Elements shows the current DOM, including changes made by JavaScript.

In SportsHub, `app/layout.php` produces `<html>`, `<head>`, `<body>`, shared navigation, `<main>` and the footer.
The `home` branch of `app/views.php` produces the hero, statistics strip, club cards and next-fixture banner.
The hero headline is literal copy; counts and club cards come from database queries.

| Element or attribute | Meaning | SportsHub example |
|---|---|---|
| `header`, `nav`, `main`, `footer` | Page regions that help users orient themselves | Workspace header, portal navigation and `main#main` |
| `section` | Related topic, normally identified by a heading | Homepage hero or feature banner |
| `article` | A unit that can make sense on its own | A club card or community post |
| `h1`, `h2`, `h3` | Heading hierarchy, not merely a font-size choice | Page title, section title, card title |
| `p`, `span`, `div` | Paragraph, inline grouping, generic grouping | Description, highlighted headline words, grid wrapper |
| `a href` | Navigation to another location | View club, Sign in, Schedule |
| `button` | An action; inside a form its default type is submit | Apply, Toggle theme, Save record |
| `label` and input `name` | Label names a control; `name` identifies the submitted field | Search clubs / `q` |
| `class`, `id`, `data-*` | Reusable styling hook, unique identity, application metadata | `.card`, `#navigation`, `data-theme` |

Using a clickable `<div>` instead of a button loses useful keyboard semantics unless you rebuild them correctly.
Using an `h3` only because its default size looks convenient can produce a confusing heading outline.
Choose meaning first, then style it.
Read [MDN's HTML introduction](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Structuring_content/Basic_HTML_syntax) and [W3C's page-structure guidance](https://www.w3.org/WAI/tutorials/page-structure/).

### Predict before inspecting

If CSS disappears, a proper button still has native button behavior and a link still navigates.
If PHP never emitted an admin control, CSS cannot reveal that missing control.
If an element exists but is `display:none`, inspect styles rather than rewriting the template.
These three failures have different causes.

## 2. Make the box model concrete

Every ordinary CSS box has content, padding, border and margin.
Padding is inside the painted box; margin separates boxes outside their borders.
SportsHub sets `box-sizing:border-box` on every element, so a declared width includes padding and border.

For an **illustrative** 240px card with 20px padding and 1px borders, the content width is `240 - 40 - 2 = 198px` under border-box.
Without border-box, declaring a 240px content width would produce a 282px outer box before margins.
The actual `.card` uses padding, border, background, shadow and `min-width:0`; it does not declare a fixed width for every card.
Grid determines the available track width.

`display:block` starts a normal block box; inline text participates in text lines.
`inline-block` combines inline participation with box sizing, as used by `.inline` forms.
`display:none` removes an element from layout/presentation; opacity zero merely makes it transparent and can leave it interactive.
Adjacent vertical block margins can collapse; Grid/Flex gaps do not behave like collapsing margins.

Use [MDN's box-model lesson](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Box_model) beside DevTools' box diagram.

## 3. Learn the cascade before adding overrides

A selector chooses elements; declarations assign property values.
When several declarations apply, the **cascade** resolves importance, specificity and source order.
Specificity is the weight of a selector: `.header-search input` is more specific than `input`.
An inherited property such as font or text color can flow from a parent; padding and display normally do not.

The generic input rule sets borders, padding and backgrounds.
The more specific `.header-search input` removes its own border and background so it fits inside the search wrapper.
That rule can win even though it appears earlier.
Equal-specificity declarations are decided by later source order when other cascade factors are equal.
The final `.metric small {font-size:10px}` overrides earlier equal-specificity metric-label rules, including responsive ones.
A media query controls whether a rule participates; it does not add selector specificity.

Do not fix every surprise with `!important`.
First inspect the winning declaration, the selector that supplied it and any inherited/custom-property value.
See [MDN's cascade explanation](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Styling_basics/Handling_conflicts).

## 4. Distinguish Flexbox from Grid

**Flexbox** lays out items along a main axis, row by default.
`justify-content` distributes along that axis; `align-items` aligns across it.
Changing `flex-direction:column` makes the main axis vertical.
`gap` spaces siblings, and `flex-wrap` allows another line instead of squeezing everything into one row.

The actual workspace header is Flex.
Its search has `flex:1` and a maximum width; actions use `margin-left:auto` and `flex-shrink:0` to remain together at the far side.
When minimum child sizes exceed available space, shrinking a gap alone may not solve overflow.
The app hides header search at 1050px and simplifies the header again at 800px.

**Grid** defines tracks across rows/columns.
`.grid.three` uses `repeat(3,minmax(0,1fr))`: three equal shares after subtracting gaps, each allowed to shrink below its intrinsic content minimum.
A 900px content area with two 20px gaps gives approximately `(900 - 40)/3 = 286.7px` per track.
`min-width:0` lets a card shrink with its track; a very long unbroken string may still require a wrapping policy.
Do not assume `width:100%` fixes every overflow.

The source uses explicit breakpoints, not `auto-fit` as its current card strategy.
`auto-fit` and `auto-fill` are useful later alternatives to study; replacing the design's layout strategy is a reviewed change.
Read [MDN Flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_flexible_box_layout/Basic_concepts_of_flexbox) and [MDN Grid](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_grid_layout/Basic_concepts_of_grid_layout).

## 5. Position things without losing their context

`static` is normal placement.
`relative` keeps a box in flow and can establish a containing block for absolutely positioned descendants.
`absolute` removes the child from normal flow; a parent containing only absolute children may need its own height.
`fixed` positions against a viewport-related containing block; `sticky` remains in flow and sticks within its scrolling context when an inset is reached.

SportsHub's `.hero-photo` is relative and has a minimum height; its absolute image fills that box with `object-fit:cover`.
Cover preserves the image ratio while cropping; contain preserves the whole image with possible empty space.
The desktop sidebar is fixed, so `.portal .app-frame` has a matching left margin.
The header is sticky at top 0.
The profile menu is relative; its dropdown is absolute below the summary.

A high `z-index` is not a universal escape hatch.
Positioned or transformed ancestors can create **stacking contexts**, groups whose descendants are painted together.
At mobile size the sidebar's layer sits above the scrim, which sits above the main header.
A dropdown inside a lower stacking context does not automatically cover an unrelated higher one.
See [MDN positioning](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/CSS_layout/Positioning).

## 6. Treat responsiveness and themes as states

The viewport is measured in CSS pixels, which need not match hardware pixels.
`px` gives CSS-pixel sizes; percentages depend on a containing dimension; `vw` depends on viewport width.
`rem` refers to root font size and `em` to a local font context; these are useful concepts even though this stylesheet mostly uses px and viewport units.
`clamp()` bounds a preferred responsive value, as with headings.

| Actual query | Important changes to investigate |
|---|---|
| Minimum 1700px | Larger main padding and performance hero |
| Maximum 1250px | Narrower sidebar and more compact header/metrics |
| Maximum 1050px | Search hidden; three-card grid becomes two; dashboard/community columns collapse |
| Maximum 800px | Sidebar becomes a drawer; portal offset disappears; hero/auth/two-column layouts stack |
| Maximum 500px | Card grid becomes one; filters and club banner stack; compact controls |
| Reduced motion | Transitions are disabled |
| Print | Navigation/buttons disappear; tables and surfaces use a report-friendly light presentation |

Test 801/800 and 501/500 as well as representative 1440px and 390px widths.
`app.js` closes mobile modal state when `matchMedia('(min-width: 801px)')` becomes true.
Changing only the CSS threshold can leave focus/scroll behavior out of step.

Dark defaults live in `:root`; `:root[data-theme="light"]` overrides tokens such as `--bg`, `--text`, `--surface` and `--accent`.
A declaration using `var(--surface)` automatically follows the chosen theme.
Hardcoded photo-banner colors are intentional exceptions that still need a contrast review.
`theme-init.js` runs before CSS so a saved light preference can apply before the first paint.
`app.js` toggles the root attribute and the `sportshub-backup-theme` preference; it does not store authentication there.

## 7. JavaScript is event-driven browser behavior

`const` prevents reassignment of a binding, not all changes to the referenced object.
`querySelector` finds the first match; `querySelectorAll` finds a collection.
`?.` skips an operation on a missing optional value, useful when the public and authenticated shells contain different nodes.
`classList` updates styling states; attributes such as `aria-expanded` describe a control's state to assistive technology.
Neither one automatically updates the other.

Clicking the menu changes expansion/classes.
For the sidebar only, JS sets dialog/modal attributes, reveals a scrim, locks page scrolling and schedules focus into navigation after the visible paint.
Tab/Shift+Tab wrap at the current first/last link or button; Escape and scrim close the drawer and return focus.
The public nav has a simpler dropdown behavior and does not become that sidebar dialog.
The profile uses native `details`/`summary`; JS adds outside-click and Escape closing.

The current focusable selector is simple and the background is not made inert.
Do not call this a complete accessibility audit merely because Tab works once.
Check keyboard paths, zoom, reduced motion, visible focus and meaningful names.
Read [MDN events](https://developer.mozilla.org/en-US/docs/Learn_web_development/Core/Scripting/Events) and [W3C keyboard guidance](https://www.w3.org/WAI/WCAG22/Understanding/keyboard.html).

## 8. Move a known component into PHP understanding

The production component already lives in PHP.
For learning, compare a small synthetic HTML sketch with the real view loop rather than replacing the app with static copies.
`foreach` repeats markup for each row; `<?= ... ?>` echoes an expression; `e()` encodes text safely for the current HTML uses.
A stored name is still untrusted text; MySQL storage does not make it safe HTML.
`rows()` and `one()` return results through PDO helpers; parameterized queries keep input values separate from SQL syntax.

Prepared SQL and escaped HTML solve different problems.
A prepared query does not make raw HTML output safe; HTML escaping does not make concatenated SQL safe.
The [source walkthrough](source-walkthrough.md) shows each boundary with actual excerpts.
Backend/database detail belongs to the corresponding learning track, while A/B must understand the display contract.

## 9. Make the native form work before enhancing it

A GET form is appropriate for reading/filtering clubs: its `page`, `q` and `sport` values appear in the query string.
A POST form requests a mutation, such as applying to a club.
The native join form includes an action name, target ID and CSRF token; PHP checks the session, permissions, validation and database state before writing.
A hidden input is editable by the user and is not a trusted source of identity or role.

With JavaScript enabled, `app.js` maps the same action to a REST route.
`FormData` collects successful named form controls, `JSON.stringify` creates the body and Fetch sends same-origin cookies plus the CSRF header.
`await` waits for a Promise; it does not freeze the whole browser.
Fetch resolves for HTTP errors too, so the code checks `response.ok` after parsing JSON.
The page shows errors using `textContent` rather than parsing them as HTML.

Native and REST paths share server business rules; their exact statuses and response formats can differ.
A native success redirects with 303; API success returns JSON and may include redirect metadata.
A network failure can occur after the server committed the write, so automatically resending it can duplicate an operation.
Check current state before a human retries.
Read [MDN Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API/Using_Fetch) and [MDN FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData).

## 10. Know where frontend authority stops

JavaScript cannot securely decide that a user is an admin, that a booking slot is free or that a membership is approved.
Users can edit the DOM, disable scripts or send requests without the page.
Client validation improves usability; PHP validation and authorization remain authoritative.
CSRF ties a mutation to its session context; it does not replace role/ownership checks.
Avoid logging FormData, cookies, tokens, passwords or recovery data during debugging.

To finish this lesson, explain one real component using **source → DOM → computed style → event → request → server rule**.
Then do one [exercise](exercises.md), rather than memorizing every CSS property before you practise.
