# PhyST-Net: Physics-Aware Differentiable Spatio-Temporal Transformer Network

Wind power forecasting paper targeting IEEE Transactions.

## Module Naming

| Abbreviation | Full Name |
|---|---|
| PhyST-Net | Physics-aware Differentiable Spatio-Temporal Transformer Network |
| AGSP | Adaptive Gaussian-soft-windowed Patching |
| R-STMF | Relational Spatio-Temporal Manifold Fusion |
| K-AMC | KAN-enhanced Adaptive Meteorological Conditioning |
| PI-DCH | Physics-Informed Dynamic Capping Head |

## Project Structure

```
source/
  main.tex            # Master file (includes inline bibliography)
  nomenclature.tex    # Symbol table
  intro.tex           # Introduction
  related_work.tex    # Related work
  methodology.tex     # Methodology
  experiments.tex     # Experiments & results
  discussion.tex      # Discussion
  case_studies.tex    # Case studies
IEEEtran.cls          # IEEE Transactions class file
figures/              # Figure assets (pending)
output/               # Compiled PDF
```

## Build

Local TeX Live required. Two passes of pdflatex suffice (no bibtex needed):

```
cd source
pdflatex main.tex
pdflatex main.tex
```

## LaTeX Conventions

- ASCII only, no Chinese characters
- Use `` ``quoted text'' `` instead of straight quotes
- Em-dash `---`, en-dash `--`, hyphen `-`
- Always use `\label{}` / `\ref{}`, never hard-code numbers
- Abbreviations in Abstract must be redefined in Main Text
- `where` after equations: no indentation
