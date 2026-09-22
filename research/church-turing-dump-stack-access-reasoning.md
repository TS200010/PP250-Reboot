# From “Church” to Access

## A reasoning note on the PP250 capability, process and Dump Stack model

**Working research note — 22 September 2026**

## Purpose and evidence discipline

This note preserves the chain of reasoning developed while asking what Ken Hamer-Hodges meant by describing part of System 250 in “Church” terms. It is deliberately not a compressed architecture summary. The aim is to retain the intermediate conceptual steps, the mathematical model that emerged, the points at which that model had to be revised, and the unresolved questions.

The discussion unexpectedly ends at the nine-bit access/form field, which may be a key link between protected binding, capability authority, process state and the unusual status of the Dump Stack.

Throughout this note:

- **Documented** means behaviour or terminology found in the PP250 primary material already under study.
- **Interpretation** means a mathematical or conceptual model we are using to understand that behaviour.
- **Hypothesis** means a question suggested by the model but not yet established by the sources.
- The later Andrews/Wheatley patent is treated as evidence about a later PP250-family capability design and is **not** silently back-projected onto the 1972–76 machine.

---

## 1. The starting question: what could “Church” mean in a computer?

The investigation began with Hamer-Hodges's apparent distinction between a **Church side** and a **Turing side** of System 250.

The obvious historical reference is Alonzo Church and the lambda calculus, set against the conventional stored-program, state-transforming machine associated with Alan Turing. The useful question was not whether PP250 literally implements lambda-calculus instructions, but whether the distinction exposes two different kinds of machine operation.

The lambda calculus can be introduced with only three forms. A term M may be a variable x, an abstraction λx.M, or an application (M N).

M ::= x | (λx.M) | (M N)

Its fundamental evaluation rule is beta reduction:

(λx.M) N → M[x := N]

The notation M[x := N] means “M with occurrences of x replaced by, or more practically bound to, N.”

Real implementations normally avoid literal textual substitution. They maintain an **environment**: a mapping from names to values.

ρ = { x ↦ a, y ↦ b, … }

The Greek letter rho, ρ, denotes an environment. x ↦ a means that the name x is bound to the value a.

A **closure** combines executable code with the environment in which its free names are to be interpreted:

Closure = ⟨ λx.M , ρ ⟩

This supplied the first useful conceptual bridge to PP250.

## 2. The first PP250 correspondence: C6 as a protected environment

This suggested a way of looking at the PP250 C6/C7 structure.

C7 identifies the currently executing code block. C6 supplies the current capability environment — the protected set of references through which that code can reach the objects it is authorised to use.

C7 ≈ executable code

C6 ≈ protected environment ρ

CALL then begins to look less like a conventional jump-to-subroutine and more like **protected application**.

An Enter capability allows code to enter another protected domain. The machine establishes the called code's C7 and the capability environment represented by C6, while preserving the caller's C6/C7/IAR context for RETURN.

protected closure ≈ (C7 code, C6 capability environment)

At this point the analogy was still incomplete, because an ordinary closure does not carry the PP250 concept of authority.

## 3. The crucial refinement: PP250 bindings carry authority

An ordinary lambda-calculus environment binds a name to a value:

x ↦ V

That is not enough to describe PP250. A PP250 capability binds a reference to an object together with the **authority permitted over that object**:

x ↦ (V, R)

V denotes the referenced object or segment. R denotes the set of rights conveyed by the capability.

R ⊆ { EC, WC, RC, ED, WD, RD }

The same underlying object may therefore be denoted by capabilities conveying different authority:

C1 = (S, {RD})

C2 = (S, {RD, WD})

Both designate segment S, but the second binding conveys more authority.

This changes the closure analogy. The access rights are not an awkward extra feature attached to an otherwise lambda-like mechanism. They may be precisely what makes the binding **protected**.

In particular, an Enter capability can convey the authority to apply or enter a protected object without conveying ordinary authority to read or rewrite its internals.

**Enter Capability ≈ hardware-enforced opaque closure reference**

“Opaque” is important. Possession of the reference authorises an operation on the protected abstraction without necessarily exposing the abstraction's internal representation.

Thus the access rights may be the mechanism that turns an ordinary closure-like binding into a protected binding.

## 4. A two-part running-state model

Let P denote the architectural state of a running PP250 computation. Split it conceptually into T and Γ (capital Greek gamma):

P = (T, Γ)

T denotes ordinary mutable computational state — data registers, arithmetic state, instruction progress and similar conventional machine state.

Γ denotes the protected capability environment — capability registers and the authority-bearing context in which the computation runs.

The labels “Turing” and “Church” are interpretive shorthand, not claims that the processor literally contains two independent physical machines.

Ordinary instructions predominantly transform T while leaving the protected environment unchanged:

(T, Γ) —ADD→ (T′, Γ)

Capability operations can use or transform protected bindings under hardware rules:

(T, Γ) —LC/SC/CALL/RET…→ (T′, Γ′)

This suggested an important security invariant:

**T ⇏ Γ**

In plain language: the Turing side may compute using the world made available to it, but ordinary bit manipulation must not allow it to manufacture a different authority-bearing world for itself.

## 5. LDP at the boundary

LDP, Load Pointer, became interesting because the processor self-test description says:

> D := Pointer associated with A.

Whatever its exact detailed semantics prove to be, it appears to expose some pointer/reference information into an ordinary D register without thereby turning arbitrary D-register contents into capabilities.

That makes LDP a candidate **boundary operation between reference and authority**. It may reveal or manipulate a name/reference associated with a capability-mediated address while preserving the rule:

ordinary data ⇏ capability authority

This is an interpretation, not yet a complete instruction definition.

## 6. CHP breaks the two-level model

CHP, Change Process, does not merely manipulate Γ in the way CALL changes protected execution context. It saves the state of the current computation and restores another.

It therefore acts on the pair (T, Γ) as a whole.

If M denotes a complete runnable machine/process state:

M = (T, Γ)

then a conceptual CHP transition is not merely Γ1 → Γ2. It is closer to:

M1 → DS1

followed by:

DS2 → M2

where DS denotes a Process Dump Stack.

The first transformation preserves the outgoing process as a stored resumable state; the second establishes the incoming process from its stored state. This is a semantic description, not an asserted microinstruction sequence.

CHP therefore does not fit comfortably in the simple “Church instruction” bucket. Its architectural operand is the complete computation.

## 7. The Dump Stack as a different kind of object

This explains why the Dump Stack has repeatedly felt anomalous.

It contains ordinary data-register state, protected capability-register state, watchdog state, indicators, a CALL-stack pushdown pointer, and C6/C7/IAR frames.

It is therefore not merely an ordinary program stack, nor merely a collection of capabilities.

A useful mathematical word is **reification**: turning something that exists implicitly as active machine state into an explicit object that can be stored and subsequently acted upon.

DS = reify(M)

M = restore(DS)

PP250 documentation does not necessarily use the word “reification”; this is our model of what the architecture is doing.

Initially this tempted us to say that the Dump Stack was “outside” both the Turing and capability worlds. That formulation is too strong.

C(D) is itself a capability designating the active process's Dump Stack.

**C(D) is a capability; the object designated by C(D) has machine-state semantics.**

The capability mechanism controls access to the object, but the meaning of the object is the preserved representation of a computation.

## 8. One structure joins procedure state and process state

The Dump Stack appears to have two related roles.

Its fixed portion preserves process-level state such as C0–C5, D0–D7, watchdog state, indicators and other machine state.

Its variable portion contains C6/C7/IAR frames associated with nested CALL/RETURN activity.

| Dump Stack region | Role |
|---|---|
| Fixed process-save area | Preserves sufficient ordinary and capability state for suspension/resumption of the process |
| C6/C7/IAR call frames | Preserves protected invocation/environment transitions within the continuing process |

The same architectural object therefore sits at the join between **protected procedure invocation** and **complete process suspension**.

CALL/RET use it to preserve changes of protected execution environment. CHP uses it in changing the complete active process.

## 9. Faults, interrupts and watchdog activity suggest a third semantic level

Faults — and possibly watchdog and normal/interval interrupts, subject to exact source verification — appear to belong at the same semantic level as CHP rather than at the level of ordinary computation.

This suggests three semantic mechanisms.

### Level 1 — ordinary/Turing computation

Operations primarily on T: arithmetic, logic, ordinary data movement and branching.

### Level 2 — capability/Church computation

Operations involving protected bindings and environments Γ. LC, SC, LDP, CALL and RET are candidates for this class, although their exact historical classification still needs checking.

### Level 3 — process-state transition

Operations or events on the complete state:

M = (T, Γ)

CHP clearly belongs here conceptually. Faults and interrupts may invoke comparable hardware state-transition mechanisms.

A tentative abstract transition is:

M_A —event/CHP→ DS_A —restore→ M_B

For an explicit CHP the event is an instruction. For an interrupt or fault, the trigger is external or exceptional.

The precise microsequences may differ. The shared architectural idea is a protected transition between complete process states.

These levels are **not privilege rings**. They describe what kind of state a mechanism acts upon.

This is particularly attractive for System 250 because fault recovery need not imply a conventional supervisor mode. Hardware can transfer execution from one capability-constrained process state to another without first entering an unrestricted privileged software state.

## 10. Why this sharpens the capability-genesis problem

If arbitrary ordinary software could manufacture a Dump Stack containing arbitrary saved capability state and then cause that state to be restored, ordinary data computation would have acquired a route to manufacture Γ.

That would violate:

T ⇏ Γ

Therefore at least one of the following must be constrained by the architecture:

- how capability-bearing Dump Stack entries are represented;
- who can modify them;
- how CHP interprets them;
- how they are validated;
- how the first valid instance is created.

Which mechanism PP250 actually uses remains to be established.

This reframes the bootstrap problem. Instead of asking only “Who writes the first C6 and C7?”, ask:

**How does inert hardware first establish a valid machine-state representation from which legitimate protected authority can be restored?**

That question naturally reconnects C(S), the fault start-up block, checkout, the initial Change Process sequence and capability genesis.

## 11. The access-field question appears

The investigation then returned to something that had previously looked like a separate bit-format puzzle.

The capability pointer has a nine-bit region associated with access, yet the familiar PP250 capability vocabulary names only six rights.

The Pocket Reference transcription shows:

COS: 1 1 EC WC RC ED WD RD 0

POS: 0 1 1 EC WC RC ED WD RD

This establishes that the complete nine-bit quantity cannot safely be described as six permission bits plus three irrelevant spare bits.

The additional positions have prescribed values, and the six named rights appear in different positions in the two shown layouts.

The better question is therefore:

**What is the complete semantics of the nine-bit access code, of which the six named rights are only part?**

## 12. “Access code”, not merely six Boolean rights

Halton describes an access field containing a **linear code** defining permitted operations. The Pocket Reference labels the patterns **ACCESS CODES**.

This raises the possibility that a modern mental model — six independent Boolean permission flags — is too simple.

A more general model is:

A = encoded access/form value

where A is interpreted by the machine according to capability representation or form.

The named operations EC/WC/RC/ED/WD/RD may be semantic properties encoded within A rather than the complete meaning of A.

This possibility also fits the suspicion that the microcode may not intrinsically treat six universally fixed bit positions as six capability permissions. The full code may have to be interpreted in context.

That remains to be proved.

## 13. Andrews/Wheatley: FORM appears explicitly

The later Andrews/Wheatley capability patent provides an important clue, but it must be version-qualified.

In that design, the first and last bits of the nine-bit high-order region are **form discriminators**. Between them lies a seven-bit primary access field which, for a store capability, includes a propagation permission plus the six familiar rights.

F P EC WC RC ED WD RD F

Here F denotes a FORM discriminator and P denotes a propagation permission.

The important conceptual step is that **FORM determines how the remaining fields are to be interpreted**.

The patent describes different capability forms/classes. In at least some forms, the middle bits do not simply mean the six store rights.

This suggests a more general capability model:

**Capability = (form, authority-or-type, identity)**

- **identity** — what protected thing is being referred to;
- **form** — what kind of protected reference it is and therefore how its fields are to be interpreted;
- **authority/type** — what operations or resource semantics are conveyed within that form.

The apparent mismatch with the Pocket Reference must not be normalised away.

The 1976 COS/POS layouts and the later Andrews/Wheatley layout do **not** line up directly. That discrepancy is evidence to investigate.

## 14. Why FORM may matter to the Church/Turing/Dump-Stack argument

The FORM discovery does **not** prove that a Dump Stack has a special capability form. There is currently no basis for asserting that.

Its significance is more general.

The PP250 family contains an architectural idea in which a protected reference carries not merely an address and a set of permissions, but information controlling **how the reference itself is to be interpreted**.

That links back to the original Church question.

A protected binding is not simply:

name → bits

nor even necessarily merely:

name → (object, six rights)

The emerging possibility is closer to:

**name → (object identity, form, authority)**

The word **access** may therefore sit at the junction between several ideas that initially appeared separate:

- what an object is;
- how a protected reference to it is interpreted;
- what authority the holder has;
- whether that authority can be propagated;
- which machine operation is legitimate on the referenced object.

That is why a discussion that began with Church and lambda calculus has ended at access.

The common thread is the machine's control over the **meaning and authority of protected references**.

## 15. Consolidated working model

Running process:

M = (T, Γ)

where T is ordinary mutable computational state and Γ is the protected capability/binding environment.

Simple protected binding:

x ↦ (V, R)

More general protected binding suggested by FORM:

x ↦ (identity, form, authority)

Dump Stack:

DS = reify(M)

Security invariant:

T ⇏ Γ

CALL/RET operate within a continuing process, changing and restoring protected execution environments.

CHP acts on the complete process state.

Fault/interrupt mechanisms may cause comparable whole-process transitions through hardware paths.

The Dump Stack is capability-addressed but represents resumable computation.

Access/form semantics may be the mechanism by which the architecture controls legitimate operations on protected objects, but the exact relationship among the 1972 architecture, the 1976 Pocket Reference layouts and the later Andrews/Wheatley FORM scheme remains unresolved.

## 16. Evidence status

| Status | Proposition | Comment |
|---|---|---|
| **Documented** | C6/C7 define current protected capability/code context; CALL establishes called context and RETURN restores prior context. | Exact wording and version details still matter. |
| **Documented** | The Process Dump Stack saves capability registers, data registers and other process state and contains C6/C7/IAR call frames. | Pocket Reference and contemporary architecture descriptions support this. |
| **Documented** | C(D) designates the active process Dump Stack and is involved in process switching. | C(D) is therefore a capability even though its target has unusual semantics. |
| **Interpretation** | P or M = (T, Γ) is a useful decomposition of process state into ordinary and protected-authority components. | Mathematical model, not PP250 source notation. |
| **Interpretation** | The Dump Stack reifies a computation: DS = reify(M). | Explains why it is more than a conventional stack. |
| **Interpretation** | T ⇏ Γ expresses the anti-forgery requirement that ordinary bit computation cannot manufacture authority. | Security invariant inferred from capability-machine principles. |
| **Hypothesis** | CHP, faults and some interrupt/watchdog mechanisms form a process-state transition layer above ordinary and capability operations. | Strongly suggested; exact commonality of mechanism/microcode must be checked. |
| **Documented, later design** | Andrews/Wheatley uses FORM discriminator bits around a primary access field and supports multiple capability forms. | Must not be assumed identical to earlier PP250. |
| **Hypothesis** | The full access/form mechanism may provide a deeper unifying explanation of protected object semantics. | Research question, not conclusion. |

## 17. Questions to resume with

1. Find the earliest primary-source definition of the complete capability access field, including any use of the word **FORM**.
2. Trace the first and last bits of the nine-bit region through the original patents and papers, before the later Andrews/Wheatley extensions.
3. Collect complete access-code values from primary sources rather than decoding only the six named rights. Record the object/context associated with each value.
4. Determine the actual access/form values carried by or used for C(D), C(C), C(N), C(I) and C(S), where evidence exists.
5. Determine exactly what ordinary software may read or modify within a Process Dump Stack, especially saved capability entries and C6/C7/IAR frames.
6. Establish whether CHP validates or reconstructs saved capability state through the SCT and how this prevents arbitrary fabrication of authority.
7. Separate the mechanisms for explicit CHP, normal interrupt, interval-timer event, watchdog expiry and fault interrupt. Identify which genuinely share the same process-state transition machinery.
8. Revisit Hamer-Hodges's original Church/Turing terminology after the access/form evidence is assembled. Was “Church” describing protected naming/binding, capability manipulation, or a broader semantic division?
9. Resolve, rather than paper over, the mismatch between the 1976 Pocket Reference COS/POS nine-bit layouts and the later Andrews/Wheatley F-P-six-rights-F layout.

## 18. Why this line of reasoning matters

The reconstruction began as an attempt to understand a historical phrase.

It has produced a candidate conceptual architecture in which System 250 is not adequately described as a conventional processor with memory-protection checks added.

Ordinary computation, protected authority-bearing bindings, and complete process-state transitions appear to be distinct semantic concerns.

The Dump Stack is the object at which those concerns meet. It is reached through capability authority, contains both ordinary and protected state, supports protected CALL/RETURN history, and is used to suspend and restore a process.

The access field may be equally fundamental.

If access is an encoded statement not only of permission but, through FORM, of how a protected reference is to be interpreted, then it may explain why System 250's capability mechanism feels qualitatively different from a simple bounds-and-permissions memory scheme.

That possibility is sufficiently important that the next research step should be primary-source reconstruction of **access/form semantics** before the Church/Turing model is formalised further.

## Source anchors for resuming the investigation

- **System 250 Pocket Reference Book / Instruction Codes, Issue 1, May 1976**, especially pp. 4–7: access-code layouts, process structure, Dump Stack format and special-purpose CPU registers.
- **D. Halton, “Hardware of the System 250 for Communication Control” (1972)**: capability format, C6/C7, special capability registers, CALL/RETURN and interrupt discussion.
- **D. M. England, “Architectural Features of System 250” (1972)**: capabilities, domains, CALL, process management and Change Process.
- **US 4,408,274, Andrews/Wheatley, “Memory protection system using capability registers”**: later FORM discrimination, propagation and multiple capability forms; use as version-qualified evidence.
- **US 4,486,831, “Multi-programming data processing system process suspension”**: later detailed Process Dump Stack and process-suspension description.
- **Processor self-test transcription**: particularly the operational description of LDP and evidence about processor mechanisms.
