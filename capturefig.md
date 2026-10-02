# capturefig

When instructed to **capturefig** a figure from a patent:

1. Identify the requested patent and figure.

2. Locate the patent PDF and the corresponding existing transcription file in `Transcriptions`. The transcription may be `.txt` or `.rtf`.

3. Inspect the actual patent figure at sufficient resolution to read all reasonably legible labels, reference numerals, arrows, connections and annotations. Consult the patent text where necessary to identify numbered elements or understand connections shown in the figure.

4. Append the figure capture to the **end of the existing transcription file**. Do not replace, rewrite, reformat, summarise or otherwise alter the existing transcription.

5. Append a section headed:

   `FIGURE <n> — TEXTUAL CAPTURE`

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

10. The purpose is to make the information content of the patent figure available in searchable textual form while retaining the original PDF drawing as the primary source.

11. Do not modify any other repository material unless explicitly instructed.

## Example invocation

`capturefig Fig 3 of 712 patent`

means: locate the patent whose identifier unambiguously corresponds to “712”, inspect Figure 3, locate its existing TXT or RTF transcription in `Transcriptions`, and append the textual capture to that transcription according to these rules.
