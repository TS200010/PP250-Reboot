# PP250-G2 — Recovered System 250 Architecture

## Status

**WORKING ARCHITECTURAL RECONSTRUCTION**

This document gives a coherent architectural description of the mature System 250 represented by the c. 1975–76 evidence. The purpose is to describe the machine on its own terms. It is not a source review, an evolutionary history, or a catalogue of every unresolved implementation detail. Where surviving evidence leaves a detail open but that detail is not required to explain architectural behaviour, it is left unspecified.

Detailed provenance and the reasoning trail remain in the repository's research and transcription material.

## The proposition

**System 250 was not just a capability machine; it was a pure capability machine. No machine like it existed at the time, and none has been created since.**

System 250 has usually been understood and described through its most visible architectural feature: capabilities. That description is correct, but incomplete. It obscures a deeper structure which appears never to have been fully described outside the relatively small community that designed, implemented and programmed the machine. This reconstruction attempts to recover that structure.

We will begin with the architecture as it appeared to the programmer and, in particular, with the unusual mechanism at its heart: the **Enter Capability**. Understanding what an Enter Capability actually does provides the key to understanding System 250. From there we can ask the questions that naturally follow: where do capabilities come from, what establishes and protects a process, and what lies beneath the capability architecture itself? Following those questions down into the processor reveals a structure considerably more interesting than the conventional description of System 250 as an early capability machine suggests.

## Method of reconstruction

This reconstruction proceeds in two directions.

It begins at the highest architectural level and **descends through the architecture**, identifying the hierarchy of concepts from which the PP250-G2 system is constructed. At this stage the concern is what the architecture means, rather than how the processor implements it.

At the bottom of the descent, the processor mechanisms and state available to realise that architecture are **inventoried explicitly**.

The reconstruction then reverses direction. In the **ascent**, those mechanisms are taken one by one and combined from the bottom upward, showing how the processor realises the architectural structures established during the descent.

The descent therefore establishes **what the architecture is**; the inventory establishes **what machinery is available**; and the ascent shows **how that machinery constructs the architecture**.

## 1. Architectural character

### 1.1 System 250 as a protected system

**PP250 provides an unforgeable authority substrate from which software can construct and enforce arbitrary programmer-defined authorities. It enables an authority-structured hardware and software system built recursively upon that substrate: authority protects computation and resources, contains faults and supports recovery from them, while the mechanisms that manage, protect and recover the system are themselves governed by authority from that very same substrate.**

*_(PP250 implementation: Store; C0–C7; Microprogrammed execution.)_*

### 1.2 No privileged operating system

**System 250 has no conventional operating system.** Software may provide services conventionally associated with an operating system—storage management, filing, communications, scheduling, device access and so forth—but those services do not collectively constitute a privileged software layer standing between applications and the machine. They are protected software structures constructed within the same architecture as everything else.

*_(PP250 implementation: C0–C7; Internal Mode; Microprogrammed execution.)_*

### 1.3 Hardware-enforced software structure

Authority is not confined to hardware-defined operations such as reading, writing or executing store. The architectural authority primitives in PP250 provide the foundation from which software can construct higher-level authorities with arbitrary software-defined semantics.

The architecture is described below at machine-instruction level, not because authority is defined at that level, but because this exposes where its enforcement ultimately resides. Higher-level software—languages, compilers and system structures—can create richer semantics and impose additional constraints. Where those structures express authority through the PP250 authority mechanisms, that authority survives their translation into machine code: its enforcement does not depend on the correctness or continued cooperation of the higher-level abstraction, but ultimately rests on mechanisms enforced by the processor itself.

PP250 enforces a software-defined protected structure in hardware: once authority has been restricted to that defined structure, there is no path around it. A program can exercise only that defined authority. The authority restriction survives all the way down to machine code; it does not depend on programming convention, the compiler, or a privileged operating system. It is baked into the very wiring of the processor.

The protected structure and its semantics are defined by software. It may represent a file, process, device, directory, allocator, operating-system service or application object. Such structures can themselves hold and expose authority, allowing them to be composed recursively into complete software systems.

Conceptually, a protected structure combines state with a defined set of operations upon that state. It is readily recognisable as the encapsulated abstraction represented in modern programming languages by objects, classes and other structured types.

*_(PP250 implementation: C0–C7; MIF; Microprogrammed execution.)_*

## 2. The protected-object model

### 2.1 Segment, access and unforgeable token

Before going any further, it is useful to introduce a few definitions and important concepts:

- **Segment** — a block of storage with defined bounds.
- **Access (authority)** — the operations permitted upon that segment.
- **Unforgeable token** — a reference to a segment together with the access permitted through that reference.
- **Protected-token segment** — a segment containing unforgeable tokens.
- Different unforgeable tokens may refer to the same segment while granting different access to it.

*_(PP250 implementation: Store; C0–C7.)_*

### 2.2 Constructing a protected object

A protected object is defined by a **protected-token segment** containing unforgeable tokens. Those tokens give access to the code and data segments from which the object is constructed.

The segments accessible through those tokens constitute the object's **protected space**.

*_(PP250 implementation: Store; C0–C7.)_*

### 2.3 Controlled access to an object

**A further unforgeable token provides controlled access through the protected-token segment.** It does not expose the tokens contained within that segment. Instead, it permits invocation of the functions made available through the object.

**Possession of this token permits its holder only to invoke the functions made available through it.** The implementation of those functions, the data upon which they operate, and the accesses required to perform those operations remain within the protected space. The holder cannot bypass those functions to obtain access to the underlying code, data or tokens. That restriction is enforced by the hardware.

**At this point the primary architectural concept can be stated simply.** An object's protected space is defined by a collection of unforgeable tokens: tokens giving access to the functions that implement its operations and to the data upon which those functions operate. A further unforgeable token provides controlled access to the object without exposing the tokens from which its protected space is constructed.

This construction is deliberately general. Such protected objects can represent structures at essentially any level of a computing system: application objects and complete applications, files and filing systems, memory and resource managers, devices and communications services, network access, or system services themselves. Protected objects may themselves hold tokens giving controlled access to other protected objects, allowing larger structures to be composed recursively from the same architectural primitive, **with the hardware enforcing access and enforcing the bounds of each object's protected space.**

*_(PP250 implementation: C0–C7; Microprogrammed execution.)_*

### 2.4 Different access to the same object

**A protected object need not have a single form of access.** Different unforgeable tokens may refer to the same object while granting different access to it. The object and its protected space remain the same; what differs is the access permitted through each token.

*_(PP250 implementation: Store; C0–C7.)_*

## 3. Creation and propagation of access

### 3.1 Enduring tokens

An unforgeable token has an enduring existence independent of its immediate use by a processor. The segment it identifies, its bounds and the access it grants remain unchanged while the token is stored and when it is subsequently used in computation.

*_(PP250 implementation: Store; C0–C7; C(C).)_*

### 3.2 Creation of unforgeable tokens

Unforgeable tokens are created only by **identically protected mechanisms of the system** (described later). Users and programs may request the creation of new unforgeable tokens, but such a token cannot be created by ordinary computation.

*_(PP250 implementation: Store; C0–C7; C(C); Microprogrammed execution.)_*

### 3.3 Transfer of access

An unforgeable token may be passed to another protected object, thereby giving that object the access represented by the token.

Different tokens may give different access to the same protected object.

*_(PP250 implementation: Store; C0–C7.)_*

### 3.4 The protected system as a graph of access

Protected spaces form **impenetrable islands**, isolated from one another by hardware-enforced boundaries. An object has no access outside its own protected space except through unforgeable tokens.

The islands and the tokens connecting them therefore form a **directed graph of access**.

*_(PP250 implementation: Store; C0–C7; Microprogrammed execution.)_*

## 4. Computation within the protected system

### 4.1 The process

A process is an executing computation defined by its protected space.

*_(PP250 implementation: D0–D7; C0–C7; D17; C(D); Change Process.)_*

### 4.2 Process state

A process has computational state sufficient for its execution to be suspended and subsequently resumed. That state includes its working state, its current protected environment, and its point of execution.

The state exists independently of whether the process is currently executing on a processor.

*_(PP250 implementation: D0–D7; C0–C7; D10; D11; D17; C(D); MIP; Change Process.)_*

### 4.3 The process's protected space

The protected space defines the code that a process may execute and the data and other objects it may access. The process cannot operate outside that space except through another unforgeable token.

*_(PP250 implementation: C0–C7; Microprogrammed execution.)_*

### 4.4 Execution within a protected object

A process executes code within its protected space. That code can operate only upon the data and objects accessible within that space.

*_(PP250 implementation: C0–C7; D17; MIF; Microprogrammed execution.)_*

### 4.5 Invocation of another protected object

A process may invoke another protected object through an unforgeable token possessed by the invoking object. The invoked code then executes within the protected space of the invoked object.

*_(PP250 implementation: C0–C7; D17; Microprogrammed execution.)_*

## 5. System activity and change

### 5.1 Multiple processes

Multiple processes may exist independently, each defined by its own protected space.

*_(PP250 implementation: C(D); Change Process.)_*

### 5.2 Events and interruption

Execution of a process may be interrupted by an event. The event may cause another process to execute.

*_(PP250 implementation: D15; C(I); C(N); MIP; Interval timer; Change Process.)_*

### 5.3 Fault containment

The protected structure of the system confines software faults within protected spaces. Hardware faults that could compromise those boundaries are detected and the faulty hardware isolated, preserving the protected structure of the remaining system.

*_(PP250 implementation: MIP; MIF; Watchdog timer; Microprogrammed execution.)_*

### 5.4 Recovery

Exceptional conditions are distinguished according to their severity. Some permit the affected computation to be suspended, the condition handled, and computation subsequently resumed. More serious faults render the affected computational state unviable and invoke the system's fault-recovery mechanisms.

A condition that would normally be recoverable may itself become a fault when safe recovery cannot be performed.

*_(PP250 implementation: D12; D15; C(N); C(S); MIP; MIF; Internal Mode; Change Process.)_*

### 5.5 Fault recovery

When a fault makes the current computational state unusable, recovery begins from protected state established independently of the affected computation. This permits the faulty computation or hardware to be isolated and execution to be re-established without depending upon the state that has failed.

*_(PP250 implementation: C(D); C(S); MIP; MIF; Internal Mode; C(S) Start-Up Location Control (possibly); One-Shot Second Group LC; Change Process.)_*

### 5.6 Cold bootstrap

At initial start-up there is no existing process from which the protected system can be entered. The processor therefore begins with a protected root established by the hardware itself. From this root the initial protected execution environment is constructed and the first process entered.

*_(PP250 implementation: C(D); C(S); Internal Mode; C(S) Start-Up Location Control (possibly); One-Shot Second Group LC; Change Process.)_*

### 5.7 Resource lifetime

Protected objects may be created and may cease to exist. Their lifetime is independent of the lifetime of any particular process using them.

When an object is no longer required, the resources from which it was constructed may be recovered for reuse.

*_(PP250 implementation: Store; C0–C7; C(C).)_*

### 5.8 Peripheral devices and device control

Peripheral devices are part of the same protected system. Access to a device does not require escape into a privileged I/O mechanism or operating system.

Software controlling a device is itself a protected software structure. It may possess the access necessary to operate that device and expose selected operations to other parts of the system without exposing the device itself.

Device control can therefore be structured in exactly the same way as other protected services. A program may be given access to a service that uses a device without thereby acquiring access to the device, its controller, or the other operations that controller can perform.

*_(PP250 implementation: Store; C0–C7; MIP; MIF; Microprogrammed execution.)_*

### 5.9 What we have established

We can now return to the claim made at the beginning: **System 250 was not just a capability machine; it was a pure capability machine.**

There is no privileged operating system sitting above the applications and below the hardware. There is no supervisor mode into which trusted software escapes when it needs to do something that ordinary software cannot do. Instead, the entire software system is constructed from mutually protected pieces, each able to do only what the access it has been given allows it to do.

Some of those pieces provide application functions. Others provide filing, storage management, communications, device control, scheduling or other services that would conventionally be regarded as parts of an operating system. Architecturally there is no distinction. They are all software protected in the same way, and they interact through the same mechanisms.

Hardware enforces the boundaries, but **software defines what those boundaries mean**. A protected structure might represent a file, a device, a process manager, an application or an entire subsystem. The hardware neither knows nor needs to know which.

That is already substantially different from simply adding protected pointers to a conventional computer. But it leaves an awkward question.

**If there is no privileged operating system underneath all this, what is underneath it?**

## 6. Raw machine resources

At this point the architectural descent reaches the machine resources from which the protected system is constructed. These resources do not themselves describe the protected-object architecture developed above; they are the processor and storage substrate available to realise it.

The relationship between these resources and the architecture can be represented as:

**M⟨H,T⟩**

**T — Turing computation** — conventional instruction execution upon data.

**H — Church computation** — computation expressed through capabilities and protected functional structures.

**M — microprogram computation** — everything that executes under the control of the processor microprogram. M can initiate protected process transitions through entry capabilities available only to M.

This is not three separate processors. It is one processor in which H and T are realised within the encompassing microprogram computation M.

Much of the machinery listed below will at first appear obscure. The objective of the following sections is to show how this substrate is used to realise the architecture established on the descent.

### 6.1 Store

The machine provides shared primary storage for **data (including instructions) and capabilities**.

### 6.2 Processor registers exposed to H and T

M exposes two sets of working registers to H and T:

- **D0–D7** — eight 24-bit data registers.
- **C0–C7** — eight 48-bit capability registers.

### 6.3 Special data registers

The processor contains further data registers, not addressable by normal program instructions:

- **D10** — absolute Dump Stack pushdown pointer.
- **D11** — watchdog timer.
- **D12** — first-fault MIF copy.
- **D13** — unassigned in the Pocket Reference.
- **D14** — unassigned in the Pocket Reference.
- **D15** — interrupt accept register.
- **D16** — unassigned in the Pocket Reference.
- **D17** — instruction address register (IAR).

### 6.4 Special capability registers

The processor contains further capability registers, not addressable by normal program instructions:

- **C10 / C(D)** — Dump Stack.
- **C11 / C(I)** — contains an Enter Capability through which M initiates a protected process transition when the interval timer matures.
- **C12 / C(C)** — System Capability Table.
- **C13 / C(N)** — Normal Interrupt Block.
- **C14** — unassigned in the Pocket Reference.
- **C15** — unassigned in the Pocket Reference.
- **C16** — unassigned in the Pocket Reference.
- **C17** — unassigned in the Pocket Reference.

The processor also contains **C(S)** — the Fault Start-Up Block capability. C(S) lies outside the address space of the C registers and its value is hardwired into the processor.

### 6.5 Primary Indicator Register — MIP

The **Primary Indicator Register (MIP)** contains:

- **MIP00** — =0.
- **MIP01** — <0.
- **MIP02** — Overflow.
- **MIP03** — Spare.
- **MIP04** — Second Group.
- **MIP05** — Inhibit Interface Faults.
- **MIP06** — Odd Data Parity.
- **MIP07** — First Attempt.
- **MIP08** — Inhibit Interrupts.
- **MIP09–MIP23** — unassigned in the Pocket Reference.

### 6.6 Fault Indicator Register — MIF

The **Fault Indicator Register (MIF)** contains:

- **MIF00** — Bus Corrupt.
- **MIF01** — unassigned.
- **MIF02** — Interrupt Timeout.
- **MIF03** — unassigned.
- **MIF04** — Spare.
- **MIF05** — Slave Timeout.
- **MIF06** — Capability Parity Fault.
- **MIF07** — Sumcheck Fault.
- **MIF08** — Base/Limit Fault.
- **MIF09** — Interface Timeout.
- **MIF10** — Parity Comparison.
- **MIF11** — Read Data Parity.
- **MIF12** — Invalid Operation.
- **MIF13** — Power Failure.
- **MIF14** — Invalid Control Code.
- **MIF15** — Trap with MIP08.
- **MIF16** — Hardware Fault 1.
- **MIF17** — Watchdog Timer Expired.
- **MIF18** — Access Violation.
- **MIF19** — Hardware Fault 2.
- **MIF20–MIF23** — capability register on which failure occurred, from least-significant bit at MIF20 to most-significant bit at MIF23.

### 6.7 Secondary Indicator Register — MIS

The processor also contains the **Secondary Indicator Register (MIS)**.

Its bits describe internal microprogram execution state rather than the architectural state from which H and T are constructed. Its detailed contents are therefore left at the microprogram level.

### 6.8 Internal Mode

**Internal Mode** makes processor-internal state addressable through the normal addressing mechanism. The addressed state includes the D and C registers, Historic Registers, MIP, MIF, MIS and C(S).

Access to this state is capability controlled.

### 6.9 Historic Registers

The processor contains **sixteen 24-bit Historic Registers** retaining recent execution information. They are accessible through Internal Mode and provide a recent history of processor execution for fault investigation.

### 6.10 C(S) Start-Up Location Control

**C(S) Start-Up Location Control** is the twelve-bit field **C(S)[23:12]** that can be altered through Internal Mode. The remainder of C(S) can be read but not altered through Internal Mode. These twelve bits also participate in the fault/start-up mechanism.

Their precise architectural purpose is not yet established.

### 6.11 Interval timer

The processor contains an **interval timer**. When the interval timer matures, M initiates a protected process transition through the Enter Capability held in C(I).

### 6.12 Watchdog timer

**D11** is the **watchdog timer**. Expiry of the watchdog timer is recorded by **MIF17 — Watchdog Timer Expired**.

### 6.13 Change Process

**CHP (Change Process)** is an instruction of **M** that causes the state of one process to be dumped and the state of another to be restored through their Dump Stacks.

M also provides an **internal CHP** mechanism by which M can initiate a process change directly, without execution of the CHP instruction.

### 6.14 One-Shot Second Group LC

**One-Shot Second Group LC** permits the special capability registers **C(C)**, **C(I)** and **C(N)** to be established during start-up/recovery. It is associated with **MIP04 — Second Group**.

Its one-shot character raises an important architectural question: whether, once established, those registers constitute the only surviving authority by which M can enter the protected software that extends it.

### 6.15 Unplaced reconstruction breadcrumbs

The following documented structures or mechanisms are retained here as **placeholders** so that the architectural descent and machine inventory can be cross-checked in both directions. Their proper architectural placement or exact relationship to M has not yet been established.

- **LOKK** — documented in ROS/PDOS Dump Stack layouts; exact role unresolved.
- **SIP — State and Internal Priority Word** — documented in ROS/PDOS process state.
- **Error Control** — documented Process Error Control Parameter from the Process Template.
- **Ptarmigan words** — three words documented in the PDOS Dump Stack layout.
- **RSPC-0** — reserved segment pointer used in the fault/start-up path to identify the checkout-process Dump Stack.
- **Special Fault Block** — stored structure used by the fault/start-up machinery.
- **Normal interrupt acceptance** — the M-level mechanism that decides that a normal interrupt is accepted before entry through **C(N)**. **D15 — Interrupt Accept Register** is documented processor state associated with this area; the complete acceptance mechanism and its architectural expression remain to be reconstructed.

### 6.16 Microprogrammed execution

Processor operations are executed under microprogram control.

The microprogram operates the store interface, processor registers, indicators and the other internal processor mechanisms described above. Its internal execution state, including the detailed state represented by MIS, lies below the architectural level considered here.

This is the bottom of the architectural descent.

The following sections now reverse direction. Starting with this substrate and M⟨H,T⟩, we can reconstruct PP250-G2 from the bottom upward and show how the protected architecture established on the descent emerges.

The first structure to reconstruct is a **process**. Its persistent computational state is represented by the **Dump Stack**.


### Reconstruction roadmap

The reconstruction from this point follows the route by which the architecture becomes intelligible to a programmer, rather than mechanically assembling the inventory item by item. The sequence below is a working guide for the sections that follow.

- **B1 — Begin with the Enter Capability.** This is the unusual programmer-visible mechanism that first demands explanation: what does it mean to enter a protected software structure?

  On the way down we found that an object could make some of its functions available to the outside world while keeping everything else inside it inaccessible.

  To use those functions, another object needed the right unforgeable token. System 250 had a name for this token: an **Enter Capability**.

  By convention, the functions made available by an object are arranged at numbered offsets. Possession of an Enter Capability for the object allows its holder to call any of those functions by specifying the appropriate offset.

  **That is all the Enter Capability allows.** Any attempt to use it for anything other than a legitimate entry to one of those functions is detected by the hardware and causes a fault.

- **B2 — Arguments are passed to the called function in the data and capability registers, and results are returned in the same way.**

- **B3 — Any attempt to do anything else other than call a legitimate function is detected by hardware and generates a fault.**

  To understand what happens when a fault occurs, we first need to introduce the System 250 concept of a process.

  A System 250 process is the process already encountered in the descent: an executing computation together with its protected environment and processor state.

  A process executes with its protected environment in **C6** and its code in **C7**. Both form part of its execution context and are preserved across CALL and RETURN through the **Dump Stack**.

  The Dump Stack holds the processor state needed to preserve and resume the process.
- **Follow entry into the protected structure.** C6 establishes the capability environment and C7 the executable code; CALL and RETURN expose the relationship between controlled entry, execution and protection.
- **Ask where capabilities come from.** If every protected structure depends upon capabilities, the next question is how authority is created and protected. This leads towards capability construction, the SCT, C(C) and storage management.
- **Follow execution into the process mechanism.** CHP and the Process Dump Stack reveal that a process is not merely a software abstraction: M can preserve one protected execution and establish another.
- **Ask what protects the machinery underneath.** The powerful access associated with process state and the special processor mechanisms forces the reconstruction below the ordinary programmer-visible capability architecture.
- **Discover the absence of a conventional privileged layer.** C(D), C(I), C(C), C(N), C(S), Internal Mode, CHP and related microprogram mechanisms do not reveal a supervisor that escapes the protection architecture. They reveal protected mechanisms by which M establishes, enters and supports the software structures above it.
- **Arrive at M⟨H,T⟩.** The distinction between M, capability-structured computation H, and conventional computation T emerges as an explanation of the architecture discovered along this route, rather than as a taxonomy imposed in advance.
- **Return to the opening proposition.** The reconstruction can then show why describing System 250 merely as an early capability machine misses the deeper architecture: capability is the organising protection principle down to the boundary with the microprogrammed machine.

These are **reconstruction breadcrumbs**, not settled section headings. They are retained here to guide the order of investigation and writing; detailed ROS/PDOS policy, such as scheduling algorithms, is pursued only where it is needed to establish this architectural path.

## 7. Realisation in PP250-G2

PP250-G2 realises the unforgeable tokens of the architectural model as **capabilities**.

### 7.1 The Dump Stack

A **Dump Stack** is a segment managed by **M**. Crucially, it is not part of either **H** or **T**; it belongs to the encompassing microprogram computation.

The Dump Stack holds the processor state of a process when that process is not executing. It contains C0–C5, D0–D7, the Pushdown Pointer, Watchdog Timer and MIP, together with the C6/C7/IAR execution state and saved CALL contexts.

Thus the state from which both H and T computation can subsequently resume is preserved outside both of them, under M.

### 7.2 Loading and dumping a process

**M** transfers process state between a Dump Stack and the processor registers.

**CHP (Change Process)** is an instruction of **M**. It causes the current process state to be dumped and another process state to be restored from its Dump Stack.

Process change therefore takes place entirely within M. Neither H nor T performs the transfer; both cease in one process and are re-established from the state of the process entered.

### 7.3 Capability access

A capability identifies a segment and specifies what operations may be performed upon that segment. PP250-G2 distinguishes access to **data** from access to **capabilities stored within the segment**.

**RD — Read Data** permits ordinary data to be read from the segment.  
**WD — Write Data** permits ordinary data to be written into the segment.  
**ED — Execute Data** permits instructions held in the segment to be executed.

**RC — Read Capability** permits a capability stored in the segment to be loaded into a capability register.  
**WC — Write Capability** permits a capability to be stored into the segment.  
**EC — Enter Capability** permits the segment to be entered as a protected capability block rather than exposing its contained capabilities to the caller.

These distinctions are fundamental. **RD does not provide RC**: being able to read the data in a segment does not allow a program to obtain capabilities stored there. Similarly, the ability to invoke a protected object through **EC** does not expose the capabilities from which that object is constructed.

**M enforces these access rights in hardware whenever a capability is used.** This is the primitive hardware protection mechanism from which the protected spaces described on the architectural descent are constructed.

### 7.4 Interval timer and M-initiated process transition

The **interval timer is a mechanism of M**. It runs independently of computation in H or T.

The special capability register **C(I)** contains an Enter Capability available to M. When the interval timer matures, M uses that capability to initiate a protected process transition.

The process entered through C(I) then executes normally in H and T. M does not execute the handler as a separate kind of software computation.

For the moment, assume that the appropriate Enter Capability has already been installed in C(I). How it is established will be explained later.

The capability held in C(I) is protected in exactly the same way as every other capability in the system. M has no separate protection mechanism for this entry.

### 7.5 System Capability Table

The SCT is reached through the special capability C(C).

A PP250-G2 SCT entry is a three-word descriptor family containing the information required to validate and expand an active capability, including:

- SUMCHECK;
- BASE;
- LIMIT;
- Generation-2 object-management state.

The access authority exercised by a program originates in the capability being loaded; the SCT does not independently grant arbitrary rights to the holder.

PP250-G2 evidence also establishes SCT state used by the garbage-collection and allocation machinery, including **GARBAGE** and **VISITED**.

Changing an SCT descriptor does not by itself rewrite capability registers that have already been expanded. Where such state must be refreshed, the architecture can use protected process interruption/restoration so that saved compact identities are resolved again through the current SCT.

### 7.6 LDP

LDP exposes the compact pointer associated with a capability as ordinary data.

For example:

```text
LDP D2 C3
```

in direct form loads D2 with the compact capability pointer associated with C3.

This implies that the processor retains sufficient association between an expanded capability and its compact protected identity for that pointer to be recovered. No later pointer-register architecture is required to explain the PP250-G2 instruction.

The historical software uses of LDP are not required to define its architectural operation.

### 7.7 Enter Capability, CALL and RETURN

Protected invocation is built into the capability architecture.

A caller may possess an Enter Capability to another node's principal capability block. A CALL through that capability, with an offset selecting an executable entry, establishes the called execution domain.

Conceptually:

```text
Enter Capability
       |
       +---- offset ----> Execute capability
```

The processor then establishes:

```text
C6 = called node's principal capability block
C7 = selected executable code capability
IAR = called entry point
```

and preserves the caller's:

```text
C6
C7
return IAR
```

on the Process Dump Stack.

C0–C5 are not replaced by this transition. They can therefore carry data authority and parameters across the protected interface. Data registers likewise remain ordinary call-visible state.

RETURN restores the saved C6, C7 and IAR.

CALL is consequently a protected call, not a complete process-context replacement.

### 7.8 Process Dump Stack

Each active process has a Process Dump Stack identified by C(D).

The 1976 format has a common fixed process-state area containing:

```text
0–5       C0–C5
6–15      D0–D7
16        pushdown pointer for CALL stack
17        watchdog timer
20        MIP
```

Beyond that fixed state, the format contains system-dependent process information and the C6/C7/IAR execution frames used by protected calls.

The same protected structure therefore supports two related requirements:

1. preservation/restoration of process architectural state;
2. the nested C6/C7/IAR stack required by CALL and RETURN.

The operating systems differ in their additional Dump Stack fields and initial layouts; those differences are not part of the processor definition.

### 7.9 Change Process

**CHP (Change Process)** is distinct from CALL.

CALL changes protected execution domain while remaining within the same process and Process Dump Stack.

CHP performs a process transition: the outgoing process state is preserved and an incoming process state is established from its protected process-state structure.

The architectural distinction is:

```text
CALL / RETURN
    same process
    same Process Dump Stack
    push/pop C6, C7, IAR

CHP
    process transition
    preserve outgoing process state
    establish incoming process state
    change active Dump Stack
```

PP250-G2 provides both Store and Direct forms of CHP. The processor architecture does not require us to assign those forms to a particular operating-system process-creation policy in order to explain process switching.

### 7.10 Special processor state

PP250-G2 defines a second group of special-purpose processor registers.

The documented special capability registers are:

```text
C10   C(D)   Process Dump Stack
C11   C(I)   Interval Timer / system interrupt structure
C12   C(C)   System Capability Table
C13   C(N)   Normal Interrupt Block
```

C(S), the Fault Start-Up capability, is a separate special capability.

Documented special data registers include:

```text
D10   absolute Dump Stack pushdown pointer
D11   watchdog timer
D12   first-fault MIF copy
D15   interrupt accept register
D17   instruction address register
```

The Primary, Secondary and Fault Indicator registers contain processor control and fault state.

These structures allow protected system mechanisms to operate without introducing a conventional unrestricted supervisor address space.

### 7.11 Normal events and interrupts

Normal system interrupts are capability-mediated.

C(I) provides access to the system interrupt information and C(N) identifies the Normal Interrupt Block. The processor periodically examines the interrupt state, selects an eligible request and enters the corresponding protected system handling path.

Normal interrupt handling can cause a process transition using the same protected process-state machinery used elsewhere by the architecture.

The important architectural point is that an interrupt does not simply install an arbitrary privileged program counter. The destination and its authority are represented by protected system structures.

### 7.12 Fault and start-up

Fault handling is distinct from normal interrupt handling.

C(S) identifies the Fault Start-Up Block and provides the protected root for fault/start-up execution. The startup/fault root is established by architectural hard-wired or preset processor state rather than being authority that ordinary software must manufacture. Contemporary descriptions show this mechanism being used to enter restricted checkout/recovery code following detected processor or capability failures.

Thus PP250-G2 has two deliberately different exceptional roots:

```text
C(N)   normal interrupt/system dispatch
C(S)   fault/start-up/recovery
```

They may ultimately use common process-state machinery, but they are not the same entry mechanism. Exact generation-specific cold-load and microinstruction sequencing is an implementation/documentary matter rather than an unresolved authority mechanism.

### 7.13 Persistent storage, virtual store, Inform and Outform

Virtual storage is integrated with the capability/object architecture rather than being a separate conventional virtual-address translation layer.

An active or **Inform** capability identifies an object through the System Capability Table. A passive or **Outform** representation carries the persistent backing-store identity needed while capability-containing material is represented outside primary store.

Conceptually:

```text
Inform
active protected reference
SCT identity
       |
       | virtual-store management
       |
       v
Outform
persistent backing-store identity
```

When capability-containing blocks move between primary and secondary storage, their contained protected references can be converted between the appropriate representations by the virtual-storage machinery.

Nonresident access and materialisation are handled through the established trap/storage-management mechanism. PP250-G2 does not require a later SCT PRESENCE mechanism to explain this behaviour.

### 7.14 Resource creation and allocation

Ordinary software does not need the ability to fabricate capabilities.

System resource-allocation services are themselves reached through capability-protected interfaces. Contemporary System 250 material describes a Common Facilities Block exposing services such as store, process, flag, stream, text-file, directory and job allocation.

An allocator creates the appropriate resource and returns a capability giving the caller the permitted authority over it.

Thus new authority enters an ordinary process through an already-authorised protected operation rather than by constructing an arbitrary capability bit pattern.

### 7.15 Resource lifetime and garbage collection

The capability system forms a graph of protected references.

PP250-G2 includes background garbage-collection machinery capable of traversing capability-containing blocks and identifying reachable SCT objects. GARBAGE and VISITED state in the SCT supports this process.

At the architectural level:

```text
roots
  |
  v
capability-containing blocks
  |
  v
contained capability references
  |
  v
referenced SCT objects
```

Objects not reachable under the collection rules can eventually become eligible for reclamation. Explicit release is a distinct operation and can invalidate existing references to the released resource.

This mechanism depends on the architectural distinction between capability-containing storage and ordinary data; the collector does not need to guess which arbitrary data words might be capabilities.

### 7.16 Protected I/O and multiprocessor operation

System 250 is a symmetric multiprocessor architecture. CPUs share system work rather than having permanently assigned operating-system roles.

Store modules and device-access structures are connected through the system bus architecture. Store Access Units arbitrate concurrent access and participate in integrity checking.

I/O is deliberately integrated into the normal protected addressing model. Device registers can appear as addressed resources, while CPU processes perform polling and block transfers. System interrupts communicate system events without binding a device permanently to a particular CPU.

This model allows processors, stores and peripheral modules to be added or removed within a capability-constrained system structure.

## 8. The processor mechanisms

*The existing PP250-G2 material above already contains processor-level detail. It will be reorganised under this heading as the implementation ascent is developed.*

## 9. M⟨H,T⟩

### 9.1 T — ordinary computation

*To be developed.*

### 9.2 H — protected computation

*To be developed.*

### 9.3 M — mechanisms acting upon H and T

*To be developed.*

### 9.4 Process transition as the conjunction of H and T

*To be developed.*

### 9.5 Why M is not a third peer machine

*To be developed.*

## 10. Architectural invariants

The recovered PP250-G2 architecture is characterised by the following invariants:

1. **Ordinary store access is capability-relative.** A program does not generate an unrestricted physical address.
2. **Authority accompanies the protected reference.** The SCT supplies object representation, not arbitrary authority.
3. **Stored capability identity is distinct from expanded physical addressing state.**
4. **C6 and C7 define the current protected execution context.**
5. **CALL changes domain, not process.**
6. **CHP changes process.**
7. **The Process Dump Stack is protected architectural state, not an ordinary language stack.**
8. **Normal interrupt and fault/start-up entry are capability-rooted.**
9. **Resource creation returns capabilities through already-authorised services; ordinary programs need not fabricate them.**
10. **Virtual storage preserves protected object identity across physical movement.**
11. **Capability-containing storage is distinguishable from ordinary data, enabling capability-aware lifecycle management.**
12. **No conventional unrestricted supervisor mode is required to explain normal system operation.**

Together these properties explain the surviving programmer-visible, operating-system and protection behaviour without importing later architectural mechanisms.

## 11. Deliberately unspecified details

The following details are not required to make the PP250-G2 architecture internally coherent and are therefore not invented here:

- a universal interpretation of every non-right bit in the COS and POS access diagrams;
- functions for undocumented C14–C17 and blank special data-register entries;
- exact microinstruction sequencing of the architectural operations;
- operating-system-specific process-construction policy;
- exact cold-load/commissioning details beneath the documented hard-wired/preset startup root;
- bit-for-bit correspondence with earlier or later System 250 generations.

These are implementation, representation, system-policy or historical-evolution questions unless further evidence shows that one of them changes PP250-G2 architectural behaviour.

## 12. Reconstruction conclusion

On the current repository evidence, PP250-G2 forms an internally consistent architecture.

The processor has documented protected roots for ordinary capability resolution, process state, normal interrupt handling and fault/start-up. Stored capability identity, SCT-mediated expansion, C6/C7 protected invocation, Process Dump Stack state, CHP process transitions, virtual storage, resource allocation and capability-aware object lifetime fit together without requiring an additional undocumented privilege mechanism.

There are currently **no identified unresolved architectural blockers** meeting the repository's open-question admission rule. Further source work may refine encodings, microsequences and operating-system policy without changing that conclusion.
