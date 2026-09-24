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
