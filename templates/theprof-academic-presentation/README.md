# TheProfesseer — Academic Presentation Template

Branded Beamer (LaTeX) slide deck: crimson accents (`#7B0925`), owl mark in the
corner, frame-numbered footer. A blank skeleton — fill in the `% TODO` frames.

## Compile on Overleaf

1. Upload this folder (or a zip of it) to Overleaf.
2. Open **Menu → Settings → Compiler** and set it to **pdfLaTeX**.
3. Set the **Main document** to `theprof-academic-presentation.tex`.
4. Click **Recompile**.

## Compile locally

```bash
latexmk -pdf theprof-academic-presentation.tex
```

## Editing

Open `theprof-academic-presentation.tex`:

- `\title{...}` / `\author{...}` — set the deck title and author.
- `\section{...}` — starts a new section and auto-inserts a divider slide.
- Standard slides use `\begin{frame}{Title} ... \end{frame}` with an `itemize`.
- `\pause` reveals bullets one at a time.
- `\fullpage{...}` — one big centred statement.
- The skeleton includes a bullet slide, a two-column (text + figure/table)
  slide, a section divider, and a closing slide as starting points.

Brand colors and theme styling live in `beamercolorthemeprofesseer.sty` — you
normally don't need to touch it.

## Files

| Path | Purpose |
|------|---------|
| `theprof-academic-presentation.tex` | The deck — edit this. |
| `beamercolorthemeprofesseer.sty` | Beamer color theme (brand crimson). |
| `professeer-mark-crimson.png` | Owl mark shown in the slide corner. |
