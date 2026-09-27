# PP250 SECOND GROUP bootstrap hypothesis

Status: **RESEARCH HYPOTHESIS**, 27 September 2026. This note captures a reconstruction developed from the Pocket Reference, early System 250 patents, later Wheatley/Andrews patents, and the M⟨H,T⟩ model. It is deliberately not promoted to established architecture.

## 1. Established observations to preserve

- The early System 250 Primary Indicator/MIP bit 4 is named **SECOND GROUP**.
- The PP250 has an ordinary register group C0–C7/D0–D7 and a second/special group numbered C10–C17/D10–D17 (octal numbering).
- The Pocket Reference assigns C10=C(D) Dump Stack, C11=C(I) Interval Timer, C12=C(C) SCT, and C13=C(N) Normal Interrupt Block. C14–C17 are not assigned functions in that table and must remain UNKNOWN; they may be unused, reserved, or M-internal/scratch registers.
- C(S), the Fault Start-Up Block capability, is separate from the C10–C17 numbering.
- Later Wheatley/Andrews material places SPECIAL MODE in the corresponding PIR bit 4 position and states that it lasts for one instruction, permitting LC to address a corresponding special-purpose capability register. This is evidence for architectural lineage, not proof that every later semantic detail applies unchanged to the early PP250.
- Early capability material recognises an all-zero/null access code. Therefore `access != 0` is not universally synonymous with `defined capability`.

## 2. Working equivalence

Until contradicted by primary evidence, use the following as a **working hypothesis**:

> Early `SECOND GROUP` and later bit-4 `SPECIAL MODE` are successive descriptions/implementations of the same underlying second-register-bank selection idea.

For reconstruction purposes we additionally test the later one-instruction lifetime against the early architecture:

```text
normal instruction register field 0..7 -> C0..C7 / D0..D7
SECOND GROUP for one instruction       -> C10..C17 / D10..D17
```

The bit is therefore treated as a one-instruction bank selector rather than as a conventional persistent privileged mode.

## 3. The bootstrap problem this may solve

A one-instruction SECOND GROUP facility cannot by itself create a process, construct a Dump Stack, or manufacture a capability. It can, however, perform a final controlled installation of an already legitimate capability into a normally inaccessible special C register.

The proposed bootstrap role is therefore **installation, not capability genesis**.

## 4. Virgin Dump Stack hypothesis

A virgin Dump Stack begins the initial process transition. The hypothesis is that M can recognise that the special processor capability environment associated with startup is incomplete and, on the relevant CHP/change-process transitions, grant SECOND GROUP for exactly one instruction.

Bootstrap software uses that instruction to initialise one required special C register, then passes through CHP/M again. The cycle repeats until the required special capability registers have been populated.

Conceptually:

```text
hardware/startup state
       |
       v
virgin initial Dump Stack
       |
      CHP
       |
M sees required special-C state incomplete
       |
sets SECOND GROUP for incoming instruction
       |
LC legitimate stored capability -> one C1x register
       |
SECOND GROUP clears
       |
      CHP
       |
      repeat
       |
required special-C state complete
       |
M no longer grants SECOND GROUP
       |
normal sealed execution
```

The important security property is that H cannot obtain persistent special-register access merely by setting a bit in ordinary data. M controls the transition and the one-instruction grant.

## 5. Candidate completion test

The simplest candidate hardware implementation is deliberately modest. On reset, the relevant special C-register access fields are zero. For each **required** special register, a reduction OR detects whether any access bit has become non-zero. Those per-register signals feed an AND representing completion.

For a required set R:

```text
present(Cn) = OR(access bits of Cn)
complete    = AND(present(Cn) for Cn in R)
```

While `complete = 0`, the appropriate CHP/startup transition may grant SECOND GROUP for one instruction. When `complete = 1`, that route closes.

This requires no bootstrap counter, OS identity, conventional privilege mode, or software-settable persistent bootstrap flag. It is compatible with very small combinational logic and with the M model: M need not interpret the OS-level meaning of every access bit; it need only test the physical condition required by its transition logic.

## 6. Null capability qualification

Early patent evidence admits an all-zero/null access code. Consequently a simple non-zero test cannot distinguish an untouched/reset register from a deliberately loaded architectural null capability.

This does **not** presently kill the hypothesis. It constrains it:

- registers participating in the completion test may be expected to receive genuine non-null capabilities; and/or
- unused/unassigned second-group registers need not participate in the test at all; and/or
- a participating but otherwise unused register could be loaded with a harmless valid non-zero capability if the historical startup software required that.

Do **not** invent a separate `DEFINED` bit unless primary evidence requires one.

## 7. C14–C17

The Pocket Reference leaves C14–C17 unassigned. Their role is UNKNOWN.

If they are M-internal/scratch or otherwise not part of the OS-visible startup environment, there is no reason to include them in the completion logic. The completion test should therefore be described as ranging over the **required special capability registers**, not automatically over all C10–C17.

For the currently documented PP250 special registers, C10–C13 are the obvious candidates to investigate, but even this set must be checked individually because CHP may establish C(D) specially.

## 8. Relationship to capability genesis

This hypothesis narrows but does not solve the general capability-genesis problem.

SECOND GROUP can explain:

```text
already legitimate stored capability
             |
             | LC under one-instruction SECOND GROUP
             v
special processor capability register
```

It does not explain how the first stored capability for a newly allocated resource/SCT entry is manufactured. That remains a separate research problem.

The bootstrap and capability-genesis topics therefore intersect but must not be conflated.

## 9. M⟨H,T⟩ consequence

The hypothesis illustrates an important M⟨H,T⟩ principle:

> M may derive a temporary transition authority from protected physical processor state without exposing a forgeable privilege state to H or T.

Here the candidate condition is incompleteness of the required special-C state. M grants one narrowly bounded bank-selection operation; H supplies an already legitimate capability; ordinary T data cannot by itself manufacture the authority.

This is a reconstruction hypothesis, not yet a demonstrated historical implementation.

## 10. Primary-source tests

The hypothesis should now be tested against patents, manuals, processor diagrams and self-test material for:

1. exact early semantics and lifetime of MIP04 SECOND GROUP;
2. whether SECOND GROUP is set/cleared directly by microcode during CHP/startup;
3. reset contents/state of C10–C17;
4. any OR/reduction detection of access/type bits in special C registers;
5. any combined completion condition feeding CHP/PIR/MIP logic;
6. whether C(D) is installed automatically by CHP rather than by a SECOND GROUP LC;
7. exact startup roles of C(I), C(C), C(N), and any other required special C registers;
8. any documented functions for C14–C17;
9. whether any required special register may legitimately remain null;
10. evidence connecting virgin initial process/Dump Stack state with repeated startup/change-process transitions.

A finding of explicit completion logic over the special C-register state would strongly support the hypothesis. Evidence that SECOND GROUP can be freely requested/restored by ordinary process state, or that startup does not depend on special-register population, would weaken or falsify it.
