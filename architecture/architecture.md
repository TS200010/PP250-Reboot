# System 250 Architecture — Working Reconstruction

## Status

This document is the current working reconstruction of the Plessey System 250 architecture.

It is **not primary evidence**. Statements below are derived from source material held or transcribed in this repository. Where the available evidence does not establish semantics, this document records the fact without filling the gap by assumption.

This first pass is deliberately limited primarily to evidence in the *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7, with additional contemporary Plessey papers and patent material where explicitly identified below.

## Source basis

Primary sources used for this revision include:

- *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7.
- D. Halton, *Hardware of the System 250 for Communication Control* (1972).
- D. M. England, *Architectural Features of System 250*, Figure 7, "Examples of Commands".
- Contemporary Plessey capability-register, interrupt, and store-allocation patent material.
- Repository transcriptions under `transcriptions/`.

## Architecture sections

The working architecture reconstruction has been split by subject into the following documents:

- [Generation 2 Recovered Architecture](generation-2-system-250.md) — canonical integrated reconstruction of the mature c. 1975–76 System 250 architecture.
- [Instruction Set](instruction-set.md) — architectural word size, instruction formats, addressing, assembler syntax, programmer-visible instructions, and the existing Church/Turing interpretation.
- [Capability Representation](capability-representation.md) — capability access rights and stored capability type/form representation.
- [System Capability Table](system-capability-table.md) — SCT role and entry structure, descriptor validation, unavailable segments, and active/passive representation.
- [Process Model](process-model.md) — process data and capability register state.
- [Processor Control](processor-control.md) — special-purpose CPU registers and indicator/fault registers.
- [I/O and Interconnect](io-and-interconnect.md) — protected channel transfer, access-unit read-and-hold, and bus diagnostic evidence.
- [Faults, Interrupts and Startup](faults-interrupts-startup.md) — C(N), Normal Interrupt Block, automatic CHP, SPECIAL/C(S)-rooted startup, and trap/storage-management paths.

This file is the navigation index for the current working PP250 architectural reconstruction. Use the subject documents above for substantive architecture.
