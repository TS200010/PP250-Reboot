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

## 4. SCT failure and deferred normal-interrupt entry

The SCT investigation adds an important detail to the path.

For an active stored capability, `LC` performs the SCT lookup needed to expand the stored representation into a capability register. If the SCT descriptor is valid, the destination C register receives the normal expanded capability.

The surviving interrupt material indicates that an unavailable SCT state can instead result in a distinguished unusable/trap representation being established in the capability register. The full software recovery action need not occur during `LC` itself.

The later attempt to **use** that unusable capability causes hardware detection and normal interrupt entry.

Thus:

```text
stored active capability
        |
        v
       LC
        |
        +---- valid SCT entry ----> usable expanded capability
        |
        +---- unavailable entry --> unusable/trap C-register state
                                           |
                                           v
                                  subsequent attempted use
                                           |
                                           v
                                  normal interrupt condition
                                           |
                                           v
                                         C(N)
                                           |
                                           v
                                          NIB
                                           |
                                           v
                                     automatic CHP
```

This is more precise than saying that `LC` simply "page faults".

## 5. Software dispatch after normal interrupt entry

The hardware mechanism need not know the complete policy for resolving the condition. Its responsibility is to detect the protected exceptional state and perform the protected transition into the configured Normal Interrupt process.

The software process entered through `C(N)` can inspect the saved process/fault state and determine what action is required. Depending on the cause this can lead to storage management, backing-store transfer, I/O/resource handling, scheduling, or other system action.

For the storage-management case the reconstructed path is:

```text
ordinary process
      |
      | attempts to use unavailable segment authority
      v
hardware detects unusable capability state
      |
      v
C(N) -> NIB -> target Dump Stack
      |
      v
automatic CHP
      |
      v
Normal Interrupt process
      |
      | inspect saved condition/reference
      v
storage-management software
      |
      +-- determine required segment
      +-- allocate/identify main-store location
      +-- arrange backing-store transfer
      +-- update the SCT representation
      +-- establish valid descriptor/check state
      v
waiting process can eventually be made runnable
      |
      v
original computation resumes/retries
```

This explains how user/system storage-management code can be triggered without requiring the microcode to contain the storage-management policy itself.

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
