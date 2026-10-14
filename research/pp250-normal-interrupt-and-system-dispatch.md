# PP250 Normal Interrupt and System Dispatch — Research Reconstruction

## Status

Research reconstruction. This note captures the current understanding of the PP250 normal-interrupt path, its relationship to `C(N)`, the Normal Interrupt Block, automatic `CHP`, storage-management dispatch, and the startup chain by which `C(N)` is established.

It deliberately keeps the PP250 architectural reconstruction primary. The M/H/T model is a later abstraction over this mechanism and is not used here to define the architecture.

Evidence should be read using the repository's existing distinction between documented observations and reconstruction. The reconstruction method is the project's "elephant" method: surviving sources expose different parts of the machine; mutually constraining observations may establish the shape of the architecture even where no surviving source states the complete mechanism in one paragraph.

## 1. Two distinct processor-entry mechanisms

A central conclusion is that **normal interrupt entry through `C(N)` must not be conflated with fault/start-up entry through `C(S)`**.

`C(S)` identifies the Fault Start-Up Block and belongs to the exceptional processor fault/startup/checkout path. That path includes the deliberate invalidation/reversal of processor-state parity and checkout before a legitimate process is established. It is therefore not a plausible mechanism to execute on every ordinary storage fault, unavailable-segment condition, I/O event, or similar normal intervention.

The running system instead has a separate normal-interrupt mechanism rooted in `C(N)`.

Conceptually:

```text
catastrophic fault / startup              normal operational event
           |                                         |
           v                                         v
          C(S)                                      C(N)
           |                                         |
           v                                         v
   checkout / startup                       Normal Interrupt Block
           |                                         |
           v                                         v
 first legitimate process                    automatic CHP
```

The two paths are related at bootstrap — the `C(S)`-rooted startup must ultimately establish the running normal-interrupt machinery — but they are not the same operational path.

### Interval Timer — C(I)

Each processor has a C(I) capability register (C11), initialised during processor startup, addressing a 24-bit countdown word in memory. Hardware decrements this word every 100 µs and generates a Normal Interrupt on reaching zero. Software reloads the word without changing C(I).

**Working interpretation:** Each processor has its own interval timer to ensure that, even in the absence of other interrupts, the running process can be suspended when its allocated time slice expires.

The maximum countdown is 27 minutes 58 seconds. Wraparound after zero is plausible but unconfirmed.

**Sources:** 1976 Pocket Reference; US 4,383,297.

## 2. Why normal interrupt entry must lead to a Dump Stack

The processor's protected process-transition mechanism is `CHP` (Change Process). A process is represented by persistent architectural state in its Process Dump Stack, from which the processor can restore the incoming process context.

An explicit `CHP` instruction can identify the process to be entered through its operand. An **automatic** change process caused by a normal interrupt has no executing instruction supplying such an operand. The processor therefore requires another protected source for the incoming process's Dump Stack reference.

`C(N)` is identified by the Pocket Reference as the **NORMAL INTERRUPT BLOCK** capability. Patent descriptions of the normal-interrupt mechanism establish the extra level of indirection: `C(N)` designates the Normal Interrupt Block (NIB), and the NIB supplies the capability pointer/reference used to identify the Dump Stack of the Normal Interrupt process.

Thus the reconstructed normal-interrupt entry is:

```text
normal interrupt condition
        |
        v
      C(N)
        |
        v
Normal Interrupt Block
        |
        v
IDAP + permitted access code
(Interrupt Handler Process Dump Area Pointer)
        |
        v
Master Capability Table lookup
        |
        v
DS-IHP (Interrupt Handler Process Dump Area / Dump Stack)
        |
        v
DCR loaded for IHP
        |
        v
automatic CHP / IHP undumped
        |
        v
Normal Interrupt process
```

US3771146A / GB1410631A supplies this additional detail: the Normal Interrupt Block provides the Interrupt Handler Process Dump Area Pointer (IDAP), together with its permitted access code. The automatic process-change machinery uses IDAP through the Master Capability Table to obtain DS-IHP's Dump Area descriptor and load DCR before IHP is undumped. This describes how DS-IHP is located; it does not by itself establish how IHP subsequently accesses DS-A.

This distinction matters. **`C(N)` is not itself a C6/C7 process image.** It designates the NIB. The NIB leads to the Dump Stack. The Dump Stack contains the state from which the normal interrupt process is entered.

## 3. Automatic CHP is still process change

There is no need to posit a second, special interrupt execution environment or a conventional supervisor mode.

Once the normal-interrupt machinery has obtained the incoming Dump Stack reference through `C(N)` and the NIB, the processor can use the same architectural process-transition machinery used by `CHP`.

The incoming Dump Stack restores the process state required for execution. In the 1976 Pocket Reference the common fixed part includes:

- C0–C5;
- D0–D7;
- the CALL-stack pushdown pointer;
- the watchdog timer;
- MIP, the Primary Indicator Register, at octal offset `20`.

The OS-dependent continuation of the Dump Stack supplies the execution frames, including the initial C6/C7/IAR state.

Accordingly the Normal Interrupt process is an ordinary PP250 process in the architectural sense. What is special is **how the processor selects and enters it**, not a privileged instruction universe in which it subsequently executes.

## 4. Program Trap, non-resident segments and normal interrupt entry

### DOCUMENTED — Program Trap is distinct from Fault Interrupt

Halton explicitly distinguishes **Program Trap** from **Fault Interrupt** during capability-controlled store access.

When a store address is constructed through a capability, the processor performs a sequence of checks:

1. The ACCESS field is checked for all zeros. If it is all zeros, a **PROGRAM trap** is generated.
2. The resulting absolute address is checked against the BASE and LIMIT held in the capability register. If the address is outside those bounds, a **Fault Interrupt** is generated.
3. The operation being attempted is checked against the operations permitted by the ACCESS field. If the operation is not permitted, a **Fault Interrupt** is generated.
4. Only after these checks succeed does the requested store access take place.

An all-zero ACCESS field is therefore a deliberately distinguished architectural condition. It does **not** represent an ordinary capability-protection failure.

This distinction is important when interpreting the later `MIF18 Access Violation` indication. A reference which violates the authority represented by a capability belongs to the Fault Interrupt mechanism. The all-zero ACCESS condition belongs instead to Program Trap.

### DOCUMENTED — non-resident segments enter a trap-handler process

The System 250 operating-system description explains that movement of blocks between main store and disk is intended to be transparent to processes using those blocks.

A process can therefore attempt to use a block which is not currently resident in main store. Hardware detects this condition and causes an interrupt into a **trap-handler process**. The trap handler arranges for the required block to be transferred into main store so that execution can subsequently continue.

The surviving descriptions of the System 250 virtual-store mechanism therefore establish a direct relationship between:

```text
reference to non-resident block
        |
        v
hardware detection
        |
        v
trap-handler process
        |
        v
make block resident
        |
        v
continue interrupted computation
```

Secondary descriptions of the System 250 architecture independently describe the same mechanism: a segment can have an SCT entry while having no primary-memory allocation, and first reference to such a segment causes a trap. The operating system then allocates or restores primary storage and updates the relevant SCT state.

**DOCUMENTED — existing nonresident blocks and virgin new blocks:** England's *Operating System of System 250*, paragraphs 13–15, applies this mechanism to both cases. For an existing nonresident block, the required contents are transferred from disk into allocated main-store space. For a virgin new block, the Store Allocator has already allocated backing-store space at creation and returned its capability; first reference uses the same nonresident-block mechanism to establish main-store residence, but the actual disk transfer is short-circuited because the block does not yet contain useful information. Thus the transfer step in the dispatch path below is conditional on useful contents being present. See the [working transcription](../transcriptions/operating-system-of-system-250.md#virtual-memory---storage-media-to-blocks).

### Process-management boundary: interrupt acceptance

The generic SIW/D15/C(N) acceptance mechanism and its relationship to process transition are now maintained in [PP250 execution and process model](pp250-execution-and-process-model.md). This note retains the interrupt/trap-specific reconstruction built on that mechanism.


### STRONG RECONSTRUCTION — Program Trap is the non-resident-segment mechanism

The documentary evidence combines into a particularly coherent explanation of Program Trap.

A process possesses a legitimate capability for a segment, but the segment is not presently resident in main store. The capability is represented in a state with an all-zero ACCESS field.

Attempting to use it therefore does **not** produce an Access Violation or other protection fault. Hardware detects the zero ACCESS field and generates **Program Trap**.

The processor records the accepted condition in its protected state:

```text
D15.6      = Trap Accepted
MIF20-23   = capability register C(n)
```

The processor then enters the normal-interrupt machinery through `C(N)`. The resulting Normal Interrupt/trap-handler process can use Internal Mode to inspect this processor state.

`D15` tells the handler that the accepted event is a Program Trap rather than an SIW correlation result.

`MIF20–23` tells it which capability register was involved in the trapped reference.

From the capability/SCT state associated with that register, the operating system can determine the non-resident segment which must be made available.

The resulting reconstructed sequence is:

```text
program references segment through C(n)
        |
        v
segment is not presently resident
        |
        v
capability has ACCESS = 0
        |
        v
processor detects ACCESS = 0
        |
        v
PROGRAM TRAP
        |
        +----> D15.6 = Trap Accepted
        |
        +----> MIF20-23 = C(n)
        |
        v
normal interrupt acceptance
        |
        v
C(N) / Normal Interrupt Block
        |
        v
automatic CHP
        |
        v
Normal Interrupt / trap-handler process
        |
        v
inspect protected processor state
using Internal Mode
        |
        +----> D15 identifies Program Trap
        |
        +----> MIF20-23 identifies C(n)
        |
        v
identify segment represented through C(n)
and its SCT state
        |
        v
allocate/restore the required main-store block
        |
        v
update the relevant SCT/capability state
        |
        v
interrupted computation can subsequently continue
```

This explains why the architecture needs both **Program Trap** and **Fault Interrupt**.

They represent fundamentally different conditions:

```text
valid authority, object unavailable
        |
        v
ACCESS = 0
        |
        v
PROGRAM TRAP
        |
        v
recoverable storage-management action


invalid use of authority
        |
        +---- outside BASE/LIMIT
        |
        +---- operation not permitted by ACCESS
        |
        v
FAULT INTERRUPT
```

Program Trap therefore appears to represent **temporary unavailability of an otherwise legitimate object**, whereas Fault Interrupt represents an architectural failure or violation.

### STRONG RECONSTRUCTION — apparent singular purpose of Program Trap

No other normal architectural use of **Program Trap** has yet been found in the corpus.

The known alternatives are accounted for elsewhere:

- processor and I/O activity is represented through the System Interrupt Word;
- interval-timer events have their own mechanism;
- BASE/LIMIT violations generate Fault Interrupt;
- ACCESS permission violations generate Fault Interrupt;
- hardware, parity, sumcheck, watchdog and related failures are represented by MIF/Fault Interrupt state;
- a Program Trap occurring under the exceptional inhibited-interrupt condition is separately represented as a Trap Fault.

There is therefore presently no evidence for Program Trap being a general software-exception facility analogous to the trap mechanisms of many other architectures.

The evidence instead points strongly to Program Trap being specifically the System 250 mechanism for the recoverable **non-resident-segment/page-in condition**.

This exclusivity remains a **strong reconstruction**, rather than a documented architectural rule, because no source examined so far explicitly states that Program Trap can have no other cause.

### SECOND GROUP is not required for normal Program Trap handling

Earlier consideration of the Program Trap path raised the possibility that SECOND GROUP might be required to give the incoming handler access to processor state such as `D15`.

That hypothesis is unnecessary.

The documented Internal Mode mechanism already provides the means by which the Normal Interrupt/trap-handler process can address the relevant special-purpose processor state.

The reconstructed normal Program Trap path therefore requires no SECOND GROUP transition:

```text
Program Trap
    |
    v
processor records D15 / MIF state
    |
    v
automatic CHP through C(N)
    |
    v
Normal Interrupt / trap-handler process
    |
    v
Internal Mode access to processor state
```

SECOND GROUP remains part of the separately reconstructed startup/fault-startup mechanism. No connection between SECOND GROUP and normal Program Trap entry is asserted here.

### OPEN EVIDENCE QUESTION

One documentary question remains:

> Does any surviving System 250 source explicitly state that Program Trap is used **only** for the non-resident-segment mechanism?

No alternative Program Trap use has yet been found, and the known interrupt and fault conditions are accounted for by other mechanisms. Nevertheless, until an explicit statement of exclusivity is found, “Program Trap is exclusively the page-in/non-resident-segment trap” should remain classified as **STRONG RECONSTRUCTION** rather than DOCUMENTED.

## 5. Software dispatch after normal interrupt entry

The hardware mechanism need not know the complete policy for resolving the condition. Its responsibility is to detect the architecturally defined condition and perform the protected transition into the configured Normal Interrupt process.

For the non-resident-segment case reconstructed in §4, the attempted reference encounters the all-zero ACCESS state and the processor generates a Program Trap. The accepted trap is recorded in protected processor state, after which normal interrupt entry proceeds through `C(N)`, the NIB and automatic `CHP`.

The Normal Interrupt/trap-handler process can then inspect that protected state using Internal Mode and perform the storage-management policy that the processor itself does not need to contain.

The reconstructed storage-management path is therefore:

```text
ordinary process
      |
      | references non-resident segment
      v
capability ACCESS = 0
      |
      v
PROGRAM TRAP
      |
      +--> D15.6 = Trap Accepted
      |
      +--> MIF20-23 = capability register
      |
      v
C(N) -> NIB -> target Dump Stack
      |
      v
automatic CHP
      |
      v
Normal Interrupt / trap-handler process
      |
      | inspect D15 / MIF using Internal Mode
      v
storage-management software
      |
      +-- identify the required segment
      +-- allocate/identify main-store location
      +-- arrange backing-store transfer
      +-- update the relevant SCT/capability state
      v
interrupted computation can subsequently continue
```

This explains how storage-management software can be triggered without requiring the microcode to contain the storage-management policy itself.

## 6. SPECIAL and loading the special capability-register bank

The next question is how the running system establishes `C(N)` in the first place.

The Pocket Reference places **MIP at Dump Stack offset octal `20`**. MIP is therefore part of the process state involved in change process.

Patent material identifies **SPECIAL** as a one-instruction state in the Primary Indicator Register. When SPECIAL is set, a `LOAD CAPABILITY` instruction can address the corresponding **special-purpose capability register** instead of the ordinary general-purpose C register.

The direction of `LC` is important:

```text
stored capability  --LC-->  capability register
```

Thus SPECIAL provides a controlled mechanism by which an ordinary stored capability can be loaded into a special capability register such as `C(N)`. This is distinct from Internal Mode. It does not require inventing a supervisor mode or an unrestricted ability to fabricate processor capabilities.

## 7. Reconstructed C(S) to C(N) bootstrap

The individual observations constrain a coherent startup chain:

1. `C(S)` is the hardware-rooted Fault Start-Up Block mechanism.
2. The fault/startup path performs checkout and eventually establishes a legitimate initial process.
3. MIP is part of the Dump Stack process image and is restored as part of process entry.
4. SPECIAL is a MIP state permitting one `LC` to address the special capability-register bank.
5. The running system requires `C(N)` before normal automatic interrupt entry can operate.
6. Therefore the `C(S)`-rooted startup process provides the ancestry by which the initial running system can establish `C(N)`.

The current reconstructed bootstrap is:

```text
C(S)
  |
  v
fault/startup + checkout
  |
  v
initial legitimate process image
  |
  | includes MIP at Dump Stack offset 20
  | with SPECIAL available for the required LC
  v
initial process entered
  |
  v
LC loads C(N)
  |
  v
Normal Interrupt Block becomes the configured
normal software-entry mechanism
  |
  v
subsequent normal events can cause automatic CHP
```

This is recorded as **reconstructed architecture**, not as an unresolved mechanism merely because no surviving source examined so far states the entire chain in one sentence.

The reconstruction does **not** assert that ordinary normal interrupts go back through `C(S)`. `C(S)` roots the initial trusted transition. The running system then establishes `C(N)`, and `C(N)` is the normal operational path thereafter.

## 8. Why this reconstruction is preferable to a hidden supervisor mechanism

An alternative explanation would require some additional undocumented root of processor authority: for example a hidden supervisor mode, an unrestricted internal capability, or a special privileged instruction capable of writing `C(N)` independently of the capability system.

That would be a substantial architectural addition for which the surviving material examined here gives no need.

The `C(S)` → process MIP/SPECIAL → `LC` → `C(N)` chain instead uses mechanisms that are independently visible in the surviving architecture:

- the hardware-rooted `C(S)` startup path;
- checkout and legitimate process creation;
- Dump Stack process state;
- MIP;
- SPECIAL;
- `LC`;
- the special C-register bank;
- `C(N)`;
- the NIB;
- automatic change process.

The reconstruction therefore closes the bootstrap loop without introducing a second privilege architecture by assumption.

## 9. Evidence and reconstruction boundary

The conclusion should not be weakened to "unknown until an explicit manual sentence is found." Equally, reconstruction must not be presented as a quotation from a source.

The appropriate distinction is:

**Observed/documented components:** the named special registers; the Dump Stack structure including MIP; CHP process-state transfer; SPECIAL's one-instruction special-register selection; the role of `C(N)`/NIB in normal interrupt entry; the `C(S)` fault/startup role; and the SCT/unusable-capability mechanisms described by the surviving material.

**Reconstructed architecture:** these components jointly imply a startup ancestry in which `C(S)` establishes the first legitimate process state from which `C(N)` can be loaded, after which normal events use `C(N)` rather than repeating the `C(S)` checkout path.

This reconstruction is falsifiable. Newly recovered material describing initial special-register setup should fit this chain or require the model to be revised.

## 10. Architectural conclusion

The resulting picture is compact:

```text
                         BOOTSTRAP

C(S) -> checkout -> initial process -> MIP/SPECIAL -> LC -> C(N)
                                                           |
                                                           v
                                                    Normal Interrupt Block
                                                           |
                         NORMAL OPERATION                  v

ordinary execution -> protected exceptional state -> automatic CHP
                                                           |
                                                           v
                                                Normal Interrupt process
                                                           |
                                                           v
                                                   software policy
```

The important separation is between **protected transition** and **software policy**. The processor provides the former. Capability-constrained processes provide the latter.

This architectural result should remain distinct from any later M/H/T interpretation. The M/H/T model may use this mechanism as evidence, but it is secondary to the reconstructed PP250 architecture described here.


### Trap dispatch, IHP responsibility and process resumption

The minimum coherent trap mechanism can now be stated without assuming any particular operating-system process-management structure.

Consider a running process **A**, whose current Dump Stack is **DS-A**. When A incurs a trap, the processor records the trap state and dumps A into DS-A. The processor then uses `C(N)` and the Normal Interrupt Block to obtain the Interrupt Handler Process Dump Area Pointer (IDAP). Through the Master Capability Table this identifies **DS-IHP**, the Dump Stack of the Interrupt Handler Process (**IHP**). `C(D)` is changed to DS-IHP and IHP is undumped and begins execution.

Thus the hardware transition is:

```text
A executing
C(D) = DS-A
      |
      | trap
      v
record trap state
      |
      v
dump A -> DS-A
      |
      v
C(N) -> NIB -> IDAP
      |
      v
MCT lookup -> DS-IHP
      |
      v
C(D) := DS-IHP
      |
      v
undump IHP
      |
      v
IHP executing
```

This should be understood as a **process change**, not as a subroutine call with an implicit return to A. Once IHP has been entered, the processor has preserved A and transferred responsibility for dealing with the trap to software. The IHP may ultimately cause A to resume, or a richer operating system may select some other process.

No additional hardware mechanism for passing DS-A to IHP is required by the minimum architecture. In the simplest coherent system, consisting only of A and IHP, IHP can already possess the authority needed to access DS-A. More elaborate process tables, Process Bases, scheduling structures and mappings between processes and Dump Stacks are operating-system mechanisms layered above this minimum requirement.

For a recoverable trap, such as an access to a virtual-memory block which is not currently resident, IHP corrects the condition and causes A to be resumed. A resumes at the instruction which caused the trap: the saved IAR has not advanced past that instruction, so the instruction is retried after the condition has been corrected.

```text
A executing instruction X
      |
      | recoverable trap
      v
A dumped -> DS-A
      |
      v
IHP entered from DS-IHP
      |
      | correct condition
      v
resume A from DS-A
      |
      v
IAR -> instruction X
      |
      v
instruction X retried
```

**Evidence status:** the transparent virtual-memory behaviour establishes the requirement for restart of the trapping operation. An explicit primary-source reference for the precise statement that the IAR remains at the trapping instruction, and hence that the instruction is retried on resumption, is still to be supplied.

## Evidence update — 2 October 2026: relocation-triggered normal interruption

**DOCUMENTED OBSERVATION:** US3771146A, Description 121, explicitly uses interruption followed by process restoration to refresh capability registers expanded before relocation. This is an additional use of the existing normal interrupt/CHP path, not evidence for a second general dispatcher. See [canonical SCT refresh mechanism](../architecture/system-capability-table.md#already-expanded-capabilities-during-relocation).

US3757307A, Description 40, 46–48, 49–65 and 68–78, corroborates completion of the current instruction before polling, transient operand/control state at that boundary, atomic interrupt-word handling and automatic CHP restoration. It does not change the conclusions of the completed [normal-interrupt source review](../transcriptions/US3757307A-program-interrupt-facilities-source-notes.md). That record is retained unchanged.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.


### 12 October 2026 — scheduler interval versus timer resolution (open design consideration)

**WORKING RECONSTRUCTION / TO INVESTIGATE:** The documented 100 µs decrement period is the **resolution** of the C(I) interval timer, not evidence that the operating system schedules at that frequency. A proposed 1 ms timer interrupt period would cause 1,000 scheduler entries per second per processor, potentially imposing excessive interrupt, process-dump and scheduling overhead on a 1970s PP250. The reconstructed scheduler must choose an operational interval large enough to allow useful execution between interrupts; do **not** yet assign a numerical time slice without source evidence or a measured implementation model. Look for a documented COS/POS/ROS/PDOS scheduler quantum, C(I) reload value or timer-setting policy, and assess handler/context-switch cost against it. The 300 ms interrupt-acceptance watchdog is a separate deadline and must **not** be assumed to be the scheduler quantum.


## 12 October 2026 — Normal Interrupt Acceptance and Dispatch

**WORKING RECONSTRUCTION — normal interrupt mechanism reconstructed.** This is a consolidated model derived from the available evidence and engineering reasoning, not a claim that every microinstruction has been recovered. The documentary facts and proposed implementation choices are distinguished below.

### Documented architectural constraints

- The interval timer word addressed through C(I) is decremented every 100 µs; reaching zero generates a Normal Interrupt.
- Program Trap and interval-timer expiry are Normal Interrupt sources. The Interrupt Accept Register (D15) has bit 6 set **when a Program Trap is accepted** (US4383297A, para. 13).
- MIP bit 8 inhibits timer interrupts. A Program Trap while MIP bit 8 is set produces a Trap Fault (MIF bit 15).
- The interrupt-acceptance timeout tests whether the Interrupt Accept Register has been accessed within 300 ms following timer expiry **when interrupts are not inhibited** (US4383297A, para. 62).
- C(N) supplies the protected Normal Interrupt entry structure. Normal interrupt entry invokes the process-change/Dump Stack mechanism.
- A nonresident block is detected by hardware; the trap handler queues the affected process and required block, the disc handler arranges residency, and the trapped instruction is retried on rescheduling (England, *Operating System of System 250*, paras. 13–14).

### Reconstructed acceptance and dispatch sequence

1. At an eligible interrupt boundary, hardware selects the Normal Interrupt to accept. If timer expiry and Program Trap coincide, **timer expiry has priority**.
2. **At acceptance**, the microcode immediately sets MIP bit 8 (INHIBIT INTERRUPTS). This belongs to interrupt acceptance, **not** to the general CHP instruction.
3. The accepted interrupt's identity is represented in D15 bit 6: set for an accepted Program Trap, not set for an accepted timer interrupt. This is an indication of **acceptance**, not merely detection.
4. The processor automatically enters the handler through the C(N)-addressed Normal Interrupt Block and Dump Stack/CHP path.
5. The handler reads D15 to distinguish the accepted Program Trap from timer expiry. The access also meets the documented Interrupt Accept Register access requirement for the timeout watchdog; **reading does not need a special read-to-clear effect**.
6. For an accepted Program Trap, the handler **explicitly clears D15 bit 6** with a permitted Internal Mode data-store operation, then follows the trap/storage-management handling path. This is a working software protocol, not a proven register-specific historical instruction sequence.
7. For timer expiry, scheduling machinery chooses the next runnable process. Immediately before dispatch, the scheduler **reloads the timer word addressed by C(I)** with that process's round-robin time-slice count, then dispatches it through CHP.
8. Interrupt inhibition remains in force during protected handling until the next process's suitable indicator state is established. The scheduler, trap-handler and disc-handler **need not be the same software routine**; this model describes the protected entry and dispatch path.

### Timer inhibition and remaining implementation considerations

MIP bit 8 inhibits **acceptance of timer interrupts** while set; therefore expiry during protected handling does not interrupt that handler. The timer still counts down. The 300 ms interrupt-acceptance timeout expressly excludes timer expiry while interrupts are inhibited.

Two narrow implementation details remain for modelling and testing, **not unresolved architectural mechanisms**:

- What occurs if the timer reaches zero with interrupts inhibited and they are subsequently re-enabled (deferred delivery versus another explicitly programmed timer event).
- The historical numerical round-robin quantum and its performance implications. **100 µs is timer resolution, not a scheduling quantum**; a 1 ms scheduling period risks prohibitive overhead, and no numerical quantum is presently established.

**Related authority requirement:** restricted Internal Mode capabilities needed for D15 access are tracked in [capability genesis and resource lifecycle](capability-genesis-and-resource-lifecycle.md); do not duplicate the Store Manager API design here.


## 12 October 2026 — C(N), reentrant scheduler and Program Trap dispatch

**WORKING RECONSTRUCTION — software completion of the normal-interrupt path.** Preserve the earlier reconstruction of acceptance, D15 and automatic CHP. The C(N) Normal Interrupt Block names the incoming Dump Stack, not an instruction address: restoring that prepared process loads its C7/IAR and enters ordinary capability-protected code. We reconstruct that process as a **per-CPU execution instance of England's reentrant scheduler**, incorporating an interrupt-dispatch entry. This does not claim that the scheduler and disc-handling routines are the same code.

**DOCUMENTED (England, *Operating System of System 250*, paragraph 10 and figure 2):** scheduler subroutine code is reentrant, a common Scheduler Capability Block is shown, and all CPUs share **one Ready List**. **DOCUMENTED (ROS/PDOS Pocket Reference SIP):** a process records its running status and CPU number (`r`, `cccc`), and an `x` bit records Ready List membership. These per-process status fields are distinct from the scheduler's operational indexes.

**MINIMUM ADDITIONAL HYPOTHESIS:** the scheduler maintains a **Running List**, alongside the documented Ready List. It gives a direct association from the local CPU to its currently assigned process, avoiding a scan through every Process Base and SIP. Its exact representation (terminated list versus small table), membership conventions while the scheduler itself executes, and atomic update protocol are not specified. We do **not** reconstruct other process-management queues here. The Running List and SIP may contain related but deliberately different state.

### Accepted Program Trap

1. C(N)'s prepared execution context enters the scheduler/normal-interrupt dispatch routine. D15 bit 6 identifies the accepted Program Trap.
2. The documented MIF bits 20–23 identify the offending **capability-register number**, not the identity of the nonresident segment itself (US4383297A, paragraph 74).
3. Using its local CPU identity and the proposed Running List, the scheduler finds the interrupted process's **Process Base Enter capability**.
4. **PROPOSED Process Base interface:** an authorised entry operation returns the **saved capability pointer** for the specified register (for example C2), following the documented Process Base-to-Dump Stack relationship (offset 3). This must not be represented as an existing documented command, nor as unrestricted direct Dump Stack access.
5. The scheduler passes that capability pointer, together with the suspended-process association needed for completion, to **Storage Manager** and requests residency. The exact capability-safe parameter representation and service interface remain implementation choices.
6. The process waits while the VM/disc handler arranges residency. England paragraphs 13–14 explicitly describe the trap handler's disc queue, the disc handler and the eventual return of the process to the Ready List.
7. When residency is established, the affected process is returned to the **Ready List**.
8. **The scheduler is invoked.** When it subsequently selects that process, CHP restores its saved context. England explicitly states that the trapped instruction is **reattempted**. Our mechanism inference is that the saved IAR still identifies the faulting instruction rather than its successor, so the normal CHP fetch retries it; no special return-to-trap-microcode mechanism is needed.

### Timer expiry and idle boundary

On timer expiry, this same per-CPU scheduler instance identifies the displaced process using the Running List, updates ready/running state as appropriate, selects a successor, loads the C(I) timer word with that process's round-robin time slice, and dispatches through CHP. The quantum is not established. When no process is ready, the CPU may do ordinary background/polling work; neither its exact idle-loop implementation nor its MIP transitions are inferred here. In particular, do not confuse interrupt-entry inhibition with a claim that an idle scheduler must remain interrupt-inhibited.

**Boundary:** The Running List, scheduler-as-C(N)-process interpretation, and Process Base retrieval interface are reconstructed software design, whereas the hardware C(N)/CHP path, SIP fields, shared Ready List, reentrant scheduler and instruction reattempt are supported by contemporary sources.


## 14 October 2026 — Program Trap pointer provenance and displaced-process lookup

**PRIMARY EVIDENCE — US3771146A, paragraphs 66–84 and 100–107.** A capability load (`LC`) saves the **original compact 24-bit capability pointer** in the current process Dump Stack's reserved slot for the capability register. For a non-active/passive pointer, the described load microsequence zeros the expanded destination capability-register halves; the register itself therefore cannot be assumed to retain the backing-store identity. Attempted use raises a Program Trap. The trap microsequence places the **offending capability-register number** in MFI bits 20–23 (not the object identity), raises the accepted trap indication, and saves the outgoing process state to its **original Dump Stack**. The Normal Interrupt Block reached through the NIB register then supplies an **incoming handler Dump Stack** and the processor changes to that handler's context.

**CRUCIAL DOCUMENTED STEP — paragraph 84:** The handler uses the offending register number to determine which reserved capability-pointer **slot in the suspended process Dump Stack** to read, tests that stored pointer, and, if the object resides in backing store, arranges for a disc handler to bring it into memory. This establishes *where the required identity is stored* and *how its slot is selected*. It does **not** explicitly describe how handler software obtains authority for the **outgoing Dump Stack** after the register holding its descriptor has been replaced.

**EXISTING WORKING RECONSTRUCTION — relate, do not reinvent:** The 12 October C(N)/reentrant-scheduler section above already supplies the proposed software bridge: the handler runs in the scheduler/dispatch context, identifies the displaced process via per-CPU identity and the proposed Running List (consistent with documented SIP running/CPU fields), and uses the protected Process Base–Dump Stack relationship to retrieve the saved pointer identified by MFI. The proposed Process Base retrieval entry and Running List representation are **not documented original software interfaces**. The patent's register number specifies the *offset*, while this scheduler/process association supplies the *outgoing process identity*; both are needed. England's disc-queue, residency and Ready List behaviour then completes the established VM sequence.

**Rejected distractions:** There is no evidence in this microsequence of a hidden persistent outgoing Dump Stack pointer transferred to the new process, an automatic C(N)/C(D) swap, or a capability register retaining Outform identity after the described zeroing. The dump-address RES REG is used as a microsequence address cursor; its observed use does not establish a handoff into software. SUMCHECK invalidation during relocation must not be equated with nonresidence. The precise historical software lookup remains unverified, but that narrower documentary detail must not be promoted into a claim that the Program Trap/VM path is architecturally unexplained.
