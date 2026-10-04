# PP250 cold-start hypothesis space

**Status:** historical hypothesis-space note, superseded as a statement of current open architecture. Retained to preserve the investigation path. Current status is governed by `pp250-open-questions.md`.

This note records the hypothesis space for reconstructing PP250 cold start where the surviving corpus does not yet describe the complete sequence. Its purpose is to preserve alternatives before evidence-driven pruning. It is not an emulator specification and does not select a preferred cold-start sequence.

The investigation should distinguish **documented mechanism**, **supported inference**, **possible but unsupported mechanism**, and **rejected mechanism**. A hypothesis should not be rejected merely because another appears simpler.

## 1. Problem boundary

The current reconstruction gets the processor from exceptional start-up/fault machinery to a native PP250 execution context through a Start-Up/Fault Block, a restricted capability environment, a Dump Stack and CHANGE PROCESS. Before normal operation, the special-purpose state associated with the normal system must be established, in particular:

- C(I) — interval/interrupt mechanism;
- C(C) — normal System Capability Table;
- C(N) — Normal Interrupt Block.

The Pocket Reference identifies these as C11, C12 and C13 respectively. C10 is C(D), the Dump Stack capability register. MIP bit 4 is named SECOND GROUP, and MIP is part of the saved/restored Dump Stack state.

**Superseded framing:** this was the question being investigated when the note was written. Subsequent corpus review establishes that startup authority is rooted in architectural hard-wired/preset startup state and structures. The remaining sequencing and generation-specific details in this note are documentary/implementation questions, not a current primordial-authority problem.

## 2. Historical caution, now superseded by the consolidated startup evidence

An earlier line of reasoning treated C(S) itself as a capability fabricated or preset by hardware at power-up. That is stronger than the evidence presently warrants.

At this stage of the investigation the distinction between preset startup addressing/control information and a hardware-established capability root was deliberately left open. The consolidated corpus now supports the architectural conclusion that the processor's exceptional startup state provides the legitimate root; ordinary software is not required to manufacture it. The following diagram is retained as historical reasoning, not as a current unresolved transition:

```text
POWER UP
    |
    v
hardware-established start-up/fault information
    |
    ?       [historical question; no longer a current architectural blocker]
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

Current reconstruction should instead follow the documented generation-specific startup mechanisms: early SSCR/Fault-Block hard-wiring and later processor-preset C(S)/Special Start-Up Block state. Do not generalise one generation's exact representation into another.

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

### B. Bootstrap-completion SECOND GROUP

The start-up transition enters a native process with SECOND GROUP already set. SECOND GROUP need not be consumed by the immediately following instruction and need not be re-established by software.

Instead, each relevant LC gets the normal one-instruction use of SECOND GROUP. The LC consumes/clears SECOND GROUP in the normal way after addressing the corresponding second-group capability register. The LC microcode then tests whether the required bootstrap special-capability state is complete. If it is incomplete, the microcode deliberately **sets SECOND GROUP again**, thereby granting a later LC another one-instruction access to the second group. When completion is reached, SECOND GROUP is left clear.

Thus the significant sequence is not necessarily three consecutive instructions. It is three relevant **LC events**, potentially separated by arbitrary non-LC bootstrap work:

```text
startup / CS
    -> initial process entered with SECOND GROUP set

    ... arbitrary non-LC work ...
    LC -> first required special capability register
         LC clears SECOND GROUP; state incomplete -> microcode sets SECOND GROUP again

    ... arbitrary non-LC work ...
    LC -> second required special capability register
         LC clears SECOND GROUP; state incomplete -> microcode sets SECOND GROUP again

    ... arbitrary non-LC work ...
    LC -> third required special capability register
         LC clears SECOND GROUP; state complete -> leave SECOND GROUP clear

    -> normal execution
```

The exact registers, their order, and the hardware test for "complete" remain to be established. C(I), C(C) and C(N) are the present candidates because they are the three identified normal special capability registers requiring establishment in the 1976 register set.

This hypothesis does **not** require software to manufacture or re-arm SECOND GROUP, and it remains compatible with the later one-instruction SPECIAL rule. The exceptional bootstrap behaviour is not persistence: LC consumes the one-instruction grant normally, after which microcode issues another one-instruction grant only while the fixed bootstrap-completion condition remains false.

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

### P8. Microcode implements bounded mechanism, not normal programming logic

PP250 microcode should be reconstructed as a stepped, tightly bounded and deterministic sequence of architectural operations: register transfers, fixed tests, selections and fixed control transitions. A hypothesis should be strongly disfavoured if it requires microcode to contain the sort of algorithmic or policy logic that would normally belong in PP250 program code — for example, walking arbitrary structures, searching tables, interpreting configurable data structures, making extended decision sequences, or orchestrating a general bootstrap algorithm — unless primary evidence explicitly documents such behaviour.

This does not exclude small fixed conditions within an instruction's microcode. For example, the bootstrap-completion SECOND GROUP hypothesis can satisfy this principle if LC clears SECOND GROUP normally, performs a fixed completion test, and re-sets SECOND GROUP only when bootstrap state remains incomplete. That is a bounded architectural state transition, not a bootstrap program implemented in microcode.

A useful working distinction is:

> **Microcode provides architectural mechanism and fixed state transitions; PP250 instructions provide algorithms and policy.**

## 6. First pruning step

The principles immediately make one **family** of explanations strongly disfavoured:

> Hardware contains a preordained set of capabilities directly identifying the normal SCT, interrupt block, timer structures, bootstrap code, or other configuration-dependent software objects.

Such a design would embed software topology and substantial initial authority in the processor instead of using the capability architecture to derive it from a minimal exceptional root.

This is **not yet recorded as a formal rejection of hypothesis N**. N remains available for direct evidential testing. The correct status is:

**STRONGLY DISFAVOURED BY ARCHITECTURAL PRINCIPLES; NOT YET REJECTED BY PRIMARY EVIDENCE.**

The distinction matters because the investigation is intended to show why alternatives were eliminated rather than retrofitting a preferred answer.

## 7. Result of first architectural pruning

Applying P1–P7 as an actual pruning pass reduces the active mechanism search space. This is architectural pruning, not a claim that every removed hypothesis has been independently contradicted by a primary source.

### Removed from the active mechanism set

- **N — hardware directly presets normal special capabilities.** Removed. Its defining mechanism requires the processor to manufacture configuration-dependent normal-system authority. There is no positive evidence for this, and it conflicts with P1, P2, P6 and P7. It can be reinstated only by direct evidence.
- **H — capability operation with hidden bootstrap side effects.** Removed. It adds undocumented authority-changing behaviour to otherwise architectural capability operations and conflicts with P3 and P5 unless direct evidence is found.
- **G — CHANGE PROCESS restores C(I), C(C), C(N).** Removed in its strong form. The documented Dump Stack does not contain these special capability registers, and treating CHP as secretly restoring them conflicts with P4. A genuinely documented start-up-specific CHP extension would constitute new evidence and would reopen the question.
- **J — controlled interrupt/fault manufactures normal roots.** Removed. It requires an undocumented exceptional authority-creation path when ordinary capability mechanisms are available.
- **I — C(C) automatically causes C(I) and C(N) to appear.** Removed in its original automatic form. A weaker proposition — that ordinary code, once given legitimate authority through C(C), can obtain capabilities subsequently loaded into C(I) and C(N) — is not a separate hypothesis and remains compatible with the surviving families.

These removals are deliberately stronger than "possible but unsupported": under the working PP250 architectural principles they are no longer active reconstruction candidates. They remain recorded above so that new primary evidence can reinstate them.

### Set aside: relevant, but answering a different question

- **K — another processor prepares state.** Relevant to processor addition/rejoin, not a complete explanation of first-processor cold start.
- **L — maintenance/loading equipment supplies primordial state.** Relevant to how stored bootstrap structures may initially be populated, but does not by itself explain the processor's architectural authority transition.
- **M — retained/preloaded memory.** Relevant to restart/rejoin, not an explanation of first start from an uninitialised system.

These are not rejected. They are environmental or system-initialisation cases and should not be mixed with the processor mechanism currently being reconstructed.

### Active hypotheses after first pruning

The remaining hypotheses are better treated as choices along three largely orthogonal dimensions rather than as competing complete narratives.

#### Authority entering native execution

- **E/F — restricted SCT environment / transition to normal C(C).** The exceptional start-up/fault path may already have established a restricted capability-table environment before the first native instruction. Bootstrap then replaces or transitions this into the normal SCT and establishes the remaining normal roots.

E and F are retained as one family until the relationship between the early MCR/special table and the 1976 C(C) can be established more precisely.

#### Access to the special capability registers

The documented **SECOND GROUP** state is the principal candidate mechanism.

- **later-SPECIAL-like interpretation** — SECOND GROUP redirects an LC to the corresponding special capability register for the relevant instruction;
- **O — broader early SECOND GROUP semantics** — the earlier mechanism may select a wider alternate register or microcode environment than the later documented one-instruction SPECIAL mechanism.

O therefore remains an important generation-sensitive alternative, not an independent source of primordial authority.

#### Sequencing the required loads

- **A/C/D — prepared-process / Dump-Stack sequencing family.** Restored process state supplies SECOND GROUP at the required moments. The variants differ over whether this is repeated prepared entries, a chain of synthetic processes/Dump Stacks, or repeated entry of one advancing process state.
- **B — bootstrap-completion SECOND GROUP.** The initial process begins with SECOND GROUP active. Arbitrary non-LC instructions may intervene between the relevant LC operations. Each relevant LC consumes/clears SECOND GROUP normally. After the LC, microcode tests the fixed bootstrap-completion condition and, while it remains incomplete, sets SECOND GROUP again to grant the next relevant LC one-instruction access. On completion, SECOND GROUP is left clear.

A, C and D are retained as one family until evidence about the sequencing mechanism distinguishes them.

### Reduced search space

```text
AUTHORITY
    |
    +-- E/F: restricted SCT environment
    |
    v
SPECIAL-REGISTER ACCESS
    |
    +-- SECOND GROUP with later-SPECIAL-like semantics
    |
    +-- O: broader early SECOND GROUP semantics
    |
    v
SEQUENCING
    |
    +-- A/C/D: prepared process / Dump Stack / CHP driven
    |
    +-- B: bootstrap-completion SECOND GROUP
    |
    v
NORMAL C(C), C(I), C(N) STATE
```

A likely eventual reconstruction may therefore be a **combination**, not one lettered hypothesis: E/F supplies legitimate initial authority, SECOND GROUP supplies access to the special capability registers, and either A/C/D or B supplies sequencing.

This reduced set is the starting point for the next evidential pruning pass.

## 8. Evaluation method

Each hypothesis should now be tested against the complete corpus and assigned one of:

- **REJECTED BY EVIDENCE** — contradicts a documented mechanism or required state.
- **POSSIBLE BUT UNSUPPORTED** — architecturally possible but without positive evidence.
- **SUPPORTED INFERENCE** — multiple documented facts make the mechanism a reasonable reconstruction.
- **INDISTINGUISHABLE** — surviving evidence cannot currently distinguish it from another hypothesis.

For every rejection or promotion, record the specific evidence and the exact dependency being tested.

The objective is not necessarily to force a unique answer. If several mechanisms survive all available evidence, a faithful PP250 reconstruction should preserve that uncertainty explicitly.


## 9. Preferred surviving hypothesis — prepared process chain and one-way C(C) transition

The subsequent investigation has materially strengthened the **A/C prepared-process / Dump-Stack sequencing family**. It is now the preferred reconstruction. This section does not remove the earlier hypothesis-space or pruning record; it records why one surviving family has become substantially better supported than the alternatives.

### 9.1 New evidence: SECOND GROUP is prepared process state

The Pocket Reference identifies **MIP04 as SECOND GROUP**, and MIP is saved in the process Dump Stack. Contemporary process documentation establishes that CHANGE PROCESS restores process state from the incoming Dump Stack.

This changes the sequencing problem. SECOND GROUP need not be manufactured by microcode, asserted by an undocumented privileged instruction, or re-armed by a special bootstrap rule. A process with legitimate authority to construct the incoming Dump Stack can prepare that process to begin with SECOND GROUP set.

```text
control/start-up process
        |
        | constructs target Dump Stack
        | target MIP04 = SECOND GROUP
        v
       CHP
        |
        v
target begins with SECOND GROUP set
        |
        v
LC can address the corresponding special capability register
```

The exact primary-source statement that the creator explicitly writes the initial MIP word has not yet been found. The reconstruction is therefore a **supported architectural inference**, not a directly documented bootstrap algorithm. It follows from the documented facts that the Dump Stack contains MIP, CHANGE PROCESS restores the incoming process state, and process Dump Stacks are allocated at process inception.

This evidence substantially weakens hypothesis **B**, because its proposed microcode re-assertion of SECOND GROUP is no longer needed to explain repeated special-register loads.

### 9.2 C(C) must be established first

The normal System Capability Table root **C(C)** must be installed before normal capabilities intended for C(I) or C(N) are resolved through the runtime capability-table environment. Otherwise those capability references would be interpreted relative to the wrong SCT.

The preferred ordering is therefore:

```text
first special load   -> C(C)
later special load   -> C(I)
later special load   -> C(N)
```

The ordering of C(I) and C(N) relative to each other is not yet established by this reasoning; the important constraint is that **C(C) is first**.

### 9.3 The transition process deliberately loses its startup path

A subtle consequence of changing C(C) initially appeared to be a problem and is now an important part of the hypothesis.

Suppose process B's Dump Stack is created and made reachable while the **startup/special SCT** is active. CHANGE PROCESS into B expands that Dump-Stack capability into **C(D)**. B therefore executes with:

```text
C(D) -> B Dump Stack        [expanded capability]
C(C) -> startup/special SCT
MIP04 = SECOND GROUP
```

B then uses its SECOND GROUP opportunity to install the normal runtime SCT:

```text
LC -> C(C) = runtime SCT
```

Changing C(C) does not invalidate the already-expanded C(D): the Dump Stack's real base, limit and access information are resident in the processor capability register. B can therefore continue to execute.

What disappears is the **stored capability path through the startup SCT by which B's Dump Stack was reached**. Once C(C) has changed, the runtime capability universe need not contain any route back to B's Dump Stack.

```text
STARTUP AUTHORITY WORLD

startup SCT
    |
    +----> B Dump Stack
              |
             CHP
              |
              v
       C(D) = expanded B DS

B: LC -> C(C) = runtime SCT

================================
        AUTHORITY BOUNDARY
================================

RUNTIME AUTHORITY WORLD

C(C) -> runtime SCT

C(D) -> B Dump Stack
        remains usable while B runs

runtime SCT ----X----> B Dump Stack
```

This is not a stranded-process defect. It provides a natural **one-way authority transition**. When B eventually performs CHANGE PROCESS, C(D) is replaced by the incoming process's Dump-Stack capability. If the runtime SCT has no path back to B's Dump Stack, B then becomes deliberately inaccessible.

No explicit revocation operation, memory clearing, hidden CHP privilege, or microcode bootstrap-completion test is required. Authority to the startup world disappears because the new capability-table root provides no path back to it.

### 9.4 Successive prepared processes supply subsequent SECOND GROUP opportunities

Once B has installed the runtime C(C), subsequent process construction can take place in the runtime capability environment. B can prepare or select an incoming process C whose Dump Stack contains MIP04 SECOND GROUP, then CHANGE PROCESS to it. C can perform the next special load and similarly prepare or select a further process.

The minimal conceptual chain is:

```text
startup process A
    |
    | prepare B Dump Stack with SECOND GROUP
    v
   CHP B

B:
    LC -> C(C) = runtime SCT
    |
    | startup authority is now cut off
    | prepare/select C under runtime authority
    v
   CHP C

C:
    LC -> C(I)
    |
    | prepare/select D with SECOND GROUP
    v
   CHP D

D:
    LC -> C(N)
    |
    v
normal online state
```

The labels A/B/C/D denote architectural roles, not documented PP250 process names. The corpus has not yet established that exactly four separately named software processes were used, nor that the three LC operations were consecutive instructions.

### 9.5 Processor rejoin uses the same mechanism

This mechanism also provides a coherent explanation for processor rejoin after successful checkout.

The fault/checkout environment is deliberately restricted and is rooted independently of the normal runtime SCT. The fault patent documents successful checkout progressing through a **rejoin/start-up process** before the processor returns to the online system. A returning processor must re-establish the normal processor-global capability state before normal scheduling can resume.

The prepared-process chain provides that transition without requiring a separate privileged "rejoin setup" operation:

```text
fault / checkout environment
        |
        v
rejoin/start-up process state
        |
        | prepared SECOND GROUP
        v
install normal C(C)
        |
        v
runtime-prepared state
        |
        | prepared SECOND GROUP
        v
install C(I)
        |
        v
runtime-prepared state
        |
        | prepared SECOND GROUP
        v
install C(N)
        |
        v
normal online processor
```

The important architectural point is that passing checkout does not by itself grant normal-system authority. The normal system can control the prepared rejoin state and thereby control the transition into the runtime capability universe.

### 9.6 Status of the surviving alternatives

The present evidential position is:

- **A/C — prepared process / preconstructed Dump-Stack chain:** **PREFERRED SUPPORTED INFERENCE.** It uses documented Dump-Stack MIP state, ordinary CHANGE PROCESS restoration and the documented SECOND GROUP mechanism without requiring additional hidden bootstrap behaviour.
- **D — repeated entry of one advancing Dump Stack:** still architecturally possible, but there is currently no need for it and the one-way loss of the first transition process favours a chain of prepared states.
- **B — bootstrap-completion SECOND GROUP:** now **strongly disfavoured**. It requires an otherwise undocumented LC/microcode rule to re-assert SECOND GROUP, whereas prepared incoming MIP state supplies the required opportunities using ordinary process machinery.
- **E/F — restricted SCT environment transitioning to normal C(C):** retained as the authority context in which the preferred sequence begins. The precise generation-dependent relationship between the early MCR/special table and the 1976 C(C) remains unresolved.
- **O — broader early SECOND GROUP semantics:** remains generation-sensitive and unresolved, but the preferred sequence does not require broader semantics than the ability to select the relevant special capability register for the LC.

### 9.7 Relationship to the reported three-instruction boot

Hamer-Hodges's recollection that **"three instructions booted the machine"** is striking in light of the three special-register establishments:

```text
LC -> C(C)
LC -> C(I)
LC -> C(N)
```

This correspondence is **not evidence that these were the remembered three instructions**. The recollection may refer to a different boundary or sequence, and the three special loads need not have been consecutive. It is retained as a research clue only.

### 9.8 Current preferred reconstruction

The preferred reconstruction is therefore:

> **A privileged startup/rejoin context constructs or selects an incoming process Dump Stack whose saved MIP has SECOND GROUP set. CHANGE PROCESS restores that state. The first transition process uses its one SECOND GROUP opportunity to install the normal C(C). Because its own C(D) is already expanded, it can continue after the SCT root changes, while the stored capability path back into the startup world disappears. Subsequent process states, now constructed or selected under the runtime SCT, provide further SECOND GROUP opportunities to establish C(I) and C(N). The transition is consequently implemented by ordinary PP250 capability and process mechanisms and is naturally one-way.**

This is a reconstruction, not yet a fully documented historical instruction trace. Its strength is that it now explains cold-start normalisation and processor rejoin with the same small set of documented architectural mechanisms while eliminating several previously necessary undocumented bootstrap behaviours.
