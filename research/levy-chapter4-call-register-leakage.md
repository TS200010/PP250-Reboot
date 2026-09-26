# Levy Chapter 4 evidence: C0-C4 authority across protected CALL

**Status:** research evidence addition, 26 September 2026  
**Evidence class:** SECONDARY EVIDENCE  
**Source:** Henry M. Levy, *Capability-Based Computer Systems* (Digital Press, 1984), Chapter 4, “The Plessey System 250”.  
**Local working source:** `transcriptions/levy-capability-based-computer-systems-chapter-4-pasted-extract.txt`.

## Domain-switch qualification

The normal description of protected CALL correctly emphasises the domain transition: CALL saves the previous instruction position and C6/C7 state on the Process Dump Stack, installs the entered Central Capability Block as the new C6 environment, and installs the selected executable code in C7. RETURN restores the previous execution domain.

Levy identifies an important qualification to any claim that this produces a completely clean authority boundary.

According to Levy, **C0-C4 are not automatically cleared by the protected procedure call**. Consequently:

- capabilities deliberately or accidentally left in C0-C4 by the caller remain available to the called procedure;
- capabilities left in C0-C4 by the called procedure remain available to the caller after RETURN;
- C0-C4 therefore form potential capability/authority channels across the C6/C7 domain transition.

Levy characterises this as a weakness in the implemented protected-procedure mechanism and notes the performance tradeoff: retaining these registers makes them efficient for passing parameters. Software concerned with leakage can clear them explicitly.

## Reconstruction consequence

A PP250 protected CALL should therefore not be described simply as replacing *all* caller authority with the callee's domain.

A more accurate conceptual description is:

```text
before CALL
-----------
C0-C4 : caller-held capabilities / possible arguments
C5    : current-process dynamic state
C6    : caller Central Capability Block
C7    : caller current code

CALL through Enter capability

inside callee
-------------
C0-C4 : survive unless software/procedure convention changes them
C5    : current-process dynamic state persists
C6    : entered Central Capability Block
C7    : selected executable code
```

Thus the C6/C7 transition establishes the protected execution environment, while C0-C4 may carry delegated authority across that boundary.

### C5 is structurally different from C0-C4

Levy's register-usage description assigns C0-C4 to general program use but gives C5 a distinct role: it points to a data structure containing dynamically allocated elements associated with current process execution. His protected-CALL vulnerability discussion identifies the persistence of **C0-C4**, not C0-C5, as the authority-leakage problem.

Taken together with CALL changing C6/C7, this supports the following **strong reconstruction inference**:

- C5 is persistent process context;
- C6/C7 identify the currently executing protected package/domain;
- C0-C4 are general capability registers that can intentionally or unintentionally convey authority across the transition.

Conceptually, two processes may therefore enter the same protected package while retaining distinct process state:

```text
Process A                         Process B
---------                         ---------
C5 -> A dynamic state             C5 -> B dynamic state
C6 -> package environment         C6 -> same package environment
C7 -> package code                C7 -> same package code
```

This is an architectural interpretation of the documented/secondary register roles, not yet proof that the processor contains special C5-only hardware treatment. The distinction matters: **C5 can have an architecturally defined role even if its persistence is implemented simply because protected CALL replaces C6/C7 and leaves C5 untouched.** Primary instruction or microcode evidence should be sought before claiming more.

## ABI consequence

This is directly relevant to future PP250 ABI work. C0-C4 cannot be treated merely as generic scratch registers if protected calls are involved. An ABI must specify, at minimum:

- which C registers may contain capability arguments across an Enter call;
- which are caller-saved or callee-saved;
- whether unused capability argument registers must be cleared at a protection boundary;
- whether capability return values are permitted in these registers;
- what confidentiality/confinement guarantee an ABI claims when registers are intentionally retained;
- how the C5 persistent process-context role is preserved across ordinary and protected procedure calls.

The mechanism can be useful rather than purely defective: retaining a C register across CALL provides an efficient way to delegate a capability argument to the callee. The security issue is that such delegation must be intentional and governed by calling convention rather than occurring as residual register state.

## Evidence discipline

The survival/leakage observation here is **SECONDARY EVIDENCE from Levy** until checked against primary instruction/microcode documentation. It is consistent with the reconstructed role of CALL in changing C6/C7, but this note must not silently promote Levy's retrospective discussion to primary evidence.

The stronger C5 interpretation is explicitly an **inference** from Levy's distinct register roles plus the asymmetry of his C0-C4 leakage discussion. It should be tested against primary CALL/RET documentation.

This note supplements `research/pp250-execution-and-process-model.md`; it does not replace that reconstruction.
