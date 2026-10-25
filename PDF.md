# Markdown-to-PDF standards

When asked to create a PDF from a Markdown file in this repository, follow these defaults unless the user specifies otherwise.

- Use Pandoc with XeLaTeX.
- Main font: Times New Roman; body size: 11 pt.
- Paper: A4; margins: 20 mm on all sides.
- Line spacing: 1.0 (single); paragraph spacing: 0 pt.
- Include referenced diagrams, keeping SVG/vector output where supported.
- Do not modify the source Markdown or its diagram files.
- Write the PDF alongside the source file, with the same basename and a `.pdf` extension, unless another destination is requested.
- Check conversion warnings and errors; verify the PDF exists and diagrams and special characters render correctly before declaring success.
- Pandoc, XeLaTeX and `rsvg-convert` must be installed in the environment running the command; local installation on the user's Mac is not accessible from a remote execution environment.

## General command (run from repository root)

```bash
input="path/to/document.md"
output="${input%.md}.pdf"

pandoc "$input" \
  -o "$output" \
  --pdf-engine=xelatex \
  -V 'mainfont=Times New Roman' \
  -V papersize=a4 \
  -V fontsize=11pt \
  -V geometry:margin=20mm \
  -V linestretch=1.0 \
  -V parskip=0pt \
  --resource-path="$(dirname "$input"):."
```

For Unicode mathematical notation such as `M⟨H,T⟩`, inspect the rendered PDF. If the chosen font lacks a glyph, fix that glyph or use an appropriate font fallback rather than silently losing it.
