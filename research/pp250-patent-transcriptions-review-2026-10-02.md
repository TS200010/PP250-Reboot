# Review of the ten patent transcriptions added on 2 October 2026

**Status: completed source review.** Repository baseline: `44f1260476a6d0710a778f02651a2009defb9a07`. This review records new evidence from the supplied RTF text against the existing reconstruction. It does not replace earlier completed source reviews or certify the transcriptions against the scanned figures.

## Corpus and provenance

There are ten files but nine distinct texts. `US4121286A.rtf` is byte-identical to `US4050059A.rtf`, including its internal patent number and read-and-hold subject. Both have SHA-256 `20f8201703ce3489186042560e2c425130db2e2590b3755adb59d83ed1af8c9b`. The allocation/deallocation patent therefore has **no independent new transcription evidence in this batch**. Its original PDF and conclusions drawn from it remain separate evidence. The supplied files are preserved unchanged.

Locators below refer to the numbered **Description** paragraphs unless otherwise specified; numbering restarts between patent sections. Source text was extracted with Pandoc without processing source images. Diagrams and missing embedded tables are not reconstructed from the surrounding prose.

| Supplied transcription | Relevant locators | Result against existing research |
|---|---|---|
| [US3771146A](../transcriptions/US3771146A.rtf) | 75, 78–81, 103, 111–121 | New explicit evidence for refreshing already-expanded capabilities during relocation; confirms dump-pointer restoration and deferred traps |
| [US3757307A](../transcriptions/US3757307A.rtf) | 40, 46–48, 49–65, 68–78 | Corroborates the completed normal-interrupt source review; no new dispatcher architecture required |
| [US3787818A](../transcriptions/US3787818A.rtf) | 4, 48–50, 63–90 | New detail on independently protected channel transfers and offline configuration |
| [US3814919A](../transcriptions/US3814919A.rtf) | 21, 48; fault steps S2, S10, S16; 120–134 | Confirms early fault-root chain; corrects six-right chronology; documents a subsequent-fault restore path bypassing outgoing dump |
| [US4050059A](../transcriptions/US4050059A.rtf) | 13, 16–21 | Bounds read-and-hold semantics and documents hold-integrity fault detection |
| [US4041460A](../transcriptions/US4041460A.rtf) | 17–19 | Received-address monitor and inverse support bus diagnosis; reset-decoder wording requires version-sensitive treatment |
| [US4121286A](../transcriptions/US4121286A.rtf) | Internal text is US4050059A | Duplicate; cannot verify allocation/deallocation or GARBAGE/VISITED from this file |
| [US4383297A](../transcriptions/US4383297A.rtf) | 38, 86, 92–93, 101 | Confirms later C(S) preset and constrained Internal Mode; no universal unrestricted register backdoor |
| [US4408274A](../transcriptions/US4408274A.rtf) | 106, 147, 154–155; LC/LCM descriptions | Corroborates later load-on-use, attenuation, propagation and SCT state; does not backdate them |
| [US4486831A](../transcriptions/US4486831A.rtf) | Background/Summary 16–18; Description 53, 58, 249–251, 291–297 | Explicit local lifetime rules and Protected Return null mask; later SPECIAL and saved primary state constrain startup hypotheses |

## Relocation and live capabilities

**DOCUMENTED OBSERVATION:** US3771146A paragraph 121 identifies the exact hazard already raised in the reconfiguration and attack-surface notes: capabilities expanded before relocation still contain old bounds. The relocating process interrupts the affected processor modules. Entering the handler and returning to the interrupted process reloads workspace capability registers through the reserved pointers saved in its Dump Stack. With the MCT sumcheck zeroed, the loaded register becomes unusable. Subsequent attempted use triggers the trap; merely executing LC does not perform the whole recovery operation.

**Constraint:** changing the SCT does not by itself invalidate every loaded register. The documented remedy is a software-coordinated process transition using hardware restoration. This establishes a relocation mechanism, not a complete multiprocessor acknowledgement/barrier protocol. It does not establish ROS semantic software replacement, channel quiescence, table-slot reuse, or general revocation. Those questions remain open.

## Early rights and generations

**DOCUMENTED OBSERVATION:** US3814919A paragraph 48 assigns bit 16 RD, 17 WD, 18 ED, 19 RC, 20 WC and 21 EC; bits 22 and 23 are spare. Its priority is 4 March 1971 and US filing 1 March 1972. This is early patent-family evidence for six independently named rights, not proof of a delivered implementation on the priority date.

The distinct PS(2)/DT(2)/RTE(4) Figure 3 representation in US3787813 remains evidence. However, the proposed clean Generation A-to-B chronology in which explicit execute/enter rights first emerge in 1975–76 is no longer supported. Structural comparison remains useful, but the two schemes are treated as **generation/version-specific representations**; an exact mapping between them is not required by the current reconstruction and is no longer an active unresolved question. Later LOU, propagation and local-store lifetime mechanisms remain later evidence.

## Startup and fault roots

**DOCUMENTED OBSERVATION:** the early fault patent describes inversion of old capability parity, loading the four-word fault block (sumcheck, base, limit and RSPC-0) through the restricted master-capability register, obtaining the checkout Dump Stack pointer and invoking automatic CHANGE PROCESS. Paragraphs 125–134 describe a subsequent-fault path through the fault interrupt identity table and another module; paragraph 132 bypasses the outgoing-register dump.

This strengthens the capability-rooted fault-entry reconstruction. It shows that process restoration can occur without a normal outgoing dump in a documented recovery case. Applying that fact to a virgin processor is **WORKING RECONSTRUCTION**, not a documented cold-start sequence.

US3771146A saves primary indicators with process state, including SECOND GROUP (bit 4). The later US4486831A saves/restores PIR (53) and makes SPECIAL (bit 4) redirect capability loading for one instruction (58). Restoring prepared initial process state is consequently a candidate explanation for initial special-register loading. Exact semantics must not be silently equated across versions. Automatic detection of incomplete special registers followed by repeated CHP grants remains **SPECULATION**. Neither this batch nor the fault-block root proves how virgin memory, the initial SCT and primordial stored capabilities were populated. The existing CHP/store-mode and capability-manufacture hypotheses are not resolved by this evidence.

## Channels, interconnect and holds

**DOCUMENTED OBSERVATION:** US3787818A describes channel-owned source/destination capability registers, a transfer Dump Stack with pointer pairs, SCT descriptor sumcheck/bounds validation and repeated transfer bounds checks. Its special registers include transfer-stack, SIW and SCT capabilities. Processor backdoor writes are permitted while the channel is offline (48). This is explicit protection for autonomous transfer, beyond the processor's ordinary access checks. It does not establish all access-right checks, online relocation coordination, or the complete authority governing channel configuration. Memory-mapped peripheral control does not require a separate I/O instruction set (4).

US4050059A paragraph 13 locks the whole access unit to the requesting bus/port during READ-AND-HOLD. A WRITE or RESET from that bus releases it; the described timeout is 10 microseconds. Paragraphs 16–21 use parity inversion on the following write to check hold integrity and route failure into the fault-handling mechanism, including multiplexed arrangements. This is an atomic access facility with an integrity check, not evidence for an arbitrary-duration software mutex or a lock spanning an entire relocation transaction.

US4041460A paragraphs 17–19 expose the address received by an access unit, and its inverse, through a monitor register. This supports localisation of bus/address failures during operation; it does not alter capability authority.

## Later local-store lifetime

**DOCUMENTED OBSERVATION:** US4486831A explicitly associates local descriptors with procedure nesting levels, automatically deallocates RLS allocations on return, invalidates registers referring to expired local storage, and prevents storing a capability at a lower associated level. The generational note's earlier question whether the names alone imply lifetime enforcement is now answered by this description. It still does not prove compiler influence or these facilities in the 1976 machine.

Protected Return also takes a low-twelve-bit mask from D(0) identifying D(1)–D(6) and C(0)–C(5) to null before unstacking (249–251). This is distinct from restoring registers selected by the saved call descriptor. The text does not justify inventing an individual mask-bit mapping beyond the documented register groups.

## Source limitations retained

These are textual discrepancies to preserve and investigate, not silently repaired source text:

- US3771146A's unavailable-entry branch names S7 where the zeroising step appears to be SY; the non-memory test wording also conflicts with its `11` memory classification (105, 114).
- US3814919A paragraph 21 reserves WCR7 for the pointer table; other early descriptions use WCR6. Its primary MIP and FIRST ATTEMPT in MIS must not be substituted for later PIR/FIR mappings.
- US4041460A's decoder treats codes other than 1, 2 and 4 as RESET; other bus descriptions give a specific reset code. Do not combine these into an unqualified universal decoder.
- US4383297A's PIR width/bit numbering and fault-timeout wording are internally problematic. Later C(C1)/C(C2), C(L) and C(S) maps must not be projected onto every early processor.
- US4408274A's propagation overview mentions source and sink permission whereas the detailed SC steps describe a source test and local exception. The full rule needs reconciliation rather than an invented combined test.
- US4486831A has inconsistent local sumcheck rotation, LSCCR wording, and a count described as eight pointers while naming C0–C5. Its high-level lifetime rule is explicit; these microsequence details are not yet an executable specification.

## Architectural consequence and research scope

The batch adds a documented refresh mechanism, protected channel detail, early six-right evidence and later lifetime/return detail. It corroborates normal interruption, fault roots and constrained Internal Mode. It does not overturn the capability-mediated process model or complete cold startup.

For M⟨H,T⟩, the relocation case is another historical observation that a process transition rematerialises capability state as well as restoring computational state. The abstraction remains theory; the evidence establishes neither a historical third machine nor a particular formal calculus.

Only living topic notes and the canonical subjects affected by these observations require additions or corrections. Earlier completed source reviews remain historical records. This review itself is a completed batch record; future interpretation belongs in the living topic notes unless an actual error in this record is found.
