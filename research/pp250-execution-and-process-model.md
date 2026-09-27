# PP250 execution and process model: research reconstruction

Status: research reconstruction, originally 19 September 2026; reorganised 27 September 2026. This note describes execution state and process transitions; it is not an emulator specification.

## Scope and evidence

**Central proposition / Strong inference:** a System 250 process is a hardware-defined unit of execution, not merely an operating-system abstraction. Instructions and processor mechanisms preserve its capability, data, execution and CALL state across suspension. The operating system creates, schedules and manages these execution entities; a physical processor temporarily executes one of them. The Process Dump Stack holds persistent architectural state, but is not itself the entire process.

This reconstruction continues the user's [PP250 Boot Sequence Knowledge investigation](https://chatgpt.com/c/6aae536e-0f68-83eb-914e-d28ce0cc3dee). Evidence labels follow the existing research note and repository policy:

- **Documented / PRIMARY EVIDENCE:** statements in contemporary manuals, papers or patents. Pocket-reference details are checked against the repository transcriptions, whose scan-verification caveat remains applicable.
- **SECONDARY EVIDENCE:** Levy's retrospective account, identified explicitly.
- **Strong inference / INFERENCE:** conclusions connecting documented mechanisms.
- **HYPOTHESIS:** a proposed explanation awaiting confirmation.
- **Unresolved / UNKNOWN:** a missing mechanism or unresolved conflict.

**Verification boundary:** the pocket-reference transcriptions, England's 1972 paper and the online descriptions of patents [P1–P4] were read for this note. Source scans and patent figures were not visually rechecked. Layout-sensitive details not recoverable confidently from text remain open. The 1976 pocket reference anchors the Dump Stack offsets; later patents describe extensions and must not silently redefine that machine. Source identifiers below match those in the boot note where possible.

## Execution context and scope

The processor has eight 24-bit data registers D0–D7 and eight 48-bit capability registers C0–C7 [E1; P2; L1, SECONDARY EVIDENCE for the quoted widths]. C6/C7 have execution-domain roles. C(D) identifies the active Dump Stack; D10 is the absolute Dump Stack pushdown pointer, D11 the watchdog and D17 the IAR [R1, p. 7].

A saved capability word need not contain all 48 live register bits: a compact protected reference can preserve identity and rights while the SCT supplies the segment descriptor on restoration. See [capability representation](pp250-capability-representation.md) and [the SCT](pp250-sct-segments-and-virtual-memory.md) for formats and version qualifications.

Processes execute on processors connected to shared store. That physical setting is described in [System 250 Overall Architecture](system-250-overall-architecture.md). The [processor-control note](pp250-processor-registers-and-internal-mode.md) owns the full register map and Internal Mode access rules.

The scope here is the resumable process context, CALL/RET, CHP and processor mobility. Bootstrap, general capability integrity, OS management policy and M⟨H,T⟩ interpretation are separate subjects; relevant links are collected below.

<a id="6-the-hardware-defined-process"></a>

## 1. The hardware-defined process

**Documented basis:** England describes a process as an execution of a reentrant program, with multiple executions possible [E1, paragraph 31]. His Dump Stack is unique to an execution and preserves registers on context change [paragraph 26]. The suspension patent explicitly describes hardware preservation of process parameters and procedure nesting [P3, background and “Process Dump-Stack”].

**Strong inference / architectural definition:** a process is an execution entity whose architectural context includes:

- capability state, including C0–C5 and current C6/C7 identities and rights;
- D0–D7 data state;
- instruction position and current execution domain;
- primary indicators and per-process watchdog state;
- pushdown state and saved CALL/domain continuations.

```text
Process A: continuing execution identity
           |
           +-- running --> architectural state in a processor
           |              plus persistent Dump Stack information
           |
           +-- suspended --> saved state in Process Dump Stack A
                             plus referenced code/data segments
```

The process is not the physical CPU, nor just a bag of memory bytes. Its Dump Stack is the structure preserving its resumable architectural context, with capability links to other required segments. It need not contain copies of all code/data or every internal CPU register. C(S), SCT-selection machinery and fault history must not automatically be classified as per-process state.

<a id="7-process-dump-stack-1976-format"></a>

## 2. Process Dump Stack: 1976 format

**Documented, [R1], p. 6. All offsets below are octal.** The common fixed portion `0–20` contains seventeen words:

| Offset | Saved state | Offset | Saved state |
|---:|---|---:|---|
| 0 | C0 | 10 | D2 |
| 1 | C1 | 11 | D3 |
| 2 | C2 | 12 | D4 |
| 3 | C3 | 13 | D5 |
| 4 | C4 | 14 | D6 |
| 5 | C5 | 15 | D7 |
| 6 | D0 | 16 | Pushdown pointer for CALL stack |
| 7 | D1 | 17 | Watchdog timer register |
| | | 20 | MIP (Primary Indicator Register) |

The OS-dependent portion is summarized below. “Blank” means blank in the source table, not necessarily unused or zero. Combined rows preserve multiword entries and frame boundaries.

| OS | Entries following fixed portion | Initial C6 / C7 / IAR | First subroutine C6 / C7 / IAR | Second subroutine C6 / C7 / IAR shown |
|---|---|---|---|---|
| COS | Initial frame immediately follows | 21 / 22 / 23 | 24 / 25 / 26 | 27 / 30 / 31 |
| POS | 21–23 blank | 24 / 25 / 26 | 27 / 30 / 31 | 32 / 33 / 34 |
| ROS | 21 MIF; 22 LOKK; 23 Error Control; 24 Process Base; 25 SIP | 26 / 27 / 30 | 31 / 32 / 33 | Not shown |
| PDOS | 21 MIF; 22 LOKK; 23–25 three Ptarmigan words; 26 Error Control; 27 Process Base; 30 SIP | 31 / 32 / 33 | 34 / 35 / 36 | Not shown |

The initial entries are labelled **C6 Initial**, **C7 Code**, **IAR Block**; subsequent triples are associated with subroutines. The source defines MIF as a copy of the CPU Fault Indicator Register, LOKK as used by privileged system facilities, Error Control as the Process Error Control Parameter from the Process Template, and SIP as the State and Internal Priority Word. “Privileged system facilities” here does not establish a processor supervisor mode. LOKK must not be conflated with the separately documented lock state associated with synchronising flags; its exact semantics remain **UNKNOWN**.

MIP is saved for all four operating systems shown; MIF is additionally saved by ROS/PDOS; MIS is not shown as a saved Dump Stack word. These are save-layout observations, not a complete classification of internal processor state. Their M/H/T interpretation is developed in [the access reasoning note](church-turing-dump-stack-access-reasoning.md#22-save-layout-constraint-on-the-mht-interpretation).

```text
Dump Stack
+--------------------------------------+
| fixed process save area, 0-20 octal   |
|   C0-C5; D0-D7; pointer; watchdog; MIP|
+--------------------------------------+
| OS-specific intervening words        |
+--------------------------------------+
| C6 Initial | C7 Code | IAR Block      |
+--------------------------------------+
| C6         | C7      | IAR            | CALL frame
+--------------------------------------+
| further nested frames ...            |
+--------------------------------------+
              ^
      current pushdown state connects
      execution with the active frame
```

**Documented, [P3], background:** the fixed area preserves machine state at suspension and the variable area preserves procedure links. **Strong inference:** the saved pushdown pointer supplies the active-frame location rather than hardware selecting a COS/POS/ROS/PDOS-specific initial constant.

**Pointer qualification:** [R1], p. 7, calls D10 an absolute pushdown pointer. [P3], “Process Dump-Stack,” distinguishes the running next-free location from the dumped pointer to the last written IAR word. These later semantics support the reconstruction but require confirmation for the 1976 CPU. An IAR's relative table offset is not necessarily the literal word-16 value: stack base and pointer encoding matter. A suspended process with nested CALLs resumes its active frame, not invariably the initial frame.

[P3] also extends links with local-store and optional saved-register information. Those extensions are not inserted into the 1976 table above.

<a id="8-c6-c7-and-execution-domains"></a>

## 3. C6, C7 and execution domains

**Documented, [E1], paragraphs 21–26:** C6 supplies the current node's main capability block, while C7 supplies its code block. This is the capability environment through which the code can reach authorized objects. The term **Central Capability Block** is also used in [L1, sections 4.3–4.4, SECONDARY EVIDENCE].

```text
C6 --> current capability environment --> accessible segments
C7 --> current code segment
IAR --> current instruction position within that code context
```

**Documented, [P2]:** the live IAR contains an absolute instruction address within the C7-defined block. [P3] describes a relativized saved instruction address. These are different representations of execution position; this note does not prescribe an unverified 1976 encoding or increment order.

C6 is **not inherently an OS Process Base pointer**. A particular management domain may use its C6 environment to reach a Process Base or may arrange for that block itself to be the current capability block. That is a structure and calling-context question, not the definition of C6. C0–C5 can carry capabilities across a domain call [E1, paragraph 25], so describing C6 as the current environment must not imply that all other held authority disappears on CALL.

<a id="9-call-versus-chp"></a>

## 4. CALL versus CHP

| Operation | Execution entity | Dump Stack | State transition |
|---|---|---|---|
| CALL | Same process | Same process's stack | Enter another procedure/domain and preserve a return context |
| CHP (Change Process) | Outgoing process suspended; incoming process activated | Outgoing and incoming stacks | Transfer the resumable process context |

**Documented, [E1], paragraphs 24–26:** CALL uses an enter capability and an offset selecting code, establishes the called domain's C6/C7, and preserves the caller's context for RETURN. [R1] records the C6/C7/IAR frames; [P3] explicitly distinguishes stack updates by CALL from a process change involving two Dump Stacks. CALL does not itself create another hardware process.

**Provenance correction:** the recalled example `CHP 3 0 C6` has been withdrawn; see the [erratum](ERRATUM-CHP-3-0-C6.md). It supplies no evidence for CHP syntax, C6 selection, offset 3 or operand semantics.

[P1], following fault step S17, independently describes normal CHANGE PROCESS using an instruction-supplied offset through a reserved segment-pointer table and the master capability table to obtain the incoming dump area. This observation does not establish the withdrawn assembly example.

Established instruction formats are treated in [the instruction-set architecture](../architecture/instruction-set.md). Competing creation/resumption interpretations, operand questions and OS-specific structural evidence are maintained in [CHP process creation and resumption](chp-process-creation-and-resumption-hypothesis.md). The process-level distinction above does not depend on selecting one of those hypotheses.

<a id="10-cd-and-process-switching"></a>

## 5. C(D) and process switching

**Documented, [R1], p. 7, and [P2], “Capability Register C(D)”:** C(D) is the special capability for the active process's Dump Stack and is changed by CHANGE PROCESS. [P3] describes the two-stack operation. The conceptual transition is:

```text
                         PROCESSOR
Before: process A active     C(D) ----------------> Dump A
                                    save outgoing state

Incoming operand --------------------------------> Dump B
                                    restore incoming state
After:  process B active     C(D) ----------------> Dump B
                         C6 / C7 / IAR = B's context
```

**Strong inference / conceptual effects, not an ordered microprogram:** outgoing data, indicators, watchdog and execution-frame state become resumable in Dump A; the incoming capability identifies Dump B; C(D) and the restored architectural context come to describe B. Incoming C6/C7/IAR establish its environment and instruction position. Restoring capability state may require reconstructing expanded registers from compact pointers through the SCT.

“Complete context” here means the process's resumable execution context, not every register or transient microcode latch in the CPU. The precise sequencing, already-maintained pointer entries and interrupted-instruction rules remain reconstruction questions. Cold-entry treatment of an invalid old C(D) belongs to [startup reconstruction](pp250-boot-and-processor-startup.md#execution-model-boundary-and-startup-provenance). It must not become literal emulator pseudocode by accident.

<a id="11-process-versus-processor-multiprocessor-mobility"></a>

## 6. Process versus processor: multiprocessor mobility

**Strong inference from the save/restore mechanism:** a process is not intrinsically attached to one CPU. Its saved context and referenced segments can supply execution on another available processor, subject to compatible system state and scheduling/exclusion rules.

```text
CPU 0 executes A --> suspend into Dump A in shared store
                                      |
                               scheduling decision
                                      |
                                      v
CPU 1 resumes A  <-- restore from the same Dump A
```

There is also direct contemporary support: **[E1], paragraph 31(2)** describes a priority-organized common ready list and says the identity of the CPU scheduling its head process is transparent to that process. Paragraph 5 says CPUs share work and no function is dedicated to a particular CPU. This documents England's described system, rather than proving every subsequent ROS/PDOS scheduler implementation.

**Documented, [R1], p. 5:** the ROS/PDOS state word records whether a process is running, its CPU number when running, priority and ready-list status. Recording the current CPU is not evidence of permanent affinity. Exact queue operations, locks, simultaneous-activation prevention, scheduling policy and fault/rejoin migration in each OS remain **UNKNOWN**. “Can resume elsewhere” must not be read as “may run the same mutable saved context concurrently on two processors.”

<a id="12-os-process-base-versus-hardware-process"></a>

## 7. OS Process Base versus hardware process

**Strong inference:** Process Base is the OS management structure layered above the architectural process. The Dump Stack supplies resumable execution state; management fields and conventions support software operations. The differing COS/POS/ROS/PDOS layouts do not make one OS Process Base format the hardware definition of a process.

The ROS/PDOS Process Base diagram and offset-3 Dump Stack link are retained as OS-specific evidence in [the CHP reconstruction note](chp-process-creation-and-resumption-hypothesis.md#relationship-to-the-rospdos-pocket-reference-diagram). The `666`/`760` decoding and its format qualification are in [the access reasoning note](church-turing-dump-stack-access-reasoning.md#21-rospdos-link-access-code-interpretation).

The frame comparison above records where OS words occur. It does not establish scheduling policy, the semantics of LOKK, Error Control or Ptarmigan fields, or a universal process-management interface.

## Relocated background

<a id="1-physical-system-250"></a>

Physical topology and the bus diagram: [System 250 Overall Architecture](system-250-overall-architecture.md).

<a id="2-processor-register-architecture"></a>
<a id="internal-mode-is-a-separate-access-mechanism"></a>

Register architecture and Internal Mode: [Processor Control](pp250-processor-registers-and-internal-mode.md).

<a id="mip-mif-and-mis-persistent-versus-transient-processor-state"></a>

MIP/MIF/MIS research interpretation: [save-layout constraint](church-turing-dump-stack-access-reasoning.md#22-save-layout-constraint-on-the-mht-interpretation).

<a id="3-stored-and-expanded-capabilities"></a>

Stored and expanded capability formats: [Capability Representation](pp250-capability-representation.md).

<a id="4-sct-segments-and-virtual-memory"></a>

SCT, segments and virtual memory: [System Capability Table](pp250-sct-segments-and-virtual-memory.md).

<a id="5-capability-integrity-the-unresolved-mixed-access-case"></a>
<a id="current-access-code-decoding"></a>

Mixed-access integrity and access-code decoding: [access reasoning](church-turing-dump-stack-access-reasoning.md#20-mixed-access-dump-stack-observations-and-unresolved-mechanisms).

<a id="13-relationship-to-boot-and-fault-research"></a>

Startup and fault context: [Boot and Processor Startup](pp250-boot-and-processor-startup.md#execution-model-boundary-and-startup-provenance).

## Related research and remaining boundaries

- [Boot and processor startup](pp250-boot-and-processor-startup.md): how the first executable context is established, C(S), checkout and cold-entry questions.
- [Capability genesis and resource lifecycle](capability-genesis-and-resource-lifecycle.md): where the required authority originates.
- [Normal interrupt and system dispatch](pp250-normal-interrupt-and-system-dispatch.md): C(N)-rooted entry and operational dispatch, distinct from startup/fault entry.
- [Church/Turing/Dump Stack reasoning](church-turing-dump-stack-access-reasoning.md): mixed-access integrity, access/form interpretation and the developing M⟨H,T⟩ account.
- [CHP creation and resumption](chp-process-creation-and-resumption-hypothesis.md): competing lifecycle mechanisms and detailed instruction questions.

The saved active-frame pointer and resumable-state reconstruction remain part of this model. Broader theories and bootstrap mechanisms are not prerequisites for understanding the execution-state transitions described here.

## Sources and provenance

- **[R1] PRIMARY EVIDENCE via repository transcription:** user's Plessey *System 250 Pocket Reference Book / Instruction Codes*, Issue 1, May 1976. [Title/contents transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg0-pg2%20transcription.txt); [pp. 3–4 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg3-pg4.pdf); [pp. 5–7 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg5-pg7.pdf). Locators: p. 3 instruction codes, p. 4 access-code diagrams, p. 5 ROS/PDOS structures/state word, p. 6 Dump Stack, p. 7 Special Purpose CPU Registers/Internal Mode. Transcriptions checked; scans not independently rechecked here.
- **[E1] PRIMARY EVIDENCE:** D. M. England, *Architectural Features of System 250* (1972), [repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf). Paragraphs 4–9: topology; 16–20: rights, integrity and SCT; 21–26: domains/CALL/Dump Stack; 30: virtual store and disk capabilities; 31: process management and CPU independence. PDF text inspected; diagram bit boundaries not treated as verified from extraction.
- **[E2] PRIMARY EVIDENCE, contextual research lead:** D. M. England, *Operating System of System 250*, International Switching Symposium, June 1972, in the [repository ISS collection](../sources/1972/1972-06-System-250-ISS-Papers-MIT.pdf). Detailed OS claims above rely on [E1], not an assumed reading of this collection.
- **[E3] PRIMARY EVIDENCE, research lead:** D. M. England, *Capability Concept Mechanism and Structure in System 250*, International Workshop on Protection in Operating Systems, August 1974. Bibliographic reference in [Levy's bibliography](https://homes.cs.washington.edu/~levy/capabook/Bibliography.pdf); no copy in the inspected repository tree. Not used as proof of an unverified mechanism.
- **[P1] PRIMARY EVIDENCE:** Plessey, US 3,814,919, *Fault detection and isolation in a data processing system*, [patent text](https://patents.google.com/patent/US3814919A/en). Locators: capability parity fault; fault microsequence S2/S10/S16/S17; automatic and normal CHANGE PROCESS discussion immediately afterwards. Retrieved directly and read for this note; not archived in the inspected repository.
- **[P2] PRIMARY EVIDENCE:** US 4,383,297, *Data processing system including internal register addressing arrangements*, [repository PDF](../patents/US4383297-internal-register-addressing.pdf), [patent text](https://patents.google.com/patent/US4383297A/en). Locators: illustrative embodiment/Figure 1 description; special data and capability registers; Internal Mode Operation General and restrictions. Text read; later register map kept distinct from [R1].
- **[P3] PRIMARY EVIDENCE:** US 4,486,831, *Multi-programming data processing system process suspension*, [repository PDF](../patents/US4486831-process-suspension.pdf), [patent text](https://patents.google.com/patent/US4486831A/en). Locators: background's existing two-part Dump Stack; summary's extensions; Process Dump-Stack/Figure 5 description. Text read; later local-store/extended-link facilities are not backdated to 1976.
- **[P4] PRIMARY EVIDENCE:** US 4,408,274, *Memory protection system using capability registers*, [repository PDF](../patents/US4408274-memory-protection-capability-registers.pdf), [patent text](https://patents.google.com/patent/US4408274A/en). Locators: Description of Prior Art, Capability Formats, LC/load-on-use and SC descriptions, Figures 5–9 references. Text read; enhanced pointer format and propagation controls are version-qualified.
- **[L1] SECONDARY EVIDENCE:** Henry M. Levy, *Capability-Based Computer Systems*, Digital Press, 1984, [chapter 4, “The Plessey System 250”](https://homes.cs.washington.edu/~levy/capabook/Chapter4.pdf), especially sections 4.2–4.5 and Figure 4-1. Read as corroboration and for terminology; it does not override the pocket reference or resolve version conflicts.

Originally prepared against repository commit `43547b29cbd426a3dfda6f8dce464f4653cb3252`. Reorganised from `1a4772c8beaf023edd7465c5fa232b8e3a98c995` on 27 September 2026: supporting material relocated to the linked subject notes, and the withdrawn CHP recollection removed as evidence. Original sources and transcriptions were not modified.
