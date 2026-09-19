# PP250 boot and processor startup: research reconstruction

Status: research note, 19 September 2026. This is not an emulator specification or a verified cold-start microprogram.

## Scope and evidence

This note records the reconstruction developed in [PP250 Boot Sequence Knowledge](https://chatgpt.com/c/6aae536e-0f68-83eb-914e-d28ce0cc3dee). It separates starting a processor from loading an initially empty system memory.

Evidence labels follow the repository policy:

- **Documented / PRIMARY EVIDENCE**: a statement in an original manual or patent. Pocket-reference details below were checked against the repository transcription, not rechecked against the scans.
- **SECONDARY EVIDENCE**: Levy's retrospective description.
- **Strong inference / INFERENCE**: a conclusion connecting documented mechanisms, rather than an explicit source statement.
- **HYPOTHESIS**: a plausible explanation awaiting evidence.
- **Unresolved / UNKNOWN**: a missing mechanism or unresolved detail.

**Verification boundary:** the patent findings below are preserved from the referenced investigation, with patent identifiers and passage locators. Online patent text could not be retrieved during preparation of this note, so those passages have not been independently reverified here. “Documented (reported)” means the investigation attributed the detail to a primary source; it does not make the conversation itself primary evidence. The England papers and Levy provide context, not proof of an exact cold-start sequence.

## 1. Reconstructed power-up path

**Documented (reported), [P2]:** C(S) defines a four-word block used for fault interrupts. The processor presets the register following power-up except for the twelve most significant base bits associated with fault-sequence incrementing. This supplies a hardware-established starting capability without first requiring an ordinary SCT lookup.

**Strong inference:** power-up uses that root to establish a restricted capability environment, obtain an initial Dump Stack, and restore a process through the CHANGE PROCESS machinery. The corresponding fault-recovery sequence is much better documented than initial system loading.

```text
POWER UP
   |
   v
C(S) preset by processor                    [reported primary evidence]
   |
   v
Special Fault/Startup Block                 [cold-start connection inferred]
   |
   +--> establish restricted/special SCT
   |
   +--> RSPC-0 --> SCT lookup --> Dump Stack capability
                                      |
                                      v
                            C(D) / CHANGE PROCESS
                                      |
                                      v
                           restore fixed state and
                           initial C6 / C7 / IAR frame
                                      |
                                      v
                         FIRST ORDINARY PP250 INSTRUCTION
```
SCT is System Capability Table
RSPC-n is Reserved Segment Pointer Capability n

This diagram assumes the required memory structures already exist and are valid. It does not explain how they were first loaded. C(S) identifies a data structure; it should not be represented as a direct pointer to bootstrap instructions.

“CHP” in this startup diagram denotes the automatic change-process operation. It does not imply that the uninitialised processor first fetches an ordinary CHP instruction.

## 2. Special Fault/Startup Block and the SCT

**Documented (reported), [P1]:** the Special Block Capability Register (SSCR) selects a four-word per-processor entry in the Special Fault Block. The investigation identifies this with the C(S) fault/startup role described in [P2] and the pocket reference [R1]. The correspondence is functional; terminology across the sources must not be treated as proof that every implementation detail is identical.

| Word | Contents reported in the fault patent |
|---:|---|
| 0 | Sum-check |
| 1 | Base address of the special capability table |
| 2 | Limit/type information for that table |
| 3 | RSPC-0, the reserved-segment pointer identifying the checkout process Dump Stack |

**Documented (reported), [P1]:** fault microcode loads words 1 and 2 into the Master Capability Register (MCR), with parity and sum-check validation. The resulting special table restricts what the recovering processor can address. RSPC-0 is then interpreted using this table to obtain the checkout Dump Stack capability, followed by automatic CHANGE PROCESS.

RSPC-0 is therefore not simply a raw instruction address, nor should it be conflated with the expanded capability ultimately installed in processor state.

The earlier patent's MCR terminology, the pocket reference's C(C)/C12 SCT register, and the later patent's C(C1)/C(C2) arrangement require version-aware comparison. This note uses “establish the SCT environment” without silently assigning the early four words to a later register layout.

## 3. C(D), CHP and the first executable context

**Documented, [R1], p. 7:** C10 is C(D), the Dump Stack capability register. It identifies the current process Dump Stack in the processor description [P2, reported].

**Documented (reported), [P3]:** a process has a fixed dump area for processor state and a variable pushdown area for procedure state. CHANGE PROCESS saves/restores process state using this structure.

**Strong inference:** a never-run process can be represented as a synthetic suspended process. Its preconstructed Dump Stack supplies the state that the change-process restore consumes. The incoming Dump Stack capability becomes C(D) as part of this transition; the exact internal ordering is not specified here.

```text
Processor                         Incoming Dump Stack
 C(D) --------------------------> fixed saved state
                                      |
                               saved pushdown pointer
                                      |
                                      v
                                  C6 Initial
                                  C7 Code
                                  IAR Block
                                      |
                                      v
                           restored execution context
```

C6, C7 and IAR are not all in the fixed area. The saved pushdown pointer connects the fixed state to the active execution frame. The first ordinary instruction can execute only after a valid C7 code capability and IAR have been established; a fault block or an SCT alone does not supply an executable process context.

The OS Process Base and the hardware Dump Stack should be distinguished. The ROS/PDOS diagram places a Dump Stack pointer at Process Base offset 3 and a backlink in the Dump Stack [R1, p. 5]. **Inference:** this is an OS management arrangement around the execution state, not evidence that startup hardware must understand every OS's Process Base layout.

## 4. Fixed Dump Stack offsets

**Documented, [R1], p. 6:** the following fixed area is common to COS, POS, ROS and PDOS. **All offsets are octal.** Thus 0 through 20 is seventeen words, not twenty-one decimal words.

| Octal offset | Saved state |
|---:|---|
| 0 | C0 |
| 1 | C1 |
| 2 | C2 |
| 3 | C3 |
| 4 | C4 |
| 5 | C5 |
| 6 | D0 |
| 7 | D1 |
| 10 | D2 |
| 11 | D3 |
| 12 | D4 |
| 13 | D5 |
| 14 | D6 |
| 15 | D7 |
| 16 | Pushdown pointer for call stack |
| 17 | Watchdog timer register |
| 20 | MIP (Primary Indicator Register) |

Page 7 identifies D10 as the **absolute** Dump Stack pushdown pointer. The investigation reports that [P2/P3] describe DSPPR as pointing to the saved IAR when dumped during process change, whereas the running pointer identifies the next available stack location.

**Important qualification:** the IAR offsets below are locations within the Dump Stack, not necessarily the literal contents of word 16. A virgin saved pointer must identify the appropriate absolute IAR location under the actual pointer encoding. Writing “word 16 = 23” for every COS stack would discard the stack's base address.

## 5. OS-dependent initial frames

**Documented, [R1], p. 6; all values octal:**

| OS | C6 Initial | C7 Code | IAR Block |
|---|---:|---:|---:|
| COS | 21 | 22 | 23 |
| POS | 24 | 25 | 26 |
| ROS | 26 | 27 | 30 |
| PDOS | 31 | 32 | 33 |

Do not normalise these layouts. The transcription leaves POS words 21–23 blank; that does not establish that they are zero or unused. ROS includes MIF, lock, error control, Process Base and SIP before its initial frame. PDOS additionally has three Ptarmigan words and moves error control, Process Base and SIP accordingly.

```text
fixed area: 00 ... 15 | 16 pointer | 17 watchdog | 20 MIP
                            |
                            +--> OS-dependent active frame
                                 [ C6 ][ C7 ][ IAR ]
                                               ^
                                      saved pointer target
```

**Strong inference:** restoring the saved pointer lets the hardware locate the active frame without selecting an OS-specific constant. A virgin stack must therefore contain both genuine initial capabilities and a correctly initialised saved pointer before its first restore. Subsequent nested frames repeat C6/C7/IAR; the initial frame is not necessarily the active one in a previously running process.

## 6. Fault recovery

**Documented (reported), [P1]:** fault entry reverses the internal capability parity convention, making the interrupted process's existing capability state invalid. This is isolation before recovery, not merely a branch to a handler with the old authority intact. Do not generalise this into a claim that every stored capability throughout system memory is invalidated.

```text
RUNNING PROCESS
      |
     FAULT
      |
      v
reverse internal capability parity convention
      |
      v
prior processor capability state invalid
      |
      v
C(S) / SSCR --> Special Fault Block
      |
      v
restricted SCT --> RSPC-0 --> checkout Dump Stack
      |
      v
automatic CHANGE PROCESS --> checkout process
```

The reported sequence also sets FIRST ATTEMPT, records fault information and controls interrupts/interface recovery. The module-selecting part of SSCR initially selects module zero; further faults can advance to equivalent fault structures in another store module. This supports recovery without trusting the failed execution environment [P1, reported].

Checkout and eventual entry to startup-supervisor/rejoin software are subsequent software activity. They should not be confused with an ordinary application resuming immediately after the fault.

## 7. Finite recovery transition, not a persistent supervisor mode

**Strong inference:** C(S) and fault microcode provide a finite hardware recovery transition whose successful outcome is another capability-controlled process. They do not establish a persistent, unrestricted supervisor instruction environment underneath ordinary execution.

This does **not** mean C(S) disappears after CHP or is forever inaccessible. The reported internal-register patent allows all its bits to be read and only twelve base bits to be altered through Internal Mode [P2]; the pocket reference explicitly includes C(S) in its Internal Mode addressing diagram [R1, p. 7]. Internal register access must be distinguished from autonomously executing the fault microsequence.

The earlier conversational suggestion that C(S) is entirely inaccessible after startup is therefore too strong. Likewise, this note does not prove that software can never provoke a fault. The architectural claim concerns the absence of a continuing unrestricted execution mode, not the impossibility of triggering recovery.

## 8. Cold startup and recovery: likely common machinery

**Strong inference:** both transitions need a trusted process context when no trustworthy current context exists. C(S), a special SCT, a Dump Stack and change-process restoration form a plausible common path.

Three situations must remain distinct:

| Situation | Memory prerequisite | Evidence status |
|---|---|---|
| Fault recovery or processor rejoin | Valid recovery structures exist | Fault mechanism documented in [P1], as reported |
| Processor startup with retained/preloaded memory | Valid startup structures exist | Common path strongly inferred |
| Whole-system startup with empty/invalid memory | Structures must first be created or loaded | Loading mechanism unresolved |

**UNKNOWN:** whether cold-start and fault-start microsequences are identical. Shared architectural machinery does not prove identical entry conditions, parity handling, outgoing-state treatment or microinstruction order.

## 9. Unresolved questions and research leads

1. **Virgin memory and stored capabilities:** who initially loads the Special Fault Block, SCT, Dump Stack and checkout code? How are genuine stored capabilities established before any ordinary process can execute? Raw data bit patterns must not simply be assumed to confer capability authority.
2. **Initial pointer and template:** which system-generation or process-template operation constructs the initial C6/C7/IAR frame and absolute saved pushdown pointer? The pocket reference mentions a Process Template in its error-control definition, but does not supply the construction algorithm.
3. **Cold versus fault entry:** are their microsequences identical, or do they merely converge? What happens to outgoing-state saving when no valid old C(D) exists?
4. **Capability validity:** what precise storage and instruction rules prevent capability fabrication in a Dump Stack containing both data and capabilities? The conversation considered and then questioned a per-word tag explanation; no tag implementation is established here.
5. **Version correspondence:** how do SSCR/MCR, C(S)/C(C), and C(C1)/C(C2) map across processor descriptions? Which later-patent details apply to the user's 1976 machine?
6. **Loading equipment:** maintenance hardware, another processor, retained memory, tape/disk loading and INFORM/OUTFORM are research possibilities, not established boot mechanisms. No ROM bootstrap is established or ruled out by this note.

Priority evidence targets are the original fault-patent figures and microsequence, processor startup/maintenance manuals, process-template documentation, and the repository's System 250 General Information material. These findings extend questions left open in [the architecture WIP](../architecture/architecture.md); they do not silently amend that document.

## Sources and provenance

- **[R1] PRIMARY EVIDENCE via transcription:** user's *System 250 Pocket Reference Book / Instruction Codes*, Issue 1, May 1976. Pages 5–7 cover Process Base, Dump Stack and special registers. [Repository transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt); [source scan](../documentation/System%20250%20Pocket%20Reference%20pg5-pg7.pdf). Exact offsets above were checked against the transcription; its scan-verification caveat remains applicable.
- **[P1] PRIMARY EVIDENCE, findings reported in prior investigation:** Plessey, US 3,814,919, *Fault Detection and Isolation in a Data Processing System*, filed 1 March 1972. [Patent](https://patents.google.com/patent/US3814919A/en). Relevant locators: SSCR, Special Fault Block, RSPC-0, parity reversal, FIRST ATTEMPT, automatic CHANGE PROCESS and checkout/rejoin. Not currently archived in the inspected repository tree.
- **[P2] PRIMARY EVIDENCE, findings reported:** US 4,383,297, *Data processing system including internal register addressing arrangements*. [Repository PDF](../patents/US4383297-internal-register-addressing.pdf); [patent](https://patents.google.com/patent/US4383297A/en). Relevant locators: C(S), power-up preset, Internal Mode, C(D), DSPPR and special capability registers.
- **[P3] PRIMARY EVIDENCE, findings reported:** US 4,486,831, *Multi-programming data processing system process suspension*. [Repository PDF](../patents/US4486831-process-suspension.pdf); [patent](https://patents.google.com/patent/US4486831A/en). Relevant locators: fixed and variable dump-stack portions, CHANGE PROCESS, saved pushdown pointer and procedure state.
- **[P4] PRIMARY EVIDENCE, supporting research lead:** US 4,408,274, *Memory protection system using capability registers*. [Repository PDF](../patents/US4408274-memory-protection-capability-registers.pdf); [patent](https://patents.google.com/patent/US4408274A/en). Relevant to capability representation/protection; not cited as proof of cold loading.
- **[E1] PRIMARY EVIDENCE, contextual reference:** D. M. England, *Architectural Features of System 250* (1972). [Repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf).
- **[E2] PRIMARY EVIDENCE, contextual reference to locate/check:** D. M. England, *Capability Concept Mechanism and Structure in System 250*, International Workshop on Protection in Operating Systems, IRIA, August 1974. No copy was identified in the inspected tree. Neither England paper is used here to establish undocumented cold-start steps.
- **[L1] SECONDARY EVIDENCE:** Henry M. Levy, *Capability-Based Computer Systems*, Digital Press, 1984, chapter 4, “The Plessey System 250.” [Author's book and chapter downloads](https://homes.cs.washington.edu/~levy/capabook/). The prior investigation cites the Start-up Block for recovery after failure, SCT translation and process context. These support context, not a literal power-on microsequence.
- **[G1] PRIMARY EVIDENCE, next research target:** [Plessey System 250 General Information (1972)](../sources/1972/1972-Plessey-System-250-General-Information.pdf). Initial loading and commissioning procedures remain to be established from suitable contemporary material.

Prepared from the referenced discussion and repository state at commit `88e9f4bc13d8ba63606bcd6e61f4249610abeab6`. No original source or transcription was modified.
