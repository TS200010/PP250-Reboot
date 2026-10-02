# PP250 cold-start hypothesis space

**Status:** working research note, 2 October 2026.

This note records the hypothesis space for reconstructing PP250 cold start where the surviving corpus does not yet describe the complete sequence. Its purpose is to preserve alternatives before evidence-driven pruning. It is not an emulator specification and does not select a preferred cold-start sequence.

The investigation should distinguish **documented mechanism**, **supported inference**, **possible but unsupported mechanism**, and **rejected mechanism**. A hypothesis should not be rejected merely because another appears simpler.

## 1. Problem boundary

The current reconstruction gets the processor from exceptional start-up/fault machinery to a native PP250 execution context through a Start-Up/Fault Block, a restricted capability environment, a Dump Stack and CHANGE PROCESS. Before normal operation, the special-purpose state associated with the normal system must be established, in particular:

- C(I) — interval/interrupt mechanism;
- C(C) — normal System Capability Table;
- C(N) — Normal Interrupt Block.

The Pocket Reference identifies these as C11, C12 and C13 respectively. C10 is C(D), the Dump Stack capability register. MIP bit 4 is named SECOND GROUP, and MIP is part of the saved/restored Dump Stack state.

The unresolved question addressed here is: **by what mechanism does cold-start execution establish the normal special-purpose capability state without violating the PP250 capability model?**

## 2. Important correction: do not assume hardware creates C(S)

An earlier line of reasoning treated C(S) itself as a capability fabricated or preset by hardware at power-up. That is stronger than the evidence presently warrants.

The evidence supports hardware preset/start-up information associated with the Start-Up/Fault Block. It does **not yet establish that hardware manufactures a valid C(S) capability at power-up**.

The reconstruction must therefore preserve the primordial transition explicitly:

```text
POWER UP
    |
    v
hardware-established start-up/fault information
    |
    ?       primordial authority/capability transition unresolved
    |
    v
restricted/special capability environment
    |
    v
Dump Stack
    |
    v
native PP250 execution
```

Any future statement that hardware "creates", "presets", or "fabricates" C(S) as a capability requires direct evidence.

## 3. Decompose the problem before selecting a narrative

The final mechanism may combine elements that appear in different hypotheses. At minimum four questions should be kept separate:

```text
SOURCE OF LEGITIMATE AUTHORITY
            |
            v
HOW IS A SPECIAL CAPABILITY REGISTER SELECTED?
            |
            v
HOW ARE C(I), C(C), C(N) POPULATED?
            |
            v
HOW ARE THE REQUIRED OPERATIONS SEQUENCED?
            |
            v
NORMAL PROCESSOR STATE
```

For example, authority might originate in the special SCT reached from the start-up structure, SECOND GROUP might redirect LC, and prepared Dump Stack states might sequence several operations. Rejecting one proposed complete narrative must not accidentally reject independently supported components.

## 4. Initial hypothesis space

The following deliberately includes obscure and weak possibilities. None is rejected merely by inclusion here.

### A. Repeated prepared-process entries

A Dump Stack restores SECOND GROUP; the first relevant instruction loads one of C(I), C(C) or C(N); another CHANGE PROCESS supplies a fresh prepared state; the operation repeats until the required registers are established.

This is the detailed hypothesis already explored elsewhere in the repository.

### B. One process re-establishes SECOND GROUP in software

A single start-up process executes several special-register loads, explicitly restoring or setting SECOND GROUP before each operation.

This requires a software-accessible mechanism for re-establishing MIP04 that has not yet been demonstrated.

### C. Chain of preconstructed Dump Stacks or processes

Several synthetic process states are prepared in advance. Each is entered with SECOND GROUP set, performs one special-register load, and transfers to the next prepared state.

Unlike a hardware completion-loop hypothesis, the sequence is encoded in the prepared process states.

### D. Repeated entry of one advancing Dump Stack

One synthetic Dump Stack/process is successively re-entered. Its restored MIP and execution frame cause successive entries to perform the required special-register loads.

This differs from C in requiring only one evolving/re-entered process state.

### E. Start-up/fault microcode establishes C(C), or its architectural ancestor

The C(S)-rooted machinery establishes the restricted special capability table in the same architectural state that becomes, or is ancestral to, normal C(C). Native code therefore begins with some C(C)-like authority already present.

The early fault patent's Master Capability Register behaviour makes this a serious version-sensitive possibility, but MCR must not be silently equated with the later C(C).

### F. Restricted C(C) is replaced by normal C(C)

A refinement of E: native execution starts with a restricted/special SCT already represented by the relevant master/SCT capability state. Start-up code replaces that restricted root with the normal SCT root and separately establishes C(I) and C(N).

This changes the problem from creating three roots from nothing to transitioning from one restricted root to the normal system roots.

### G. CHANGE PROCESS restores additional special capability state

A start-up form or undocumented aspect of CHANGE PROCESS restores some or all of C(I), C(C) and C(N), beyond the ordinary process state visible in the documented Dump Stack format.

No such fields are presently identified in the Pocket Reference Dump Stack.

### H. A capability operation has a bootstrap side effect

LC, LDP, EC or another capability operation used during start-up causes microcode to establish more special capability state than its explicit architectural destination suggests.

No such side effect is presently established.

### I. Establishing C(C) makes C(I) and C(N) automatically derivable

Start-up explicitly establishes only the normal SCT root. Interrupt/timer initialization machinery subsequently derives C(I) and C(N) from structures reachable through that SCT.

This requires evidence for such automatic derivation.

### J. A controlled interrupt/fault transition establishes further normal roots

Bootstrap deliberately invokes an architectural transition whose microcode establishes C(N), C(I), or other normal special state, analogously to the way exceptional machinery can change capability environments.

This is presently speculative.

### K. Another processor prepares the required state

In a multiprocessor configuration an already-running processor prepares memory structures, Dump Stacks or other state required for a newly starting processor.

This may explain processor addition/rejoin but cannot alone explain the first processor in a genuinely cold, uninitialised system.

### L. Maintenance/loading equipment supplies primordial state

External loading or maintenance equipment prepares memory and possibly other start-up information before the processor enters the capability architecture.

This may participate in whole-system initial loading, but the required mechanism has not yet been established from the corpus.

### M. Retained/preloaded memory makes start-up a reconnection problem

For processor restart or rejoin, valid capability tables, Dump Stacks and start-up structures already exist. The processor reconnects to previously constructed authority rather than creating it from nothing.

This is a distinct case from first start of an empty system and must not be used to explain the latter.

### N. Hardware directly presets normal special capabilities

Hardware directly establishes valid C(I), C(C), C(N), or some subset, at power-up.

There is presently no evidence that hardware fabricates such capabilities. This hypothesis remains in the search space for explicit testing rather than being rejected by assumption.

### O. Early SECOND GROUP has broader semantics than later SPECIAL

The 1976 MIP04 SECOND GROUP state may not be identical to the later one-instruction SPECIAL mechanism. It could select a broader alternate register or microcode environment with different start-up consequences.

The identification of early SECOND GROUP with later SPECIAL is useful but remains an inter-generation inference.

## 5. Architectural principles for pruning

These are **working reconstruction principles**, derived from the character of the PP250 architecture. They are not substitutes for primary evidence. Their purpose is to identify hypotheses that require unusually strong evidence.

### P1. Hardware should know architectural mechanisms, not software topology

The processor can know register roles, capability formats, CHANGE PROCESS, interrupt mechanisms and start-up structures. A hypothesis becomes suspicious if hardware must know the configuration-dependent locations of particular operating-system code, the normal SCT, interrupt processes or other software objects.

### P2. Exceptional primordial authority should be minimal

Some exceptional mechanism must break the bootstrap circularity. The PP250 architecture gives no present reason to assume a collection of independently preordained capabilities pointing at normal software objects when a smaller root can derive further authority.

### P3. Once capability machinery exists, authority should flow through capability machinery

Hypotheses that fabricate base/limit/access descriptions by ordinary data manipulation are inconsistent with the protection model unless the corpus explicitly documents an exceptional operation.

### P4. CHANGE PROCESS should not acquire undocumented privilege merely because it is used at start-up

A Dump Stack represents process state. Start-up hypotheses should use the documented CHP restoration model unless evidence establishes additional start-up behaviour.

### P5. Exceptional start-up machinery should converge onto ordinary PP250 mechanisms

Once native PP250 execution is possible, explanations using ordinary capability manipulation, tables and process transitions are preferable to continuing hidden bootstrap behaviour, unless primary evidence says otherwise.

### P6. Configuration should normally reside in memory structures rather than processor wiring

A fixed architectural route to a configurable stored description is PP250-like. A set of processor-wired capabilities pointing directly to particular software objects is much less so and requires strong evidence.

### P7. Do not solve the primordial-capability problem by assumption

Statements such as "hardware creates C(S)" or "power-up presets the system capabilities" cannot be used as premises unless the source explicitly establishes capability creation rather than merely presetting start-up addressing/control information.

## 6. First pruning step

The principles immediately make one **family** of explanations strongly disfavoured:

> Hardware contains a preordained set of capabilities directly identifying the normal SCT, interrupt block, timer structures, bootstrap code, or other configuration-dependent software objects.

Such a design would embed software topology and substantial initial authority in the processor instead of using the capability architecture to derive it from a minimal exceptional root.

This is **not yet recorded as a formal rejection of hypothesis N**. N remains available for direct evidential testing. The correct status is:

**STRONGLY DISFAVOURED BY ARCHITECTURAL PRINCIPLES; NOT YET REJECTED BY PRIMARY EVIDENCE.**

The distinction matters because the investigation is intended to show why alternatives were eliminated rather than retrofitting a preferred answer.

## 7. Evaluation method

Each hypothesis should now be tested against the complete corpus and assigned one of:

- **REJECTED BY EVIDENCE** — contradicts a documented mechanism or required state.
- **POSSIBLE BUT UNSUPPORTED** — architecturally possible but without positive evidence.
- **SUPPORTED INFERENCE** — multiple documented facts make the mechanism a reasonable reconstruction.
- **INDISTINGUISHABLE** — surviving evidence cannot currently distinguish it from another hypothesis.

For every rejection or promotion, record the specific evidence and the exact dependency being tested.

The objective is not necessarily to force a unique answer. If several mechanisms survive all available evidence, a faithful PP250 reconstruction should preserve that uncertainty explicitly.
