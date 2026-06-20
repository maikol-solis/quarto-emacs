# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

`quarto-mode` is a single-file Emacs Lisp package that provides a
[polymode](https://github.com/polymode/polymode) for editing
[Quarto](https://quarto.org) (`.qmd`) documents. All code lives in
[quarto-mode.el](quarto-mode.el). The package is distributed on MELPA.

There is intentionally no `quarto-mode` *major* mode: `.qmd` files are
associated with the `poly-quarto-mode` polymode. When ESS / poly-R are
installed the polymode inherits R support; otherwise it falls back to a
plain markdown polymode.

## Code evaluation in chunks

`polymode-eval-region-or-chunk` runs the code in a chunk:

- **R** chunks work automatically through ESS / poly-R.
- **Python** chunks are wired up by this package (see the "Python
  evaluation" section in [quarto-mode.el](quarto-mode.el)):
  `quarto-mode--python-mode-hook` sets `polymode-eval-region-function`
  buffer-locally inside polymode inner buffers, and
  `quarto-mode--eval-python-region` delegates to
  `python-shell-send-region`. The hook is attached to both
  `python-mode-hook` and `python-ts-mode-hook` via
  `with-eval-after-load`, so an inferior Python process executes
  `{python}` chunks. This is the optional-dependency pattern: `python`
  is loaded lazily and `python-shell-send-region` is `declare-function`'d
  for the byte-compiler.

## Layout

- [quarto-mode.el](quarto-mode.el) — the entire package
- [Cask](Cask) — dependency/build manifest (`package-file "quarto-mode.el"`)
- [.github/workflows/melpazoid.yml](.github/workflows/melpazoid.yml) — MELPA lint checks
- [.github/workflows/lint.yml](.github/workflows/lint.yml) — byte-compile + checkdoc + package-lint
- [TODO.org](TODO.org) — outstanding work

## Conventions

- File is `lexical-binding: t`. Keep it that way.
- Minimum Emacs is `29.1`; dependencies are declared in the
  `Package-Requires` header. Do not use APIs newer than the declared
  baseline without bumping it, and bump `Version:` when shipping a change.
- Naming: public symbols are `quarto-…`; internal symbols use a
  double dash, `quarto-mode--…`. Follow this when adding code.
- Every `defun`/`defcustom`/`defvar` needs a docstring (checkdoc /
  package-lint enforce this on CI). First line of a docstring must be a
  complete sentence ending in a period and fit on one line.
- Optional dependencies (ESS/poly-R, `python`) are loaded defensively:
  `(require … nil t)`, `with-eval-after-load`, `declare-function` for
  byte-compile. Never hard-`require` an optional package at top level.
- Prefer `setq-local` inside mode hooks (polymode reuses other modes'
  buffers, so global `setq` would leak).

## Commits

Always write commit messages as [Conventional
Commits](https://www.conventionalcommits.org/) **in English**, e.g.
`feat: add python chunk evaluation`, `fix: guard python hook to polymode
buffers`, `docs: document chunk evaluation`, `ci: add elisp lint
workflow`, `chore: bump minimum Emacs to 29.1`. This applies regardless
of the language used in conversation.

## Build, lint, test

Local checks (need Emacs on PATH):

```sh
# byte-compile and fail on warnings
emacs -Q --batch -L . \
  --eval '(setq byte-compile-error-on-warn t)' \
  -f batch-byte-compile quarto-mode.el

# checkdoc
emacs -Q --batch --eval '(checkdoc-file "quarto-mode.el")'
```

With Cask installed (`brew install cask` or the upstream installer):

```sh
cask install
cask build       # byte-compile via the manifest
```

There is no unit-test suite. CI ([.github/workflows/lint.yml](.github/workflows/lint.yml))
runs across Emacs 29.1/29.4/30.1/snapshot:

- **byte-compile** and **checkdoc** are hard gates — keep both clean.
  The only accepted byte-compile warnings are the structural
  `make-variable-buffer-local` / `easy-mmode-define-keymap` ones emitted
  by polymode's `define-polymode` macro (it is wrapped in an `if`); do
  not introduce new warnings beyond those.
- **package-lint** is advisory. Its one standing finding is a
  `with-eval-after-load` warning for the optional `python` integration,
  which is the intended pattern.

Note the header is `Package-Requires:` (canonical capitalization) — tools
silently miss the dependency list otherwise.

## Reviewing changes

Use `/review-elisp` for a review tuned to this package (see
[.claude/commands/review-elisp.md](.claude/commands/review-elisp.md)),
or `/code-review` for a general diff review. When reviewing, weight
heavily: optional-dependency safety, docstring/checkdoc compliance,
`quarto-mode--` naming for internals, lexical-binding hygiene, and
buffer-local correctness inside polymode hooks.
