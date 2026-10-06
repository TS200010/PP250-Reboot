# PP250 boot and processor startup: research reconstruction

**New evidence, 2 October 2026:** the dated evidence update at the end records which earlier claims are superseded; the original reasoning is preserved.

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

The complete COS/POS/ROS/PDOS Dump Stack layouts and their process-management fields are now maintained in [PP250 execution and process model](pp250-execution-and-process-model.md). They were moved there to avoid maintaining the process model in the startup note.

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

## 8.1 Hypothesis: C(S) is a common cold-start and fault-entry root

**HYPOTHESIS / strong reconstruction.** The processor's special capability register **C(S)** identifies the same architectural Start-Up/Fault Block during bare-metal cold startup as it subsequently identifies during fault handling. There is not a distinct startup object which is later replaced by a fault object.

**Documented evidence.** C(S) is a special-purpose capability register separate from the ordinary numbered capability-register set. Its contents are preset by processor hardware following power-up. The fault mechanism uses C(S) to address a four-word special block containing the information required to establish the Special Capability Table and a reserved segment pointer identifying a Dump Stack. Software cannot arbitrarily replace C(S)'s capability definition. The exceptional mutable portion is the high-order part of its Base address, which the fault mechanism itself can alter/increment.

**Reasoning.** If C(S) initially designated one structure for cold startup and subsequently had to designate a different structure for fault handling, some mechanism would have to transform C(S) between those two roles. No such mechanism has been identified. More importantly, the documented restrictions on modification of C(S) appear not to permit such an arbitrary transformation.

The mutable high-order Base bits instead have a natural explanation in the documented fault-recovery mechanism: they select another storage module containing another copy of the same special fault/startup information. Thus the modification changes the **physical copy selected**, rather than the **architectural object selected**.

This interpretation becomes particularly clear for configurations using 32K storage modules. If alteration of those high-order bits has no effect on the address within a 32K module, they cannot be being used to transform a startup-block address into a different fault-block address within that module. Their purpose is consistent with storage-module selection/redundancy.

Therefore the simplest reconstruction consistent with the evidence is:

```text
                         power-up
                            |
                     hardware presets
                           C(S)
                            |
                            v
                 Start-Up / Fault Block
                            |
                +-----------+-----------+
                |                       |
          cold-start use           fault-entry use
                |                       |
                v                       v
          Dump Stack /             Dump Stack /
        native PP250 process     native PP250 process
```

The documented fault-tolerant system uses the fault path to enter a **checkout process**, but checkout is software/system policy layered on the processor mechanism. At the raw PP250 architectural level the evidence establishes only that the structure can identify a Dump Stack from which **PP250 native execution** is entered; it does not require checkout, ROS, PDOS, or any other particular software component.

The diagram deliberately does **not** assert that cold startup and fault entry necessarily use identical microcode, nor that they necessarily select the same Dump Stack. The hypothesis concerns the **root structure addressed by C(S)**.

A fault retry may alter the permitted high-order C(S) Base bits:

```text
C(S) -> copy of Start-Up/Fault Block in SM0

                 fault/retry
                      |
                      v

C(S) -> corresponding copy in SM1
```

That is a change of **replica**, not a change from "startup meaning" to "fault meaning."

**Remaining uncertainty.** We still lack primary evidence describing the complete cold-power-up microsequence. Consequently, we should not yet state as fact that power-up automatically traverses C(S) in exactly the manner used by fault entry. What we can say is that **if C(S) is the cold-start root, the architecture gives us no evidence of a separate startup version of it that is subsequently converted into the fault root; the available evidence instead strongly supports a single persistent Start-Up/Fault structure.**

## 8.2 Hypothesis: initial native code must establish the normal-system roots

**HYPOTHESIS / strong architectural deduction.** C(S) provides the hardware-established authority root from which the processor can enter its first native PP250 execution context. That initial execution context does not, by itself, constitute a normally operating processor. Before normal system operation can begin, three further special capability registers must be established:

- **C(C)** — the normal System Capability Table root, required for normal capability-pointer resolution.
- **C(I)** — the system interrupt/interval mechanism root, required for normal interrupt polling.
- **C(N)** — the Normal Interrupt Block root, required for normal interrupt entry.

C(S) is not one of these because it is preset by the processor following power-up and provides the bootstrap root itself. C(D) is not one of them because the Dump Stack capability is established as part of entering the initial process.

The reconstructed cold-start sequence therefore extends one stage beyond the C(S) common-root hypothesis:

```text
POWER UP
   |
   v
hardware establishes C(S)
   |
   v
C(S) -> Start-Up Block
   |
   v
special startup capability environment
   |
   v
initial Dump Stack
   |
   v
automatic CHANGE PROCESS
   |
   v
FIRST NATIVE PP250 EXECUTION
   |
   |  initial native code must establish:
   |
   +---- C(C)  normal System Capability Table
   |
   +---- C(I)  interrupt/interval mechanism
   |
   +---- C(N)  Normal Interrupt Block
   |
   v
NORMAL PP250 PROCESSOR OPERATION
```

**Reasoning.** These three registers represent persistent machine state required for normal processor operation but which cannot simply be assumed to exist at bare-metal power-up. C(C) is required for the normal capability environment: without the normal System Capability Table root, ordinary reserved-segment-pointer/capability resolution cannot operate normally. C(I) is required for the normal interrupt/interval polling machinery. C(N) is required for normal interrupt entry.

Conversely, none of these three need be assumed merely to explain the exceptional transition from power-up into the first native PP250 process. That transition is rooted in hardware-established C(S), uses the special startup capability environment, identifies a Dump Stack, and enters its process through CHANGE PROCESS. The Dump Stack transition supplies C(D) as part of the process context, while C(S) already exists as the exceptional hardware root.

The resulting architectural boundary is therefore: **C(S) gets the processor into native PP250 execution; establishment of C(C), C(I), and C(N) is what is then required to move from that bootstrap execution environment to normal processor operation.**

## 9. Ground zero: establishing the first executable authority state

This section records the central reasoning that emerges when the documented fault/startup mechanism is considered as an architectural bootstrap mechanism rather than merely as ROS/PDOS checkout policy.

### 9.1 Start from the first instruction, not from an operating system

The reconstruction question can be reduced to a sharper one:

> Immediately before the first ordinary instruction is fetched, what legitimate capability state exists, and how did the processor obtain it?

A conventional bootstrap model tends to assume that some privileged code is already executing and can construct the machine state it needs. That assumption is inappropriate here. System 250 has no ordinary unrestricted supervisor mode that can simply manufacture capabilities from data.

It is useful to separate two aspects of processor state conceptually:

- **computational state** — data registers, arithmetic state, instruction sequencing and the other state needed to continue an ordinary computation; and
- **authority state** — the capabilities and tables that determine what code and data that computation is permitted to reach.

In the terminology used elsewhere in this research, these are the Turing and Church aspects of the machine. At genesis there is no previously running ordinary computation whose Turing state must be resumed. The first architectural requirement is therefore to establish a legitimate **Church/authority state** from which an executable process context can be entered.

There must of course be enough exceptional hardware sequencing to perform the startup transition. The point is that this is not yet an ordinary PP250 process executing arbitrary instructions.

### 9.2 C(S) provides a hardware root

The Pocket Reference explicitly names **C(S)** as the **FAULT START-UP BLOCK** capability. [P2] further reports that C(S) is preset by the processor following power-up, apart from the base bits manipulated during the fault sequence.

This is the crucial break in the apparent circularity. The processor does not need an already functioning normal SCT in order to find the first protected structure. C(S) is exceptional hardware-established capability state.

The communication-control description reinforces the role: C(S) permits access to a *special limited area of store containing the necessary parameters*, with corresponding blocks available in successive store modules for fault retry.

Thus the first trusted chain begins:

```text
processor hardware
      |
      v
     C(S)
      |
      v
four-word Start-Up / Special Fault Block
```

### 9.3 The four words establish a miniature capability universe

[P1] reports the four-word per-processor block as:

| Word | Function |
|---:|---|
| 0 | Sum-check |
| 1 | Base of the special capability table |
| 2 | Limit/type information for that table |
| 3 | RSPC-0, reserved segment pointer to the selected process Dump Stack |

This explains why the block does not need to contain bootstrap instructions or a complete process image. Its purpose is more fundamental: it supplies the parameters needed by the fault/startup microcode to establish a **small, separate SCT environment**.

Words 1 and 2 are loaded by the microcode into the capability-table/master-capability state, subject to the reported validation. The processor has therefore moved from one exceptional hardware capability, C(S), to a restricted capability namespace without executing ordinary software.

Conceptually:

```text
C(S)
 |
 v
Start-Up Block
 |
 +-- base/limit --> special SCT
 |
 +-- RSPC-0 -----> entry in that SCT
```

This special SCT is not the normal operating system's SCT. It is a deliberately small authority universe sufficient for the recovery/startup transition.

### 9.4 RSPC-0 leads to a process, not directly to an instruction

RSPC-0 is interpreted through the newly established special SCT. [P1] reports that it selects the capability for the checkout process Dump Stack.

The important architectural point is independent of the historical checkout policy. The underlying mechanism is:

```text
special SCT
    |
 RSPC-0
    |
    v
process Dump Stack capability
    |
    v
automatic CHANGE PROCESS
    |
    v
executable process context
```

The fault-tolerant ROS/PDOS system chose to make that process a **checkout process**. It could test the processor and, if successful, participate in returning it to system service. That is software policy layered above the architectural transition.

The basic mechanism does not intrinsically mean "run checkout". It means, in effect, **enter the process identified through this restricted startup authority structure**.

This distinction matters for reconstruction. An emulator or reconstructed machine should not bake the historical checkout policy into the fundamental startup mechanism unless further evidence shows that the hardware itself did so.

### 9.5 Automatic CHANGE PROCESS is the boundary

This resolves a question that had appeared circular when approached from the first instruction.

An ordinary instruction does **not** have to create C6, C7, the SCT and the authority needed to fetch itself. The documented fault machinery establishes protected capability state in microcode and then performs an **automatic CHANGE PROCESS**.

The incoming Dump Stack supplies the process state consumed by that transition. As described earlier in this note, its fixed and active-frame structures provide the saved C0-C5 and D0-D7 state, pushdown state, indicators and ultimately the C6/C7/IAR execution frame.

The sequence is therefore:

```text
NO TRUSTWORTHY ORDINARY PROCESS STATE
              |
              v
       hardware C(S)
              |
              v
        Start-Up Block
              |
              v
         special SCT
              |
              v
 RSPC-0 -> process Dump Stack
              |
              v
     automatic CHANGE PROCESS
              |
              v
 legitimate capability + computation state
              |
              v
      FIRST ORDINARY INSTRUCTION
```

The first ordinary instruction is consequently fetched **after** an executable C7/IAR context has been established. There is no need to postulate an unprotected bootstrap instruction stream that later turns capability protection on.

### 9.6 What is documented and what is inferred

The following central pieces are supported by the cited material:

- C(S) is the Fault Start-Up Block capability [R1].
- C(S) is reported as processor-preset following power-up [P2].
- C(S)/SSCR reaches a four-word special block [P1/P2 comparison].
- that block contains the special-table descriptor information and RSPC-0 [P1].
- fault microcode establishes the restricted table, resolves RSPC-0 to the checkout Dump Stack and performs automatic CHANGE PROCESS [P1].
- a Dump Stack contains the state needed for process restoration, including the route to the C6/C7/IAR frame [R1/P3].

The important **inference** is the architectural interpretation: this machinery constitutes a hardware root-of-authority transition capable of taking a processor that has no trustworthy ordinary process context into a fully capability-constrained executable process.

It is also an inference that cold power-up and fault recovery converge on exactly this path. The evidence that C(S) is preset following power-up makes that connection compelling enough to investigate, but it does not yet prove that the complete cold-start and fault-entry microsequences are identical.

### 9.7 Why this matters to the reconstruction

This substantially narrows the bootstrap problem.

We no longer need to invent a privileged bootstrap program that fabricates initial capabilities, nor assume that the first software instruction somehow loads its own C7. The architecture already contains a plausible finite transition from hardware-established authority to ordinary protected execution.

The remaining ground-zero questions are correspondingly concrete:

1. What exact protected and special-register state is established by power-up before the C(S) sequence begins?
2. Does cold power-up execute the same C(S) -> special SCT -> RSPC-0 -> automatic CHANGE PROCESS sequence as fault recovery, or merely a closely related one?
3. How were the Start-Up Block, special SCT, initial Dump Stack and first code populated in store for a completely cold system?
4. What exact state does automatic CHANGE PROCESS establish immediately before the first instruction fetch?
5. Where does Hamer-Hodges' recollection that PP250 could be booted in three instructions fit? The three instructions, if correctly remembered, now appear more plausibly **after** the hardware has established the initial protected process context rather than before capability state exists.

Until those are answered, the reconstruction should preserve this boundary:

> **Hardware establishes a minimal legitimate authority universe; automatic process transition converts it into an ordinary executable PP250 context; software policy begins on the far side of that transition.**

## 10. Unresolved questions and research leads

1. **Virgin memory and stored capabilities:** who initially loads the Special Fault Block, SCT, Dump Stack and checkout code? How are genuine stored capabilities established before any ordinary process can execute? Raw data bit patterns must not simply be assumed to confer capability authority.
2. **Initial pointer and template:** which system-generation or process-template operation constructs the initial C6/C7/IAR frame and absolute saved pushdown pointer? The pocket reference mentions a Process Template in its error-control definition, but does not supply the construction algorithm.
3. **Cold versus fault entry:** are their microsequences identical, or do they merely converge? What happens to outgoing-state saving when no valid old C(D) exists?
4. **Capability validity:** what precise storage and instruction rules prevent capability fabrication in a Dump Stack containing both data and capabilities? The conversation considered and then questioned a per-word tag explanation; no tag implementation is established here.
5. **Version correspondence:** how do SSCR/MCR, C(S)/C(C), and C(C1)/C(C2) map across processor descriptions? Which later-patent details apply to the user's 1976 machine?
6. **Loading equipment:** maintenance hardware, another processor, retained memory, tape/disk loading and INFORM/OUTFORM are research possibilities, not established boot mechanisms. No ROM bootstrap is established or ruled out by this note.

Priority evidence targets are the original fault-patent figures and microsequence, processor startup/maintenance manuals, process-template documentation, and the repository's System 250 General Information material. These findings extend questions left open in [the architecture WIP](../architecture/faults-interrupts-startup.md); they do not silently amend that document.

## 11. SECOND GROUP and primordial special-capability completion

**Status: HYPOTHESIS.** The early MIP bit 4 is named **SECOND GROUP**. The PP250 has ordinary C0–C7/D0–D7 and second/special C10–C17/D10–D17 register groups (octal). The Pocket Reference identifies C10=C(D), C11=C(I), C12=C(C) and C13=C(N); C14–C17 remain UNKNOWN and may be unused, reserved or M-internal/scratch. C(S) is separate.

Later Wheatley/Andrews material places SPECIAL MODE at the corresponding PIR bit 4 and describes it as lasting for one instruction, allowing `LC` to address a corresponding special-purpose capability register. Until contradicted by early evidence, we test the working hypothesis that SECOND GROUP and later SPECIAL MODE represent the same underlying second-bank selection idea, including the one-instruction lifetime.

### 11.1 Virgin Dump Stack and repeated CHP

The proposed trigger is the **virgin initial Dump Stack/startup condition**, not an arbitrary Dump Stack. M recognises that the required special capability environment is incomplete and, on the relevant primordial CHP/change-process transitions, grants SECOND GROUP for exactly one instruction. Bootstrap uses that instruction to install one already-legitimate capability into a required special C register. SECOND GROUP clears; bootstrap passes through CHP/M again; the cycle repeats while required special state remains incomplete.

```text
virgin initial Dump Stack -> CHP
        -> M sees required special-C state incomplete
        -> SECOND GROUP for one instruction
        -> LC legitimate stored capability into one required C1x
        -> SECOND GROUP clears
        -> CHP -> repeat
        -> required special-C state complete
        -> M no longer grants SECOND GROUP
```

Thus a one-instruction SECOND GROUP is sufficient: the bootstrap receives a fresh one-instruction grant on successive transitions rather than attempting the entire setup in one grant.

#### LC source addressing remains ordinary

A necessary refinement is that SECOND GROUP cannot sensibly be modelled as globally replacing every C0-C7 reference in the selected `LC` with C10-C17. `LC` has distinct roles for the capability register used to address the stored source capability and for the capability register that receives the loaded capability. The working reconstruction is therefore that SECOND GROUP redirects the **LC destination-register selection** to the corresponding second/special C register, while the ordinary C-register bank remains available for addressing the stored source capability.

Conceptually:

```text
ordinary Cx -> addresses prepared block of stored capabilities
                         |
                         v
                    LC reads capability
                         |
              SECOND GROUP redirects
              LC destination selection
                         |
                         v
                 corresponding C1x
```

This distinction is required for the bootstrap hypothesis to be operational: otherwise setting SECOND GROUP would remove access to the ordinary capability needed to reach the prepared source block, leaving the one-shot `LC` with no usable authority from which to fetch C(I), C(C) or C(N). The later SPECIAL description is consistent with destination redirection: it permits `LC` to load the corresponding special-purpose capability register in place of the ordinary destination. The identification of early SECOND GROUP with later SPECIAL remains a hypothesis; this refinement states how that hypothesis must operate if the identification is correct.

### 11.2 Candidate completion logic

A simple candidate implementation is:

```text
present(Cn) = OR(access bits of Cn)
complete    = AND(present(Cn) for Cn in required special set)
```

Assuming reset leaves the relevant access fields zero, M need not understand their OS-level meanings. While `complete = 0`, the primordial transition may grant SECOND GROUP; when `complete = 1`, that route closes. This is a candidate circuit-level reconstruction, not documented gate logic.

Early patent evidence recognises an all-zero/null access code, so `access != 0` is not universally equivalent to “defined capability”. The consequence is simply that a register participating in this particular completion test must receive a genuine non-zero capability representation. A register not required for bootstrap need not participate.

In particular, **C14–C17 must not automatically be included**. If they are M scratch/internal registers, they need neither be set nor feed the OR/AND completion logic. C10–C13 are the currently documented special registers to investigate, but even these must be checked individually because CHP may establish C(D) specially.

### 11.3 Boundary with capability genesis

This mechanism, if correct, explains installation of already-legitimate capabilities into normally inaccessible special processor registers. It does **not** explain creation of the first stored capability:

```text
already legitimate stored capability
             |
       LC under SECOND GROUP
             v
special processor capability register
```

**Superseded:** general runtime capability genesis is not a separate unresolved problem. Protected allocator/resource mechanisms create resources and return their capabilities. The remaining genesis issue is the cold-start provenance of the initial legitimate authority/state.

### 11.4 Evidence tests

Search primary material for: exact early MIP04 SECOND GROUP semantics and lifetime; microcode setting/clearing it during primordial CHP/startup; reset state of required special C registers; reduction/OR detection of their access/type bits; combined completion logic; whether CHP establishes C(D) automatically; startup roles of C(I), C(C), C(N); functions of C14–C17; whether any participating register may remain null; and evidence linking a virgin initial Dump Stack with repeated startup/change-process transitions.

Explicit completion logic over required special C-register state would strongly support this reconstruction. Evidence that SECOND GROUP is freely restorable by ordinary process state, or that primordial startup does not depend on special-register population, would weaken or falsify it.


## 12. MIF, MIP and MIS indicator registers

**PRIMARY EVIDENCE:** Pocket Reference page 8 gives three separate 24-bit indicator registers. They must not be conflated:

- **MIF — Fault Indicators**
- **MIP — Primary Indicators**
- **MIS — Secondary Indicators**

The page-8 transcription is retained at \`transcriptions/System 250 Pocket Reference - pg8.txt\`.

### 12.1 MIF — Fault Indicators

| Bit | Pocket Reference name |
|---:|---|
| 00 | Bus Corrupt |
| 01 | — |
| 02 | Interrupt T/O |
| 03 | — |
| 04 | Spare |
| 05 | Slave T/O |
| 06 | Cap. Parity Fault |
| 07 | Sumcheck Fault |
| 08 | Base/Limit Fault |
| 09 | Interface T/O |
| 10 | Parity Comparison |
| 11 | Read Data Parity |
| 12 | Invalid Operation |
| 13 | Power Failure |
| 14 | Invalid Control Code |
| 15 | Trap with MIP08 |
| 16 | Hardware Fault 1 |
| 17 | W.D.T. Expired |
| 18 | Access Violation |
| 19 | Hardware Fault 2 |
| 20–23 | Capability Register on which failure occurred (LS at 20, MS at 23) |

The MIF table establishes the available named **fault causes**. In particular, \`Sumcheck Fault\` is MIF07 and \`Access Violation\` is MIF18.

**Fault-interrupt provenance — keep the register generations distinct:** US 3,814,919 uses an earlier indicator-register organisation than the May 1976 Pocket Reference. In the patent, **MIP** is the primary indicator register and contains the fault indicators; **MIS** is the secondary indicator register. The patent states explicitly that setting any of **MIP bits 5 through 14** sets the **Common Fault Indicator (CFI) in MIS**, and that CFI starts the fault-interrupt microprogram regardless of other current conditions. The same patent places **FIRST ATTEMPT (F.A.T.) in MIS**.

The May 1976 Pocket Reference must be described using its own later nomenclature: **MIF** is the Fault Indicator register, **MIP** is the Primary Indicator register, and **MIS** is the Secondary Indicator register. Thus Pocket Reference **MIF07 = Sumcheck Fault**, **MIF18 = Access Violation**, and **MIP07 = FIRST ATTEMPT**.

The patent's MIP fault-bit numbering must not simply be relabelled as Pocket Reference MIF numbering. There is clear continuity in several named faults but also changed assignments: for example the patent has capability parity at MIP06, base/limit violation at MIP07, SUMCHECK at MIP08, interface timeout at MIP09, parity comparison at MIP10, read-data parity at MIP11, invalid operation at MIP12, power failure at MIP13 and invalid store control at MIP14; the Pocket Reference has the corresponding later fault names in MIF, with SUMCHECK at MIF07 and Base/Limit Fault at MIF08. Therefore the documented historical statement is **patent MIP05–MIP14 -> patent MIS.CFI -> fault-interrupt microprogram**. Establishing the precise correspondence of that earlier group to the later Pocket Reference MIF organisation is a separate version-mapping question.

### 12.2 MIP — Primary Indicators

| Bit | Pocket Reference name |
|---:|---|
| 00 | =0 |
| 01 | <0 |
| 02 | Overflow |
| 03 | Spare |
| 04 | Second Group |
| 05 | Inh. Interface Flts. |
| 06 | Odd Data Parity |
| 07 | 1st Attempt |
| 08 | Inhibit Interrupts |

**Generation distinction:** the 1976 Pocket Reference identifies **MIP07 = 1st Attempt**, and US3771146A likewise places **First Attempt at primary-indicator bit 7**, alongside **Second Group at bit 4**, with the primary indicators retained in process state. The earlier fault-recovery embodiment in US3814919A instead describes its **F.A.T. in MIS** as internal fault-microprogram state. These statements should not be collapsed into a single register assignment: they document different implementations/generations of the fault machinery. For the 1976 architecture reconstructed here, FIRST ATTEMPT is MIP07; Repton remains valid evidence for the earlier fault/checkout mechanism on its own terms. MIP04 is SECOND GROUP, already discussed in Section 11.

### 12.3 MIS — Secondary Indicators

| Bit | Pocket Reference name |
|---:|---|
| 00 | Microprogram O/F |
| 01 | — |
| 02 | Inhibit Slot Decode |
| 03 | — |
| 04 | — |
| 05 | Interval Timer Matured |
| 06 | Multiply |
| 07 | Divide |
| 08 | Set Read Capability |
| 09 | IAR Decrement |
| 10 | Out = Limit |
| 11 | Trap |
| 12 | Time Up |
| 13 | Cycle Intercomplete |
| 14 | Move |
| 15 | Fault Toggle |
| 16 | Fault Link From B.P.W. |
| 17 | Dump Process Before Int |
| 18 | Internal Mode |
| 19 | Cap. Pointer in OPP |
| 20 | HAD Increment |
| 21 | Even Parity Internal |
| 22 | Busy |
| 23 | Status |

MIS therefore contains secondary microprogram/execution-control state, including \`Trap\`, \`Fault Toggle\`, and \`Internal Mode\`; it is not the CPU Fault Indicator register.

### 12.4 Relevance to capability-genesis trap investigation

The separation is important to the rejected fault-assisted genesis ideas:

- SUMCHECK is specifically **MIF07**.
- Access Violation is specifically **MIF18**.
- FIRST ATTEMPT is **MIP07**, and belongs to the fault-checkout sequence rather than being a generic instruction-retry marker.
- MIS contains additional microprogram state but must not be substituted for either MIF fault causes or MIP FIRST ATTEMPT.

The detailed capability-genesis consequences are recorded in \`capability-genesis-outform-working-reconstruction.md\`.

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

## Addendum — evolution of C(S) fault-block addressing: 8-bit to 12-bit retry field

**Status:** working historical reconstruction. This section preserves both the documentary observations and the hypotheses evaluated while trying to explain the later C(S) addressing rule. It does not promote the current leading explanation to established architecture.

### Documented endpoints

**Earlier fault mechanism — PRIMARY EVIDENCE, [P1], as reported above:** the Special Block Capability Register (SSCR) directly selects the processor's four-word area in the Special Fault Block. The within-store address is established by a hard-wired strapping field; the alterable part selects the store module. On a further fault the store-module-number field is incremented and the corresponding location in another store module is tried. Thus the early mechanism can be represented schematically as:

```text
[ store module number ][ hard-wired corresponding location ]
       alterable                 fixed
```

This is a direct address. No preliminary memory lookup is required to discover where SSCR itself should point.

**Later mechanism — PRIMARY EVIDENCE, [P2], as reported above:** C(S) defines the four-word Special Start-Up Block. The twelve most significant bits of its Base are incremented during the fault sequence. Internal Mode permits only those same twelve Base bits of C(S) to be altered, although all bits can be read. The patent separately describes the module number as the eight most significant Base-address bits.

The later split is therefore potentially:

```text
23              16 15          12 11               0
+----------------+---------------+-------------------+
| module : 8     | high location | low offset : 12   |
|                |    : 4        |                   |
+----------------+---------------+-------------------+
|<----- twelve alterable / fault-sequence bits ---->|
```

The reason for widening the mutable/fault-sequence field from the earlier module selector to twelve Base bits is not explicitly stated in the evidence so far inspected.

### Store-size pressure — current leading reconstruction

Early implementation evidence includes 32K store modules, while System 250 descriptions allow store modules up to 64K and do not require every fitted store module to have the same capacity. Intermediate or heterogeneous module sizes must therefore not be reduced to a simple 32K-versus-64K choice.

A plausible evolutionary pressure is the placement of the Special Fault/Start-Up Block. If the original 32K implementation placed the corresponding recovery structure at a convenient reserved position associated with that store size, retaining the same absolute within-module location in a larger store could leave a permanently reserved hole inside otherwise allocatable storage. Moving the recovery structure to a suitable boundary region for the actual store size would instead require some high within-module address bits to vary as well as the module number.

On this reconstruction, the later twelve-bit field generalises:

```text
early:  [ module : 8 ]

later:  [ module : 8 ][ high within-module position : 4 ]
```

while retaining a fixed twelve-bit low-order offset. A 32K, 48K and 64K store could consequently place the same small recovery structure at the same low-order offset in different high address regions.

**HYPOTHESIS:** this variable-store-size/recovery-placement problem is the reason the later C(S) mechanism makes twelve rather than eight Base bits alterable and subject to fault-sequence incrementing.

**UNRESOLVED:** no primary source yet inspected states that this was the designers' motivation, and the exact historical location of the early Special Fault Block within a 32K store has not yet been established. In particular, placement at or near the upper boundary of the original 32K store remains a reconstruction, not a documented fact.

### Likely software use of the new writable field

The later Internal Mode facility is significant because it gives suitably authorised software controlled access to processor-internal state without making C(S) an arbitrary writable capability. All of C(S) can be read, but only the twelve most significant Base bits can be altered.

**Strong reconstruction:** system configuration/reconfiguration software is a natural consumer of this facility. While the system is healthy, such software can know the installed physical-store configuration and establish an appropriate preferred recovery location in C(S). Fault microcode can subsequently use that already prepared state without depending on the configuration software still being executable.

Conceptually:

```text
healthy configuration/reconfiguration software
        |
        | controlled Internal Mode authority
        v
set C(S) high twelve Base bits
        |
        v
preferred Special Start-Up Block location
        |
        | later processor fault
        v
fault machinery uses C(S) autonomously
```

This division of responsibility would let software supply configuration knowledge while retaining a hardware-rooted recovery path.

**UNRESOLVED:** the specific historical software process or routine that writes C(S)[23:12] has not yet been identified. Store allocator, system configuration and reconfiguration material should be searched for such use before assigning the function to a named component.

### Hypotheses considered and weakened or rejected

#### Multiple replicated Start-Up Blocks at 4K intervals

The twelve/twelve address split initially suggested that a store might contain multiple copies of the Start-Up Block, perhaps one in each 4K region.

**Status: unsupported and currently rejected as the leading explanation.** No evidence has been found for sixteen replicated recovery blocks per module. The early material instead describes corresponding fault structures in different store modules.

#### Blind 4K search as the primary purpose

If the high twelve bits are incremented as an ordinary binary field while the low twelve bits remain fixed, successive candidate addresses would be 4K apart. This suggested a hardware search through candidate regions until a valid recovery block was encountered.

**Status: mechanically plausible but weakened as an intended normal mechanism.** No evidence has been found that the architecture deliberately searches 4K regions in this way. Controlled software configuration of the writable twelve-bit field would provide a much cleaner normal path. Increment/search could still be a fallback consequence of the hardware arithmetic, but that has not been established.

#### A 4K physical bank, SAU unit or redundancy quantum

The twelve low fixed bits also suggested that 4K might be a physical storage-bank, SAU-decoding, allocation or redundancy unit.

**Status: unsupported.** Searches so far have found no evidence that System 250 physical store organisation used a 4K unit of this kind. The address split alone is insufficient evidence for such a physical organisation.

#### Fixed-address per-store indirection that loads C(S)

A further hypothesis proposed two stages: hardware first reads a permanently fixed bootstrap location in a selected store module; that location contains a store-specific C(S) value; C(S) then points to the actual Special Start-Up Block.

**Status: rejected by the evidence currently inspected.** The early SSCR directly contains the hard-wired within-store Fault Block address, and the later C(S) directly defines the four-word Special Start-Up Block. No intermediate per-store record supplying a replacement C(S) address has been found. This hypothesis should not be used unless new primary evidence requires it.

### Provisional historical reconstruction

The current best explanatory chain is:

```text
homogeneous early 32K implementation
        |
        v
corresponding Fault Block location in every store
        |
        v
fault retry need alter only the 8-bit module selector
        |
        v
larger / potentially heterogeneous store modules
        |
        v
fixed 32K-era recovery position becomes undesirable
        |
        v
C(S) variable address extended to
[module : 8][high within-module position : 4]
        |
        v
Internal Mode permits trusted software to configure
those twelve Base bits while the system is healthy
        |
        v
fault hardware retains autonomous increment/retry
```

The first three stages are grounded in early fault-mechanism evidence. The proposed causal connection from variable store sizes to the widened C(S) field, and the proposed configuration-software use, are **RECONSTRUCTION / HYPOTHESIS** pending direct documentary confirmation.

### Questions that would discriminate the hypothesis

1. Where exactly was the original Special Fault Block located within a 32K store?
2. Does the later statement that the twelve most significant Base bits are “incremented” mean an ordinary binary increment of C(S) Base[23:12], including carry between the four high within-module bits and the eight module bits?
3. Which system software writes C(S)[23:12], and under what events: power-up configuration, store addition/removal, fault isolation, or general reconfiguration?
4. Do surviving configuration or store-allocation documents reserve differently located Fault Start-Up Blocks for differently sized store modules?
5. Was widening the variable field from eight to twelve bits explicitly motivated by larger or heterogeneous store modules?
6. If differently sized modules coexist, is C(S) merely a preferred first recovery location with incrementing as fallback, or is there another documented rule for choosing the next module?

## Evidence update — 2 October 2026: verified fault roots and startup alternatives

The supplied [US3814919A](../transcriptions/US3814919A.rtf), [US4383297A](../transcriptions/US4383297A.rtf) and [US4486831A](../transcriptions/US4486831A.rtf) texts now permit direct textual checking of the patent findings previously marked “reported”. The earlier verification boundary and source-inventory statements describe the repository at preparation time. US3814919A is now held as both [PDF](../patents/US-3814919-A.pdf) and transcription. This update does not certify figures or reconcile every microsequence discrepancy.

**DOCUMENTED OBSERVATION:** US3814919A's fault steps S2, S10 and S16 confirm the parity-invalidated old capability state, four-word fault-block descriptor/RSPC-0 root, checkout Dump Stack pointer and automatic CHP. Description 125–134, especially 132, further documents a subsequent-fault path that bypasses the outgoing dump and restores a process through another module. **WORKING RECONSTRUCTION:** this makes entry without a valid outgoing process plausible for cold start; it does not prove that cold start uses this recovery branch.

**DOCUMENTED OBSERVATION:** US3771146A saves primary indicators including SECOND GROUP bit 4 with process state; US4486831A saves/restores PIR (53) and makes SPECIAL bit 4 redirect capability loading for one instruction (58). **WORKING RECONSTRUCTION:** a prepared initial Dump Stack could therefore supply relevant primary state as part of restoration. This competes with, rather than establishes, the hypothesis of automatic detection of incomplete special registers and repeated CHP grants. Early SECOND GROUP and later SPECIAL cannot be assumed identical without version-specific reconciliation.

The root chain is architecturally accounted for by the documented hard-wired/preset startup machinery. Virgin-memory loading/commissioning and exact inert-to-first-process microinstruction sequencing may remain documentary details, but they are **not** a missing primordial-authority mechanism. Nothing here implies arbitrary capability manufacture. Details and source discrepancies remain recorded in the batch review.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.
