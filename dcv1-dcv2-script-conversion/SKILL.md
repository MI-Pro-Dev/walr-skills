---
name: dcv1-dcv2-script-conversion
description: Convert question-level scripts (JavaScript/CSS running in pageReady()/validate() on a single question) between Walr's old page model (DCv1) and new Svelte-based page model (DCv2). Use when someone asks to port an existing DCv1 script to DCv2 (or vice versa), write a new question script for DCv2, explain why a DCv1 script doesn't work unchanged in DCv2, or decide whether something actually needs a script at all.
---

# Converting question scripts between DCv1 and DCv2

Walr questions can have their own JavaScript and CSS attached directly to the question. The
JavaScript is invoked by the platform itself through two functions:

- **`pageReady()`** - runs when the question loads.
- **`validate()`** - runs when the respondent tries to move on to the next question. If the
  function returns `false`, navigation is blocked; if it returns `true` (or doesn't exist),
  the respondent proceeds.

These two function names are the same in DCv1 and DCv2. What changes between versions is the DOM
structure the script reads/writes against, and which `$` library is loaded on the page. This file
covers what you need to know before writing or porting such a script - detailed tables and code
examples live in `references/`.

**How to use this skill:** the goal is for the person you're helping to be able to reason about
their next question themselves, not to hand them a corrected script and leave it at that. When you
fix or port something, say WHY it broke or WHY the new version is needed (which platform behavior
changed, which selector moved) alongside the fix - not just the corrected code on its own.

## Critical things to know before writing or porting anything

### 1. `validate()` replaces the built-in validation, it doesn't supplement it

Defining `validate()` on a question TAKES OVER from the platform's own built-in validation (e.g.
"question is required") - the built-in check no longer runs alongside it. If the question should
still be required AND have an extra rule, both must be coded explicitly inside `validate()`. Always
test by trying to advance without answering at all, not just with a valid or invalid answer - a
`validate()` that silently always returns `true` is an easy bug to miss because the page looks
completely normal until someone actually submits an empty answer.

### 2. `pageReady()` can run before the answer DOM actually exists, in DCv2

In DCv1, the question's markup was already fully present in the page by the time `pageReady()`
ran. In DCv2, `pageReady()` can fire before the question's answer elements have finished rendering.
Code that immediately does `document.querySelector(...)` on an answer element and expects to find
something can therefore get `null`/an empty result - even though the exact same code worked fine
in DCv1. Wait until the elements actually exist (see "Waiting for elements" in
`references/jquery-vs-dcquery.md`) instead of assuming `pageReady()` means "the DOM is ready".

### 3. Survey-level script loading, and the "same function name, only one runs" trap

JavaScript can be loaded at the survey level in Build -> Settings (the gear icon in the left-hand
navigation) -> Advanced settings, in the "Extra JavaScript reference" field. It accepts a script
from your own Walr file library or an external URL (e.g. a file hosted on GitHub), and multiple
references can be combined by separating them with a pipe character (`|`). This mechanism exists
for both DCv1 and DCv2 surveys.

**If more than one of the loaded scripts defines a global function with the same name, only ONE of
them actually runs - not all of them.** This is a real, recurring support issue: a survey
accumulates several loaded scripts over time that each define their own `pageReady()` (or, on
DCv1, `globalPageReady()`), and only part of the intended logic ends up executing. It isn't fully
confirmed whether the FIRST or the LAST loaded definition wins - it's likely the last, matching how
a plain `<script>` tag silently overwrites an earlier global function declaration of the same name
- but don't rely on that ordering without testing it on the actual survey.

`globalPageReady()` itself is a DCv1-specific convention: some DCv1 setups depend on it being
called automatically once per page load when defined in a script loaded this way. **DCv2 has no
equivalent automatically-called hook** - a script loaded via "Extra JavaScript reference" on a
DCv2 survey only DEFINES functions, it doesn't get any of them called for you automatically. The
only thing DCv2 calls automatically is each question's own `pageReady()`/`validate()`.

**Practical guidance:**
- Never define the same hook name (`pageReady()`, `validate()`, or the DCv1-only
  `globalPageReady()`) in more than one script loaded via this field - combine the logic into a
  single script instead.
- If you need several independently-maintained loaded scripts, give each one's own logic a
  distinct function name, and call all of them explicitly from the one hook that's actually
  invoked (a question's `pageReady()`/`validate()` in DCv2).
- Keep a personal library of reusable snippets in your Walr file library or on GitHub, and load
  only the specific ones a given survey actually needs, rather than always loading one large
  combined file - see `references/code-examples.md` for a worked example of defining such a
  library and calling into it from a question's `pageReady()`.

### 4. Check whether the functionality actually needs a script in DCv2

Quite a few things that required JavaScript in DCv1 are now built-in settings in DCv2. **Never
write a new script for something that's already solved natively** - suggest the setting that
covers the need instead. Full list in `references/key-differences.md`; the clearest example:

> An exclusive answer in a numeric question (e.g. a "Don't know" row that clears the other
> fields) required JavaScript in DCv1. In DCv2 you mark the row as exclusive directly in the
> question builder - no script needed.

If someone asks for a solution to something listed in that table, point out the built-in setting
BEFORE you start writing or porting code for it - even though it could still be solved with
JavaScript in DCv2 too.

Beyond that table, a large amount of question/section logic (show/hide, auto-answering based on a
parameter, routing a respondent to a specific point in the survey) is handled natively through
filters, Answer Control expressions, and parameter-driven answers - none of which need a script at
all. See the separate `walr-filters-and-answer-control` skill before writing a script for that kind
of need.

## DOM selectors: what has changed

Full table in `references/selector-map.md`. The most important ones to remember right away:

| Purpose | DCv1 | DCv2 |
|---|---|---|
| Locating the question's DOM element | `document.getElementById(rsQno)` / `"#" + rsQno` | `rsQno` is NOT a valid DOM id in DCv2 and can contain dots. Use `document.getElementById('question-' + rsSubqIndex).parentElement` |
| Next button | `#btnNext` / `.buttonNext` | `.main-next-button` |
| Back button | `#btnPrevious` / `.buttonPrevious` | `.main-back-button` |
| Inserting custom HTML right after the question | an `.after(html)`-style call | `questionEl.querySelector('.custom-question-canvas').insertAdjacentHTML('beforeend', html)` |
| Radio input | `.cRadio` | `.cRadio` (unchanged) |
| Checkbox input | `.cCheck` | `.cCheck` (unchanged) |
| Numeric input field | `.cFInput` | `.numeric-decimal-input` |
| Open-text field | `.cTextInput` (`input[type=text]`) | `.open-end-input` (`textarea`) |
| A single answer row | `.rsRow` (`<tr>`) | `.answer-container` |
| Hidden direction field (forward/back navigation) | `#rs_dir` | **Confirmed absent in DCv2** (verified by testing). Code that gates validation with `document.getElementById('rs_dir').value == '1'` stops validating completely silently, because the condition is always false. Remove any dependency on `#rs_dir` when porting to DCv2 - treat it as a required fix, not just a risk to flag. |

`.cRadio`/`.cCheck` are confirmed unchanged for simple selection questions - but don't assume the
same for every type. Numeric questions with an exclusive answer, for example, have NO radio/
checkbox in DCv2 at all (see below).

### Structural changes, not just renamed classes

Some question types aren't just "same structure with new class names" - the structure itself has
changed:

- **Numeric question with an exclusive answer**: in DCv1 this was two linked sub-questions (one
  column of number fields, one column of radio buttons, linked row by row). In DCv2 it's ONE flat
  list where every row - including the exclusive one - is identical markup
  (`input[type=number]`, no separate radio/checkbox). See `references/code-examples.md` for how
  this affects the code.
- Grid types likely have similar structural changes - confirm with a real markup copy (copied
  from the browser's developer tools) before assuming selectors derived from CSS alone.

**Rule of thumb:** without a real HTML copy of the question in DCv2, say clearly that the
selectors below are assumed and must be confirmed before production use - ask for a copy of the
markup from the browser's developer tools.

## jQuery -> DcQuery

DCv1 pages load real jQuery. DCv2 pages instead load an internal library with a `$` API that
resembles jQuery but isn't identical. `$(...)` syntax still works in DCv2, then - it's not an
absence of jQuery, just a library with gaps compared to real jQuery. Full overview in
`references/jquery-vs-dcquery.md`. Most important when porting:

| | jQuery (DCv1) | DcQuery (DCv2) |
|---|---|---|
| Setting a value programmatically | `.val('x')` alone is often enough | `.val('x')` alone triggers NO event. Always chain `.trigger('input')` (text/numeric) or `.trigger('change')` (radio/checkbox) after `.val()` |
| AJAX | `$.ajax()`, `$.get()`, `$.post()` | Not implemented - use `fetch()` |
| Mouse/hover shortcuts | `.hover()`, `.mouseenter()`, `.one()` etc. | Not implemented - use `.on('eventname', handler)` |
| Waiting for elements that don't exist yet | Usually unnecessary | `.waitFor(selector)` (promise-based) - useful precisely because `pageReady()` can run before the answer DOM exists, see point 2 above |

## Write vanilla JavaScript for new DCv2 scripts

A `$` library is available in DCv2, but vanilla JavaScript is more robust and easier to debug
regardless of which library is actually loaded. Common replacements:

```js
function qsa(root, selector) { return root ? Array.prototype.slice.call(root.querySelectorAll(selector)) : []; }
function qs(root, selector) { return root ? root.querySelector(selector) : null; }
// .addClass('a')                -> el.classList.add('a')
// .removeClass('a')             -> el.classList.remove('a')
// .val()                        -> el.value
// .val('x').trigger('input')    -> el.value = 'x'; el.dispatchEvent(new Event('input', { bubbles: true }))
// .on('change', fn)             -> el.addEventListener('change', fn)
// .html(x)                      -> el.innerHTML = x
// .append(x)                    -> el.insertAdjacentHTML('beforeend', x)
```

## Other known differences to check

- **CSS breakpoints**: DCv2 often uses `max-width: 1023px` / `767px`, DCv1 often used `980px`.
  Don't reuse a DCv1 breakpoint without checking the actual DCv2 page.
- **Elements the platform itself owns can be redrawn without warning.** Never set your own
  styling as an inline style on an element the platform controls (typically
  `.answer-container`) - it can be silently overwritten on the next update. Use an injected
  `<style>` block targeting your own class/id instead, with `!important` where you need to beat a
  specific DCv2 base style - this is one of the few cases where reaching for `!important`
  immediately is justified rather than a shortcut; see the `walr-css-and-themes` skill for CSS
  delivery options that don't need a script at all, and for why `!important` should stay scoped
  narrowly even when it is justified.
- **Your own markers on such elements should be `data-*` attributes, not classes.** The platform
  can overwrite the entire `class` attribute on an element it controls (e.g. when an answer is
  selected/deselected), and a class you've added there then silently disappears. A `data-*`
  attribute the platform doesn't know about survives such updates. Always test that your styling
  survives the respondent actually clicking/selecting an answer - not just that it looks right
  before anyone has answered.
- **Make the script idempotent.** Remove your own previous elements/classes/injected `<style>`
  (identified by a stable id) BEFORE building again in `pageReady()` - preview and editing can
  trigger the script multiple times on the same page.
- **Multiple questions on the same page**: check
  `document.querySelectorAll('[id^="question-"]').length` before assuming a script only needs to
  deal with one question.

## Checklist before delivering a conversion

1. Is `validate()` used? Confirm it re-implements a required-answer check wherever one needs to be
   kept (point 1).
2. Does `pageReady()` read answer elements immediately? Add waiting logic if DCv2 is the target
   (point 2).
3. Is any of the functionality already built into DCv2 (`references/key-differences.md`)? Suggest
   that instead of writing/porting a script.
4. Is `rsQno` used as a DOM id? Switch to the `question-<rsSubqIndex>` pattern.
5. Is `#btnNext`/`#btnPrevious` used? Switch to `.main-next-button`/`.main-back-button`.
6. Is `#rs_dir` used to gate validation? Flag this explicitly and ask for testing.
7. Is `.val(x)` used without a `.trigger(...)` afterward? Add an explicit trigger.
8. Is `$.ajax`/a mouse shortcut used? Rewrite to `fetch()`/`.on()`.
9. Could the question type have a structural change (numeric + exclusive, grid)? Ask for a real
   markup copy if you don't already have one.
10. Are CSS breakpoints checked against the actual DCv2 page, not assumed from DCv1?

## Reference files

- `references/selector-map.md` - full DOM selector table across question types.
- `references/key-differences.md` - full list of platform functionality that has changed between
  versions, beyond selectors. Check this BEFORE writing a new script.
- `references/jquery-vs-dcquery.md` - detailed jQuery/DcQuery comparison, incl. waiting for
  elements that don't exist yet.
- `references/code-examples.md` - complete before/after code examples: setting answers
  programmatically (numeric/radio/open-text/multi), extracting age from date of birth, capturing
  IP address, exclusivity logic (incl. the numeric example mentioned in point 4), device-based
  logic, and a ready-made setup for a shared helper-function library at the survey level (point 3).
