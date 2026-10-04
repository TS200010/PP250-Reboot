# Anthony J. Stanners — System 250 first-hand recollections

**Status:** living first-hand source record  
**Compiler-project period recalled:** 1975–1977  
**Record consolidated:** 4 October 2026

## Purpose and evidence policy

This document consolidates first-hand recollections contributed by Anthony John Stanners during the PP250-Reboot investigation. Stanners worked on the CORAL 250 compiler at Plessey, Taplow, from 1975 to 1977 and recalls becoming project leader during the maintenance phase.

These recollections are historical source evidence in their own right. They must not be silently converted into documentary fact. Where surviving contemporary material independently supports a recollection, that corroboration is noted separately. Conversely, later reconstruction must not be written back into the recollection.

The wording below is a faithful consolidation rather than a verbatim interview transcript. Where Stanners has expressed uncertainty, that uncertainty is retained.

## 1. CORAL 250 work and people

### 1.1 CORAL 250 compiler

**Recollection:** Stanners worked on the CORAL 250 compiler from about 1975 until 1977, initially as a programmer and later as project leader during its maintenance phase.

**Status:** first-hand recollection. This is also the basis of the existing entry in `people/people.md` and the project README.

### 1.2 Ian David Cottam and compiler metrication

**Recollection:** Ian David Cottam was a colleague on the CORAL 250 work and worked at the desk opposite Stanners. Cottam developed a metrication/profiling mechanism for the compiler.

Stanners recalls that the compiler maintained a capability-based structure identifying its procedures, using Enter Data (ED) capabilities. Cottam duplicated this structure and altered the duplicate so that calls passed through a short instrumentation path before reaching the original procedure, allowing time spent in individual compiler procedures to be measured.

Stanners does not now remember which capability register held or referenced this procedure structure.

**Status:** first-hand recollection; implementation detail currently uncorroborated by surviving contemporary compiler material.

**Existing record:** `research/coral-250-capability-based-profiling.md`.

### 1.3 Martyn Andrews

**Recollection:** Martyn Andrews was a project manager associated with the CORAL 250 work.

**Status:** first-hand recollection. Andrews's involvement in later System 250 architectural development is independently documented by the Andrews/Wheatley patent families; the specific CORAL project-management role remains recollection.

### 1.4 Charlie Repton

**Recollection:** Stanners remembers Charlie Repton personally as a PP250 colleague/person.

**Status:** independently corroborated as a System 250 participant by patent evidence naming Charles S. Repton. Whether Charles S. Repton is the published D. J. Repton remains unresolved.

### 1.5 John Purdy / Purdey

**Recollection:** a John Purdy or Purdey is remembered in connection with the PP250/CORAL project.

**Status:** uncorroborated; spelling and role unresolved.

## 2. CALL, Enter Capability and RETURN

**Recollection (4 October 2026):** a CALL through an Enter Capability behaves as a call rather than a wholesale register-context replacement. If, for example, C3 contains an Enter Capability and the caller calls an entry at an offset through C3, the called domain receives that domain as C6 and the selected executable capability as C7. The other general capability registers pass across the call rather than being automatically replaced.

Every CALL creates a frame on the Process Dump Stack containing:

```text
C6
C7
IAR
```

RETURN restores that saved execution context. The reason this is a stack is that nested calls add successive C6/C7/IAR frames to the same Process Dump Stack used by the process; this call-stack region is distinct from the fixed full-process state saved for process suspension/context change, not a separate stack object.

Stanners also recalls that capability registers left with unintended contents on return were a known security weakness: capabilities could unintentionally cross the protection boundary because the non-C6/C7 capability registers were not automatically sanitised by CALL/RETURN.

**Corroboration:** strong contemporary corroboration.

- D. M. England, *Architectural Features of System 250* (1972), §§24–26, explicitly describes CALL through an enter capability, loading C7 with the selected execute capability and C6 with the called node's capability block, stacking old C6/C7, restoring them on RETURN, and allowing C0–C5 to carry parameters between nodes. England identifies the stack as the Dump Stack.
- D. Halton, *Hardware of the System 250 for Communication Control* (1972), independently describes C6/C7 and the same enter/CALL/RETURN mechanism.
- *System 250 Pocket Reference Book*, Issue 1 (May 1976), Process Dump Stack format, shows successive C6/C7/IAR triples and identifies the pushdown pointer for the CALL stack.
- H. M. Levy, *Capability-Based Computer Systems* (1984), later discusses capability-register survival across protected CALL as a leakage weakness. The repository analysis is in `research/levy-chapter4-call-register-leakage.md`.

**Status:** the core CALL/Enter/RETURN mechanism and C0–C5 parameter passage are independently documented. The characterization of unintended residual capability contents as a weakness is Stanners's first-hand recollection and is independently supported by Levy's later account.

## 3. CORAL calling convention and local storage

**Recollection:** the Generation-B-era CORAL compiler handled ordinary high-level-language procedure activation in compiler/software convention. Parameters could be passed in registers or on the stack; register preservation across calls therefore required calling convention/ABI rules. The compiler maintained its own local-store/stack mechanism rather than depending on the later architectural local-store machinery.

**Status:** first-hand recollection. Later patents document architectural local-lifetime machinery, but that later mechanism must not be projected backwards onto the CORAL implementation.

**Existing record:** `research/pp250-architectural-generations.md`.

## 4. LDP

**Recollection:** Stanners is fairly sure that the CORAL compiler generated or used LDP, although he does not presently remember what compiler operation required it.

**Status:** first-hand recollection with explicit uncertainty. The existence of LDP and aspects of its architectural operation are independently documented in the processor material, but the compiler use and purpose remain to be established. Do not infer the compiler use merely from the instruction's documented semantics.

## 5. CHP cost

**Recollection:** CHP (Change Process) was remembered as an expensive/heavyweight operation.

**Status:** first-hand qualitative recollection. This should not be converted into a cycle count or precise performance claim without documentary evidence.

**Existing record:** `research/interrupts-events-faults-and-process-switching.md`.

## 6. Withdrawn recollection

### CHP instruction example

An earlier discussion recorded a remembered example:

`CHP 3 0 C6`

Stanners subsequently withdrew this as unreliable.

**Status: WITHDRAWN — MUST NOT BE USED AS EVIDENCE.**

Its preservation here is solely to prevent the obsolete recollection from being rediscovered elsewhere in the research corpus and treated as evidence. The surviving startup/CHP reconstruction does not depend upon it.

## 7. Source-handling rules

When this record is cited:

- identify the relevant statement as **first-hand recollection of Anthony J. Stanners**;
- separately identify documentary corroboration where it exists;
- preserve uncertainty explicitly recorded here;
- do not strengthen a recollection because it happens to fit a later reconstruction;
- do not treat a later patent as proof that the same mechanism existed in the 1975–77 compiler environment;
- never use the withdrawn CHP example as evidence.

Further substantive PP250 recollections from Stanners should normally be added to this document first, with date and evidence status, and then cited from architectural or research notes as appropriate.
