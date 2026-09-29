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
incoming-process Dump Stack capability/pointer
        |
        v
automatic CHP
        |
        v
Normal Interrupt process
```

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
