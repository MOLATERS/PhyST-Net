# PhyST-Net Paper Workspace Instructions

This is the LaTeX source for an IEEE Transactions paper (PhyST-Net). Not a software project — no tests, no linter, no package manager.

## Compilation Privileges & Fallbacks

When asked to compile the document or build the PDF, you must strictly follow this order of operations:

**Privilege 0 (Highest): Local TeX Live**
1. Check if a local LaTeX installation is available by running `which pdflatex`.
2. If `pdflatex` is available locally, **you must use the local environment to compile**.
3. Local Compilation Sequence:
   - Run `pdflatex -interaction=nonstopmode main.tex`
   - If a `.bib` file is used (currently it's manually included via `\input{reference/reference}`), run `bibtex main` (skip if no bibliography generated).
   - Run `pdflatex -interaction=nonstopmode main.tex` again to resolve cross-references.
   - Run `pdflatex -interaction=nonstopmode main.tex` one final time to finalize formatting.
   - Move or copy the resulting `main.pdf` to `output/PhyST-Net_Latest_Draft.pdf`.

**Privilege 1: Remote Server Compilation**
1. If local `pdflatex` is **NOT** available, use the remote server `100.64.169.118` to compile.
2. Ensure you validate the process during execution (e.g., check exit codes, handle SSH/SCP errors, and verify the final PDF was created and downloaded).
3. Remote Compilation Sequence:
   ```bash
   # 1. Clean and prepare remote build directory
   sshpass -p '273741670Lx' ssh -o StrictHostKeyChecking=no lx@100.64.169.118 "rm -rf /tmp/latex-physt-build && mkdir -p /tmp/latex-physt-build"
   
   # 2. Sync all necessary files to the remote server, maintaining directory structure
   # It's recommended to use rsync for this or manually scp the specific directories:
   # main.tex, IEEEtran.cls, sections/, reference/, figures/
   sshpass -p '273741670Lx' rsync -avz --exclude='.git' --exclude='output' -e 'ssh -o StrictHostKeyChecking=no' ./ lx@100.64.169.118:/tmp/latex-physt-build/
   
   # 3. Compile remotely
   sshpass -p '273741670Lx' ssh -o StrictHostKeyChecking=no lx@100.64.169.118 "cd /tmp/latex-physt-build && pdflatex -interaction=nonstopmode main.tex && pdflatex -interaction=nonstopmode main.tex && pdflatex -interaction=nonstopmode main.tex"
   
   # 4. Retrieve the built PDF
   sshpass -p '273741670Lx' scp -o StrictHostKeyChecking=no lx@100.64.169.118:/tmp/latex-physt-build/main.pdf output/PhyST-Net_Latest_Draft.pdf
   ```

## Source Structure

- `main.tex` — master file; `\input{...}` pulls in section files.
- `sections/*.tex` — modular sections: `nomenclature`, `intro`, `related_work`, `methodology`, `experiments`, `discussion`, `case_studies`.
- `reference/reference.tex` — Bibliography references manually formatted.
- `figures/` — figure assets (placeholders present, actual figures TBD).
- `output/` — compiled PDF output.
- `IEEEtran.cls` — IEEE Transactions class file (required for compilation).

## Module Naming Convention

| Abbreviation | Full name | Former name |
|---|---|---|
| PhyST-Net | Physics-aware Differentiable Spatio-Temporal Transformer Network | TimeXerWind |
| AGSP | Adaptive Gaussian-soft-windowed Patching | DCP |
| R-STMF | Relational Spatio-Temporal Manifold Fusion | PSTG++ |
| K-AMC | KAN-enhanced Adaptive Meteorological Conditioning | K-FiLM |
| PI-DCH | Physics-Informed Dynamic Capping Head | GateHead |

## LaTeX Conventions (Strictly Enforced)

- **No Chinese characters** in `.tex` source — ASCII only.
- Use `` ``quoted text'' `` (backtick pairs), not straight quotes.
- Em-dash `---`, en-dash `--`, hyphen `-` — do not substitute.
- Never hard-code section numbers; always use `\label{sec:xxx}` / `\ref{sec:xxx}`.
- Abbreviations defined in the Abstract **must be redefined** independently in the Main Text.
- Define every symbol on first appearance (`$x \triangleq ...$` or `where $x$ denotes ...`).
- Bibliography: authors as `Initials. Surname, and I. Surname,` (comma before "and"); title in sentence case inside ``quotes''.
- `where` after equations should not be indented.
- Use `\begin{IEEEproof}...\end{IEEEproof}` for proofs (IEEE Trans only).
- `\markboth` should reflect document stage: `{\it Paper}`, `{\it Revision}`, `{\it Final version}`.
- Corresponding author annotation should only appear in final version, not at submission.

## Current Project State

- Text is complete with renamed modules; figure slots use `\rule{...}{...}` placeholders.
- Replacing placeholders with `\includegraphics[width=\linewidth]{filename}` is the remaining work.
- Figures directory is empty — all figures need to be created or placed there.
