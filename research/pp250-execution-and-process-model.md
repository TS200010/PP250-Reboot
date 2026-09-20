# PP250 execution and process model: research reconstruction

Status: research note, 19 September 2026. Architectural preamble to [PP250 boot and processor startup](pp250-boot-and-processor-startup.md); not an emulator specification.

## Scope and evidence

**Central proposition / Strong inference:** a System 250 process is a hardware-defined unit of execution, not merely an operating-system abstraction. Instructions and processor mechanisms preserve its capability, data, execution and CALL state across suspension. The operating system creates, schedules and manages these execution entities; a physical processor temporarily executes one of them. The Process Dump Stack holds persistent architectural state, but is not itself the entire process.

This reconstruction continues the user's [PP250 Boot Sequence Knowledge investigation](https://chatgpt.com/c/6aae536e-0f68-83eb-914e-d28ce0cc3dee). Evidence labels follow the existing research note and repository policy:

- **Documented / PRIMARY EVIDENCE:** statements in contemporary manuals, papers or patents. Pocket-reference details are checked against the repository transcriptions, whose scan-verification caveat remains applicable.
- **SECONDARY EVIDENCE:** Levy's retrospective account, identified explicitly.
- **Strong inference / INFERENCE:** conclusions connecting documented mechanisms.
- **HYPOTHESIS:** a proposed explanation awaiting confirmation.
- **Unresolved / UNKNOWN:** a missing mechanism or unresolved conflict.

**Verification boundary:** the pocket-reference transcriptions, England's 1972 paper and the online descriptions of patents [P1–P4] were read for this note. Source scans and patent figures were not visually rechecked. Layout-sensitive details not recoverable confidently from text remain open. The 1976 pocket reference anchors the Dump Stack offsets; later patents describe extensions and must not silently redefine that machine. Source identifiers below match those in the boot note where possible.

## 1. Physical System 250

**Documented, [E1], paragraphs 4–9:** System 250 comprises multiple processors, shared Store Modules and peripheral subsystems. A processor can reach each Store Module. Devices expose controller registers through the interconnection and can be addressed using ordinary data instructions.

The following is a **conceptual common system/CPU-bus view**, showing peer subsystems, not a wiring diagram:

```text
   +-------------+     +-------------+     +-------------+
   | Processor 0 |     | Processor 1 | ... | Processor n |
   +------+------+     +------+------+     +------+------+
          |                   |                   |
==========+===================+===================+===========
          Common system / CPU-bus interconnection (conceptual)
==========+===================+===================+===========
          |                   |                   |
   +------+------+     +------+------+     +------+----------+
   | Store       |     | Store       | ... | Disk / secondary|
   | Module A    |     | Module B    |     | storage subsystem|
   | shared      |     | shared      |     | via controller /|
   +-------------+     +-------------+     | bus interface   |
                                          +-----------------+
```

Disk storage is off the bus as a peer bus-connected subsystem. It does **not** hang from a Store Module. This does not imply that disk sectors are directly accessible like primary-memory words: controller-register access and backing-store transfer are distinct operations.

**Physical-topology qualification:** England describes each CPU's own parallel CPU bus, with a port at each Store Module's access unit; bus multiplexors connect CPU buses to peripheral buses. [P2], description of Figure 1, likewise has separate CB1/CB2 paths terminating at access-unit ports. Thus “common bus” above means a common reachable system interconnection, not one electrically shared CPU wire bundle. Multiplexors, duplicated peripheral paths and access-unit arbitration are collapsed in the diagram.

**Strong inference from [E1], paragraphs 16–19, and [P2]:** the bus transports addresses, information and control/status signals; it is capability/data agnostic in the sense that a transmitted bit pattern does not acquire authority merely by travelling on it. The processor's capability checks and microcode enforce permitted operations and bounds. This does not deny transport parity, control codes or interface checks, and should not be read as saying the bus carries only unqualified data bits.

## 2. Processor register architecture

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

The general register sets are documented in [E1], paragraph 16, and [P2], processor description; the 24/48-bit sizes are also stated by [L1], section 4.2 (**SECONDARY EVIDENCE**). “Programmer-visible” does not mean that every capability register has an interchangeable role or accepts arbitrary data as its contents.

**Documented, [R1], p. 7:** the manual's term is **Special Purpose CPU Registers**. The following are its named capability registers; register numbers retain the manual's octal notation.

| Manual register | Name | Function named by the pocket reference |
|---|---|---|
| C10 | C(D) | Dump Stack |
| C11 | C(I) | Interval Timer |
| C12 | C(C) | SCT |
| C13 | C(N) | Normal Interrupt Block |
| No C10–C17 number assigned | C(S) | Fault Start-Up Block |

C14–C17 are blank in the table: no functions are assigned here. The named special data registers are D10, absolute D/S pushdown pointer; D11, watchdog timer; D12, first-fault MIF copy; D15, interrupt accept register; and D17, IAR. Blank D13/D14/D16 entries remain unspecified. C(I) designates the timer's store block in [P2]; it is not the watchdog value saved for an individual process.

**Documented mechanism, [P2]; synthesis:** these special capability registers are internal processor architectural state. Ordinary instructions do not normally name and manipulate them as ordinary C0–C7 operands. Defined instructions and processor events use or modify them through microcode: CHP changes C(D); CALL/RET and CHP affect the pushdown and execution state; fault/startup machinery uses C(S). They are not all process state that must be saved with every process.

### Internal Mode is a separate access mechanism

**Documented, [P2], “Internal Mode Operation General” and its restrictions:** possession of an appropriate capability permits addressing internal processor registers through a reserved module-address interpretation. General-purpose instructions can thereby access permitted internal state, with capability bounds restricting the accessible set. This is capability-controlled access, not a conventional privileged/supervisor execution mode that bypasses protection.

The patent's general statement that special registers can be read and altered must be read with its specific restrictions: capability registers are read-only to data stores except for twelve alterable high base bits of C(S); all C(S) bits may be read. The remaining capability-register loading is through capability manipulation mechanisms. Consequently, “internal” must not be strengthened into “never accessible by software,” nor does Internal Mode imply unrestricted fabrication of capability registers. The pocket reference independently lists C(S) in its Internal Mode addressing diagram [R1, p. 7].

**Version boundary:** [P2] describes C(C1)/C(C2), C(L) and C(P), whereas the 1976 table names C(C) and leaves other slots blank. Those later names are evidence for that patent embodiment, not a completed 1976 register map.

## 3. Stored and expanded capabilities

**Documented, [E1], paragraphs 16 and 19:** a stored capability combines access rights with a reference to an SCT entry. Loading it obtains base/limit information from that entry and combines it with the stored access rights. The result addresses a bounded segment with specified permitted operations.

```text
24-bit stored capability pointer
+------------------+------------------------------------+
| access/form code | SCT reference / index              |
+------------------+------------------+-----------------+
                                      |
                                      v
                              SCT entry: base / limit
                                      |
                                      v
48-bit capability-register representation
+-------------------------------------------------------+
| base address                                          |
+-------------------------------------------------------+
| access information and limit                          |
+-------------------------------------------------------+
       conceptual fields; not a universal bit allocation
```

The compact stored pointer is not a raw physical address. Nor is saving C0 at one Dump Stack word evidence for storing all 48 register bits there. Capability identity and rights can be preserved compactly and the expanded descriptor reconstructed.

**Exact layouts available, with provenance:** [L1], Figure 4-1, labels its stored format as an 8-bit rights field and 16-bit SCT index (**SECONDARY EVIDENCE**). [R1], p. 4, instead supplies the following nine-position access/prefix diagrams; it does not establish a complete universal 24-bit layout:

```text
COS:  1 1 EC WC RC ED WD RD 0
POS:  0 1  1 EC WC RC ED WD RD
```

The diagrams are reproduced as transcribed. They are not interchangeable. [P4], “Capability Formats,” explicitly describes a 24-bit pointer with form/access information in the high nine bits and an identity in the low fifteen; its later form discrimination and propagation-permit rules introduce further distinctions. These source differences must be reconciled by machine/version and capability form before selecting emulator bit fields. The complete expanded-register bit map is likewise not reconstructed from schematic text alone.

The named rights are enter capability (EC), write/read capability (WC/RC), execute data (ED), and write/read data (WD/RD); [E1], paragraph 16 and Figure 2, explains the corresponding operations. EC permits entry through a capability block; ED permits instruction execution. They are distinct rights.

## 4. SCT, segments and virtual memory

**Documented, [E1], paragraphs 19–20:** the **System Capability Table (SCT)** holds the physical base/limit information for store blocks. A special processor capability identifies that table. Programs traverse capability structures without supplying physical addresses for the referenced segments.

```text
process's capability environment
          |
          v
stored capability --SCT reference--> SCT entry
          |                              |
       rights                         base / limit
          +---------------+--------------+
                          v
                 expanded capability
                          |
                checked segment access
                          v
                 shared Store Module
```

Multiple processes can possess different rights to the same segment [P4, “Description of Prior Art”]. A shared table does not mean universal authority: the reachable capability network and each capability's rights constrain a process's accesses.

**Documented, [E1], paragraph 30:** virtual store extends the capability structure to disk. The store-management package moves blocks between backing store and main store, and an attempted access can trigger bringing a block into main store. A main-store capability still needs an SCT entry when its target block exists only on disk. On-disk capabilities replace the SCT offset with disk identity/address information; moving a capability block entails converting its contained capabilities.

**SECONDARY EVIDENCE, [L1], sections 4.3–4.5:** the SCT is shared by processors, with synchronization required during updates. Primary-memory capabilities are called inform/active, and disk capabilities outform/passive; the latter identify the disk object rather than its current SCT slot. LC retains the SCT index in the Process Dump Stack, and SC combines that identity with register rights to reconstruct the stored capability.

Thus PP250 virtual memory is **segment/capability oriented**, not a conventional separate flat paged address space for each process. Sharing follows segment identity and authority. The disk representation is documented at this conceptual level; exact outform fields, conversion procedures and the applicability of each OS's implementation remain **UNKNOWN**. This does not establish an INFORM or OUTFORM machine instruction, a page-table mechanism, or a disk-based cold loader.

**Strong inference:** a common SCT supports processor-independent segment identity. Updating an SCT entry must also account for any descriptors already expanded in running processors; changing the table alone must not be assumed sufficient for coherent relocation. The exact synchronization protocol for the target machine remains to be established.

## 5. Capability integrity: the unresolved mixed-access case

Authority cannot safely be created merely by writing arbitrary data and then treating it as a capability. An SCT lookup checks/resolves a reference; it does not by itself prove that a process was entitled to manufacture that reference.

**Documented, [E1], paragraph 18:** England explicitly identifies data-write followed by capability-read as a way to manufacture authority and says access combinations permitting it are forbidden, with blocks separated into capability and data types. This is a primary-source protection rule, not a conjectured memory tag.

**Documented, [R1], pp. 5–6:** the Dump Stack contains saved data and capability state, and the ROS/PDOS Process Base diagram labels its forward link `666`. Under the COS-style field interpretation examined in section 12, that link permits both WC/RC and WD/RD. The two observations do not yet establish how the 1976 implementation enforces England's earlier general rule around this exceptional structure.

The evidence establishes several distinct mechanisms:

| Mechanism | What is established | What is not established |
|---|---|---|
| LC and SC | The manual lists separate capability load/store instructions; [E1], paragraph 19, explains SCT expansion. [P1], automatic CHANGE PROCESS discussion, says corresponding reserved-segment pointers are recorded in the dump area when capability registers are loaded. | A complete validation rule for loading a word after an arbitrary data store into a mixed-access block. |
| Capability provenance | [E1], paragraphs 19–25, obtains authority through existing capability blocks and controlled CALL entry. | A per-word provenance tag or other hidden storage encoding in the target machine. |
| Parity and sum-checks | [P1] checks transmitted/stored descriptor values and reverses internal parity on initial fault entry; [P4] describes descriptor validation. | That parity or a sum-check proves software authorization or prevents deliberate forgery. |
| Later pointer handling | [P4] adds pointer registers, load-on-use, access reduction and propagation control. | That these extensions existed in 1976 or solve the mixed-access question in that implementation. |

**Unresolved / UNKNOWN:** precisely which restrictions apply to holders of the Dump Stack capability, how ordinary data and capability operations interact there, and where the `666` exception is admitted and controlled. Restricting such powerful capabilities to trusted management code is a **HYPOTHESIS**, not a demonstrated complete mechanism. Per-word tags, cryptographic validation, parity-as-type-tag, and unrestricted data-to-capability conversion must not be invented.

This is a concrete research issue: reconcile [E1], paragraph 18, with [R1], pp. 4–6, using the target CPU's LC/SC and capability-access validation documentation. The existing [architecture WIP](../architecture/architecture.md) already leaves complete LC/SC semantics open; this note records the sharper conflict without altering that document or any transcription.

## 6. The hardware-defined process

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

## 7. Process Dump Stack: 1976 format

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
| ROS | 21 MIF; 22 LOCK; 23 Error Control; 24 Process Base; 25 SIP | 26 / 27 / 30 | 31 / 32 / 33 | Not shown |
| PDOS | 21 MIF; 22 LOCK; 23–25 three Ptarmigan words; 26 Error Control; 27 Process Base; 30 SIP | 31 / 32 / 33 | 34 / 35 / 36 | Not shown |

The initial entries are labelled **C6 Initial**, **C7 Code**, **IAR Block**; subsequent triples are associated with subroutines. The source defines MIF as a copy of the CPU Fault Indicator Register, LOCK as used by privileged system facilities, Error Control as the Process Error Control Parameter from the Process Template, and SIP as the State and Internal Priority Word. “Privileged system facilities” here does not establish a processor supervisor mode.

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

## 8. C6, C7 and execution domains

**Documented, [E1], paragraphs 21–26:** C6 supplies the current node's main capability block, while C7 supplies its code block. This is the capability environment through which the code can reach authorized objects. The term **Central Capability Block** is also used in [L1, sections 4.3–4.4, SECONDARY EVIDENCE].

```text
C6 --> current capability environment --> accessible segments
C7 --> current code segment
IAR --> current instruction position within that code context
```

**Documented, [P2]:** the live IAR contains an absolute instruction address within the C7-defined block. [P3] describes a relativized saved instruction address. These are different representations of execution position; this note does not prescribe an unverified 1976 encoding or increment order.

C6 is **not inherently an OS Process Base pointer**. A particular management domain may use its C6 environment to reach a Process Base or may arrange for that block itself to be the current capability block. That is a structure and calling-context question, not the definition of C6. C0–C5 can carry capabilities across a domain call [E1, paragraph 25], so describing C6 as the current environment must not imply that all other held authority disappears on CALL.

## 9. CALL versus CHP

| Operation | Execution entity | Dump Stack | State transition |
|---|---|---|---|
| CALL | Same process | Same process's stack | Enter another procedure/domain and preserve a return context |
| CHP (Change Process) | Outgoing process suspended; incoming process activated | Outgoing and incoming stacks | Transfer the resumable process context |

**Documented, [E1], paragraphs 24–26:** CALL uses an enter capability and an offset selecting code, establishes the called domain's C6/C7, and preserves the caller's context for RETURN. [R1] records the C6/C7/IAR frames; [P3] explicitly distinguishes stack updates by CALL from a process change involving two Dump Stacks. CALL does not itself create another hardware process.

The user's concrete instruction is:

```text
CHP 3 0 C6
```

**Provenance:** this exact assembly example is supplied by the user's investigation, not printed in the inspected pocket-reference pages. [R1], p. 3, establishes CHP's name and opcodes; its instruction list is not a full operand or microsequence specification.

**Strong inference / current working interpretation:** C6 plus offset `3` locates the stored capability designating the incoming Dump Stack in the ROS/PDOS arrangement:

```text
C6 in the calling management context
             |
        offset 3
             v
   stored Dump Stack capability
             |
             v
            CHP --> incoming C(D) --> incoming execution state
```

[P1], the discussion after fault step S17, independently says normal CHANGE PROCESS uses an instruction-supplied offset down a reserved segment pointer table to obtain the incoming dump area through the master capability table. This supports the pointer-to-Dump-Stack interpretation, while using earlier terminology.

Use C6 for this mechanism, as in the supplied example; an arbitrary C2 example would contradict the investigation. However, the inspected evidence does not prove whether every alternative CAP-field encoding is illegal. The exact role of `0`, effective-address checks, direct-mode CHP semantics, accepted capability forms, failures and microinstruction order remain **UNKNOWN**. The C6 used to find the operand is outgoing state; incoming C6 is restored from the incoming context and may be different.

## 10. C(D) and process switching

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

“Complete context” here means the process's resumable execution context, not every register or transient microcode latch in the CPU. This model leaves the precise sequencing, already-maintained pointer entries, interrupted-instruction rules and cold-entry treatment of an invalid old C(D) to source verification. It must not become literal emulator pseudocode by accident.

## 11. Process versus processor: multiprocessor mobility

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

## 12. OS Process Base versus hardware process

**Documented, [R1], p. 5:** the ROS/PDOS diagram shows an EC arrow entering Process Base, a pointer beginning `666` at Process Base offset `3` directed to Dump Stack, and a Dump Stack pointer beginning `760` returning to Process Base.

```text
external holder
      |
      | EC (as drawn)
      v
+------------------+       666 at offset 3      +------------------+
| OS Process Base  | ------------------------> | Process Dump     |
| management      | <------------------------ | Stack            |
| structure       |       760 backlink         | execution state  |
+------------------+                            +------------------+
                                                       ^
                                                       | C(D)
                                                   active CPU
```

### Current access-code decoding

**Documented fields plus arithmetic inference:** interpreting the three octal digits with the COS-style nine-position diagram on [R1], p. 4, gives:

| Prefix | Binary | `1 1 EC WC RC ED WD RD 0` interpretation |
|---|---|---|
| 666 | 110 110 110 | WC + RC + WD + RD; no EC and no ED/execute |
| 760 | 111 110 000 | EC + WC + RC; no ED or data read/write |

The POS diagram has different alignment and fixed bits. The above arithmetic is exact **under the COS-style layout**; applying that layout to the ROS/PDOS prefixes is the current interpretation and needs explicit format confirmation. No meaning for the fixed prefix/trailing bits is invented. `760` is stronger than an enter-only capability, and `666` contains no execute authority.

**Strong inference:** Process Base is the OS management structure layered above the architectural process. The Dump Stack link supplies hardware execution state; other management fields and conventions support software operations. COS/POS/ROS/PDOS can organize management differently while using the architectural process mechanism. The pocket reference's different layouts are evidence against treating one OS Process Base format as the hardware definition of a process.

**HYPOTHESIS, with direct diagram support:** external holders may receive an ENTER capability to Process Base, invoking management operations while stronger internal links reach the Dump Stack and associated structures. The EC arrow supports an entry interface, but does not prove that every external holder receives only EC or that all OS versions implement identical object protection. In particular, it does not resolve who can obtain the mixed-access Dump Stack link. Do not generalize this into “all anyone ever gets” without the process-management interface documentation.

## 13. Relationship to boot and fault research

This document provides the register, capability, SCT and process concepts assumed by [PP250 boot and processor startup](pp250-boot-and-processor-startup.md).

**Strong inference:** at power-up there is no valid running process to supply ordinary execution authority. Startup machinery must establish sufficient valid capability/SCT state to identify a Process Dump Stack and obtain an executable C6/C7/IAR context.

```text
no valid running process
          |
C(S) / special startup-fault machinery
          |
sufficient valid SCT and capability state
          |
identify initial / checkout Dump Stack
          |
change-process restoration --> normal process execution
```

**Documented, [P1], fault steps S2, S10 and S16–S17 and following text:** fault recovery invalidates prior internal capability parity, establishes a special table, obtains the checkout Dump Stack pointer and performs automatic CHANGE PROCESS. [P2] documents a power-up preset for C(S). **Inference:** related startup machinery can establish a first process; this does not prove that cold startup and fault recovery execute identical microsequences, nor explain loading virgin memory.

The boot note expands `RSPC-n`; this note deliberately retains **RSPC-0** without adopting an unverified expansion. [P1] identifies its functional role as a reserved segment pointer to the checkout dump area. The inspected wording establishes that role more securely than the precise acronym expansion. SSCR/MCR/DCR and C(S)/C(C)/C(D) are compared functionally, not asserted to be identical layouts across generations.

The main outstanding research questions are the mixed-access anti-forgery mechanism; version-specific capability fields; exact CHP operand/restore rules; saved-pointer and initial-frame construction; and how valid startup structures first enter memory. These remain research questions rather than implicit implementation requirements.

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

Prepared against repository commit `43547b29cbd426a3dfda6f8dce464f4653cb3252`. This note adds a research reconstruction only; no architecture document, original source, transcription or previous research note was modified.
