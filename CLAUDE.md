# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A LaTeX résumé/CV for Hasan Forghani Ramandy, built on a customized fork of Vebjørn S. Førde's
two-page CV class (CC BY 4.0). It is a document, not an application — "the code" is the class file
plus a set of content fragments.

## Build

```bash
latexmk -pdf -interaction=nonstopmode main.tex   # full build (pdflatex + biber + reruns)
latexmk -c                                       # clean aux files (keeps main.pdf)
latexmk -pvc -pdf main.tex                       # watch mode while editing
```

Engine is **pdfLaTeX** with **biber** (not bibtex). `pdflatex main.tex` alone will not resolve the
bibliography — publications come from `Txt/Publications/pub.bib` via biblatex.

There is no test suite. The build *is* the test: a clean exit and a 2-page `main.pdf` means it passes.

Shipping a new version means copying the build output by hand — `main.pdf` and `Forghani_Resume.pdf`
are byte-identical in the current commit, and the older release is kept as `OLD_Forghani_Resume.pdf`:

```bash
cp main.pdf Forghani_Resume.pdf
```

### Build artifacts are tracked

`.gitignore` lists the usual LaTeX aux files, but `main.pdf`, `main.log`, `main.aux`, `main.bbl`,
`main.bcf`, `main.blg`, `main.fls`, `main.fdb_latexmk`, `main.run.xml` and `main.synctex.gz` were
committed before those rules applied and remain tracked. Any build dirties the working tree. Either
`git checkout --` them after an exploratory build, or commit them along with the content change —
past commits do the latter. `main.out` is tracked and empty; leave it.

## The two-page hard limit

`Code/CV.cls` deliberately raises a **fatal** error (`\msg_fatal:nn {CV}{pagenumber}`) if the document
exceeds `\thepagelimit` (2). A build that dies with "Page limit: 2 pages, exceeded" is the template
working as designed, not a bug — the fix is to cut content, not to patch the class. The escape hatch is
the `multipleCVs` class option, intended for documents holding several CVs, not for a longer one.

## Architecture

`main.tex` is the only file compiled. It sets class options, declares identity (`\firstname`,
`\lastname`, `\tagline`, `\personalinfo`), draws the header, and then `\input`s content fragments:

| File | Role |
|---|---|
| `Code/CV.cls` | The class. Options, colors, all custom macros/environments, the page limit, biblatex wiring. |
| `Code/Preamble.tex` | `\addbibresource`, babel language, extra color definitions (`deep_blue`, `medium_blue`, `deep_red`, `deep_purple`). |
| `Txt/1-CV-body.tex` | Page 1 main column — Experience entries. Wrapped in `CVbody`. |
| `Txt/p1_sidebar.tex` | Page 1 right column — Education, Skills, ratings. Wrapped in `CVsidebar`. |
| `Txt/2-page2.tex` | Page 2 — `\printbibliography` calls, one per publication type. Full width; its `CVbody` wrapper is commented out. |
| `Txt/p2_Sidebar.tex` | Empty stub, not currently `\input` from `main.tex`. |
| `Txt/Publications/pub.bib` | Bibliography. Mixed real entries and leftover template dummies. |
| `Tips-for-CV.tex` | Upstream author's usage notes. Never compiled — reference only. |

The two-column layout is achieved with `\begin{CVbody}` (10cm minipage) followed by `\begin{CVsidebar}`
(5cm minipage) side by side. To go single-column on a page, comment out both the `CVbody` wrapper and
the corresponding sidebar `\input` — the environments are `minipage`s, so removing one without the
other breaks the layout.

`\nocite{*}` in `main.tex` means **every** entry in `pub.bib` is printed. Filtering happens in
`Txt/2-page2.tex`, where each `\printbibliography[type=...]` line selects one entry type; commented-out
lines there are how unwanted categories are suppressed. Adding a bib entry of an uncommented type makes
it appear automatically — and can silently push the document past the page limit.

## Class conventions worth knowing

- **Class options** are documented in the comment block at the top of `main.tex`; the current set is
  `bib=normal,page2=display,nophoto`. Options are declared the old-fashioned way (`\DeclareOption`), so
  they are literal strings — `page2=display` is one token, not a key-value pair.
- **Colors** are indirections: `icons`, `soft_text`, `background_color`. Recolor via
  `\colorlet{icons}{medium_blue}` in `main.tex` (as is done now), not by editing the class.
- **Entry macros**: `\CVrating{Label}{n}{max}` (filled circles), `\softtext{}` (gray),
  `\dotline`/`\linia` (separators), `\CVevent{date}{location}`, `CV_table`/`CV_text` (aligned
  three/two-column entry blocks), `Summary` (the tcolorbox profile box), `linkheader`, `itemlist`,
  `\CVref` for referees.
- **Experience entries** in `Txt/1-CV-body.tex` do *not* use `CV_table`; they follow a hand-rolled
  pattern of `\textbf{Title} \hfill {\small \textit{Company}\softtext{\textit{, City}}} \\` then a
  `{\small ...}` paragraph, separated by `\vspace{0.75em}`. Match that pattern when adding entries.
- **Social macros** take the handle, not the URL: `\github{hsnfrghn}` builds the link itself. Note
  `\researchgate` is declared with two arguments but only uses the first, so call sites pass a second
  brace group or let it swallow the following token — copy the existing call in `main.tex` rather than
  inventing one.
- Large amounts of content in `Txt/*.tex` are commented-out template scaffolding (languages, referees,
  certificates, About me). It is kept intentionally as a menu of available features — leave it alone
  unless asked to clean up.

## Editing

Most requests are content changes to `Txt/*.tex`. Prefer those over touching `Code/CV.cls`; the class
is upstream code carrying its own copyright notice, and local changes to it are hard to distinguish
from upstream when diffing. After any content edit, rebuild and confirm the page count is still 2.
