# PP250 Beyond the “Capability Machine”

**Status:** Working reconstruction and publication hypothesis  
**Date:** 2026-09-24

## Thesis

The conventional description of the Plessey System 250 as a **capability machine** is correct but may be seriously incomplete. It risks identifying the most visible protection mechanism — capability — with the architecture as a whole.

The current PP250-Reboot reconstruction suggests a deeper structure:

```text
M<H,T>
```

where:

- **T** is the Turing/general computational machine;
- **H** is the Church/authority machine;
- **M** is the governing relation/machine that admits and performs legitimate transitions involving H and T;
- **capability** is an orthogonal protected representation/mechanism of authority that can be incorporated wherever protected authority is required, rather than being synonymous with H.

The resulting research hypothesis is:

> **System 250 may have been historically classified by one of its mechanisms — capability — while its more distinctive architectural contribution was the separation of computation, authority and sovereign state transition, with capability used across that structure.**

This is a working reconstruction, not yet a claim that the historical literature has overlooked the point. Establishing whether the interpretation is genuinely absent from the literature requires a dedicated novelty review.

---

## 1. How the conventional classification can hide the architecture

Once System 250 is labelled a “capability machine”, later analysis naturally asks familiar capability-machine questions:

- How are capabilities represented?
- How do capabilities protect memory?
- What bounds and access rights do they contain?
- How are capabilities passed and stored?
- How are protection domains constructed?

Those are legitimate questions, but they encourage an implicit equation:

```text
capability mechanism = capability architecture = System 250
```

The current reconstruction rejects that equation.

A capability is a protected representation of authority. It is not itself the Church machine H, and it is not the definition of the complete System 250 architecture.

This distinction matters because capability-bearing state appears relevant in more than one architectural role. H is extensively capability-based, but the special start-up capability C(S) is a candidate piece of M-state. If that reconstruction is correct, then capability is demonstrably orthogonal to the M/H/T decomposition.

---

## 2. The overlooked object may be M<H,T>, not capability

The current reconstruction is:

```text
                     M
              governing machine
                 /       \
                /         \
               H           T
          authority     computation
```

Capability cuts across the diagram rather than defining H:

```text
             protected authority
                 capability
                 /       \
                /         \
               M           H
          e.g. C(S)?    ordinary authority
                        machinery/state
```

This produces a different interpretation of the processor.

T does not become sovereign. It performs ordinary computation within an authority environment.

H represents and manipulates authority.

M governs the legitimate evolution of both machines and can perform transitions that ordinary computation is never empowered to perform.

Thus the deepest PP250 distinction may not be “data versus capability”. It may be:

```text
computation versus authority versus governance
```

with capability providing protected authority representation where required.

---

## 3. Apparently unrelated PP250 peculiarities become one mechanism

The value of the reconstruction is that features normally described separately begin to look like manifestations of the same architecture.

### CALL and RETURN

Rather than merely unusual subroutine instructions, protected CALL/RETURN can be interpreted as M-mediated transitions in which both computational context and authority context may change.

### CHP

Change Process is not merely an unusually powerful context-switch instruction. It is naturally interpreted as an M transition over a larger portion of H/T state.

### Fault handling

Fault recovery need not make T privileged. A fault can cause M to replace an invalid or failed H/T state with a legitimate recovery state.

### Power-up

The first legitimate capability state does not need to be fabricated by an already-running authority machine. Power-up is itself an operation of M, which may establish initial protected state as part of the machine definition.

### C(S)

C(S) is diagrammatically separated from the ordinary internal and external C-register sets and is associated with fault/start-up behaviour. Its being a capability does not require it to belong to H. It is therefore a concrete candidate for capability-bearing M-state.

### Absence of conventional supervisor mode

The absence of an unrestricted privileged software mode ceases to look like an omission. If M remains sovereign, T need never acquire unrestricted machine authority in order to perform system transitions.

These observations are important because the M<H,T> reconstruction was not invented separately to explain each one. A single model is beginning to make several formerly awkward or disconnected features coherent.

### Microprogram state: a possible concrete boundary between M and resumable H/T state

The Pocket Reference's Internal Mode material and Dump Stack layouts now provide a more concrete clue about where M may exist in the implementation.

MIP, the Primary Indicator Register, is saved in the common fixed Process Dump Stack state. ROS and PDOS additionally save MIF, the Fault Indicator Register. MIS, the Secondary Indicator Register, is exposed through Internal Mode but is not shown in the documented Process Dump Stack layouts. This does not prove a formal M/H/T boundary, but it is consistent with a processor in which some control state belongs to the resumable execution context while other state exists only as transient machinery governing the next internal transition.

The named MIS bits are particularly suggestive. `MIS08 Set Read Capability` and `MIS19 Cap. Pointer in OPP` indicate that the microprogram retains semantic information about capability-related transfers while values move through internal processor paths. Combined with the processor self-test description of slot-displaced microprogram control, this suggests that at least part of M may be implemented not as a separate high-level subsystem but as a small amount of microprogram state and gating carried from one slot into the next.

That matters to the M<H,T> reconstruction. M need not be large, nor need it understand an operating system's abstract objects. It may enforce primitive distinctions — capability versus data transfer, permitted access operation, legitimate state transition — while H and T provide the architectural state on which those primitives operate. Higher-level operating-system meanings can remain outside M.

This is a **working reconstruction**, not an identification of MIS with M. M is an architectural/theoretical concept; MIS is a documented processor register. The useful observation is narrower: the implementation exposes transient semantic control state of exactly the sort a small M-level mechanism would require.

### Architectural instructions and the lower-level microprogram machine

**Documented observation:** Pocket Reference CPU-display controls distinguish `SINGLE SLOT` from `SINGLE INSTRUCTION`, permit stopping after or on a selected slot, and permit inhibition of microprogram decode. Page 8 separately names `Microprogram O/F` and `Inhibit Slot Decode`. The processor self-test paper independently describes execution and simulation at microprogram level, including slot-to-slot conditional timing, and distinguishes tracing after every slot from tracing after every instruction.

**Architectural inference:** the PP250 architectural instruction set is implemented by a lower-level, explicitly observable microprogrammed execution machine organised into slots. This gives concrete implementation evidence for machinery below the H/T architectural state and identifies the microprogram as an important place to investigate how M-level governing transitions are realised.

This does **not** identify M with the microprogram. M is the architectural/theoretical governing relation over legitimate H/T transitions; the microprogram is an implementation mechanism and also implements ordinary instruction-level work. The justified claim is therefore narrower: **some of the mechanisms reconstructed as M may be realised in microprogram state, sequencing and gating, and the surviving engineering material provides a route for reconstructing that implementation.**

### Normal interrupt entry: a callback from M into software

The reconstructed PP250 normal-interrupt path adds an important refinement:

```text
protected exceptional condition
        |
        v
      C(N)
        |
        v
Normal Interrupt Block
        |
        v
incoming Dump Stack capability
        |
        v
automatic CHP
        |
        v
Normal Interrupt process
        |
        v
software policy / dispatch
```

In the M/H/T abstraction, **C(N) has the structural character of a capability-protected callback from M into T executing under an H-defined authority environment**.

M determines **that intervention is required** and performs the legitimate transition. It does not need to contain the higher-level policy for resolving the condition. The process entered through C(N) executes ordinary computation under capability-defined authority and determines **what policy is to be applied** — for example storage management, I/O handling or scheduling.

Thus:

```text
             T executing under H₁
                    |
                    | intervention required
                    v
                    M
          protected transition
                    |
                    | callback through C(N)
                    v
             Tn under Hn
       Normal Interrupt process
                    |
                    | software policy
                    v
                    M
          subsequent legitimate transition
                    |
                    v
             T continues
```

The word **callback** is an abstraction, not historical PP250 terminology. It describes the structural relationship: M invokes a software-defined continuation point when machine-level intervention is required.

This sharpens the meaning of M. **M is not “all operating-system code”, nor need it be imagined as a third conventional instruction processor alongside H and T.** It is better understood as the governing relation/mechanism over legitimate state transitions. Software invoked through an M callback executes on T and under H; its system role does not make the software itself M.

Schematically:

```text
state S₁ = <H₁,T₁>
        |
        | M admits/performs transition
        v
state S₂ = <H₂,T₂>
```

The C(N) mechanism also shows how M can remain small while system policy remains extensible. M needs sufficient machinery to recognise a protected condition, identify the configured normal-interrupt authority, preserve/restore state and perform the legitimate process transition. It can then delegate storage, I/O, scheduling and other policy decisions to capability-constrained software.

This is stronger than the conventional statement that an operating system installs an interrupt handler. The callback target is itself authority-bearing: C(N) designates the Normal Interrupt Block, which leads to the Dump Stack defining the process to be entered. The destination is therefore reached through protected authority structures rather than through an arbitrary raw instruction address.

### C(S) bootstraps the configurable C(N) callback

The reconstructed startup chain is:

```text
C(S)
  |
  v
fault/startup + checkout
  |
  v
initial legitimate process
  |
  v
MIP/SPECIAL
  |
  v
LC establishes C(N)
  |
  v
normal interrupt callback available
```

In the M/H/T abstraction, C(S) and C(N) expose two different stages of governance.

**C(S)** belongs to the root/recovery machinery by which M can establish a legitimate H/T state when no ordinary running H/T context can be relied upon.

**C(N)** is the configurable operational callback. Once legitimate execution exists, software arising from that state can establish C(N); thereafter M can use C(N) to request software policy during normal operation.

The concise relationship is:

> **C(S) establishes the first trusted transition; software arising from that transition establishes C(N); C(N) thereafter supplies M's normal callback path into T executing under capability-controlled H.**

This provides a bootstrap for a configurable M/software boundary without introducing an unexplained second source of sovereign authority or a conventional supervisor mode. The callback is configurable, but the authority to configure it traces back through the startup chain to the machine's root transition mechanism.

It also reinforces the orthogonality of capability to H/T/M. Capabilities occur as ordinary H authority, as candidate M-state in C(S), as the protected callback designation C(N), and as Dump Stack capabilities defining process-transition targets.


### H as binding as well as authority: a lambda-calculus/closure correspondence

The software-evolution investigation adds a potentially important refinement to the meaning of H.

Until now H has primarily been described as the **authority machine**: the protected capability environment within which T executes. The resource and process model suggests that this is incomplete. H also supplies the **bindings** through which a computation acquires its particular objects, services, code and other resources.

Abstractly, the same reentrant computation may execute with different environments:

```
H_A = {
    x -> X_A,
    y -> Y_A,
    ...
}

H_B = {
    x -> X_B,
    y -> Y_B,
    ...
}
```

The T-level code need not contain an absolute reference to either `X_A` or `X_B`; the relationship is supplied through the capability environment.

This has a genuine structural resemblance to environment-based accounts of lambda-calculus evaluation, in which the meaning of a variable is supplied by a binding environment. It also resembles a closure, conventionally understood as callable code together with an environment.

A PP250 protected callable object has a suggestive shape:

```
Enter capability
      |
      v
capability block
      |
      +--> executable operations
      |
      +--> protected state/resources
```

The important additional PP250 property is **authority**. A conventional environment binding can be written conceptually as:

```
x -> A
```

whereas a PP250 capability binding is closer to:

```
x -> (A, permitted authority)
```

The binding not only identifies an object; it carries machine-enforced authority over that object. H may therefore be better understood, provisionally, as an **authority-and-binding environment/graph**, rather than merely a protection environment.

This observation does **not** establish that PP250 is a lambda-calculus machine, nor that its designers implemented a particular formal lambda-calculus evaluator. The comparison is structural and should be tested against formal environment and closure models before any stronger claim is made.

#### ENTER as a protected environment transition

This refinement also sharpens the interpretation of protected CALL/ENTER.

A cross-domain call does not merely transfer control to another instruction address. It establishes the called capability environment and called code context while preserving the previous context for return.

At the M/H/T level this can be represented:

```
<H1,T1>
    |
   ENTER
    |
    v
<H2,T2>
```

This is therefore naturally interpretable as a transition between **protected computational environments**.

The closure analogy is again useful but deliberately limited: callable code is associated with an environment, while M enforces the legitimate transition into that environment. Unlike an ordinary language-level closure, possession of the ability to invoke the protected object does not imply arbitrary inspection, fabrication or modification of its environment.

A concise working formulation is:

> **H supplies protected bindings carrying authority; T computes within those bindings; M enforces legitimate transitions between H/T environments.**

#### Consequence for coexistence and software evolution

This interpretation arose while considering online software evolution.

If H contributes the bindings that give a process instance its effective world, then a new software generation need not necessarily transform an existing `H_A<T_A>` into `H_B<T_B>`. Distinct environments may coexist:

```
H_A<T_A1>  -----> terminates
H_A<T_A2>  ----------> terminates

             changeover for new instances

H_B<T_B1>  ---------------->
H_B<T_B2>  -------------------->
```

Old instances can retain their old bindings and representations while new instances are born with new bindings. This provides a theoretical explanation for how generational/draining software replacement could fit naturally into M/H/T, but **there is not yet primary evidence that ROS actually used this update mechanism**.

The stronger and more general M/H/T insight does not depend on that historical hypothesis:

> **H is not only the answer to “what may this computation access?” It also contributes the answer to “which objects and services constitute this computation's world?”**

That distinction moves H beyond a simple memory-protection interpretation and provides a more precise theoretical reason for retaining the Church side of the M/H/T model.


---

## 4. Why later capability systems can encourage the misreading

Modern discussion can make the historical classification still more misleading. A later architecture may employ sophisticated capabilities for memory safety, provenance, bounds, permissions, compartmentalisation or protected invocation and therefore naturally be compared with PP250 as another member of the broad family of “capability machines”.

That comparison can be too shallow.

The relevant research question is not simply:

> How does a PP250 capability compare with a modern capability?

It is:

> **What architectural structure is the capability mechanism serving?**

For PP250 the candidate answer is the complete `M<H,T>` structure.

Therefore comparison with CHERI or any other modern capability architecture should be performed along several independent dimensions:

1. **Capability representation:** what authority does a capability represent and how is it protected?
2. **Computation:** what constitutes ordinary computational state and execution?
3. **Authority:** what machinery represents, transfers and changes authority?
4. **Sovereignty:** what mechanism is ultimately allowed to change authority-bearing and computational state?
5. **Privilege:** does ordinary computation ever enter an unrestricted privileged state?
6. **Boot/fault:** what establishes protected state when no ordinary execution context can legitimately manufacture it?

The outcome of that comparison must not be prejudged. A modern architecture may contain an equivalent decomposition under different terminology. The point is that “both have capabilities” is no longer an adequate architectural comparison.

---

## 5. Why this could have been historically overlooked

There is a plausible historiographic mechanism.

The original System 250 literature naturally emphasized capabilities, protection, naming, protected procedure invocation, processes and operating-system construction. Later histories therefore had an obvious category available: **capability computer**.

Once that classification became conventional, individual mechanisms could be interpreted as special features within a capability machine:

```text
CHP              -> unusual process-switch instruction
C(S)             -> unusual special capability register
fault recovery   -> reliability mechanism
CALL/RETURN      -> protected procedure mechanism
no supervisor    -> consequence of capability protection
cold start       -> implementation detail
```

The reconstruction now being developed suggests the inverse interpretation:

```text
CALL/RETURN  \
CHP           \
fault          > manifestations of M governing H and T
power-up      /
C(S)         /

no supervisor -> T never needs to become sovereign
```

If this survives detailed reconstruction, the conventional capability-machine description has not been false; it has selected the wrong level of abstraction as the defining feature.

---

## 6. Relationship to the “six blind men” reconstruction method

No surviving document is expected necessarily to describe this complete structure. Different authors documented capability formats, instructions, operating systems, process changes, recovery, microcode, start-up and protection from different viewpoints.

The research task is therefore not to find a sentence saying:

> “System 250 is M<H,T> and capability is orthogonal to those machines.”

Instead the claim must be evaluated through the repository's reconstruction method:

```text
Observation -> Constraint -> Reconstruction -> Prediction -> Falsification
```

The candidate “elephant” is `M<H,T>` with capability orthogonal to the decomposition.

Its strength depends on whether it:

- explains all credible observations;
- contradicts none of the reliable evidence;
- eliminates otherwise unexplained special mechanisms;
- predicts properties not used to construct the model;
- requires fewer unsupported mechanisms than competing reconstructions.

---

## 7. Predictions and tests

The reconstruction produces useful tests.

### Prediction: C(S) is not ordinary H-state

If C(S) is capability-bearing M-state, ordinary H/T operations should not treat it simply as another normal programmable capability register, while fault/start-up machinery should be able to establish or use it through M.

### Prediction: fault and power-up expose M most clearly

Documentation describing fault and start-up should contain transitions that cannot naturally be expressed as ordinary privileged T execution or ordinary H capability operations alone.

### Prediction: CHP is architectural rather than merely OS convention

The process transition should operate on protected state at a level below the particular operating-system meaning subsequently assigned to the restored process.

### Prediction: normal software intervention decomposes into M/H/T roles

If the refined model is sound, other mechanisms that appear to require privileged software should often decompose into an M-level protected transition, an H-defined authority environment/target, ordinary T computation implementing policy, and a subsequent M-mediated transition. I/O completion, scheduling, timer handling, process creation and protected CALL/RETURN provide candidate tests.

### Prediction: capability semantics recur across machine roles

The protected authority representation used by C(S), C(N), Dump Stack targets and ordinary H capability state should share enough machinery to justify calling them capabilities, while their architectural ownership and purposes differ.

### Falsification

The reconstruction would be weakened if C(S) proves to be ordinary H state, if all apparently M-level transitions reduce cleanly to conventional privileged T execution, if normal system intervention requires unrestricted T to rewrite H/M state, or if M adds no explanatory or predictive power beyond established descriptions of the processor.

---

## 8. Publication and PhD significance

This is potential **paper and PhD material** under the repository publication criteria.

The candidate paper contribution is not that PP250 used capabilities. That is well established.

The candidate contribution is:

> **The conventional classification of System 250 as a capability machine may obscure a deeper architecture in which capability is an orthogonal authority mechanism used within a separation of computation T, authority H and governing transition M.**

Why this may matter academically:

- it offers a unified interpretation of several PP250 mechanisms normally treated separately;
- it changes the appropriate unit of comparison between PP250 and later capability architectures;
- it may explain the architectural significance of the absence of conventional supervisor privilege;
- it connects bootstrap, fault recovery, normal interrupt callbacks and protected invocation to the same authority model;
- it suggests that historical work may have concentrated on the capability representation while overlooking the machine that governs capability-bearing state;
- if generalizable, it may identify a useful architectural abstraction beyond PP250 itself.

Before claiming novelty, perform a literature review across historical PP250 analysis, capability machines, protection machines, reference monitors, microcoded control architectures, security automata, CAP, Hydra, PSOS, KeyKOS/EROS and CHERI. Search by concept, not by the new M/H/T notation.

---

## 9. Current concise statement

The current working proposition is:

> **PP250 was certainly a capability machine, but “capability machine” may describe its protected authority mechanism rather than its deepest architecture. The emerging reconstruction is a sovereign governing relation M over an authority machine H and a computational machine T, with capability orthogonal to those roles. M need not contain system policy: C(N) shows how it can invoke T under an H-defined authority environment when policy is required. The C(S)-rooted startup establishes the first legitimate state from which C(N) can be configured; thereafter C(N) provides the normal protected callback from M into software. If correct, CALL/RETURN, CHP, fault recovery, power-up, C(S), C(N), capability genesis and the absence of supervisor mode are parts of one coherent architecture.**

This proposition should be preserved as a reconstruction to be tested, not promoted to historical fact until the evidence and literature comparison justify doing so.

## Evidence update — 2 October 2026: process transition and current capability state

**DOCUMENTED OBSERVATION:** US3771146A, Description 121, explicitly uses an interrupt/restore cycle to rematerialise workspace capability registers through the master table after a relocation decision. The operation restores computational state while obtaining capability bounds/state afresh rather than reviving stale expanded descriptors.

For the existing M⟨H,T⟩ investigation this supplies a further concrete observation of machinery acting on combined process and capability state. It supports testing the abstraction against restoration and relocation. It does not establish M as historical terminology, a third peer machine, or a particular mathematical formulation. No new mechanism is proposed here; the addition is historical evidence constraining the existing theory.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.
