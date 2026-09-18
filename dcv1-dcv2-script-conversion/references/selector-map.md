# DOM selector overview: DCv1 vs. DCv2

A general selector overview across question types. Use this as a first lookup before spending time
deriving selectors from CSS or asking for a copy of the markup - but it doesn't replace a real
markup copy when you're converting a specific question type. Some types have gotten an entirely
new structure in DCv2, not just new class names on the same structure (see "Structural changes" in
`SKILL.md`) - always confirm the selectors below against a real page before relying on them
blindly.

| Category | Purpose | DCv1 class/selector | DCv2 class/selector |
|---|---|---|---|
| Question container | Radio question | `.rsSingle` | `.single-question` |
| Question container | Multi question | `.rsMulti` | `.multi-question` |
| Question container | Single grid | `.rsSingleGrid` | `.single-grid-question` |
| Question container | Multi grid | `.rsMultiGrid` | `.multi-grid-question` |
| Question container | Numeric | `.rsNumeric` | `.numeric-input-question` |
| Question container | Outer wrapper | `.cTABLEContainQues` | `.question-container` |
| Question text | Text wrapper | `.cQuestionText` | `.question-texts` |
| Question text | Main text | `.cReg.reg` (inside `.cQuestionText`) | `.qText-Regular` |
| Question text | Instruction text | `.cDo` (inside `.cQuestionText`) | `.qText-Instruction` |
| Question text | Supplementary text | `.cSupp` (inside `.cQuestionText`) | `.qText-Supplementary` |
| Answer text | Answer label | `.cRowText` (inside `label`) | `.ansText-Regular` |
| Answer text | Grid row heading | `.cRowText` (inside `th`) | `.statementText-Regular` |
| Radio/multi input | Radio input | `.cRadio` | `.cRadio` (unchanged) |
| Radio/multi input | Checkbox input | `.cCheck` | `.cCheck` (unchanged) |
| Radio/multi input | Exclusive answer (radio/multi) | `.cRadio` with `onclick="clearAll()"` | `.answer-texts.exclusive` > parent `.cRadio` |
| Radio/multi input | Answer label, clickable target | `.cCellRowText` > `label` | `.answer-container` > `label` |
| Grid input | Grid radio input | `.cRadio` inside `.cCell` | `.cRadio.input-grid` |
| Grid input | Grid checkbox input | `.cCheck` inside `.cCell` | `.cCheck.input-grid` |
| Grid carousel | Answer button | `.rsBtn` (`div`) | `.answer-button.answer-button-{n}` (`button`) |
| Grid carousel | Exclusive carousel button | `.rsBtn.exclusive` | `.answer-button.exclusive-answer-button` |
| Open text | Open-text field | `.cTextInput` (`input[type=text]`) | `.open-end-input` (`textarea`) |
| Open text | Open-text answer row | `.rsRowOpen` (on `tr`) | `.open-end-question` (on `.answer-container`) |
| Numeric | Numeric input field | `.cFInput` | `.numeric-decimal-input` |
| Numeric | Single numeric answer row | `.rsNumRow` (on `tr`) | `.answer-container.decimal` (on `li`) |
| Navigation | Next button | `.buttonNext` / `#btnNext` | `.main-next-button` |
| Navigation | Back button | `.buttonPrevious` / `#btnPrevious` | `.main-back-button` |
| Piping | Reference to a prior answer | `.cRef` | `.cRef` (unchanged) |

## Notes

- `.cRadio`/`.cCheck` are confirmed unchanged for simple selection questions and grids - these
  class names appear to be deliberately kept as scripting hooks even where the rest of the markup
  is new. Numeric questions with an exclusive answer are the exception: there, NO radio/checkbox
  exists in DCv2 at all, every row is `input[type=number]` (see `code-examples.md`).
- `rsQno` is no longer a valid DOM id in DCv2 (it can contain dots). Locate the question via
  `document.getElementById('question-' + rsSubqIndex).parentElement` instead.
- A hidden direction field (`#rs_dir`, forward/back navigation) is **confirmed absent in DCv2**
  (verified by testing) - code that gates on it stops validating completely silently, since the
  condition is always false. If you need to know what it actually represented in DCv1 (e.g. to
  find an equivalent signal), some starting points: check the DCv1 page's full HTML source (not
  just the per-question markup fragment - it's likely a plain hidden `<input>` near the nav
  buttons, outside the question itself), search any inline platform `<script>` blocks for the
  string `rs_dir` (it's probably set right before form submission, inside the Next/Previous button
  handlers), or watch the Network tab while clicking Next vs. Previous to see if a form field with
  that name is submitted with different values depending on direction.
- Additional question types exist beyond the ones mapped above (grid single/multiple, an open-text
  type, and a rarely-used time/hours-and-minutes type). Selectors for these aren't mapped yet -
  ask for a markup sample before scripting against them. Routing a respondent to a specific point
  in the survey based on a condition ("goto") and other no-code survey logic (filters, Answer
  Control, parameter-driven answers) are out of scope for this skill - see the
  `walr-filters-and-answer-control` skill for that.
