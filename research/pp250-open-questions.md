# PP250 / System 250 — Current Open Architectural Questions

## Status

**CANONICAL CURRENT-STATUS INDEX**

This file is the authoritative index of architectural questions that are **currently open** in the PP250-Reboot reconstruction.

Historical research notes preserve the state of investigation at the time they were written. Their labels `UNKNOWN`, `HYPOTHESIS`, “unresolved”, “open question” and similar wording are **not by themselves evidence that an issue remains open now**.

## Admission rule

A question belongs here only when all of the following are true:

1. the current repository corpus and reconstruction have been checked;
2. a specific missing fact, conflict or mechanism can be stated precisely;
3. the missing item materially prevents or weakens reconstruction of historical System 250 behaviour, rather than merely leaving a bit layout, chronology or implementation detail undocumented;
4. the issue is not simply a difference between architectural generations, operating systems, forms or source viewpoints.

Before proposing a new architectural unknown, first check this file and the current canonical architecture. If an older note appears to expose a new issue, reconcile it against later evidence and conclusions before adding it here.

Generation-specific representations are allowed to differ. There is no requirement to derive a bit-for-bit evolutionary mapping between generations unless such a mapping is itself necessary to explain observed behaviour.

## Current open questions

### Initial protected processor state at cold power-up

**Open question:** what protected and special processor state exists immediately after cold power-up, before the C(S)-rooted start-up sequence establishes the first protected process environment?

The hardware/preset C(S) root accounts for primordial authority, but the exact initial state of the other protected/special registers and indicators has not yet been established. This matters because it defines the architectural starting state from which the documented start-up mechanisms operate.

### Provenance of the capability installed in C(C)

**Open question:** how is the legitimate capability that is ultimately installed in C(C) made available to the transition process?

The present reconstruction distinguishes two cases. At genuine cold start there is no existing runtime SCT, so an empty SCT block must first be created or prepared and a legitimate capability to it supplied for installation in C(C). On processor restart/rejoin, the runtime SCT already exists in a memory module being used by the running system, so the returning processor instead needs a capability identifying that existing SCT. The transition mechanism may be the same in both cases; what remains unresolved is the provenance and delivery of the appropriate source capability.

### Bootstrap from the initial empty SCT

**Open question:** after cold start has installed an initially empty SCT in C(C), how are the first SCT entries and the capability structures needed to make normal store management, process management and the rest of the runtime system available established?

Creating an empty SCT and installing its capability in C(C) establishes the normal capability-table root but does not by itself populate the runtime capability universe. The mechanism by which that initially empty table is bootstrapped into a usable normal-system environment remains to be reconstructed.

### Multiprocessor cold-start coordination

**Open question:** when a multiprocessor System 250 is powered up from cold, how are the simultaneously starting processors coordinated so that one normal runtime capability environment is established rather than independent competing start-up environments?

In particular, it remains to be established whether all processors initially execute the C(S)-rooted sequence, whether one processor becomes the bootstrap processor while the others wait or remain in a restricted state, how any such processor is selected, and how the remaining processors subsequently acquire the same C(C) and join the newly established runtime system. This is distinct from processor restart/rejoin, where an existing running system and runtime SCT are already available.

### M-extension sealing after One-Shot Second Group LC

**Hypothesis:** C(C), C(I) and C(N) are established from the C(S)-rooted start-up/recovery environment using **One-Shot Second Group LC**. Once that operation has been consumed, no authority accessible to H or T can replace those registers or reproduce their contents. If confirmed, the protected software reached through C(C), C(I) and C(N) is effectively sealed into M until the next C(S)-rooted recovery/start-up sequence.

**Open question:** after C(C), C(I) and C(N) have been established, is there any surviving route by which H or T can replace them, or recreate equivalent authority, without a new C(S)-rooted start-up/recovery sequence?

This matters architecturally because it determines whether the protected software entered through these registers is merely privileged system software or a protected software extension of M whose installation authority disappears after construction.

### Software access to and garbage collection of SCT entries

**Established context:** C(C) identifies the SCT used by the processor to expand compact active capability pointers into capability-register state. Stored active capabilities contain an SCT index/reference; they are not themselves capabilities to the SCT. The SCT entries themselves require lifecycle management, including garbage collection, and later evidence associates garbage-collection state such as `GARBAGE` and `VISITED` with SCT entries.

**Open question:** by what capability or architectural mechanism does the software responsible for SCT garbage collection traverse and manage the SCT entries?

If C(C) is the only reference that identifies the current SCT, it remains to be established whether software can use that authority directly to traverse the table, whether a separate ordinary capability to the storage containing the SCT exists, or whether some other protected mechanism provides the required access.

This also bears on SCT replacement during start-up/rejoin. Replacing C(C) establishes a new SCT for processor capability expansion, but until the management path is understood we must not infer that this necessarily removes every software-accessible route to the previous SCT or establishes how its entries/storage are subsequently reclaimed.

### Fault within the fault-handling process

**Open question:** what happens if the fault-handling process entered by the processor itself generates a capability fault?

The architectural path for a fault arising within the fault-handling process has not yet been established.

## Resolved, demoted or deliberately parked issues

The following must **not** be resurfaced as current architectural unknowns merely because older research records contain unresolved wording:

- **`PS/DT/RTE` versus `EC/WC/RC/ED/WD/RD`:** generation/version-specific representations. No cross-generation bit mapping is required.
- **SCT state/flag layouts across generations:** generation/version-specific representations. Reconstruct semantics within the relevant generation; cross-generation bit continuity is not required.
- **Inform/Outform:** part of the virtual-memory/storage-management mechanism. Outform/Inform conversion is managed by VM/storage management and VM traps. Exact representation may be generation-specific and is not, by itself, an architectural blocker.
- **Capability creation/resource allocation:** the allocator creates the resource and returns the appropriate capability; this is established architecture, not a missing genesis mechanism for ordinary runtime allocation.
- **VM trap and restart path:** reconstructed; do not reopen merely because an older source note calls some part unresolved.
- **LDP:** accepted architectural meaning is loading the compact capability pointer associated with its operand into a D register. Historical software uses are deliberately parked pending naturally arising evidence.
- **CALL / Enter Capability / RETURN:** reconstructed. CALL saves C6/C7/IAR, establishes the entered C6/C7 context, and RETURN restores it; other general registers survive the call boundary as documented.
- **Exact bit encodings or chronology:** an undocumented generation-specific encoding is a documentary gap, not automatically an architectural open question. Promote it only if the missing encoding prevents reconstruction of behaviour.
- **Cold start / primordial authority:** the startup root is architectural hardware/preset state, not an authority that ordinary software must manufacture. The corpus documents the hard-wired early SSCR/Fault-Block route and the later processor-preset `C(S)`/Special Start-Up Block route, leading through the special capability environment, reserved pointer/Dump Stack and process entry. Exact generation-specific microsequence or physical loading/commissioning details are documentary/implementation questions, not a missing architectural authority mechanism.

## Research-record rule

Do not rewrite completed source reviews merely to make their historical questions agree with this file. They are records of what was learned at that time.

Living topic-research notes may be updated when later evidence resolves an issue. Where retaining old reasoning is useful, mark it as superseded rather than allowing it to masquerade as a current open question.

## Maintenance

When a current open question is resolved:

1. remove it from **Current open questions**;
2. add a concise entry under **Resolved, demoted or deliberately parked issues** if recurrence is likely;
3. update the canonical architecture or relevant living topic note;
4. leave completed source reviews historically intact unless they contain an actual error.

When proposing a new open question, state exactly what historical behaviour cannot presently be reconstructed and why existing evidence does not already answer it.
