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

**None presently identified that meet the admission rule.**

The current corpus is sufficient to account for the architectural mechanisms presently under reconstruction. Missing documentary details, generation-specific encodings, software policy and exact microcode sequencing may remain research targets, but they are not architectural blockers unless a future audit demonstrates a specific behaviour that cannot be reconstructed.

Do not invent a replacement open question merely because this list is empty.

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
