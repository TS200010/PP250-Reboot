# US3757307A — Program interrupt facilities in data processing systems

## Source status

**Primary source.**

This file records the machine-readable source and the architecture-critical content of the patent corresponding to British application **41951/70**.

The complete patent specification is available as searchable machine-readable HTML from Google Patents:

- US publication: https://patents.google.com/patent/US3757307A/en
- Publication: **US3757307A**
- Title: **Program interrupt facilities in data processing systems**
- US application: **US00176464A**
- Great Britain priority application: **41951/70**
- Priority date: **1970-09-02**
- US filing date: **1971-08-31**
- US publication/grant date: **1973-09-04**
- Inventors listed by Google Patents: M. O. Halloran, F. Trapnell, D. Cosserat, J. Cotton
- Assignee listed: Plessey Overseas Ltd (Google also records Plessey AG as original assignee)

Related British publication identified during the source search:

- **GB1332797A**

## Transcription/provenance note

The Google Patents HTML is machine-readable OCR of the patent specification and includes OCR defects. It should therefore be treated as a searchable transcription, not as a diplomatic transcription of the printed patent.

This repository record deliberately separates:
1. facts and terminology explicitly present in the patent;
2. later PP250 terminology;
3. reconstruction.

A complete verbatim OCR import has not yet been made into this repository. The canonical machine-readable source URL above is retained so that such an import can be made and checked against the patent images.

## Architectural terminology in this patent

This is an early form of the architecture. The patent uses terminology which predates the later Pocket Reference names.

Processor capability registers described include:

- workspace capability registers WCR0–WCR7;
- **DCR** — Dump Area Capability Register;
- **ICR** — capability defining the System Interrupt Word storage area;
- **MCR** — capability defining the Master Capability Table;
- **LSCR** — capability defining the processor's Dedicated Local Start-up Area.

The patent says WCR6 conventionally defines the main reserved-segment-pointer table for the current process, while WCR7 defines the current instruction segment.

The later C(D), C(I), C(C), C(N)/C(S) terminology must not simply be substituted into this source without documenting the historical mapping.

## Dedicated Local Start-up Area

Each processor has a **Dedicated Local Start-up Area (DLSA)** addressed through LSCR.

The patent explicitly establishes that this area contains at least:

- the processor's **Interrupt Accept Mask Word (IMW)**;
- information identifying the Interrupt Handler Process Dump Area, described during the process-change sequence as **IDAP**.

The patent states that the local start-up area stores both the interrupt accept mask and a link to the interrupt-handler programme.

This is important evidence for the ancestry of the later **Normal Interrupt Block** described in System 250 material.

No word offsets are assigned here unless recovered directly from the patent figures/text.

## Interrupt state

The processor includes:

- **STR** — Scheduler Timer Register;
- **ITR** — Interval Timer Register;
- **IAR** — Interrupt Accept Word Register;
- **DSPPR** — Dump Stack Push-Down Pointer Register.

The interrupt interrogation trigger is activated periodically by an interrupt clock generator (the embodiment suggests about 100 microseconds) and asynchronously when STR or ITR reaches zero.

The IAR is significant because the accepted interrupt identity is placed in it and the patent explicitly states that IAR is **not included in the dump/undump operations**. It therefore survives the process change and tells the newly entered Interrupt Handler Process why it was entered.

## Interrupt interrogation and process-change sequence

Figure 7 and the accompanying description give the following sequence.

1. Finish the current instruction.
2. Read the processor's interrupt accept mask IMW from its DLSA through LSCR.
3. Access the System Interrupt Word through ICR using a read-modify-write operation, locking the relevant storage module during acceptance.
4. Merge the SIW with the interrupt mask.
5. If there is no acceptable interrupt, write the SIW back unchanged and continue.
6. If one or more acceptable bits are present, correlate/select an accepted bit.
7. Place the accepted interrupt identity in IAR.
8. Clear the accepted SIW bit.
9. Write the modified SIW back and release the storage-module lock.
10. **Dump the parameters of the currently running process into the Dump Area defined by DCR.**
11. Obtain the Interrupt Handler Process Dump Area pointer (IDAP) from the processor's dedicated local start-up area.
12. Resolve/load the Interrupt Handler Process Dump Area descriptor into DCR.
13. **Undump the Interrupt Handler Process parameters from its Dump Area into the processor.**

The patent then explicitly describes the change process to the Interrupt Handler as complete.

This establishes that entry to the Interrupt Handler Process is not merely a CALL executed in the interrupted process's existing context. The currently executing process is dumped and the handler's independent processor context is restored.

## Handler interrupt masking

The specification describes a software operation at entry to the Interrupt Handler Process which can make the handler non-interruptable by swapping zero into the DLSA interrupt mask while retaining the old mask in an accumulator register.

At the end of the handler, a corresponding SWAP restores the original mask.

This is further evidence that the handler executes as an ordinary protected process after the automatic process change, rather than as a special hidden processor mode.

## Important unresolved question

The patent establishes how the processor enters the Interrupt Handler Process, but the repository still needs to establish precisely how the handler/process manager subsequently obtains authority to select and resume the process that was interrupted.

Do not infer a conventional interrupt-return instruction.

In particular, distinguish:

- the hardware-preserved identity of the accepted interrupt (IAR);
- the outgoing process state saved through the old DCR;
- IDAP, which identifies the **incoming Interrupt Handler Process Dump Area**;
- whatever operating-system structure identifies the previously running/suspended process for subsequent scheduling.

## Relationship to later System 250 evidence

Halton's 1972 description says C(N) defines the **Normal Interrupt Block**, and Figure 7 is transcribed as C(N) pointing to a mask word and a pointer to interrupt block.

The 1970-priority patent provides strong earlier evidence for the same architectural pattern:

```
processor-local protected capability
        |
        v
dedicated interrupt/start-up area
        |
        +-- interrupt accept mask
        |
        +-- pointer/link identifying
            Interrupt Handler Process
            Dump Area
```

However, the exact historical equivalence

```
LSCR / DLSA  <->  C(N) / Normal Interrupt Block
```

must remain a reconstruction until the evolution between the documents is explicitly established. Later System 250 sources also distinguish C(S) (start-up/fault start-up) from C(N) (normal interrupt), whereas this earlier patent's DLSA participates directly in interrupt handling.

## Related sources

- British priority application: **41951/70**
- British publication: **GB1332797A**
- US family member: **US3757307A**
- Later trap-related patent: **US3771146**
- System 250 paper: `transcriptions/hardware-of-the-system-250-for-communication-control.md`
- Pocket Reference: `transcriptions/System 250 Pocket Reference pg5-pg7 transcription.txt`
- Existing reconstruction: `research/pp250-normal-interrupt-and-system-dispatch.md`

## Follow-up

A useful next source-preservation step is to import and proof the complete US3757307A specification (including Figure 6 and Figure 7 labels) against the patent images. The machine-readable Google Patents text contains obvious OCR corruption, so architectural names, offsets and figure labels should be checked visually before being promoted into the architecture documentation.
