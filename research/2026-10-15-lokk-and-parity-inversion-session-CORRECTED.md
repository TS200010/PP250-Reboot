# Session Closeout: LOKK and Parity Inversion — With Critical Review
**Date:** 15 October 2026  
**Participant:** TS200010  
**Session Start:** "There is a LOKK word in the dumpstack, see if you can find out what it is"  
**Status:** Research record with critical assessment included

---

## Executive Summary

This session investigated two unresolved questions:
1. **LOKK** — purpose of a word at Dump Stack offset 22
2. **Parity Inversion** — mechanism used in processor fault isolation

The investigation produced valuable findings about why processors deliberately invalidate capability registers following faults. However, subsequent architectural conclusions overreached the evidence. This document preserves both the research AND the critical review that followed.

**Key distinction:** Evidence-based findings vs. reconstructed hypotheses vs. unverified assertions.

---

# Part 1: LOKK Investigation

## What We Found

### Location
**Dump Stack offset 22 (octal)**
- Present in ROS/PDOS process state
- Absent from COS/POS layouts
- Saved/restored as 24-bit word like other Dump Stack entries

### Definition from Primary Source
**System 250 Pocket Reference:**
> "LOKK = Used by privileged system facilities"

That is the complete documented definition.

### Current Status
**From repository research:**
> "LOKK must not be conflated with the separately documented lock state associated with synchronising flags; its exact semantics remain UNKNOWN."

Listed as unresolved placeholder in `generation-2-system-250.md`.

## What Remains Unknown

| Question | Status | Evidence |
|----------|--------|----------|
| Actual purpose | Unknown | Only definition is "used by privileged system facilities" |
| Relationship to locking | Unresolved | Denied connection to synchronization locks, but evidence supports neither inclusion nor exclusion |
| Relationship to Error Control | Unknown | Adjacent in Dump Stack (offset 23), but connection undemonstrated |
| Relationship to fault handling | Unknown | No evidence LOKK is read/modified during fault microsequence |
| Why absent from COS/POS | Unknown | Absence does not prove facility was absent |
| Which "privileged system facilities" | Unknown | No documentation identifies them |
| Modification rules | Unknown | No evidence of read/write authority rules |

## Critical Assessment of LOKK Section

**Reviewer noted:** Document correctly concludes LOKK purpose is unknown, but then:
- Denies lock semantics while later considering locking interpretations (inconsistent)
- Treats proximity to Error Control as suggestive without establishing connection
- Absence from COS/POS treated too strongly as evidence of evolution

**Recommendation:** Retain LOKK as documented fact (location, definition, unknown purpose). Treat any functional interpretations strictly as hypotheses, not conclusions.

---

# Part 2: Parity Inversion — Evidence and Overreach

## Established Evidence (Strong)

### Primary Source: US 3,814,919A, S2 (INV PAR)

**Direct quote:**
> "Each time a fault interrupt occurs, with the first attempt indicator in the reset state, the state of the internal parity indicator is inverted. Hence **all the data currently resident in the processor module at this stage will be adjudged, if used erroneously, to have bad parity.** This has particular significance with respect to the capability registers as the stored parity bits for the base and limit addresses of each loaded capability register will now be invalidated. This arrangement ensures that the program currently being performed cannot be corrupted by the faulty processor as any attempt to use the currently loaded capability registers after the fault condition has been detected will result in a capability register parity fault condition."

**What this establishes:**
1. ✅ Internal parity indicator exists and can be inverted
2. ✅ Inversion invalidates existing capability-register contents
3. ✅ This prevents faulty processor from corrupting running system through old capabilities
4. ✅ This is a documented protection mechanism

**What this does NOT establish:**
- Exactly which hardware paths are affected by parity indicator
- Whether ALL store accesses use inverted parity or only resident data
- When/how parity indicator returns to normal state
- Exact timing through the fault microsequence

## The Architectural Insight (Valuable)

**This is important:** Parity inversion provides a way to invalidate authority already resident in a potentially faulty processor **without requiring that faulty processor to correctly clear each individual capability.**

This is a significant distinction:
- **Containing a faulty process:** Restrict what it can access through permissions
- **Containing a faulty processor:** Invalidate what's already loaded in its registers, regardless of whether the processor executes correctly

The Pocket Reference's approach: **invalidate by changing parity expectation, not by requiring the processor to execute trustworthy code.**

## Overreached Conclusions (Identified by Critical Review)

### 1. Parity Scope — Too Broad

**What I asserted:**
```
C(S) MICROCODE RUNS (with inverted parity)
    └─ All reads/writes use inverted parity
```

**Critical issue:** This is not established by the evidence.

The documented mechanism is: invalidate **capability information already resident in the processor.**

This is NOT the same as asserting that **every subsequent store access operates with inverted parity interpretation.**

**What needs verification:**
- Which specific hardware paths are affected by parity indicator?
- Does it affect C(S) microsequence store access? If so, how?
- Is parity interpretation during fault microcode different from normal execution?
- When store accesses occur during C(S), what parity rules apply?

**Status:** Unverified. Essential for microprogram reconstruction.

### 2. CHP Restores Parity — Unverified Assertion

**What I asserted:**
> "CHP restores parity to normal (as part of CHANGE PROCESS operation)"

**Critical issue:** No evidence in the commit establishes this.

**Possibilities:**
- Parity restoration could be an explicit microinstruction step
- Could be a side effect of CHP operation
- Could occur elsewhere in the fault sequence
- Timing and mechanism completely unestablished

**For accurate emulation/microprogram modeling:** This is essential detail.

**Status:** Plausible but unverified. Requires source verification.

### 3. Processor Containment — Security Claims Too Absolute

**What I asserted:**
> "Processor can execute with ANY internal faults... CANNOT escape SCT boundary"

**Critical issue:** These claims exceed the evidence.

The special SCT can restrict authority exercised through **functioning capability-checking hardware.**

It cannot automatically guarantee containment if:
- The fault compromises the SCT enforcement mechanism itself
- Address calculation logic is broken (generates wrong addresses)
- Capability checking hardware is corrupted
- Control signals are damaged

**What should be stated instead:**
- SCT provides important protection boundary for properly functioning hardware
- SCT restricts authority accessible through valid capability pointers
- Parity invalidation prevents use of existing capability registers
- Combination provides strong but not absolute containment

**For which fault classes is containment demonstrable?**
- Bit flips in capability register contents ✓
- Instruction fetch/decode errors ✓
- Arithmetic unit failures ✓
- ???

**Status:** Security claim too absolute. Should distinguish intended properties from demonstrably covered fault classes.

### 4. Cold Start Model — Hypothesis Presented as Established

**What I asserted:**
> "Same mechanism used for both fault recovery and cold start"
> "Seamless integration"
> "Architectural reuse"

**Critical issue:** This is presented as conclusion when it's a working hypothesis.

**The two-CHP model:**
1. Reset: direct/internal CHP 0 → checkout at 034
2. Checkout faults: capability fault occurs
3. C(S) establishes special SCT
4. Automatic CHP back to checkout
5. Checkout resumes with special SCT

**What's unverified:**
- Does cold start actually execute CHP 0?
- Is the primordial Dump Stack at absolute zero?
- Does checkout deliberately fault to trigger C(S)?
- Or does it work differently?
- How is parity handled at cold start?

**Critical questions remaining:**
- What exact protected state exists at power-up?
- Does cold power-up execute same C(S) sequence as fault recovery?
- What initial SCT/authority structure exists before checkout?

**Status:** WORKING HYPOTHESIS, explicitly marked as such. Should NOT be stated as established architectural integration.

## Summary: What's Evidence vs. What Needs Verification

| Claim | Status | Evidence | Action |
|-------|--------|----------|--------|
| Parity indicator exists | ✅ Documented | US 3,814,919A S2 | Use as established |
| Parity inversion invalidates resident capabilities | ✅ Documented | US 3,814,919A quote | Use as established |
| Special SCT restricts authority | ✅ Documented | Fault patents, Pocket Ref | Use as established |
| All store accesses use inverted parity | ❌ Unverified | NOT in sources | **Verify scope precisely** |
| CHP restores parity to normal | ❌ Unverified | Plausible but not shown | **Verify timing/mechanism** |
| Processor absolutely cannot escape SCT | ❌ Overstated | Depends on fault class | **Distinguish covered vs. uncovered faults** |
| Cold start = fault recovery path | ⚠️ Hypothesis | Circumstantial alignment | **Verify against S1-S20 microsequence** |

---

# Part 3: The Critical Review

## Review Summary

A detailed critical review was conducted after the initial commit. Key findings:

### Main Contributions (Valuable)
1. Explanation of why processor deliberately invalidates existing capability registers
2. Distinction between containing a faulty process vs. faulty processor
3. Recognition that parity inversion allows authority invalidation without trusting processor execution

### Principal Issues Requiring Correction

**1. Parity inversion during fault microsequence**
- Scope of parity effect not clearly established
- Distinction between invalidating resident data vs. changing all store-access parity
- Need to identify exactly which hardware paths are affected

**2. Assertions about CHP restoring normal parity**
- Plausible but unverified
- Timing and mechanism not established
- Essential detail for microprogram reconstruction

**3. Claim that special SCT makes containment absolute**
- Overstates evidence
- Assumes SCT enforcement hardware remains functioning
- Should distinguish intended properties from demonstrably covered fault classes

**4. Two-CHP cold-start model presented as established**
- Explicitly marked as hypothesis when introduced (good)
- But later sections state it as conclusion
- Remains working hypothesis pending microsequence verification

**5. LOKK speculation**
- Document denies lock semantics while considering locking interpretations (inconsistent)
- Should either exclude or include as hypothesis, not both
- Absence from COS/POS treated too strongly

### Editorial Issues

1. **Duplication:** Material overlaps with existing startup/fault/process notes. Should cross-reference.

2. **Source attribution:** Broad patent citations, but specific assertions not consistently linked to passages that establish them.

3. **Commit-ready language:** Describes architectural conclusions as ready when only file added is session record.

### Reviewer's Recommendation

> "I would retain the commit as a research record, without treating every conclusion in it as established architecture.
>
> The most valuable result is the recognition that parity inversion provides a means of invalidating authority already resident in a potentially faulty processor, without having to rely on that processor correctly clearing each individual capability.
>
> My recommendation is not to embark on further LOKK speculation. Instead, we should verify the complete fault microsequence against US 3,814,919A, specifically the transitions from parity inversion through C(S) to the automatic CHP.
>
> That should tell us which parts of this commit can safely be promoted from research into the processor architecture."

---

# Part 4: Next Steps — Verified Research Path

## Immediate Action: Microsequence Verification

**Trace US 3,814,919A fault steps S1 through S20:**

1. **S1/S2: Fault entry and parity inversion**
   - Establish exactly when parity indicator is inverted
   - Identify which hardware paths use parity indicator
   - Document scope of invalidation

2. **S3-S9: C(S) microcode execution**
   - How does C(S) access Special Fault Block with inverted parity?
   - Which capabilities/pointers does C(S) read?
   - How are validity checks performed?

3. **S10-S16: Loading special SCT**
   - What is the exact SCT loading sequence?
   - When is parity state relevant?
   - How is RSPC-0 resolved?

4. **S17-S18: Preparation for automatic CHP**
   - What state is established before CHP?
   - When does parity return to normal?
   - Is it explicit or implicit in CHP operation?

5. **S19-S20: Retry and module selection**
   - How does retry path differ from first attempt?
   - What role does FIRST ATTEMPT indicator play?
   - Module selection logic?

**Deliverable:** Annotated microsequence with:
- Each step's exact operation
- Evidence from primary sources
- Identified gaps requiring reconstruction
- Clear distinction between documented and reconstructed

## LOKK Research: Suspend Speculation

**Action:** Do not pursue LOKK functional hypotheses without additional evidence.

**What to do instead:**
- Search for ROS/PDOS source code (if available)
- Look for scheduler documentation mentioning LOKK
- Examine whether LOKK appears in fault microsequence descriptions
- Cross-reference with SIP (State and Internal Priority Word)

**Current status:** Documented location and minimal definition. Purpose unknown. Retain as placeholder.

## Cold Start: Remain Hypothesis

**Do not promote two-CHP model to architecture until:**
- Primordial Dump Stack location confirmed
- Initial processor context established
- C(S) behavior at cold start verified
- Relationship between direct CHP 0 and fault-entry CHP established

---

# Session Outcome

## What This Session Established

✅ **Parity inversion is a real documented mechanism** with significant architectural properties

✅ **The distinction matters:** Containing faulty process vs. invalidating faulty processor authority

✅ **LOKK exists at documented location** with minimal documented purpose

✅ **Primary source (US 3,814,919A) is essential** for microprogram-level understanding

## What Remains Unresolved

❌ Exact scope and timing of parity indicator effects

❌ When/how parity is restored to normal state

❌ Security boundaries under various fault classes

❌ Integration of cold start with fault recovery

❌ LOKK purpose and relationship to fault handling

## Status for Emulator/Microprogram Use

**DO NOT USE:** As specification without further verification

**USE AS:** Research foundation requiring detailed microsequence trace

**NEXT:** Verify against US 3,814,919A S1-S20 with explicit source citations

---

## Files and Sources

### Primary Sources Referenced
- **US 3,814,919A** — Fault Detection and Isolation (Plessey, 1974)
  - Critical sections: S2 (INV PAR), S3-S20 (microsequence)
  
- **System 250 Pocket Reference** (May 1976, Pages 5-7)
  - Dump Stack layout, LOKK definition

- **US 4,383,297A, US 4,486,831A** — CHP mechanism and process state

### Repository Documents
- `pp250-boot-and-processor-startup.md` — Fault recovery path
- `pp250-execution-and-process-model.md` — Dump Stack layout
- `generation-2-system-250.md` — Unresolved placeholders

---

## Conclusion

This session produced valuable research that correctly identifies an important architectural mechanism (parity inversion for authority invalidation). However, subsequent analysis moved from evidence into reconstruction without consistently identifying the boundary.

The work should be retained as research record, with:
1. Overreached assertions clearly marked as requiring verification
2. Source citations tied to specific passages
3. Distinction maintained between documented and reconstructed
4. Cold start and containment claims downgraded to working hypotheses

The recommended next step is detailed microsequence verification against US 3,814,919A, which will establish which parts can be promoted to architecture and which require further investigation.

---

*Session completed 15 October 2026 — Research record with critical review integrated. Updated based on detailed architectural review.*
