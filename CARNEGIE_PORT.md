# Porting Carnegie Learning changes onto upstream MathQuill

This file tracks how the Carnegie Learning commits in the
`CarnegieLearningWeb/mathquill` fork are being brought onto the latest
upstream MathQuill (`mathquill/mathquill`).

## Background

- `release/baseline` branched from upstream in January 2019 (`248691db`) and
  holds the Carnegie changes on top of it.
- Since then upstream rewrote the source from JavaScript to TypeScript and
  removed its jQuery dependency. The Carnegie commits edit `.js` files that no
  longer exist, so they can't be cherry-picked. Each one is rewritten by hand in
  the new `.ts` files.
- The fork's `master` was synced with upstream on 2026-11-10 (PR #32, `3cd53526`).

## How the port is done

- Branch: `release/baseline-v2`, created from `master` (`3cd53526`) and pushed
  to `CarnegieLearningWeb/mathquill`.
- One new commit per original commit, in the original order.
- Each new commit keeps the original author, author date, and message. A line
  at the end of the message names the original commit and notes any deliberate
  difference.
- After each commit:
  - `make lint` (TypeScript type-checks) and Prettier pass.
  - The unit tests (`test/unit.html`) run in headless Chrome: 692 pass and 3
    fail. The 3 failures are scrolling tests that also fail on unmodified
    upstream in headless Chrome.
  - The new behavior is checked with a throwaway browser page, which is deleted
    afterwards.

## Status of each commit

Status values:

- **Ported**: rewritten in TypeScript and committed on `release/baseline-v2`.
- **Already upstream**: the same change is already in `master`, so nothing to do.
- **Skipped**: deliberately left out.
- **Included in #N**: folded into another ported commit.

### Commits on `release/baseline`

| # | Original | Author | Title | Status | New commit | Notes |
|---|---|---|---|---|---|---|
| 1 | `1d169a96` | Andreya Piplica | Expose cursor methods for setting and clearing selection | Ported | `a5928d8c` | Adds `setSelection(pointA, pointB)` on editable fields, plus `containsPoint`/`childForPoint` on tree nodes. `clearSelection` was already in `master`. The original `childForPoint` returned `true` instead of a node, which TypeScript doesn't allow, so it returns the child. It still only looks one level deep, as in the original. |
| 2 | `3e1d252b` | Andreya Piplica | Make childForPoint recursive | Ported | `dce4a0d3` | `childForPoint` returns the deepest node containing the point. Fixes selecting a fraction's denominator: before this, it selected the numerator. |
| 3 | `a45cfd80` | Karl Schaefer | Update ReadMe | Ported | `4c95eb12` | Docs for `setSelection`. The original heading said `.select(startPoint, endPoint)`; corrected to `.setSelection(...)`. |
| 4 | `e90b023d` | Andreya Piplica | Ease selection of contents of delimiters | Ported | `0f3cb9f4` | Clicking or dragging onto a bracket puts the cursor inside it. Applied to `Bracket` only: the original never affected `\binom`, whose middle element isn't a block. |
| 5 | `b299f569` | Andreya Piplica | Add make static functionality to inner fields | Already upstream | n/a | In `master` as `23500741` (same author and date), merged upstream in PR #902 from `CarnegieLearningWeb/feature-makeStatic`. `master`'s version is the corrected design and has a unit test. |
| 6 | `f94ac7c1` | Karl Schaefer | Add documentation on makeStatic and makeEditable. | Already upstream | n/a | In `master` as `9040f97e` (same PR #902). Same text, in its own "Inner MathField methods" section. |
| 7 | `61b36d58` | Karl Schaefer | Add prevent spaces from behaving like tabs in root blocks. | Ported | `573f02bb` | Adds `spaceBehavesLikeTab: 'exceptRootBlock'`, with docs and the 3 original unit tests. The root check uses the field's own root so it also works in inner fields, and other truthy values still behave like `true`. |
| 8 | `870a71d7` | Karl Schaefer | Add new Tabbing stragety for root blocks. | Ported | `13be42ca` | Adds `tabAction: 'escapeAndNextTemplate'`, with docs and the Tabbing demo in `test/visual.html`. The message says `'escapeAndTemplate'`; the code and docs use `'escapeAndNextTemplate'`. |
| 9 | `3166c20a` | Andreya Piplica | Select operators in a mathematically meaningful way | Ported | `5c21abee` | Adds `strictOperatorSelection` with a built-in operator list. Includes the guard from #16; without it 5 existing `Cursor::select()` unit tests fail. |
| 10 | `f9474e6f` | Andreya Piplica | Specify strict operators in a config | Ported | `5f94b5da` | `strictOperatorSelection` becomes `{ prefixOperators, binaryOperators }`. Adds the `StrictOperatorSelection` type. Falsy values turn it off instead of throwing. |
| 11 | `5e87d624` | Andreya Piplica | Update ReadMe to include config options for strict operator selection. | Ported | `71ffe9e1` | Docs for `strictOperatorSelection`. Curly quotes in the example replaced with straight quotes. |
| 12 | `92b51527` | Karl Schaefer | Add bower.json file for supporting bower. | Skipped | n/a | Bower is deprecated, and the file lists jQuery, which MathQuill no longer needs. |
| 13 | `f7e22299` | Karl Schaefer | Moved setSelection and clearSelection to AbstractMath. | Ported | `22b30dd0` | Both methods are now available on static math too; `select()` stays editable-only. Types and docs moved to match. |
| 14 | `198e1bf7` | Karl Schaefer | Update git ignores. | Ported | `2bc2e03d` | Adds `/mathquill-*.zip`. `script/prep-release.sh` still creates that file. |
| 15 | `679c2692` | Karl Schaefer | Update version to 0.10.1-6. | Skipped | n/a | Only changes `bower.json`. |
| 16 | `44a7562a` | Karl Schaefer | The Cursor options can be undefined. Adding a safeguard to the check for strictOperatorSelection. | Included in #9 | `5c21abee` | |
| 17 | `492f9a61` | Karl Schaefer | Binary operators were not correctly defined when the operator is the first element in a style block. | Already upstream | n/a | In `master` as `6abcd4eb`, merged upstream in PR #673 from `CarnegieLearningWeb/fix-binaryOpInStyleBlock`. |
| 18 | `5a0ee260` | Karl Schaefer | Update version number for release. | Skipped | n/a | Only changes `bower.json`. |
| 19 | `350fc094` | Karl Schaefer | Add \overarc command. | Already upstream | n/a | In `master` as `a926747b` (upstream #679, "Add \overarc and \overleftrightarrow"). |
| 20 | `adc55075` | Karl Schaefer | Add \overleftrightarrow command. | Already upstream | n/a | In `master` as `cd24d80d` (same upstream change). |
| 21 | `ebcbbea5` | Karl Schaefer | Update to version 0.10.1-8. | Skipped | n/a | Only changes `bower.json`. |
| 22 | `7a3ab671` | Karl Schaefer | Fix bad merge from feature-makeStatic that inlucded changes to command.js. | Already upstream | n/a | Removes a broken `makeStatic` from the `\MathQuillMathField` command. `master` never had that code; its `makeStatic` is on `InnerMathField`. |
| 23 | `6c8effd1` | Karl Schaefer | Update version numbers for next release. | Skipped | n/a | Only changes `bower.json`. |
| 24 | `75827cf1` | Karl Schaefer | DO NOT PUSH | Ported | `d4e542df` | `makeStatic()`/`makeEditable()` swap the `mq-editable-field-static` class. Despite the title, it shipped in Carnegie releases `v0.10.1-10` and `v0.10.1-11`, so apps may rely on the class. Also adds `mq-editable-field-static` checks to the existing `makeStatic` unit test. |
| 25 | `53db5cc6` | Karl Schaefer | Update version. | Skipped | n/a | Only changes `bower.json`. |

### Carnegie branches never merged into `release/baseline`

Carnegie's last release, `v0.10.1-11` (2019-01-24), was built from
`release/baseline` and its source is identical to it. None of these branches
were in a Carnegie release.

| Branch | Original | Author | Title | Status | Notes |
|---|---|---|---|---|---|
| `feature/improve-docs` | `8d042caa` | Karl Schaefer | Add missing documentation for features. | Skipped | Not needed. Docs for `mouseEvents` and `autoSubscriptNumerals`; the text was checked and is still accurate for `master` if it's wanted later. |
| `feature/update-package` | `7e77eb1b` | Karl Schaefer | Update the package file to properly list jQuery dependency. | Skipped | Not needed. Adds jQuery as a peer dependency, and upstream removed jQuery. |
| `feature/unbased-super-sub-scripts` | `003120ea` | Karl Schaefer | Initial commit of :before box for unbased exponents | Skipped | Not needed. Superscripts and subscripts with no base; an unfinished feature. |
| | `7903db16` | Karl Schaefer | Correctly pass options into render tree | Skipped | Same branch. |
| | `15cf0788` | Karl Schaefer | Oops, too aggressive with cursor add in finalizeXXX methods | Skipped | Same branch. |
| | `e9b212dd` | Karl Schaefer | Add ability to render unbiased subscript | Skipped | Same branch. |

The `dependabot/*` branches are automated dependency bumps, not Carnegie work,
and are not part of this port.

## Summary

The port is complete: every Carnegie commit has a status, and none are pending.

Commits on `release/baseline` (25):

| Status | Count |
|---|---|
| Ported | 12 (#1–4, #7–11, #13, #14, #24) |
| Already upstream | 6 (#5, #6, #17, #19, #20, #22) |
| Included in another commit | 1 (#16, in #9) |
| Skipped | 6 (#12, #15, #18, #21, #23, #25: `bower.json` and version bumps) |

Commits on the extra branches (6): all skipped, not needed.

### Check against a one-step merge

`release/baseline-v2` was compared with `backup/merge-into-baseline-v1`
(`880bb17a`), an earlier attempt that applied all of `release/baseline` in one
merge commit. The code differs only in comments, spacing, and how the same
code is organized, apart from these deliberate differences:

- The one-step merge kept Carnegie's copy of the `makeStatic`/`makeEditable`
  docs next to upstream's, so they appeared twice. `release/baseline-v2` keeps
  only upstream's.
- The one-step merge applied the bracket-selection change (#4) to `\binom` too.
  `release/baseline-v2` keeps the original behavior: brackets only.
- `spaceBehavesLikeTab: 'exceptRootBlock'` (#7) also works in inner fields, and
  other truthy values still act like `true`.
- `release/baseline-v2` adds unit-test checks for `mq-editable-field-static`
  (#24).

### Status of the branch

- `release/baseline-v2` has 12 ported commits, plus the commit adding this
  file, on top of `master` (`3cd53526`).
- It is pushed to `CarnegieLearningWeb/mathquill` as `release/baseline-v2`.
  `release/baseline` on GitHub is unchanged.

## Related local branches

`release/baseline-v2` was named `carnegie/master` until it was renamed.

These local branches are kept for reference and have not been pushed:

- `release/baseline-v1`: the old `release/baseline` (`4dee95f7`), unchanged.
- `backup/merge-into-baseline-v1` (`880bb17a`) and `backup/ported-merge`: an
  earlier attempt that merged `master` into `release/baseline` in a single
  commit instead of porting commit by commit.
- `update-baseline`, `sync-upstream`: earlier working branches.
