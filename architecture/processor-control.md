# System 250 Architecture — Processor Control

## Special-purpose CPU registers

Page 7 identifies:

| Register | Name / description |
|---|---|
| C10 | C(D) DUMPSTACK |
| C11 | C(I) INTERVAL TIMER |
| C12 | C(C) SCT |
| C13 | C(N) NORMAL INTERRUPT BLOCK |
| C14–C17 | not described |

A separate `C(S)` capability identifies the FAULT START-UP BLOCK and is not assigned one of C10–C17 in the Pocket Reference table.

Named special data registers include D10 (absolute D/S pushdown pointer), D11 (watchdog timer), D12 (first-fault MIF copy), D15 (interrupt accept register), and D17 (IAR).

## Indicator and fault registers

The source identifies MIP (Primary Indicator Register) and MIF (CPU Fault Indicator Register). MIP is saved at Dump Stack offset octal `20`; the dump-stack description says its MIF entry is a copy of the CPU Fault Indicator Register in the OS-specific portion where present.

## Processor context and Internal Mode

```text
+------------------------------------------------------------+
| PROCESSOR                                                  |
| Programmer-visible register set                            |
|   D0-D7: eight 24-bit Data Registers                        |
|   C0-C7: eight 48-bit Capability Registers                  |
|          (C6/C7 have execution-domain roles)                |
+------------------------------------------------------------+
| Special Purpose CPU Registers                              |
|   special capability registers, special data registers     |
|   and the processor control state identified by the manual |
+------------------------------------------------------------+
| Other internal execution machinery                         |
|   instruction sequencing, microcode and fault machinery    |
+------------------------------------------------------------+
```

The general register sets are documented in [EP-E1], paragraph 16, and [EP-P2], processor description; the 24/48-bit sizes are also stated by [EP-L1], section 4.2 (**SECONDARY EVIDENCE**). “Programmer-visible” does not mean that every capability register has an interchangeable role or accepts arbitrary data as its contents.

Blank D13/D14/D16 entries remain unspecified. C(I) designates the timer's store block in [EP-P2]; it is not the watchdog value saved for an individual process.

**Documented mechanism, [EP-P2]; synthesis:** these special capability registers are internal processor architectural state. Ordinary instructions do not normally name and manipulate them as ordinary C0–C7 operands. Defined instructions and processor events use or modify them through microcode: CHP changes C(D); CALL/RET and CHP affect the pushdown and execution state; fault/startup machinery uses C(S). They are not all process state that must be saved with every process.

### Internal Mode is a separate access mechanism

**Documented, [EP-P2], “Internal Mode Operation General” and its restrictions:** possession of an appropriate capability permits addressing internal processor registers through a reserved module-address interpretation. General-purpose instructions can thereby access permitted internal state, with capability bounds restricting the accessible set. This is capability-controlled access, not a conventional privileged/supervisor execution mode that bypasses protection.

The patent's general statement that special registers can be read and altered must be read with its specific restrictions: capability registers are read-only to data stores except for twelve alterable high base bits of C(S); all C(S) bits may be read. The remaining capability-register loading is through capability manipulation mechanisms. Consequently, “internal” must not be strengthened into “never accessible by software,” nor does Internal Mode imply unrestricted fabrication of capability registers. The pocket reference independently lists C(S) in its Internal Mode addressing diagram [EP-R1, p. 7].

**Version boundary:** [EP-P2] describes C(C1)/C(C2), C(L) and C(P), whereas the 1976 table names C(C) and leaves other slots blank. Those later names are evidence for that patent embodiment, not a completed 1976 register map.

### Saved state and research interpretation

The Internal Mode diagram also exposes MIS (Secondary Indicator Register), but MIS is not shown as a saved Dump Stack word in the documented COS/POS/ROS/PDOS layouts [EP-R1, pp. 6–7]. Absence from that table alone does not establish that every MIS bit is transient.

The M/H/T interpretation of MIP/MIF/MIS, MIS08/MIS19 and the MIP04 hypothesis is retained in [the Church/Turing reasoning note](../research/church-turing-dump-stack-access-reasoning.md#19-microprogram-state-clues-to-access-code-interpretation).

### Sources for the relocated material

Source identifiers prefixed `EP-` retain the provenance and verification limits of the execution/process note; this reorganisation does not constitute a new source verification.

- **[EP-R1] PRIMARY EVIDENCE via repository transcription:** user's Plessey *System 250 Pocket Reference Book / Instruction Codes*, Issue 1, May 1976. [Title/contents transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg0-pg2%20transcription.txt); [pp. 3–4 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg3-pg4.pdf); [pp. 5–7 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg5-pg7.pdf). Locators: p. 3 instruction codes, p. 4 access-code diagrams, p. 5 ROS/PDOS structures/state word, p. 6 Dump Stack, p. 7 Special Purpose CPU Registers/Internal Mode. Transcriptions checked; scans not independently rechecked here.
- **[EP-E1] PRIMARY EVIDENCE:** D. M. England, *Architectural Features of System 250* (1972), [repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf). Paragraphs 4–9: topology; 16–20: rights, integrity and SCT; 21–26: domains/CALL/Dump Stack; 30: virtual store and disk capabilities; 31: process management and CPU independence. PDF text inspected; diagram bit boundaries not treated as verified from extraction.
- **[EP-P2] PRIMARY EVIDENCE:** US 4,383,297, *Data processing system including internal register addressing arrangements*, [repository PDF](../patents/US4383297-internal-register-addressing.pdf), [patent text](https://patents.google.com/patent/US4383297A/en). Locators: illustrative embodiment/Figure 1 description; special data and capability registers; Internal Mode Operation General and restrictions. Text read; later register map kept distinct from [EP-R1].
- **[EP-L1] SECONDARY EVIDENCE:** Henry M. Levy, *Capability-Based Computer Systems*, Digital Press, 1984, [chapter 4, “The Plessey System 250”](https://homes.cs.washington.edu/~levy/capabook/Chapter4.pdf), especially sections 4.2–4.5 and Figure 4-1. Read as corroboration and for terminology; it does not override the pocket reference or resolve version conflicts.
