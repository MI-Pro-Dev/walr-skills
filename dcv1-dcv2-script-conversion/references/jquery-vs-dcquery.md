# jQuery (DCv1) vs. DcQuery (DCv2)

DCv1 pages load real jQuery. DCv2 pages instead load an internal, lightweight library with a `$`
API that resembles jQuery's, but is NOT identical. `$(...)` syntax works fine on DCv2 pages,
then - it's not that jQuery-style code is impossible there, just that the API has gaps compared to
real jQuery.

## Main pitfalls when porting DCv1 -> DCv2

| Category | jQuery (DCv1) | DcQuery (DCv2) |
|---|---|---|
| Setting a value programmatically | `.val('x')` alone is often enough - jQuery's internal event system and many listeners reliably pick up `.trigger('change')`/`.val()` | `.trigger(event, data)` is implemented (via a native `CustomEvent`), so `.val('x').trigger('change')`/`.trigger('input')` works, the same pattern as DCv1. **BUT**: code that relied on jQuery's implicit event propagation and only called `.val('x')` WITHOUT an explicit `.trigger()`/`.change()` will fail silently here, because `.val()` alone triggers no event. Always chain `.trigger('input')` (open-text/numeric) or `.trigger('change')` (radio/checkbox) explicitly after `.val()` when porting - or use native `el.dispatchEvent(new Event(...))` if you're not going through the `$` wrapper |
| AJAX | `$.ajax()`, `$.get()`, `$.post()`, `$.getJSON()` | **Not implemented** - use native `fetch()` instead. Any DCv1 script making AJAX calls needs to be rewritten |
| Mouse/hover shortcuts | `hover()`, `one()`, `mouseenter()`, `mouseleave()`, `mousedown()`, `mouseup()`, `mousemove()`, `mouseover()`, `mouseout()`, `select()`, `resize()`, `scroll()` | **Not implemented as shortcuts** - switch to `.on('eventname', handler)` |
| Waiting for elements that don't exist yet | Usually unnecessary - the markup was already fully in the page when `pageReady()` ran | `.waitFor(selector)` (promise-based, built on `MutationObserver`), with no jQuery equivalent. Solves the problem of `pageReady()` in DCv2 potentially running before the answer DOM has finished rendering (see `SKILL.md` point 2) |

## Waiting for elements in vanilla JavaScript

Without the `$` library, the same waiting logic can be built with a `MutationObserver`:

```js
function waitFor(selector, root, timeoutMs) {
  root = root || document;
  timeoutMs = timeoutMs || 3000;
  return new Promise(function (resolve, reject) {
    var found = root.querySelector(selector);
    if (found) return resolve(found);

    var observer = new MutationObserver(function () {
      var el = root.querySelector(selector);
      if (el) {
        observer.disconnect();
        resolve(el);
      }
    });
    observer.observe(root, { childList: true, subtree: true });

    setTimeout(function () {
      observer.disconnect();
      reject(new Error('Timed out waiting for ' + selector));
    }, timeoutMs);
  });
}

function pageReady() {
  waitFor('.numeric-decimal-input').then(function (input) {
    input.addEventListener('input', function () {
      // ...
    });
  });
}
```

## Consequences when porting

- Don't assume `$(...).val(x)` alone still works when porting a DCv1 script that itself uses
  jQuery - check whether the code relied on implicit event propagation, and add an explicit
  `.trigger(...)` where needed.
- `$.ajax`/mouse shortcuts in DCv1 code MUST be rewritten, not just translated selector-for-
  selector - they simply don't exist in DcQuery.
- `.waitFor(selector)` (or the vanilla variant above) is useful for any `pageReady()` that needs to
  manipulate answer elements in DCv2, not just code that was originally written with `$`.
