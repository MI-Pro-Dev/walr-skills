# Platform differences between DCv1 and DCv2

These are FUNCTIONAL differences (what the platform itself supports natively), not DOM selectors -
see `selector-map.md` for that. Check this table BEFORE writing or porting a script: a lot of what
required JavaScript in DCv1 no longer needs code in DCv2.

| Need | DCv1 | DCv2 |
|---|---|---|
| Auto-coding answers | - | Send the answer's VALUE when auto-coding, not just sys_range |
| How a question stores an answer | One question could hold several values together, semicolon-separated | One question always holds exactly one value |
| Splitting a long answer list into columns | Required a custom script | "Display Rows" setting to split into columns. Exclusive answers (e.g. "None") and open-text add-ons always sit on their own row and don't count toward the column split. Example: 14 answers + open-text add-on + "None", split into 2 columns -> set Display Rows to 6 |
| Randomizing order within a loop | Required a custom script | Built in via section settings - but only works when the loop iterates over a list |
| Randomizing grid columns | Required a custom script | Built in, same pattern as row/statement randomization |
| Prioritizing which quota cells fill first | Required a custom script | Built-in "priority quota" - automatically fills the cells furthest from target first |
| Hiding the language switcher | Required a custom script | Its own setting in the survey settings |
| Hiding the logo | Required a custom script | Its own theme setting |
| Number of languages per survey | Supports 7-8 | Supports up to 10 |
| Theme/appearance | Mostly dependent on CSS | Own option to customize the theme in the editor |
| Survey stop | - | Now supported, linked to the dashboard page |
| Question randomization | - | Can now be set up the same way as answer randomization, without using stop rotation |
| Ranking + exclusive answer | Required a custom script | Just mark the answer as exclusive directly in the ranking question |
| Grid carousel navigation | Required a custom script for the navigation button | Available as a built-in display format |
| Dropdown with open-text add-on | Not possible | Supports both single and multi, with or without an open-text dropdown |
| Excluding a question from reporting | Didn't exist | Supports excluding from analysis/reporting, translation export, and all data exports |
| Hiding the checkbox on an open-text question | - | Hides automatically once the open-text field's line count is set to 3 or more |
| Quota | - | Can be set for up to 3000 rows |
| Answer options | - | Supports up to 1500 answer rows |
| Making an open-text answer non-mandatory | Not possible | Its own on/off toggle |
| **Exclusive answer in a numeric question** | **Required JavaScript** | **Mark the row as exclusive directly in the question builder - no script needed** |
| The "always" system variable | Treated as a regular variable | Set as a system variable, the same way as respondent ID until it's changed |
| Setting the value of an open-text answer via logic | Set via a script function in answer logic, then at the answer level, e.g. `script:echo('Test')` | Can only be set with "set answer" at the answer level, e.g. `\script:echo('Test') c` |
| Respondent ID | Can use `sys_respguid` | `sys_respguid` doesn't exist |
| Loop iteration | No fixed variable | `\@sys_iteration` returns the iteration value, can be used in filters |

## Most important to remember

**"Exclusive answer in a numeric question" no longer requires JavaScript in DCv2** - the row is
marked as exclusive directly in the question builder. If someone asks for a script for this
"because that's how it was in DCv1", check first whether the built-in setting covers the need,
before writing or porting code. The script in `code-examples.md` is still relevant for DCv1 (which
still requires JavaScript), and as a starting point for edge cases the built-in feature doesn't
cover (e.g. if the behavior needs to deviate from the platform's default).

The same principle applies to every row marked "required a custom script" in the table above: check
whether DCv2 already solves the need natively before assuming a script is required to port the
functionality.
