---
description: Review the current diff as an Emacs Lisp / MELPA package change
---

Review the current changes to this `quarto-mode` Emacs Lisp package.

## Scope

By default review the uncommitted diff. If the working tree is clean,
review the diff of the current branch against `main`. If the user passed
an argument (`$ARGUMENTS`), treat it as the target to review (a branch,
commit range, or PR number).

First gather context:

```sh
git status --short
git diff
```

## What to check (in priority order)

1. **Optional-dependency safety.** ESS/poly-R and `python` are optional.
   Top-level hard `require` of an optional package, a call into an
   optional package without `with-eval-after-load` / `declare-function`,
   or a missing `(require … nil t)` guard is a bug. Byte-compile must
   stay warning-free.
2. **lexical-binding hygiene.** Closures, `setq-local` vs `setq`,
   dynamic vs lexical variable capture. Polymode reuses other modes'
   buffers, so mode hooks must use `setq-local`.
3. **Docstrings / checkdoc / package-lint.** Every
   `defun`/`defcustom`/`defvar`/`defmacro` needs a docstring whose first
   line is a single complete sentence ending in a period. Flag arguments
   not documented in UPPER-CASE, lines over the fill column, etc.
4. **Naming.** Public = `quarto-…`; internal = `quarto-mode--…`. Flag
   new internals that omit the double dash and new public symbols that
   aren't really meant to be public.
5. **Header hygiene.** If behavior or the public API changed, was
   `Version:` bumped? If a newer Emacs/dependency API is used, was
   `Package-Requires` / the Emacs baseline (29.1) updated?
6. **Correctness & resource handling.** Process/buffer lifecycle
   (`make-process`, `delete-process`, `kill-buffer`), temp file cleanup,
   advice add/remove symmetry, error signaling that's intentional vs
   accidental.

## Output

Group findings by severity (blocker / should-fix / nit). For each, cite
`quarto-mode.el:LINE`, explain the concrete failure mode, and give the
minimal fix. If you can run `emacs -Q --batch` to byte-compile or
checkdoc the file, do so and report the result. Be concise; prefer a few
high-confidence findings over an exhaustive list.
