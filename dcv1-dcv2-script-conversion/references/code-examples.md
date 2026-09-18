# Code examples: DCv1 vs. DCv2

Ready-made, commented examples for common needs. Use them as a starting point, not necessarily as
drop-in code - always confirm selectors against the actual page first (see `selector-map.md`).

## Setting (auto-punching) answers programmatically

### Advance automatically (DCv1 and DCv2)

```js
function pageReady(){
    document.querySelector('.main-next-button').click();
}
```

The DCv1 version of the same thing with jQuery: `$('.main-next-button').click()` (or `#btnNext` in
older setups - see `selector-map.md`).

### Setting a numeric answer (DCv2)

```js
function pageReady(){
    const input = document.querySelector('.numeric-decimal-input');
    input.value = 5;
    input.dispatchEvent(new Event('input', { bubbles: true }));
}
```

With `$` (DcQuery): `$(".numeric-decimal-input").val(5).trigger("input");` - note that `.trigger()`
is mandatory here, see `jquery-vs-dcquery.md`.

### Setting a radio answer (DCv1, jQuery)

```js
function pageReady(){
    $(".cRadio").eq(0).prop("checked", true).trigger("change");
}
```

### Setting an open-text answer and checking its checkbox (DCv1, jQuery)

```js
function pageReady(){
    $(".open-end-input").val("Test").trigger("input");
    $(".cCheck").eq(0).prop("checked", true).trigger("change");
}
```

### Setting a multi-select answer (DCv1, jQuery)

```js
function pageReady(){
    $(".cCheck").eq(0).prop("checked", true).trigger("change");
}
```

## Extracting age from date of birth (DCv2)

```js
function pageReady() {
  // Get the date-of-birth string from a hidden reference element
  var dobEl = document.querySelector(".cRef");
  var dobStr = dobEl ? dobEl.textContent.trim() : "";
  if (!dobStr) return;

  // Expecting dd/mm/yyyy
  var parts = dobStr.split("/");
  if (parts.length !== 3) return;

  var day   = parseInt(parts[0], 10);
  var month = parseInt(parts[1], 10) - 1; // JS months are 0-based
  var year  = parseInt(parts[2], 10);
  if (isNaN(day) || isNaN(month) || isNaN(year)) return;

  var dob   = new Date(year, month, day);
  var today = new Date();

  var age       = today.getFullYear() - dob.getFullYear();
  var monthDiff = today.getMonth()    - dob.getMonth();
  var dayDiff   = today.getDate()     - dob.getDate();
  if (monthDiff < 0 || (monthDiff === 0 && dayDiff < 0)) age--;

  if (age < 0 || age > 120) return; // sanity check

  var input = document.querySelector(".numeric-decimal-input");
  input.value = age;
  input.dispatchEvent(new Event("input", { bubbles: true }));

  document.querySelector(".main-next-button").click();
  document.querySelector(".main-back-button").style.display = "none";
}
```

## Capturing IP address (DCv1, jQuery)

```js
async function getIPAddress() {
  const response = await fetch('https://api.ipify.org?format=json');
  const data = await response.json();
  return data.ip;
}

function pageReady() {
  $('.question-container').hide();

  getIPAddress().then(
    function (value) {
      $('.open-end-input').val(value).trigger('input');
      $('.main-next-button').click();
    },
    function (error) {
      $('.open-end-input').val('Error searching for IP Address').trigger('input');
      $('.main-next-button').click();
    });
}
```

`$.ajax` doesn't exist in DCv2 - `fetch()` works the same in both versions, so the actual IP lookup
needs no changes when porting, only the `.val()`/`.trigger()` part if you switch to vanilla JS.

## Making answers mutually exclusive without a radio/checkbox input (vanilla JS)

Make row 1 and 3 mutually exclusive, and row 2 and 4 mutually exclusive:

```js
function pageReady(){
  const pairs = [
    ["1", "3"],
    ["2", "4"]
  ];

  const conflictMap = {};
  pairs.forEach(([a, b]) => {
    conflictMap[a] = b;
    conflictMap[b] = a;
  });

  document.querySelectorAll(".cCheck").forEach(chk => {
    chk.addEventListener("change", function () {
      if (this.checked) {
        const conflictVal = conflictMap[this.value];
        if (conflictVal) {
          const conflictChk = document.querySelector(`.cCheck[value="${conflictVal}"]`);
          if (conflictChk) conflictChk.checked = false;
        }
      }
    });
  });
}
```

## Numeric question with an exclusive answer (DCv2, vanilla JS)

**Check first whether the question builder already solves this natively** (mark the row as
exclusive there) - see `key-differences.md`. Use the script below only if the built-in feature
doesn't cover the need, or for DCv1 (which still requires a script for this).

Important structural difference: in DCv1 this was two linked sub-questions (one column of number
fields, one column of radio buttons, linked row by row). In DCv2 it's one flat list where every
row - including the exclusive one - is identical markup (`input[type=number]`, no separate radio/
checkbox). The code therefore has to work out for itself which row is the exclusive one, since the
markup no longer distinguishes them:

```js
const SEL = {
  container: '.numeric-input-question-answers-container',
  row: '.answer-container',
  input: '.numeric-decimal-input',
};

function splitRows(container) {
  const rows = Array.prototype.slice.call(container.querySelectorAll(SEL.row));
  // No "exclusive" class is guaranteed in the markup - if no row is marked, the last row in the
  // list is used as the exclusive one (common pattern: the options first, "Don't know" last).
  let exclusive = rows.filter(r => r.classList.contains('exclusive') || r.querySelector('.exclusive'));
  if (exclusive.length === 0 && rows.length > 1) exclusive = [rows[rows.length - 1]];
  const normal = rows.filter(r => exclusive.indexOf(r) === -1);
  return { exclusive, normal };
}

function pageReady() {
  document.querySelectorAll(SEL.container).forEach(function (container) {
    const { exclusive, normal } = splitRows(container);

    normal.forEach(function (row) {
      const input = row.querySelector(SEL.input);
      if (!input) return;
      input.addEventListener('input', function () {
        if (input.value.length > 0) {
          exclusive.forEach(r => { const i = r.querySelector(SEL.input); if (i) i.value = ''; });
        }
      });
    });

    exclusive.forEach(function (row) {
      const input = row.querySelector(SEL.input);
      if (!input) return;
      input.addEventListener('input', function () {
        if (input.value.length > 0) {
          normal.forEach(r => { const i = r.querySelector(SEL.input); if (i) i.value = ''; });
        }
      });
    });
  });
}

function validate() {
  // validate() replaces the platform's built-in required-answer check (see SKILL.md point 1) -
  // this function now enforces on its own that the question has been answered, the platform no
  // longer does it alone.
  let allAnswered = true;

  document.querySelectorAll(SEL.container).forEach(function (container) {
    const { exclusive, normal } = splitRows(container);
    const exclusiveHasValue = exclusive.some(r => {
      const i = r.querySelector(SEL.input);
      return i && i.value.length > 0;
    });
    if (exclusiveHasValue) return; // the exclusive answer covers the whole question

    normal.forEach(function (row) {
      const input = row.querySelector(SEL.input);
      if (!input || input.value.length === 0) {
        allAnswered = false;
        row.classList.add('validation-error');
      } else {
        row.classList.remove('validation-error');
      }
    });
  });

  return allAnswered;
}
```

## Different answer for mobile vs. desktop (DCv2)

```js
function pageReady() {
  document.querySelector('.question-container').style.display = 'none';

  const isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|Windows Phone/i.test(navigator.userAgent);
  const radios = document.querySelectorAll('.cRadio');
  const radio = isMobile ? radios[1] : radios[0];

  radio.checked = true;
  radio.dispatchEvent(new Event('change', { bubbles: true }));
  document.querySelector('.main-next-button').click();
}
```

## Shared helper-function library at the survey level

DCv2 has no automatic "run on every page load" hook the way some DCv1 setups did with
`globalPageReady()` (see `SKILL.md` point 3). Load one script at the survey level - Build ->
Settings -> Advanced settings -> "Extra JavaScript reference" - that defines a reusable library,
then call into it from each question's own `pageReady()`/`validate()`. Give the library object a
unique, unlikely-to-collide name (not `pageReady`/`validate` themselves): if this survey ever loads
a second script via the same field that happens to define a global with the same name, only one of
the two definitions survives (see `SKILL.md` point 3).

```js
// Loaded once at the survey level - defines the library, runs nothing on its own.
window.SurveyLib = {
  isValidEmail: function (value) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value);
  },

  waitFor: function (selector, root, timeoutMs) {
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
};
```

```js
// In an individual question's script:
function pageReady() {
  SurveyLib.waitFor('.open-end-input').then(function (input) {
    input.addEventListener('blur', function () {
      if (!SurveyLib.isValidEmail(input.value)) {
        input.style.border = '2px solid red';
      }
    });
  });
}
```

The benefit: validation rules and DOM helpers that recur across many questions are written and
maintained in one place. The downside compared to the old global hook: the library doesn't run
itself - each question has to call into it via its own `pageReady()`/`validate()`.
