---
name: walr-filters-and-answer-control
description: Reference for Walr's built-in, no-code survey logic - filters (section/question/statement level), Answer Control expressions (which options show and/or auto-answer), and parameter-driven answers via the sys_/sms_ Container/Question/Answer menu. Use before writing or porting any script: check whether the need is already covered by this native logic instead.
---

# Filters, Answer Control, and parameter-driven answers

A large amount of survey logic in Walr - showing/hiding sections, questions, or individual
statements; auto-answering a question from a parameter; routing a respondent to a specific point
in the survey - is handled natively, without any JavaScript. This is deliberately kept separate
from question-level scripting (`pageReady()`/`validate()`, covered in the
`dcv1-dcv2-script-conversion` skill): check here first whenever a request sounds like "show this
only if...", "skip ahead to...", or "pre-fill this from...", before reaching for a script.

**How to use this skill:** point to the specific menu/field that already solves the need, and
explain what it does, rather than only delivering a working configuration. Someone who understands
why `sys_range e` behaves differently from `sys_range c` can set up their next question alone;
someone who was just handed a working value has to come back and ask again.

This area covers two genuinely different mechanisms that are easy to conflate:

1. **Answer Control** - an expression typed directly into a question (or grid statement) that
   controls which answer options are shown and/or auto-answered.
2. **The `sys_`/`sms_` parameter menu** - a separate settings menu (at Container, Question, or
   Answer level) for reading external parameters into the survey and mapping them to specific
   answers. Confirmed to exist and to interact with Answer Control (via `sys_range`), but its full
   behavior beyond the examples below has not been fully mapped out yet - treat anything not
   explicitly confirmed here as needing verification before relying on it in production.

## Filters

Filters exist at three levels:

- **Section level**
- **Question level**
- **Statement level** (for an individual row/statement inside a grid)

A related routing feature is the **"goto" module**: it can send a respondent to a specific point
in the survey based on a filter condition. (Exact UI/syntax for filters and for "goto" has not
been detailed yet - this section will need to be expanded with concrete examples.)

## Answer Control

Answer Control is a simple expression syntax that controls which options are shown and/or how a
question gets auto-answered. Confirmed examples:

| Expression | Meaning |
|---|---|
| `*` | An answer to this question/statement is required. |
| `(1:9)` | Only answer options 1 through 9 are shown. If `sys_range c` (concealed) is set for the question, options 1-9 are automatically answered instead of just shown. If `sys_range e` (exposed) is set, it marks/selects those answers and keeps them visible - but every OTHER option gets hidden, so `sys_range e` should **not** be used for pre-filling (it doesn't just add a value to an otherwise-normal visible list, it hides the rest of the list too). |
| `(1:3;12) 4:11 try \kjonn=2` | Options 1-3 and 12 are always shown. Options 4-11 are shown only if the parameter `kjonn` equals `2` (a semicolon-separated list of codes/ranges, combined with a `try \<parameter>=<value>` condition). |
| `1 when \q2=1 2 when \q4=12 else 3` | Selects option 1 if `q2` equals 1, option 2 if `q4` equals 12, otherwise always falls back to option 3. **Rule of thumb: always pair a `when` condition with an `else` fallback** - without one, there's no defined outcome when none of the `when` conditions match. |

Answer Control can also hide an entire question and auto-fill it based on a parameter:

- For **single/multi/grid** questions, this is done through Answer Control itself (as in the
  examples above).
- For **open-text and numeric** questions, auto-filling based on a parameter is instead done by
  entering an expression/code directly in the question's own field(s) - not through Answer
  Control.

Only four expression forms are confirmed so far (`*`, a range with an optional `try` condition, and
`when ... else`). There is likely more grammar than this - expand this table as new forms are
confirmed rather than guessing at syntax not seen yet.

## The `sys_`/`sms_` parameter menu (not Answer Control)

`sys_` and `sms_` prefixed fields live in a **separate** right-hand settings menu, available at
**Container, Question, or Answer level only**. This is a distinct system from Answer Control, with
its own logic that hasn't been fully explained/documented yet - confirmed pieces:

- **`sys_range e` / `sys_range c`** - determines whether a question executes Answer Control to
  answer the question itself. `e` = exposed/visible, `c` = concealed/hidden (see the `(1:9)` row
  above for how this changes Answer Control's behavior).
- **`sms_<name>` fields** (e.g. `sms_kjonn c`) read in a same-named parameter (e.g. `kjonn=x`) -
  either from the survey link when the respondent starts, or from an uploaded sample/customer
  list. The variable name correlates between the parameter and the field name, and the same `e`
  (exposed)/`c` (concealed) distinction applies here too.
- This can be set at **field/answer level** or at **question level**. Example: the parameter
  `kjonn=1` correlates with the answer option whose code is `1` in a single-select question, at
  question level, if that question has "set answer" configured in its menu.

**Open question - needs more detail before treating this as fully understood:** the precise scope
and behavior of this menu beyond the two confirmed field types (`sys_range`, `sms_<name>`), and
whether other `sys_`-prefixed system fields exist beyond `sys_range` (the `dcv1-dcv2-script-conversion`
skill's `key-differences.md` separately notes `sys_respguid` - DCv1 only - and `\@sys_iteration`, a
loop-iteration variable - so `sys_` is at minimum a family of several distinct reserved names, not
just `sys_range`).

## Question types this logic applies to

- Selection - single and multiple choice
- Open text ("open field") - internally categorized as a multi-type question with open fields
- Numeric
- Grid - single and multiple, with per-statement filters/Answer Control
- Time (hours and minutes) - exists, but rarely used

DOM-selector detail for scripting against any of these belongs in the `dcv1-dcv2-script-conversion`
skill, not here - this skill is strictly about the no-code logic layer.

## Before writing a script

If a request sounds like "only show these options when...", "pre-fill this question with...", or
"skip the respondent ahead to...", check whether Answer Control, a filter, or the `sys_`/`sms_`
menu already covers it before reaching for `pageReady()`/`validate()`. Native logic here is
generally preferable to an equivalent script: it's configured in the question/section editor
rather than code, so it doesn't carry the DCv1-DCv2 porting concerns documented in the
`dcv1-dcv2-script-conversion` skill.

## Open questions for future updates

- Exact filter syntax/UI at section, question, and statement level.
- Full "goto" module syntax and how its filter condition is authored.
- Full grammar of Answer Control beyond the four confirmed expression forms above.
- Full behavior of the `sys_`/`sms_` Container/Question/Answer menu, and a complete list of
  reserved `sys_` names.
