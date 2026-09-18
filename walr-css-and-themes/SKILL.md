---
name: walr-css-and-themes
description: Reference for styling a Walr survey without a script - editable survey themes (a project can have more than one), CSS loaded at survey/section/question level (the same pattern as JavaScript loading), and inline styling written directly into survey builder text fields (since they render as HTML). Use before writing a script just to change appearance, and whenever a fix reaches for !important.
---

# CSS and theming in Walr surveys

Walr offers several ways to change how a survey looks without touching `pageReady()`/`validate()`
at all. Reach for these before writing a script whose only job is visual.

**How to use this skill:** teach the underlying mechanism (which level to load CSS at, why a
narrowly-scoped selector is safer than `!important`), not just supply a finished stylesheet. The
aim is for the person you're helping to make their next small styling change on their own.

## Survey themes

A survey has an editable theme, and a project can have more than one theme available to switch
between. Exactly where themes are managed and how one is assigned to a survey hasn't been pinned
down yet in this skill - confirm the menu location before describing it to someone (see "Open
questions" below).

## Loading CSS at survey, section, or question level

CSS can be loaded at the same levels JavaScript can: survey level, section level, and question
level. This is presumably the same delivery mechanism documented for JavaScript in the
`dcv1-dcv2-script-conversion` skill (Build -> Settings -> Advanced settings -> "Extra JavaScript
reference", loading from your Walr file library or an external URL such as GitHub, with multiple
references separated by `|`) - but the exact field label for CSS hasn't been confirmed, so verify
it in the actual settings menu (likely named analogously, e.g. an "Extra CSS reference" field)
before pointing someone to a specific field name.

Loading CSS this way is the right tool when the same style needs to apply consistently across many
questions or the whole survey. For a one-off tweak to a single element, inline styling (below) is
usually simpler.

## Inline CSS directly in builder text

Every text field in the survey builder - question text, answer text, instructions - renders as
HTML. That means a `style` attribute can be written directly into the text, with no separate
stylesheet or script involved:

```html
<p style="display:none;">Option 1</p>
```

This hides that specific piece of text from the respondent without any CSS file or JavaScript at
all. This is worth teaching directly: for a single answer or a single question, inline styling in
the text editor is more self-contained and easier to find again later than a script or a loaded
stylesheet - reserve the loaded-stylesheet route for styling that needs to be consistent across
many elements.

## Be careful with `!important`

`!important` forces a rule to win regardless of the platform's own CSS specificity, and it is
sometimes genuinely necessary - the `dcv1-dcv2-script-conversion` skill notes a real case where
DCv2's own base stylesheet is more specific than a simple class, and no plain-specificity selector
can beat it. That said, encourage restraint rather than reaching for `!important` as a default
habit:

- **Try a more specific selector first** (an id, a `data-*` attribute, or nesting two classes)
  before adding `!important`.
- **Scope the rule as narrowly as possible** - target the specific class/id/attribute involved,
  not a broad selector - so the override doesn't leak onto elements nobody meant to touch.
- **Treat `!important` as a last resort for platform-owned elements the framework can redraw**
  (as in the DCv2 case above), not a routine way to win an ordinary styling disagreement.
- Once several rules on the same property all carry `!important`, which one wins depends only on
  source order, not on which selector is more specific or more intentional - that's a harder thing
  to reason about later than an ordinary specificity difference, so it's worth avoiding
  accumulating more of it than necessary.

## Cross-references

- For the actual DOM class names/selectors to target in CSS (which differ between DCv1 and DCv2),
  see the `dcv1-dcv2-script-conversion` skill's `references/selector-map.md`.
- For DCv1-to-DCv2 porting concerns specific to CSS (breakpoint differences, elements the DCv2
  framework can redraw and silently undo styling on), see that skill's "Other known differences to
  check" section.

## Open questions for future updates

- Exact menu location for editing a theme, and how multiple themes are managed/assigned to a
  survey.
- Exact field label and location for CSS loaded at survey/section/question level - assumed to
  mirror "Extra JavaScript reference" but not confirmed.
- Whether loading CSS at multiple levels (survey/section/question) at once has any surprising
  interaction beyond normal CSS cascade/specificity rules.
