# System 250 Architecture — Process Model

## Data and capability registers

The process Dump Stack preserves D0–D7 directly as 24-bit data-register values. Its fixed locations corresponding to C0–C5 are also 24-bit words, but they do **not** contain copies of the expanded 48-bit capability registers. They contain the corresponding 24-bit capability pointers. Those pointers retain the protected capability state from which C0–C5 are reconstructed through the System Capability Table when the process is resumed.

C6 and C7 are represented separately as execution state in the Dump Stack's protected CALL/RETURN structure.
