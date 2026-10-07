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

### MIP, MIF and MIS: persistent versus transient processor state

**Documented, [R1], pp. 6–7:** MIP is the Primary Indicator Register and is part of the common fixed Process Dump Stack state at offset `20`. ROS and PDOS additionally preserve MIF, the Fault Indicator Register, in their OS-dependent Dump Stack area. The Internal Mode diagram also exposes MIS (Secondary Indicator Register), but MIS is not shown as a saved Dump Stack word in the documented COS/POS/ROS/PDOS layouts.

This difference is architecturally significant. A current **working reconstruction** is that MIP contains execution state that must survive process suspension/resumption, MIF carries fault state that ROS/PDOS deliberately preserve for recovery/management, while at least some MIS bits represent more transient M-level sequencing or semantic state that need not form part of the resumable H/T process context. This is an inference from the save layouts and Internal Mode exposure, not a documented definition of the three registers.

The named MIS bits strengthen that interpretation. `MIS08 Set Read Capability` and `MIS19 Cap. Pointer in OPP` appear to retain the semantic status of capability-related transfers through internal processor sequencing. Their exact timing and the meaning of OPP remain unresolved; they must not yet be turned into emulator behaviour merely from their names.

`MIP04 Second Group` may refer to selection of the documented second group of special-purpose registers D10–D17 and C10–C17. This is a useful **HYPOTHESIS**, not an established decoding of the bit. If correct, it would be consistent with MIP retaining processor-visible selection/execution state while MIS carries more transient internal control state.

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

The compact stored pointer is not a raw physical address. The Pocket Reference shows each fixed Dump Stack location corresponding to C0–C5 as a single 24-bit word, while each workspace capability register is 48 bits. US3771146A makes the relationship explicit: these locations hold the corresponding reserved capability pointers, and the appropriate pointer is recorded whenever a workspace capability register is loaded. The expanded descriptor is reconstructed through the capability table rather than being stored as a 48-bit register image in the Dump Stack.

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

**SECONDARY EVIDENCE, [L1], sections 4.3–4.5:** the SCT is shared by processors, with synchronization required during updates. Primary-memory capabilities are called inform/active, and disk capabilities outform/passive; the latter identify the disk object rather than its current SCT slot. Levy states that LC retains the SCT index in the Process Dump Stack and that SC combines that identity with register rights to reconstruct the stored capability. The retention mechanism is now independently supported by primary evidence: US3771146A states that the corresponding reserved capability pointer is recorded in the Dump Stack whenever a workspace capability register is loaded.

Thus PP250 virtual memory is **segment/capability oriented**, not a conventional separate flat paged address space for each process. Sharing follows segment identity and authority. Inform/Outform conversion belongs to the established VM/storage-management and VM-trap path; exact physical encoding is generation/version-specific and is not an architectural open question. This does not imply an INFORM or OUTFORM machine instruction, a conventional page-table mechanism, or a disk-based cold loader.

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

This is a concrete research issue: reconcile [E1], paragraph 18, with [R1], pp. 4–6, using the target CPU's LC/SC and capability-access validation documentation. The existing [architecture WIP](../architecture/capability-representation.md) already leaves complete LC/SC semantics open; this note records the sharper conflict without altering that document or any transcription.

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

**Documented, [R1], p. 6. All offsets below are octal.** The common fixed portion `0–20` contains seventeen 24-bit words. The entries labelled C0–C5 are therefore six 24-bit locations, not 48-bit register images. Read together with US3771146A, they are the reserved capability pointers corresponding to the six workspace capability registers:

| Offset | Saved state | Offset | Saved state |
|---:|---|---:|---|
| 0 | C0 capability pointer | 10 | D2 |
| 1 | C1 capability pointer | 11 | D3 |
| 2 | C2 capability pointer | 12 | D4 |
| 3 | C3 capability pointer | 13 | D5 |
| 4 | C4 capability pointer | 14 | D6 |
| 5 | C5 capability pointer | 15 | D7 |
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

The save layout itself provides an additional constraint on the processor-state model. MIP is saved for all four operating systems shown; MIF is additionally saved by ROS/PDOS; MIS is not shown as a saved Dump Stack word. This is consistent with, but does not prove, the working distinction above between persistent execution state, explicitly preserved fault state, and transient M-level sequencing state.

```text
Dump Stack
+--------------------------------------+
| fixed process save area, 0-20 octal   |
|   C0-C5 capability pointers; D0-D7;     |
|   pushdown pointer; watchdog; MIP        |
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

## 8. C6, C7, CALL, Enter Capability and RETURN

**Documented, [E1], paragraphs 21–26, and independently described by [H1]:** C6 identifies the current node's main capability block/domain and C7 identifies the currently executing code block.

For an ordinary CALL within the current domain, C6 remains the current domain while C7 identifies the called code.

For a CALL through an **Enter Capability**, the instruction specifies the Enter Capability and an offset within the entered capability block. The processor loads **C6 with the entered capability block** (supplying read-capability access) and **C7 with the executable capability selected by the offset**. The previous C6, C7 and return IAR are preserved on the Process Dump Stack.

**Documented, [E1], paragraph 25:** the remaining general capability registers C0–C5 may carry parameters across the interface between nodes. A protected CALL is therefore not a complete register-context replacement: it changes the protected execution context represented by C6/C7 while allowing other register state to cross the boundary.

**Documented, [R1], p. 6:** the Process Dump Stack contains successive **C6/C7/IAR triples**, confirming the three-word CALL frame. `RETURN` restores the saved execution context. CALL frames occupy the variable stacked part of the same Process Dump Stack whose fixed area is used to preserve process state; they are not a second stack.

**FIRST-HAND EVIDENCE, [S1]:** Anthony J. Stanners independently recalled this CALL/Enter/RETURN mechanism from his 1975–77 CORAL 250 work, including the C6/C7/IAR frame and the fact that the other capability registers survive the domain transition. Stanners also recalls unintended capabilities left in those registers on return as a known security weakness.

**SECONDARY CORROBORATION, [L2]:** Levy later identifies surviving capability-register contents across protected CALL as an authority-leakage weakness. Thus the mechanism itself is established by contemporary primary sources; Stanners's recollection supplies independent participant evidence and the remembered security significance, with Levy independently corroborating the latter.

```text
caller                         callee
C6 -> caller domain            C6 -> entered domain
C7 -> caller code     CALL     C7 -> selected code
IAR -> return point   ---->    IAR -> callee execution
C0-C5 ------------ survive across boundary ------------>

Process Dump Stack:
    ... | saved C6 | saved C7 | saved IAR | ...
                                      ^
                              restored by RETURN
```

C6 is **not inherently an OS Process Base pointer**. A particular management domain may use its C6 environment to reach a Process Base or may arrange for that block itself to be the current capability block.

## 9. CALL versus CHP

| Operation | Execution entity | Dump Stack | State transition |
|---|---|---|---|
| CALL | Same process | Same process's stack | Enter another procedure/domain; push C6/C7/IAR return frame |
| CHP (Change Process) | Outgoing process suspended; incoming process activated | Outgoing and incoming stacks | Transfer the resumable process context |

**Documented, [E1], paragraphs 24–26, [H1], and [R1]:** CALL uses an Enter Capability and offset to establish the called C6/C7 context and pushes the C6/C7/IAR return frame. CALL does not itself create another hardware process. CHP is the distinct heavyweight process-state transition.

The exact operand semantics of CHP remain under investigation. An earlier recalled example, `CHP 3 0 C6`, was subsequently withdrawn by Stanners as unreliable and **must not be used as evidence** [S1]. Any occurrence of offset 3 in the ROS/PDOS reconstruction derives instead from the documented Process Base diagram and must be treated as an OS-specific structural observation.

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

## 11A. ROS/PDOS process-management state

**Documented, [R1], p. 5:** the ROS/PDOS **State and Internal Priority Word (SIP)** records software process-management state alongside, but distinct from, the hardware execution state restored by CHP.

| Field | Documented meaning |
|---|---|
| r | 0 = currently running on a CPU; 1 = not running |
| cccc | CPU number when running |
| ppppp | 23 - current priority |
| qqqqq | 23 - standard priority |
| x | process is on the Ready List |
| s | suspended on WAITFOR |
| f | process has faulted |
| bb | 00 double unblocked; 01 unblocked; 10 blocked; 11 double blocked |
| w | may be 0 or 1; meaning unresolved |

This is direct evidence that ROS/PDOS process management distinguished **running**, **ready**, **WAITFOR-suspended**, **faulted** and **blocked** states, and maintained both current and standard priority. It also recorded which CPU was executing a running process. The exact state-transition rules, Ready List operations, WAITFOR semantics, blocking protocol and meaning of `w` remain to be reconstructed.

The Pocket Reference also places **LOKK**, **Error Control**, **Process Base** and **SIP** in the ROS Dump Stack, and adds three **Ptarmigan words** in PDOS. It defines LOKK only as “used by privileged system facilities” and Error Control as the “Process Error Control Parameter from Process Template”. Their detailed semantics remain unresolved. Their presence in the OS-dependent Dump Stack area must not be mistaken for evidence that they are intrinsic CHP hardware state.

### Consolidated from capability/resource-lifecycle research: Scheduling, watchdog and interval activity

Known/recalled structures suggest that processes execute until a scheduling event such as blocking/waiting, yielding/change-process activity, interval timer activity, or watchdog expiry/fault.

The Watchdog Timer is part of process state and provides runaway-process containment. Interval timing provides a separate source of normal system activity. System 250 processors share work through a common work list.

Periodic resource discovery therefore need not require conventional device interrupts.



### Synchronising flags, WAITFOR and return to the Ready List

**DOCUMENTED, [E4], paragraphs 30–31:** the process-management package provides a **flag** resource for synchronisation and inter-process communication. A process may **POST** a message on a flag or **WAIT FOR** one. If WAITFOR occurs before a message has been posted, the receiving process is suspended until a POST occurs. A flag can support multiple waiting processes and multiple posted messages, and a process may wait for one of several possible events.

England's process-management description [E1, paragraph 31(3)–(4)] gives the corresponding queueing model. A flag may contain either queued messages or a queue of suspended processes. A process may also wait for expiry of a time interval; such processes are held on a time-ordered chain. The time chain is serviced by **interrupt processes** in response to interval-timer interrupts. When one of several awaited events occurs, the other outstanding waits are cancelled and the process is entered in the **process Ready List**, with a parameter identifying the event that occurred.

Together with the SIP fields in [R1], this establishes a substantial part of the ROS/PDOS process-state path:

```text
running process
      |
   WAITFOR
      |
      v
suspended on flag queue / time chain
      |
   POST or time expiry
      |
      v
other waits cancelled
      |
      v
Ready List
      |
      v
eligible to be scheduled
```

The SIP `s` and `x` fields therefore correspond to documented process-management states in this path rather than merely unexplained status bits.

**Important separation:** expiry of a waited-for time interval is not itself evidence that the interval mechanism is the scheduler. England explicitly says **interrupt processes service the time chain**. Those processes can wake a suspended process and put it on the Ready List. Selection of a Ready-List process for execution is a separate scheduling operation.

### Current scheduler reconstruction

**Working reconstruction:** scheduling is required when the currently executing process ceases to continue normally — in particular when it blocks/suspends itself, or when its allocated execution interval expires. The scheduler selects an eligible process from the common Ready List and the selected process is ultimately established on a CPU through the process-change machinery.

**HYPOTHESIS:** the scheduler is a single protected process rather than a separate scheduler instance permanently associated with each CPU. This fits the documented common Ready List and CPU-independent process execution, but the surviving evidence inspected so far does not yet prove the number of scheduler instances or the exact entry path.

The two paths still to reconstruct explicitly are therefore:

```text
WAITFOR / blocking  -> scheduler -> CHP -> selected process
time-slice expiry   -> scheduler -> CHP -> selected process
```

They must not yet be assumed to have identical entry sequences.

### LOKK remains outside the reconstruction

The Pocket Reference documents `LOKK` only as **"used by privileged system facilities"**. No process-management mechanism reconstructed above currently requires it. In particular, the existence of a common Ready List does not establish competing per-CPU scheduler instances requiring a per-process scheduler lock.

`LOKK` therefore remains **UNKNOWN**. It should not be assigned a locking, scheduling, ownership or synchronisation role unless further evidence requires or documents one.

### Process-management boundary exposed by the current evidence

The current evidence therefore separates:

- **hardware execution context** preserved/restored by CHP;
- **M-level process transition** which chooses and restores another execution context;
- **software process-management structures and policy**, including Process Base, SIP, Ready List state, WAITFOR, priorities, Error Control and still-unresolved LOKK/Ptarmigan state.

How those layers cooperate to create, queue, block, wake, fault, schedule and destroy a process remains an active reconstruction problem.

## 11B. Normal interrupt acceptance as process-management machinery

The following material was moved here from `pp250-normal-interrupt-and-system-dispatch.md` because it describes the generic M-level transition into a managed process, not merely one interrupt-handler implementation.

### DOCUMENTED — the System Interrupt Word is polled, not a conventional device interrupt

Halton's System Interrupt description must be distinguished from a conventional asynchronous peripheral-interrupt architecture.

Special capability register `C(I)` defines the **System Interrupt Word (SIW)**.

The SIW contains 24 bits, with one bit associated with each processor and I/O channel. Pending system activity is represented by setting the corresponding bit in this shared word.

The processor periodically examines this state. Using `C(I)` it accesses the SIW, and using `C(N)` it obtains the associated interrupt mask information. Pending unmasked bits are correlated and one request is selected.

The selected SIW bit is cleared and its position is represented as a **correlation count**.

Thus ordinary processor/I/O activity does not cause a peripheral to supply an interrupt vector or directly seize processor execution. The activity is represented in shared system state which the processor periodically examines.

### DOCUMENTED — D15 records SIW correlation or Program Trap acceptance

The May 1976 Pocket Reference identifies special-purpose data register `D15` as the **INTERRUPT ACCEPT REGISTER**.

Direct inspection of Halton Figure 7 gives the following fields:

| D15 bits | Meaning |
|---|---|
| 0–5 | Correlation count of System Interrupt Word |
| 6 | Trap Accepted |
| 7–23 | Not presently identified |

For the SIW path, bits 0–5 contain the correlation count identifying the position selected from the System Interrupt Word.

For the Program Trap path, bit 6 records that a trap has been accepted.

The Interrupt Accept Register therefore brings two architecturally different sources of normal processor attention together:

```text
processor / I/O activity                  Program Trap
          |                                    |
          v                                    |
 System Interrupt Word                         |
          |                                    |
periodic examination                           |
and correlation                                |
          |                                    |
          v                                    v
D15[0:5] = correlation count          D15[6] = Trap Accepted
          |                                    |
          +----------------+-------------------+
                           |
                           v
                 normal interrupt machinery
```

This should not be interpreted as evidence for conventional I/O interrupts. The SIW side is the result of processor polling/correlation of shared state. Program Trap is an internally generated processor condition.

### DOCUMENTED — MIF identifies the capability register associated with the event

The May 1976 Pocket Reference identifies `MIF20–23` as the **capability register on which failure occurred**.

Consequently, the processor state available following the trapped reference contains at least two important pieces of information:

- `D15.6` records **Trap Accepted**;
- `MIF20–23` identifies the **capability register associated with the failure**.

`MIF20–23` should not be described as directly identifying an SCT entry or an object. It identifies a capability register. The relationship from that register to the referenced segment is a separate part of the capability/SCT mechanism.

### DOCUMENTED — C(N) and Internal Mode provide the handler environment

Halton identifies special capability register `C(N)` as defining the **Normal Interrupt Block**.

Normal interrupt entry uses this protected processor mechanism to enter the Normal Interrupt process. The process change is an architectural process change — an automatic `CHP` through the state supplied by the Normal Interrupt Block — rather than an ordinary branch or conventional interrupt-vector transfer.

`C(N)` is established by the running system during startup and thereafter supplies the processor with the protected state required for normal interrupt entry.

The special-purpose processor registers are not normally available to ordinary program addressing. They can, however, be accessed by code operating through the documented **Internal Mode** addressing mechanism.

The Normal Interrupt process can therefore inspect the processor-generated interrupt/trap state without requiring that state to be exposed through the ordinary capability namespace.


**Process-model implication:** normal interrupt acceptance is a boundary between event recognition in M and software process management. M identifies/accepts the event, records protected cause/source state in D15, and enters the protected Normal Interrupt process through C(N)/the Normal Interrupt Block and automatic CHP. What that process then does with Ready Lists, waiting/faulted processes, priorities or resource-specific handlers is software process-management policy.

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

The main outstanding research questions are the mixed-access anti-forgery mechanism; version-specific capability fields; exact CHP operand rules; initial-frame construction; and how valid startup structures first enter memory. The representation and restoration role of the fixed C0–C5 Dump Stack words is no longer open: they are 24-bit capability pointers used to rematerialise the workspace capability registers through the capability table. These remaining issues are research questions rather than implicit implementation requirements.

## Sources and provenance

- **[R1] PRIMARY EVIDENCE via repository transcription:** user's Plessey *System 250 Pocket Reference Book / Instruction Codes*, Issue 1, May 1976. [Title/contents transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg0-pg2%20transcription.txt); [pp. 3–4 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg3-pg4.pdf); [pp. 5–7 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg5-pg7.pdf). Locators: p. 3 instruction codes, p. 4 access-code diagrams, p. 5 ROS/PDOS structures/state word, p. 6 Dump Stack, p. 7 Special Purpose CPU Registers/Internal Mode. Transcriptions checked; scans not independently rechecked here.
- **[E1] PRIMARY EVIDENCE:** D. M. England, *Architectural Features of System 250* (1972), [repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf). Paragraphs 4–9: topology; 16–20: rights, integrity and SCT; 21–26: domains/CALL/Dump Stack; 30: virtual store and disk capabilities; 31: process management and CPU independence. PDF text inspected; diagram bit boundaries not treated as verified from extraction.
- **[H1] PRIMARY EVIDENCE:** D. Halton, *Hardware of the System 250 for Communication Control* (1972), [repository transcription](../transcriptions/hardware-of-the-system-250-for-communication-control.md). Enter mechanism: C7 receives selected executable capability, C6 the called node capability block, old C6/C7 are stacked and restored by RETURN.\n- **[S1] FIRST-HAND EVIDENCE:** Anthony J. Stanners, consolidated [System 250 first-hand recollections](../sources/anthony-stanners-first-hand-recollections.md), including CALL/Enter/RETURN recollection and evidence status.\n- **[L2] SECONDARY EVIDENCE:** Henry M. Levy, System 250 discussion of capability-register leakage across protected CALL; repository analysis in [levy-chapter4-call-register-leakage.md](levy-chapter4-call-register-leakage.md).\n- **[E2] PRIMARY EVIDENCE, contextual research lead:** D. M. England, *Operating System of System 250*, International Switching Symposium, June 1972, in the [repository ISS collection](../sources/1972/1972-06-System-250-ISS-Papers-MIT.pdf). Detailed OS claims above rely on [E1], not an assumed reading of this collection.
- **[E4] PRIMARY EVIDENCE via repository transcription:** D. M. England, *Operating System of System 250* (International Switching Symposium, June 1972), [repository transcription](../transcriptions/operating-system-of-system-250.md), paragraphs 30–31: synchronising flags, POST/WAITFOR, suspension and multiple waits. Read directly for the process-management reconstruction above.
- **[E3] PRIMARY EVIDENCE, research lead:** D. M. England, *Capability Concept Mechanism and Structure in System 250*, International Workshop on Protection in Operating Systems, August 1974. Bibliographic reference in [Levy's bibliography](https://homes.cs.washington.edu/~levy/capabook/Bibliography.pdf); no copy in the inspected repository tree. Not used as proof of an unverified mechanism.
- **[P1] PRIMARY EVIDENCE:** Plessey, US 3,814,919, *Fault detection and isolation in a data processing system*, [patent text](https://patents.google.com/patent/US3814919A/en). Locators: capability parity fault; fault microsequence S2/S10/S16/S17; automatic and normal CHANGE PROCESS discussion immediately afterwards. Retrieved directly and read for this note; not archived in the inspected repository.
- **[P2] PRIMARY EVIDENCE:** US 4,383,297, *Data processing system including internal register addressing arrangements*, [repository PDF](../patents/US4383297-internal-register-addressing.pdf), [patent text](https://patents.google.com/patent/US4383297A/en). Locators: illustrative embodiment/Figure 1 description; special data and capability registers; Internal Mode Operation General and restrictions. Text read; later register map kept distinct from [R1].
- **[P3] PRIMARY EVIDENCE:** US 4,486,831, *Multi-programming data processing system process suspension*, [repository PDF](../patents/US4486831-process-suspension.pdf), [patent text](https://patents.google.com/patent/US4486831A/en). Locators: background's existing two-part Dump Stack; summary's extensions; Process Dump-Stack/Figure 5 description. Text read; later local-store/extended-link facilities are not backdated to 1976.
- **[P4] PRIMARY EVIDENCE:** US 4,408,274, *Memory protection system using capability registers*, [repository PDF](../patents/US4408274-memory-protection-capability-registers.pdf), [patent text](https://patents.google.com/patent/US4408274A/en). Locators: Description of Prior Art, Capability Formats, LC/load-on-use and SC descriptions, Figures 5–9 references. Text read; enhanced pointer format and propagation controls are version-qualified.
- **[L1] SECONDARY EVIDENCE:** Henry M. Levy, *Capability-Based Computer Systems*, Digital Press, 1984, [chapter 4, “The Plessey System 250”](https://homes.cs.washington.edu/~levy/capabook/Chapter4.pdf), especially sections 4.2–4.5 and Figure 4-1. Read as corroboration and for terminology; it does not override the pocket reference or resolve version conflicts.

Prepared against repository commit `43547b29cbd426a3dfda6f8dce464f4653cb3252`. This note adds a research reconstruction only; no architecture document, original source, transcription or previous research note was modified.

## Evidence update — 2 October 2026: restoration is capability rematerialisation

**Resolution — 7 October 2026:** the Pocket Reference word layout and US3771146A together resolve the representation of the fixed C0–C5 Dump Stack entries. Each is one 24-bit capability pointer corresponding to its workspace capability register, not a saved 48-bit register image. US3771146A further states that the corresponding pointer is loaded into the Dump Stack whenever the capability register is loaded and that process restoration reloads the workspace capability registers using those Dump Stack pointers through the master capability table. The persistent process state is therefore the compact capability-pointer state; the expanded base/limit register representation is transient processor state rematerialised from the current table state.

**DOCUMENTED OBSERVATION:** US3771146A, Description 75, 81 and 121, saves compact capability pointers and reloads workspace capability registers through the master table on process restoration. The relocation case deliberately uses a handler transition and return to replace previously expanded bounds with the table's current unavailable state. Thus a process dump does not freeze physical capability bounds independently of the table. This supports the existing capability-mediated process model; it constrains any reading of “restore” as a bit-for-bit resurrection of old expanded descriptors. See [canonical SCT mechanism](../architecture/system-capability-table.md#already-expanded-capabilities-during-relocation).

The earlier statement that US3814919A was not archived is now superseded by its [repository PDF](../patents/US-3814919-A.pdf) and [new transcription](../transcriptions/US3814919A.rtf). Its subsequent-fault restore path bypasses the outgoing dump (Description 132); generalising this to cold start remains reconstruction, as discussed in the startup note.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.
