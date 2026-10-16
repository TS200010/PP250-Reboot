# Parity-state toggling and fault recovery — preferred working reconstruction (16 October 2026)

**Status: PREFERRED WORKING RECONSTRUCTION**, not an established complete historical rejoin algorithm.

This note records a second, better hardware-grounded candidate for parity/fault recovery, developed by tracing US 3,814,919 Figures 2a/2b, 5 and 6. It supersedes the **parity-restoration assumption** in the previous CHP-centred explanation, **not** the documented role of automatic CHANGE PROCESS. Preserve the earlier research and its reasoning in `research/2026-10-15-lokk-and-parity-inversion-session-CORRECTED.md`, `research/pp250-boot-and-processor-startup.md` and `architecture/faults-interrupts-startup.md`; do not conflate fault entry with cold power-up.

## Documented observations

- **PRIMARY EVIDENCE — US 3,814,919, description paragraph 70 / Fig. 5 step S2:** On a fault with First Attempt reset, microprogram control changes the internal parity indicator in MIS. Signals PS control the inversion circuit IP and inform PGC and COMP which internal parity convention is selected. The patent explicitly says this renders previously loaded capability base/limit parity invalid, thereby preventing use of pre-fault authority.
- **PRIMARY EVIDENCE — Fig. 2a/2b and description paragraphs 40–41, 95–104:** SOHCS carries the stored-parity signal SPB, which passes through switchable inverter IP; the descriptor base/limit words are loaded into MCR, including parity bits, using gates G8 and G10. Figure 2 signal tracing in the 16 October session identifies the path `SOHCS (SPB) → IP → PGC, G8, G10`; the `BASE STK → BPB → COMP` path identifies BPB as the base-address parity signal (abbreviation inferred from label and connection). The description independently confirms that COMP checks base/limit register parity.
- **PRIMARY EVIDENCE — paragraphs 86–124 / Fig. 6:** Microcode uses SSCR to address the per-processor four-word SFB entry (sumcheck, special SCT base, special SCT limit, RSPC-0), installs the special SCT descriptor in MCR, loads a checkout Dump Stack capability from the special SCT and enters checkout through automatic process change.
- **PRIMARY EVIDENCE — paragraphs 126–127 / Fig. 5:** A *second fault during the same checkout/recovery attempt* bypasses S2, deliberately maintaining the already-inverted parity state.
- **PRIMARY EVIDENCE — paragraph 134:** Successful checkout resets First Attempt and may seek re-admission/rejoin; no explicit restoration of the original internal parity convention is stated there.

## Preferred candidate: parity toggling as a change of internal capability-parity generation

**WORKING RECONSTRUCTION.** A first fault toggles the processor's internal parity convention. Previously loaded capability-register base/limit parity bits remain unchanged and therefore fail comparison under the new convention. The memory-resident normal SCT itself is **not** rewritten or invalidated by this toggle. Recovery microcode independently reaches the SFB via SSCR and replaces MCR with the restricted special SCT descriptor. Incoming descriptor parity is conditioned through IP before being stored in the newly loaded capability registers, so those newly loaded capabilities can be valid under the selected internal convention. Checkout can thus execute **without first restoring** the former parity convention. Automatic CHANGE PROCESS remains part of establishing the new process state; it does not independently explain the parity transition.

After successful checkout and authorised rejoin, normal execution can proceed under the currently selected internal parity convention. The mechanism does not require restoring the former convention: that requirement was introduced by our earlier hypothesis, not by the parity circuit. With First Attempt reset, a later independent fault toggles the convention again, invalidating that generation of loaded capabilities. No separate research task to prove parity restoration or permanent non-restoration is required for this reconstruction.

The pre-fault normal SCT remains an intact memory structure but is no longer selected by the replacement MCR. An authorised rejoin path must supply access to the running system's normal SCT and restore legitimate process authority under the current internal parity convention. **UNKNOWN:** the exact authority transfer, register load and rejoin sequence; the passages examined do not establish it.

## Comparison with the earlier CHP-centred candidate

The earlier account correctly recognised C(S)/SSCR, restricted SCT, Dump Stack and automatic process change, but did not explain why a processor with inverted parity can use newly loaded capabilities. The figure-traced `SPB → IP → G8/G10` and stored-parity checking paths provide that missing hardware mechanism. Prefer **parity toggling + fresh capability loading + restricted SCT + automatic process change** to any account requiring an unsupported intermediate parity restoration. Do **not** classify CHP itself as disproven: it is a complementary documented state-transfer mechanism.

## Discriminating checks

1. Reinspect the original scan of Fig. 2a/2b to verify exact routing and signal labels (`SPB`, `IP`, `PS`, `G8`, `G10`, `BPB`, and corresponding limit parity signal), keeping diagram observation separate from textual description.
2. Do not introduce an unsupported parity-restoration step into checkout or rejoin. The selected internal parity convention already supports newly loaded capabilities.
3. Reconcile the normal-SCT restoration route with version-specific special-register/One-Shot Second Group LC evidence. Do not invent an additional privileged path.
4. Distinguish the independent read-and-hold bus parity inversion in US 4,050,059 from the internal capability-parity convention switched by US 3,814,919.

**Principal source:** `transcriptions/US3814919A.rtf`, especially descriptions (40–41), (70), (86–104), (115–124), (126–127) and (134). Figures 2, 5, 6 in `patents/` should be visually checked before treating the traced gate wiring as independently audited.
