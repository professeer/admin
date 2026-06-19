# TheProfesseer — Letterhead Template

Branded LaTeX letterhead/document template (crimson header band, owl footer,
Manrope typography). The sample document is a job description, but the layout
works for any one-page branded letter.

## Compile on Overleaf

1. Upload this zip to Overleaf (**New Project → Upload Project**).
2. Open **Menu → Settings → Compiler** and set it to **LuaLaTeX**.
   This is required — the house style uses `fontspec` and bundled fonts, which
   only work under LuaLaTeX (not pdfLaTeX).
3. Set the **Main document** to `professeer-jd.tex` if not auto-detected.
4. Click **Recompile**.

## Compile locally

Requires a TeX distribution with `lualatex` and `latexmk`:

```bash
latexmk -lualatex professeer-jd.tex
```

## Editing

Open `professeer-jd.tex` and edit the content blocks:

- `\jdtitle{Title}{Meta line}` — document heading + meta (e.g. location · type).
- `\jdsection{Heading}` — a crimson section heading.
- Body text and `itemize` lists fill each section. Every `itemize` needs at
  least one `\item`, or the build fails.
- `\jdplaceholder{text}` — grey italic placeholder; delete as you fill content.

The brand styling (colors, header band, footer, fonts) lives in
`sty/professeer.sty` — you normally don't need to touch it.

## Files

| Path | Purpose |
|------|---------|
| `professeer-jd.tex` | Main document — edit this. |
| `sty/professeer.sty` | House style: colors, header/footer macros, typography. |
| `fonts/` | Manrope font family (loaded by the style). |
| `public/` | Logo and owl-mark assets (PNG + SVG). |
