# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is the LaTeX source for **PhyST-Net** — an IEEE Transactions journal paper: "PhyST-Net: An Exogenous-Aware Spatio-Temporal Graph Transformer for Domain-Constrained Network Forecasting." Not a software project — no tests, no linter, no package manager.

## Compilation

**Local TeX Live (preferred):**
```bash
pdflatex -interaction=nonstopmode main.tex
bibtex main   # only if .bib used
pdflatex -interaction=nonstopmode main.tex
pdflatex -interaction=nonstopmode main.tex
# copy main.pdf to output/PhyST-Net_Latest_Draft.pdf
```

**Remote server fallback (if no local pdflatex):**
```bash
sshpass -p '273741670Lx' ssh -o StrictHostKeyChecking=no lx@100.64.169.118 \
  "cd /tmp/latex-physt-build && pdflatex -interaction=nonstopmode main.tex && pdflatex -interaction=nonstopmode main.tex && pdflatex -interaction=nonstopmode main.tex"
```

## Source Structure

| Path | Description |
|---|---|
| `main.tex` | Master file; `\input{...}` pulls in sections |
| `sections/*.tex` | Modular sections: intro, related_work, problem_definition, methodology, experiments, discussion, case_studies, architecture_figure |
| `reference/reference.tex` | Bibliography references |
| `figures/` | Figure assets (placeholders use `\rule{...}{...}`) |
| `output/` | Compiled PDF output |
| `IEEEtran.cls` | IEEE Transactions class file |

## Module Naming Convention

| Abbreviation | Full Name | Former Name |
|---|---|---|
| PhyST-Net | Physics-aware Differentiable Spatio-Temporal Transformer Network | TimeXerWind |
| AGSP | Adaptive Gaussian-soft-windowed Patching | DCP |
| R-STMF | Relational Spatio-Temporal Manifold Fusion | PSTG++ |
| K-AMC | KAN-enhanced Adaptive Modulation and Conditioning | K-FiLM |
| PI-DCH | Physics-Informed Dynamic Capping Head | GateHead |

## Architecture Pipeline

The model follows this computation flow:
```
X, E^h → AGSP → Temporal tokens Z → Transformer encoder → H_T
                                            ↓
                                       R-STMF → H_G (graph-enhanced)
                                            ↓
                           K-AMC ← E^f → Modulated H_F
                                            ↓
                           PI-DCH → Feasible forecast Ŷ
```

## LaTeX Conventions (Strictly Enforced)

- **No Chinese characters** in `.tex` source — ASCII only
- Use `` ``quoted text'' `` (backtick pairs), not straight quotes
- Em-dash `---`, en-dash `--`, hyphen `-` — do not substitute
- Never hard-code section numbers; always use `\label{sec:xxx}` / `\ref{sec:xxx}`
- Abbreviations defined in the Abstract **must be redefined** independently in the Main Text
- Define every symbol on first appearance (`$x \triangleq ...$` or `where $x$ denotes ...`)
- Bibliography: authors as `Initials. Surname, and I. Surname,` (comma before "and"); title in sentence case inside ``quotes''
- `where` after equations should not be indented
- Use `\begin{IEEEproof}...\end{IEEEproof}` for proofs
- `\markboth` should reflect document stage: `{\it Paper}`, `{\it Revision}`, `{\it Final version}`

## Current Project State

- Text complete; figure slots use `\rule{...}{...}` placeholders
- Replacing placeholders with `\includegraphics[width=\linewidth]{filename}` is remaining work
- Figures directory empty — all figures need creation or placement
- All experimental results use `\textbf{TBD}` placeholders