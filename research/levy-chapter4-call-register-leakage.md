# Levy Chapter 4 evidence: C0-C4 authority across protected CALL

**Status:** research evidence addition, 26 September 2026  
**Evidence class:** SECONDARY EVIDENCE  
**Source:** Henry M. Levy, *Capability-Based Computer Systems* (Digital Press, 1984), Chapter 4, “The Plessey System 250”, especially §§4.3, 4.4, 4.7 and 4.10.  
**Local working source:** `transcriptions/levy-capability-based-computer-systems-chapter-4-pasted-extract.txt`.  
**External source:** Henry M. Levy, Chapter 4 PDF, `https://homes.cs.washington.edu/~levy/capabook/Chapter4.pdf`.

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
C5    : by convention, current-process dynamic state
C6    : caller Central Capability Block
C7    : caller current code

CALL through Enter capability

inside callee
-------------
C0-C4 : survive unless software/procedure convention changes them
C5    : unchanged by the C6/C7 domain switch; retains its conventional role
C6    : entered Central Capability Block
C7    : selected executable code
```

Thus the C6/C7 transition establishes the protected execution environment, while C0-C4 may carry delegated authority across that boundary.

### C5 is a documented register convention, not a special hardware register

Levy's research account explicitly distinguishes the conventional uses of the program-accessible capability registers. In §4.3 he says that C6 is used “by convention” to point to the program's Central Capability Block. In §4.4 he gives the register usage more fully:

- C0-C4 are freely usable by the program to address accessible segments;
- **C5 points to a data structure used to store dynamically allocated elements associated with current process execution**;
- C6 points to the process's Central Capability Block;
- C7 contains a capability for the currently executing code segment.

This is evidence for a **System 250 software/register convention**. It is not evidence that the hardware assigns C5 special semantics merely because it is register number 5.

That distinction also clarifies the protected-CALL evidence. Levy §4.7 describes CALL in terms of saving and replacing C6/C7. His §4.10 leakage discussion specifically identifies **C0-C4**, not C0-C5. The natural interpretation is therefore:

- C5 retains the process-execution dynamic-state convention across an entered procedure;
- C6/C7 are the registers explicitly changed by the protected domain transition;
- C0-C4 are general capability registers whose survival can intentionally or unintentionally transfer authority.

Conceptually, two process executions may therefore enter the same protected package while retaining different dynamic process contexts:

```text
Process A                         Process B
---------                         ---------
C5 -> A dynamic state             C5 -> B dynamic state
C6 -> package environment         C6 -> same package environment
C7 -> package code                C7 -> same package code
```

The diagram is a reconstruction consequence of the documented convention and CALL behaviour. It should not be restated as a claim that C5 has dedicated hardware treatment.

## ABI consequence

This is directly relevant to future PP250 ABI work. C0-C4 cannot be treated merely as generic scratch registers if protected calls are involved. An ABI must specify, at minimum:

- which C registers may contain capability arguments across an Enter call;
- which are caller-saved or callee-saved;
- whether unused capability argument registers must be cleared at a protection boundary;
- whether capability return values are permitted in these registers;
- what confidentiality/confinement guarantee an ABI claims when registers are intentionally retained;
- how software observes the documented C5 dynamic-process-state convention across ordinary and protected procedure calls.

The mechanism can be useful rather than purely defective: retaining a C register across CALL provides an efficient way to delegate a capability argument to the callee. The security issue is that such delegation must be intentional and governed by calling convention rather than occurring as residual register state.

## Evidence discipline

The register-use descriptions and the C0-C4 leakage observation are **SECONDARY EVIDENCE from Levy**. The repository's local pasted extract is a working transcription and must still be checked against the original PDF before being treated as a verified transcription.

The important evidential distinction is now explicit:

- **documented in the research literature:** the C5 dynamic-process-state register convention;
- **documented in Levy's account of CALL:** C6/C7 are saved/replaced/restored during protected CALL/RETURN;
- **documented in Levy's discussion:** C0-C4 survive the domain change and constitute a possible authority-leakage channel;
- **reconstruction consequence:** C5 can carry process-specific dynamic context while different process executions enter a shared C6/C7 package environment;
- **not claimed:** any special C5-only hardware semantics.

This note supplements `research/pp250-execution-and-process-model.md`; it does not replace that reconstruction.
