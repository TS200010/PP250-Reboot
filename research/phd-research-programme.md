# Potential PhD Research Programme: The Authority Machine and the Sovereign Machine

**Status:** Working research proposal  
**Project:** PP250-Reboot  
**Created:** 2026-09-24  

## Purpose

This note consolidates two closely related doctoral-scale research ideas that have emerged from PP250-Reboot.

The first is already developed in `research/pp250-capability-architecture-provenance.md` under **Potential PhD thesis: The Authority Machine**. It asks where the hardware/software semantic boundary should lie and whether **authority**, rather than object semantics, privilege levels or operating-system abstractions, can be the minimal hardware foundation of a general-purpose computer.

The second emerged subsequently from reconstruction of the PP250's Church/Turing/process-transition structure. It proposes that System 250 may be understood as a composition:

```text
M<H,T>
```

where:

- **H** is the Church/authority machine: protected authority, capability state, domains and the rules governing legitimate access;
- **T** is the Turing/general computational machine: ordinary data computation, arithmetic, instruction progress and conventional mutable state;
- **M** is the governing machine/mechanism that mediates and transforms the state of H and T while preserving the architectural invariants between them.

**Notation rule:** reserve **C** for PP250 capability registers and capability-register names such as C0-C7 and C(S). Do not use C as the symbol for the abstract Church/authority machine.

The important new possibility is that these are not competing PhD ideas. They may be two views of the same research programme.

The **Authority Machine** asks what the minimum hardware abstraction should be.

The **Sovereign Machine** asks what kind of machine must govern computation and authority so that the abstraction remains true under ordinary instructions, protected calls, process changes, faults and power-up.

---

## 1. Integrated thesis hypothesis

A candidate overarching thesis is:

> **A general-purpose capability-native computer can be structured as a Turing/general computational machine T and a Church/authority machine H governed by a transition machine M. The hardware need not understand application-level object meaning; instead, M preserves the integrity of authority and controls the legitimate interaction between H and T. The Plessey System 250 may constitute an early concrete realization of this separation.**

This is a research hypothesis, not an established description of PP250.

The historical reconstruction, formal model, comparative analysis and experimental implementation would be used to test it.

---

## 2. Why M<H,T> is more than new notation

The research contribution would not be the symbols M, H and T themselves. Their value depends on whether the decomposition explains the machine better than existing descriptions and produces testable consequences.

The working interpretation is:

```text
                     M
             governing mechanism
                    / \
                   /   \
                  H     T
             authority  computation
```

T determines or requests computation.

H determines the authority within which that computation may act.

M ensures that transformations of T and H obey the architecture's invariants.

An ordinary instruction can therefore be represented abstractly as:

```text
M(H,T,instruction) -> (H',T')
```

A protected CALL may change both computational and authority context:

```text
M(H,T,CALL) -> (H',T')
```

A Change Process operation acts on a still larger portion of machine state:

```text
M(H,T,CHP) -> (H',T')
```

Fault and power-up are especially significant because they need not be instructions executed by T at all:

```text
M(H,T,FAULT) -> (H_recovery,T_recovery)

M(POWER_UP) -> (H0,T0)
```

This suggests that the architectural centre of gravity may not be the ordinary instruction-executing machine T. T is itself governed by a more fundamental mechanism.

---

## 3. The inversion: from subservient microcode to a sovereign M

In a conventional account of a microcoded processor, the architectural instruction-set machine is primary and microcode is an implementation technique beneath it:

```text
architectural ISA machine
          |
          | implemented by
          v
       microcode
```

Under the M<H,T> reconstruction, this relationship is potentially inverted conceptually.

The microcode and hardwired control logic implement **M**, and M governs both the ordinary computational machine T and the Church/authority machine H:

```text
microcode + hardwired control
             |
             | implements
             v
             M
            / \
           H   T
```

It is therefore more precise to say **M is sovereign** than simply “microcode is king”. Some operations of M may be microcoded and others may be hardwired physical behaviour, including reset values and fault-state transitions.

The research question is whether this is merely a convenient implementation description or a genuine architectural distinction with explanatory and security consequences.

---

## 4. A possible explanation for the absence of conventional privilege

Conventional machines commonly manage exceptional authority by allowing computation to enter a privileged state:

```text
user computation
       |
       v
privileged computation
       |
       v
machine/resource manipulation
```

The M<H,T> interpretation suggests a fundamentally different possibility:

```text
M<H,T>  --event-->  M<H',T'>
```

T need never become unrestricted or sovereign.

Operations that would conventionally require privileged software can instead be controlled state transitions performed by M. The resulting T continues to execute within a capability-constrained authority environment H.

This gives a candidate explanation for an important PP250 characteristic: the absence of a conventional supervisor-mode escape hatch need not mean that privileged functionality is missing. The functionality may instead reside in the governing transition mechanism rather than in a privileged version of T.

A possible general security principle is therefore:

> **Do not make computation privileged in order to change authority. Make authority-changing transitions operations of the governing machine.**

Whether PP250 actually realizes this principle throughout the architecture must be established by reconstruction.

---

## 5. Boot and fault as operations of M

This model has particular explanatory power for the unresolved bootstrap problem.

A machine in which ordinary data computation cannot fabricate capabilities must nevertheless create legitimate initial capability state somehow.

If power-up is an operation of M, no circularity is required:

```text
physical power-up
       |
       v
M(POWER_UP)
       |
       v
initial legitimate H and T state
```

The same reasoning applies to fault recovery:

```text
running M<H,T>
       |
      fault
       |
       v
M fault transition
       |
       v
M<H_recovery,T_recovery>
```

M is not an ordinary process requiring a capability that authorizes it to create the first capability. It is the mechanism whose correct operation defines which H states can legitimately exist.

This does **not** by itself establish the historical PP250 boot algorithm. It removes an apparent architectural paradox and produces a framework in which the surviving observations can be assembled.

---

## 6. C(S) as a candidate piece of M-state

The PP250 documentation diagrammatically distinguishes C(S), the start-up capability register/state, from the other internal and external C registers. Other surviving descriptions associate C(S) with start-up and fault/recovery behaviour.

Under the M<H,T> reconstruction this peculiar placement becomes potentially significant.

A working reconstruction to test is:

```text
ordinary authority/capability-machine state -> H
ordinary computational state                -> T
start-up / recovery state C(S)               -> M-state
```

or schematically:

```text
M[C(S)]<H,T>
```

This would explain why C(S) can survive or participate in transitions in which ordinary processor/capability state is invalidated, and why it does not fit naturally into the normal programmable capability-register set.

The important methodological point is that the research should not demand a lost document containing the sentence “C(S) belongs to M”. Instead it should ask whether that reconstruction is the simplest model that explains all credible observations and whether it predicts further properties of C(S) that can be tested against the surviving material.

A strong prediction would be that ordinary T/H operations cannot manipulate C(S) in the same unrestricted manner as ordinary capability registers, while power-up/fault machinery can establish or use it through M.

---

## 7. The reconstruction methodology is itself part of the research

There is no known definitive description of the complete PP250 architecture. Surviving sources resemble partial observations of one machine from different viewpoints.

The repository therefore now uses the method:

```text
Observation
    |
    v
Constraint
    |
    v
Reconstruction
    |
    v
Prediction
    |
    v
Falsification
```

This is particularly important to a doctoral treatment. The thesis must distinguish:

- directly documented observations;
- necessary inferences required to reconcile observations;
- working architectural reconstructions;
- speculation;
- unresolved unknowns.

The reconstruction becomes stronger when it explains observations that were not used to construct it.

The standard is therefore not simply:

> Where does a source explicitly say this?

but:

> Does the proposed architecture explain all reliable observations, contradict none, make useful predictions, and introduce fewer unsupported mechanisms than competing reconstructions?

This method is now recorded in `AGENTS.md` as a repository-wide research rule.

---

## 8. Relationship to the earlier Authority Machine thesis

The earlier PhD proposal asks:

> **What is the minimal hardware abstraction required to support a general-purpose capability-native software system, and does PP250's separation of hardware-enforced authority from software-defined meaning represent such an abstraction?**

Its central boundary is:

```text
SOFTWARE MEANING
      |
      v
interfaces / objects / services
-------------------------------
HARDWARE AUTHORITY
      |
      v
capabilities / Enter / CALL /
protected environments / SCT
```

The new M<H,T> insight adds another dimension beneath and across that boundary.

The Authority Machine asks **what hardware must know**.

The Sovereign Machine asks **what governs the legitimate evolution of the machine state**.

Together they suggest:

```text
                SOFTWARE MEANING
                      |
          objects / services / interfaces
                      |
================================================
                 M<H,T> MACHINE

          H                       T
      authority               computation
          \                       /
           \                     /
            +--------- M -------+
               governing
               transitions
```

M need not understand the semantic meaning of the object. It must understand enough about the protected representations and permitted transitions to preserve authority integrity.

This may sharpen the earlier principle:

> **Put only the mechanisms required to preserve authority integrity below the trusted boundary. Leave meaning above it.**

M is a candidate formal description of the mechanism that makes that rule operational.

---

## 9. Candidate doctoral contributions

### 9.1 Historical architectural reconstruction

Produce the most rigorous reconstruction possible of the original PP250, including:

- capability representation and SCT semantics;
- C registers and special capability-related state;
- C6/C7 and execution domains;
- Enter, CALL and RETURN;
- CHP and complete process-state transitions;
- dump-stack semantics;
- interrupts, watchdog and fault recovery;
- C(S) and start-up machinery;
- capability genesis;
- transition from inert hardware to the first legitimate process.

The purpose is not merely historical preservation. PP250 provides the concrete machine against which the proposed abstraction can be tested.

### 9.2 Formal M<H,T> model

Define H, T and M sufficiently precisely to state invariants and transition rules.

Candidate invariants include:

```text
ordinary T computation cannot manufacture authority

T !-> arbitrary H
```

and:

```text
all legitimate changes of authority-bearing state
occur through transitions admitted by M
```

The model should distinguish events initiated by ordinary instructions from asynchronous or implementation-level events such as faults and power-up.

### 9.3 Explanatory test against PP250

Determine whether M<H,T> gives a unified explanation of mechanisms that otherwise appear as unrelated special cases:

- ordinary data instructions;
- capability instructions;
- protected CALL/RETURN;
- CHP;
- process dump stacks;
- interrupts;
- faults;
- power-up;
- C(S);
- initial capability genesis;
- absence of conventional privileged mode.

A successful reconstruction should reduce rather than multiply special mechanisms.

### 9.4 Comparative architectural analysis

Compare the model with relevant architectures and research traditions, including PP250, CAP, Hydra, KeyKOS, EROS, PSOS, IBM System/38, Intel iAPX 432, CHERI and related capability machines.

The comparison should ask at least two orthogonal questions:

1. **Semantic boundary:** what does the hardware have to understand — authority alone, or richer object/type semantics?
2. **Sovereignty boundary:** where does the power to change authority reside — privileged computation, a reference monitor, capability operations, a governing transition mechanism, or some combination?

This is a more precise comparison than simply classifying systems as “capability machines”.

### 9.5 Independent top-down derivation

Retain the experiment proposed in the Authority Machine thesis.

Begin with modern requirements for a general-purpose capability-native software system and derive the minimum hardware mechanisms without assuming PP250.

Then ask whether the result naturally decomposes into something equivalent to H, T and M.

If the top-down derivation and bottom-up PP250 reconstruction converge, that is a significant result.

If they diverge, the differences identify historical contingencies or missing mechanisms and remain a valid doctoral result.

### 9.6 Executable validation

Implement the reconstructed/formal model first in software and then, if practical, in FPGA hardware.

The implementation should demonstrate that:

- ordinary computation cannot fabricate authority;
- protected invocation changes authority context without exposing unrestricted privilege;
- complete process transitions can occur without granting T sovereign machine access;
- boot can establish legitimate initial authority without a hidden software privilege escape;
- faults can transfer the machine into a legitimate recovery context;
- application-level object/service meaning remains in software rather than being embedded unnecessarily in hardware.

---

## 10. Falsifiability and negative results

The thesis must not depend on proving that PP250 was uniquely correct.

The M<H,T> reconstruction would be weakened or falsified if, for example:

- PP250 requires ordinary privileged T execution with unrestricted authority beneath the capability model;
- supposedly M-like operations turn out to be ordinary software conventions with no architectural distinction;
- C(S) behaves exactly like ordinary capability-register state despite its apparent diagrammatic separation;
- the proposed M layer adds no explanatory or predictive power over conventional ISA/protection descriptions;
- a top-down derivation of a minimal capability-native machine requires substantially different primitives;
- existing literature already contains an equivalent formal abstraction and the PP250 analysis adds no material new result.

Any of these outcomes would still sharpen the historical and architectural understanding and could support a thesis framed around why the apparently attractive decomposition fails or requires modification.

---

## 11. Novelty review required

Before treating the integrated thesis as an original academic contribution, conduct a serious conceptual literature review. Search by mechanism and abstraction rather than by the notation M<H,T>.

Relevant areas include:

- capability-machine formal models;
- reference monitors;
- security automata;
- protection machines;
- microcoded architectures;
- tagged architectures;
- separation kernels;
- hardware/software co-design;
- privilege and supervisor-mode alternatives;
- fault/recovery state machines;
- CAP, Hydra, KeyKOS, EROS, PSOS and CHERI;
- formal models of authority and protected state transition.

The key novelty question is not whether anyone has used the letters M, H and T. It is whether prior work has already articulated essentially the same architectural decomposition and consequences: computation and authority as distinct state spaces governed by a machine whose transitions remain sovereign over both, including boot and fault transitions, without resort to privileged computation.

---

## 12. Possible thesis titles

The existing title remains strong:

> **The Authority Machine: Finding the Minimal Hardware Abstraction for Capability-Native Computing**

The new insight suggests alternatives:

> **The Sovereign Machine: Computation, Authority and Protected State Transition in Capability-Native Computer Architecture**

or:

> **Authority as the Hardware Abstraction: Reconstructing the Plessey System 250 and the M<H,T> Model of Capability-Native Computing**

or, for a historically centred paper within the PhD:

> **System 250 Reconsidered: Computation, Authority, and the Sovereign Machine**

The final title should wait until the novelty review establishes which contribution is genuinely original.

---

## 13. Possible thesis structure

A plausible eventual thesis could be organized as:

1. **Problem and research questions** — why authority and privilege boundaries matter.
2. **Method** — reconstruction from fragmentary historical evidence using observation, constraint, reconstruction, prediction and falsification.
3. **PP250 reconstruction** — the machine as supported by surviving evidence.
4. **Authority versus meaning** — the semantic-boundary question from the Authority Machine work.
5. **The M<H,T> model** — formal definition of computation, authority and governing transitions.
6. **Boot, fault and process transition** — testing the model against the hardest PP250 mechanisms.
7. **Comparative architecture** — CAP, Hydra, KeyKOS, EROS, PSOS, System/38, iAPX 432, CHERI and other relevant systems.
8. **Top-down derivation** — derive the minimum capability-native hardware abstraction independently of PP250.
9. **Convergence/divergence analysis** — compare the independent derivation with reconstructed PP250 and M<H,T>.
10. **Implementation** — executable model and FPGA validation where practical.
11. **Security and architectural evaluation** — authority paths, trusted mechanisms, privilege, fault containment and no-bypass properties.
12. **Conclusions** — what should hardware know, and what should govern authority-changing transitions?

---

## 14. Core research questions

The integrated programme can currently be reduced to five questions:

1. **Where should a general-purpose computer place the boundary between software-defined meaning and hardware-enforced authority?**
2. **Can the state of a capability-native machine usefully and rigorously be decomposed into computation T, authority H and a governing transition machine M?**
3. **Does that decomposition explain PP250's apparently disparate mechanisms, especially CHP, fault recovery, power-up, C(S), dump stacks and the absence of conventional privilege?**
4. **Can a modern capability-native architecture be independently derived from software requirements and converge on the same abstraction without assuming PP250?**
5. **What security and engineering properties follow if computation never becomes sovereign and all authority-changing transitions remain governed by M?**

---

## 15. Current working proposition

The integrated PhD idea can presently be summarized as follows:

> **The Authority Machine investigates whether authority can be the minimal hardware abstraction of a general-purpose computer. The Sovereign Machine hypothesis adds that authority (H) and computation (T) may be distinct architectural state spaces governed by a third mechanism, M, which alone admits legitimate transitions between machine states. PP250 provides a fragmentarily documented historical machine from which this structure can be reconstructed bottom-up; a modern top-down derivation and executable implementation provide independent tests. If the two directions converge, the result may identify a general capability-native architectural principle rather than merely a historical peculiarity of System 250.**

This proposition should remain explicitly provisional until the historical reconstruction, formalisation and novelty review have progressed further.

---

## Relationship to existing repository notes

This document consolidates rather than replaces the earlier work.

- `research/pp250-capability-architecture-provenance.md` preserves the development of the **Authority Machine** thesis and its provenance context.
- `research/authority-integrity-and-semantic-boundary.md` develops the distinction between software-defined meaning and hardware-enforced authority integrity.
- the boot/capability-genesis research supplies the principal historical test case for the **Sovereign Machine / M<H,T>** reconstruction.
- `AGENTS.md` now records the architectural-reconstruction method to be used when working from fragmentary PP250 evidence.

Future doctoral-scale ideas should preferably be integrated here so that the project retains one coherent research-programme statement rather than accumulating disconnected thesis proposals.
