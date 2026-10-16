# capturefig

When instructed to **capturefig** a figure from a patent:

1. Identify the requested patent and figure.

2. Locate the patent PDF in `patents/` and consult the existing patent transcription in `transcriptions/` (which may be `.txt` or `.rtf`) when useful. **Never append a figure capture to the patent transcription.**

3. Inspect the actual patent figure at sufficient resolution to read all reasonably legible labels, reference numerals, arrows, connections and annotations. Consult the patent text where necessary to identify numbered elements or understand connections shown in the figure.

4. **Create a separate UTF-8 Markdown file for each captured figure** under `transcriptions/figures/` using the convention:

   `US-<patent-number>-fig-<figure>.md`

   For example, `transcriptions/figures/US-3814919-A-fig-3.md`. Use the patent number consistently with its PDF filename. A figure with separately drawn parts that form **one continuous diagram** (for example, Figures 2a and 2b displayed side by side) must be captured **together in one file**, named `US-3814919-A-fig-2a-2b.md`. Describe the join and every discernible cross-boundary connection; do not treat the halves as independent circuits. For distinct figures, use separate files.

5. Start the file with `# FIGURE <n> — TEXTUAL CAPTURE` (or `# FIGURES <n> AND <m> — TEXTUAL CAPTURE` for combined parts). Immediately identify the patent PDF, figure/page identifiers and the transcription consulted, if any. Keep these identifiers searchable. Existing figure captures in other formats or locations should not be moved or rewritten without explicit instruction.

6. The capture must preserve the technical information conveyed by the drawing in words. Record, as applicable:
   - named components, blocks and registers;
   - patent reference numerals;
   - labelled signals, buses and paths;
   - connections between elements;
   - arrow directions and direction of information/control flow;
   - branching, convergence and feedback paths;
   - containment, hierarchy and grouping;
   - switches, gates, selectors, storage elements and other symbols;
   - significant spatial relationships where they convey structure;
   - figure annotations and legends.

7. Describe **what the figure actually shows**, not merely what it appears to mean. Do not silently fill gaps from architectural knowledge. If something is illegible, ambiguous or uncertain, say so.

8. Where the patent text explicitly explains an element or connection shown in the figure, that explanation may be used to make the capture intelligible, but distinguish information supplied by the prose from information directly visible in the drawing.

9. After the factual capture, add:

   `INTERPRETIVE NOTE`

   only when the figure provides useful architectural evidence requiring explanation. Keep interpretation separate from transcription. Clearly distinguish:
   - explicit evidence from the figure;
   - explicit evidence from the patent text;
   - architectural inference.

10. The purpose is to make the information content of the patent figure available in searchable textual form while retaining the original PDF drawing as the primary source. Individual figure files make the set of captured figures inspectable by filename, so uncaptured figures can be identified by comparison with the patent's figure list. **Do not create placeholder files for figures that have not been inspected.**

11. Do not modify any other repository material unless explicitly instructed.

## Example invocations

`capturefig Fig 3 of 712 patent`

means: locate the patent whose identifier unambiguously corresponds to “712”, inspect Figure 3 and create its own appropriately named Markdown capture file under `transcriptions/figures/`. Leave existing transcription files unchanged.

`capturefig Figs 2a and 2b of 919 patent`

means: inspect Figures 2a and 2b together as the two halves of a single processor diagram, follow connections across the join and create one file, `transcriptions/figures/US-3814919-A-fig-2a-2b.md`.
