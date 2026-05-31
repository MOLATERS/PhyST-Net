# AGENTS.md

## What this repo is

LaTeX source for an IEEE Transactions paper (PhyST-Net). Not a software project — no tests, no linter, no package manager.

## How to build

**No local TeX Live.** Compilation must happen on remote server `100.64.169.118`. The full build sequence (order matters):

```
sshpass -p '273741670Lx' ssh -o StrictHostKeyChecking=no lx@100.64.169.118 "rm -rf /tmp/latex-physt-build && mkdir -p /tmp/latex-physt-build"

sshpass -p '273741670Lx' scp -o StrictHostKeyChecking=no -r source/*.tex source/*.bib lx@100.64.169.118:/tmp/latex-physt-build/

sshpass -p '273741670Lx' ssh -o StrictHostKeyChecking=no lx@100.64.169.118 "cd /tmp/latex-physt-build && pdflatex -interaction=nonstopmode main.tex && bibtex main && pdflatex -interaction=nonstopmode main.tex && pdflatex -interaction=nonstopmode main.tex"

sshpass -p '273741670Lx' scp -o StrictHostKeyChecking=no lx@100.64.169.118:/tmp/latex-physt-build/main.pdf output/PhyST-Net_Latest_Draft.pdf
```

The `pdflatex → bibtex → pdflatex → pdflatex` order is required to resolve cross-references and citations.

## Source structure

- `source/main.tex` — master file; `\input{...}` pulls in section files (no path prefix — pdflatex runs in same dir)
- `source/*.tex` — modular sections: `nomenclature`, `intro`, `related_work`, `methodology`, `experiments`, `discussion`, `case_studies`
- `source/ref.bib` — BibTeX database
- `figures/` — figure assets (placeholders present, actual figures TBD)
- `output/` — compiled PDF output
- `IEEEtran.cls` — IEEE Transactions class file (required for compilation)

## Module naming convention

| Abbreviation | Full name | Former name |
|---|---|---|
| PhyST-Net | Physics-aware Differentiable Spatio-Temporal Transformer Network | TimeXerWind |
| AGSP | Adaptive Gaussian-soft-windowed Patching | DCP |
| R-STMF | Relational Spatio-Temporal Manifold Fusion | PSTG++ |
| K-AMC | KAN-enhanced Adaptive Meteorological Conditioning | K-FiLM |
| PI-DCH | Physics-Informed Dynamic Capping Head | GateHead |

## LaTeX conventions (non-obvious)

- **No Chinese characters** in `.tex` source — ASCII only
- Use `` ``quoted text'' `` (backtick pairs), not straight quotes
- Em-dash `---`, en-dash `--`, hyphen `-` — do not substitute
- Never hard-code section numbers; always use `\label{sec:xxx}` / `\ref{sec:xxx}`
- Abbreviations defined in the Abstract **must be redefined** independently in the Main Text
- Define every symbol on first appearance (`$x \triangleq ...$` or `where $x$ denotes ...`)
- Bibliography: authors as `Initials. Surname, and I. Surname,` (comma before "and"); title in sentence case inside ``quotes''
- `where` after equations should not be indented
- Use `\begin{IEEEproof}...\end{IEEEproof}` for proofs (IEEE Trans only)
- `\markboth` should reflect document stage: `{\it Paper}`, `{\it Revision}`, `{\it Final version}`
- Corresponding author annotation should only appear in final version, not at submission

## Current state

- Text is complete with renamed modules; figure slots use `\rule{...}{...}` placeholders
- Replacing placeholders with `\includegraphics[width=\linewidth]{filename}` is the remaining work
- Figures directory is empty — all figures need to be created or placed there
