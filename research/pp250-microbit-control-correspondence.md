# PP250 microbit-to-hardware control correspondence — first evidence ledger

**Working research ledger · 19 October 2026 · historical reconstruction, not a new circuit design**

## Question and method

Reconcile the **83 clocked slots × 150 individual control-signal columns** described by the processor self-test simulator [P4] with the processor datapath gates, named μPROG outputs, register selectors, and observable condition registers in primary evidence. The target is a *control vocabulary*, not a conveniently chosen allocation that totals 150.

**Evidence classes:**
- **D / DOCUMENTED OBSERVATION:** source actually identifies a gate, control group, bit, function, or signal direction.
- **N / NECESSARY INFERENCE:** a documented operation requires some physical control, but not necessarily a standalone UPB microbit.
- **W / WORKING RECONSTRUCTION:** a plausible allocation of microbits not pinned down by a source.
- **U / UNKNOWN:** historical number, sharing, encoding, circuitry or assignment not established.

**Counting rule:** An output group label is not one bit; an operation name is not one bit; a four-bit architectural selector is not automatically four UPB bits; an individual register state bit is not automatically two microbits; a condition *input* to μPROG is not an output-column in the self-test matrix merely because it affects sequencing. A 24-bit AND gate array can be commanded by a *single enable function*. Separate the destination's electrical width, the required independent control actions, and the number of columns used to *generate* them.

**Generational boundary:** [P1/P2/P5] show an earlier patent embodiment; [R1–R4] describe the mature CPU MK3, not a proven identical physical build. A cross-generation functional correspondence is not an identity of wires. US4383297A's later A/B register files and ALU are not silently substituted for [P1] gates.

## A. The 24 explicitly drawn microprogram-controlled gate groups

[P1,P2] identify **19 numbered gates, GS, and four physically drawn but unnumbered AND-gate groups**. Each represents a documented independently describable gated *path*. Each requires a controllable gating *function*, but **the drawing does not prove one unique UPB bit per path**: qualified control, shared enable signals and decoding are all possible.

| Ledger | Gate | Documented gated path | Evidence | Count / qualification |
|---|---|---|---|---|
| G-01 | G1 | MIP → MHW | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-02 | G2 | MHW → ACC STK | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-03 | G3 | SSCR → MILL input A | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-04 | G4 | MHW → SDIREG | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-05 | G5 | SDIREG → SIH | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-06 | G6 | SOH/GS → OPREG | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-07 | G7 | SOH/GS → BASE STK data | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-08 | G8 | SPB → BASE STK parity | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-09 | G9 | SOH/GS → TC/LMT STK data | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-10 | G10 | SPB → TC/LMT STK parity | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-11 | G11 | BASE STK → MILL input A / COMP | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-12 | G12 | TC/LMT STK → MILL input B / COMP | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-13 | G13 | MHW → RES REG | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-14 | G14 | RES REG → MILL input A | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-15 | G15 | OPREG → MILL input B | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-16 | G16 | ACC STK → MILL input A | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-17 | G17 | MHW → OPREG | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-18 | G18 | OPREG → HIS STK | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-19 | G19 | MHW → SSCR | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-20 | GS | SOH → incoming-data distribution | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-21 | G?1 | SOH → IR | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-22 | G?2 | MIS → MHW | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-23 | G?3 | PGC → HIS STK | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |
| G-24 | G?4 | HIS STK → MILL input B | Gate group, AND gating under μPROG [P1,P2] | **One independently selectable *enable function*** per depicted gate; exact UPB column assignment and shared enables **UNKNOWN** |

**Count A:** 24 documented gate groups and 24 candidate enable functions; **zero verified one-to-one mappings to named UPB columns**. Do not interpret G11's two COMP inputs, or G12's two COMP inputs, as extra gates/enable functions. The four OR-isolation junctions are not additional independently actuated gates. G7/G8 and G9/G10 load capability data/parity separately in the figure, but may or may not share control terms in an actual slot.

## B. Direct named μPROG control groups and functions not reducible to the 24 gates

| Ledger | Source name | Destination/action | Evidence | Independently supported control requirement | UPB allocation |
|---|---|---|---|---|---|
| B01 | **RSEL** | SELA selects ACC/data register | D [P1,P2,P5]; 16 registers [R2] | Four-bit **logical register address**, including hidden D10–D17; source can be instruction or μPROG [P5] | U: neither four direct UPB bits nor a selector-source microbit is proved |
| B02 | **CRSEL** | SELB/SELL select two halves of capability register | D [P1,P2,P5]; 16 registers [R2] | Four-bit **logical capability address**, including C10–C17; both stacks refer to corresponding entry | U: four direct UPB bits, shared decoded lines or other source arrangements |
| B03 | **AUμS** | MILL operation/function selection | D [P1,P2] | N: distinguish operations needed for ADD/SUB/logic/shift etc. [R1] | U: number of bits, bit encoding, increment/pass functions and multiply/divide control |
| B04 | **CμS** | COMP comparison/check operation controls | D [P1,P2] | N: activate/check access, bound and parity conditions as appropriate | U: independent signals vs autonomous combinational checking |
| B05 | **FIS** | MIP fault/indicator control | D [P1,P2] | N: some condition-bit changes are selectively enabled; direct comparator fault feeds may coexist | U: individual SET/RESET controls |
| B06 | **SIμCS** | MIS secondary state controls | D [P1,P2] | N: some MIS state transitions are commanded | U: per-bit enable, set/reset, common clear or autonomous update |
| B07 | **SIHCS** | Outgoing store-interface command/control | D [P1,P2] | Three-wire legal operation coding **001 read, 010 read-and-hold, 100 write, 111 reset** [P2] | U: three code wires do **not** establish three independent UPB columns or the precise outbound/inbound physical field assignment |
| B08 | **GOS** | Generated offset for dump-stack addressing | D [P5] | N: internal offset selection/generation required | U: width, coding, independent microbits |
| B09 | **INC** | Advance historic-register selector SELH | D [P1,P2] | One **named scalar advance control function** | U: dedicated UPB column unverified |
| B10 | **+1S** | Increment SDIREG | D [P1,P2] | One **named scalar increment control function** | U: dedicated UPB column unverified |
| B11 | **μPGCS** | Grouped timed gate/reg/control signals | D [P1,P2] | Umbrella for gates and other control outputs | **NOT** an extra count: would double-count A/B controls |
| B12 | **Select next instruction** | End of machine instruction / slot sequencing | D [P3] | Control action explicitly referenced by patent | U: count as separate UPB/next-slot control only after confirming implementation; next-address selection is a distinct mechanism [P4] |

**Count B:** at least **10 named external-to-μPROG control groups/functions** (B01–B10), plus the specifically referenced sequencing action B12. None has a verified UPB column count. RSEL+CRSEL = **eight bits of logical selection payload**, NOT eight documented microbit columns. Store operation code = **three logical command wires**, NOT three documented microbit columns.

## C. State and operation requirements — do not convert indicator names straight into microbits

**Primary indicators [R3]:** MIP00 zero, MIP01 negative, MIP02 overflow; MIP04 second group, MIP05 inhibit interface faults, MIP06 odd data parity, MIP07 first attempt, MIP08 inhibit interrupts. [R1] distinguishes arithmetic instructions that modify LT/EQ or can set overflow from instructions that preserve them. **N:** conditional indicator *update/retention* must be supported. **U:** no per-indicator microbit/SET/RESET count; some indicators are produced by arithmetic logic or specialised hardware.

**Secondary indicators [R3]:** MIS00 microprogram overflow; MIS02 inhibit slot decode; MIS05 interval timer matured; MIS06 multiply; MIS07 divide; MIS08 set read capability; MIS09 IAR decrement; MIS10 out=limit; MIS11 trap; MIS12 time up; MIS13 cycle intercomplete; MIS14 move; MIS15 fault toggle; MIS16 fault link from BPW; MIS17 dump process before interrupt; MIS18 internal mode; MIS19 capability pointer in OPP; MIS20 HAD increment; MIS21 even parity internal; MIS22 busy; MIS23 status. **D:** these are named bits/state. **N:** some transitions require control or autonomous detection and can condition microsequencing. **U:** no basis for assuming one SET plus one RESET UPB per MIS bit, and it is not established which bits are controlled solely from μPROG.

**Fault indicators [R3]:** MIF00 bus corrupt; 02 interrupt timeout; 05 slave timeout; 06 capability parity; 07 sumcheck; 08 base/limit; 09 interface timeout; 10 parity comparison; 11 read-data parity; 12 invalid operation; 13 power failure; 14 invalid control code; 15 trap with MIP08; 16 and 19 hardware faults; 17 watchdog expired; 18 access violation; MIF20–23 offending capability-register number. **D:** named fault bits. **N:** fault capture/clear/acknowledgement exists. **U:** fault detectors may assert bits independently of the MBM; do not count one microbit for every fault.

**Register and address correction [R2,R3,P5]:** D17 is IAR, MIS09 is 'IAR Decrement', MIS11 is 'Trap'; TD produces TI and the documented trap sequence saves interrupted context. **N:** for retry the stored return address must identify the trapping instruction, whether preserved or corrected. **U:** no direct TI→IAR-decrement wire or fixed microbit allocation established. See [existing investigation](pp250-microarchitecture-reconstruction.md#e1-ti-trap-handling-and-restart-address-correction).

**Instruction sequencing [R4,P3,P4]:** UPAL, UPAC, UPAN/UPA, FUN, UPB0–UPB9 display positions, decoder inhibit and single-slot controls are evidenced. These **display labels are not ten individual control columns**. Slot selection is separately modelled in the self-test paper; exclude UPA/next-state address bits from the 150-output budget until the actual register/MBM boundary is proved. Diagnostic control bits (clock, stop after N slots, etc.) are diagnostic interface controls, not automatically part of an ordinary microinstruction's 150 output columns.

**Instruction operations [R1]:** ADD, SUB, CMP, logic, shifts, correlate, multiply/divide, conditional jumps, CALL/CHP/RET, capability load/store and MOVE demand appropriate functional sequencing. Their *mnemonics* cannot be counted as microbits; operation and state may be implemented through shared control codes and conditional slot transitions.

## D. Incoming conditions: trace, but exclude from the 150-column output count

| Condition | Documented origin/meaning | Relevance |
|---|---|---|
| SOHCS | Store → processor timing, response, parity/control information [P1,P2] | Qualifies store waits, completion and errors |
| AUCS | MILL arithmetic conditions [P1,P2] | Branches, arithmetic outcomes |
| CIS | COMP comparator conditions [P1,P2] | Bounds, access and parity checks |
| ICS | Indicator state [P1,P2] | Trap, fault, busy, operation state |
| PCS | PGC parity conditions [P1,P2] | Fault/protection response |
| FICS | Fault indicator conditions [P5] | Fault sequencing |
| OP=0 | Operand-zero detector ZD [P5] | Zero conditional action |
| TI | Trap detector TD output [P5]; 'Trap Indicator' is inferred expansion | Trap microsequence |
| AC | Access code tested by TD [P5] | Protection/trap qualification; exact μPROG connectivity unknown |
| Function/mode fields | Instruction register [R1,P3] | Instruction-conditioned control and next slot |

In particular **SOHCS is not missing from the model**: it is a major **input** group, not evidence of additional outgoing microbits. The exact amount of response conditioning and the code-wire topology remain unknown.

## E. Quantitative reconciliation: what can be counted without manufacturing precision?

| Measure | Value | Valid interpretation |
|---|---:|---|
| Control matrix columns [P4] | **150** | Documented distinct control-signal columns in the self-test simulation |
| Control matrix rows [P4] | **83** | Documented slots |
| Drawn controlled gate groups [P1] | **24** | 24 distinct identifiable *controlled datapath functions*, **not** 24 proven distinct microbit columns |
| Named μPROG output groups/functions (B01–B10) | **10** | Ten *named categories*, not ten columns |
| RSEL/CRSEL logical address payloads | **4 + 4** | Binary selection width; no proved UPB allocation |
| Store command-code payload | **3** | Physical/logical coding stated in early patent; no proved UPB allocation |
| INC, +1S | **2 named actions** | Two source-identifiable scalar commands, UPB assignment unknown |
| Distinct actually recovered UPB column identities → destinations | **0** | Neither the 150-column listing nor named bit-to-wire correspondences have been recovered |

**Do not carry forward the previously illustrative “100 allocated / 50 unexplained” calculation as a discovery.** Its assumed six ALU bits, four comparator bits, 28 secondary-indicator bits, five GOS bits, ten primary-indicator bits, etc. are not evidence-based, and summing the groups risks double counting. Likewise **24+8+3+…** is *not* a proved hardware minimum because selector/command encoding and gating may be derived or shared.

A **conditional implementation experiment**, explicitly not historical evidence, could model each drawn gate enable as a separate UPB column. This accounts for up to 24 provisional column identities under that assumption; it does not constrain remaining columns without knowledge of shared enables, register-select and arithmetic encodings, reset/set granularity, and auxiliary control circuitry. It must not be used as the reconstructed historical 150-bit microcode word.

## F. Evidence-to-control correspondence template for future additions

For every proposed UPB column, record:
1. **Source-observed signal name or gate label** (not just an instruction name).
2. **Controlled sink(s)** and **clocked behaviour**, with patent figure/step/paragraph.
3. **Whether it is an independent enable, a selector-code bit, an internally decoded consequence or a condition input**.
4. **Whether it can coincide with another candidate within one slot** (proof of independent control may arise from required simultaneous/distinct actions).
5. **Source generation** (early processor patent vs 1976 CPU MK3 vs later amendments).
6. **Assignment status:** verified UPB-column correspondence, documented control but unknown UPB mapping, necessary function or speculative allocation.

**Best discriminating checks within the current corpus:**
- Compare patent steps that independently operate G7/G8 and G9/G10; do simultaneous controls force shared bits or prove separate ones?
- Compare two operations that need different MILL functions while the same input gates are active; this distinguishes ALU function selection from routing gates.
- Distinguish instruction-field RSEL/CRSEL sourcing from μPROG generated selection when accessing hidden D10–D17 and C10–C17.
- Trace separate FIS/SIμCS actions during documented trap and fault microsequences, without assuming one control per named indicator.
- In store sequences, compare read, read-and-hold, write and reset command formation with SOHCS responses; preserve direction and never count incoming condition bits as outgoing microbits.

## Sources

[P1]: [US3814919A combined Figures 2a–2b capture](../transcriptions/figures/US-3814919-A-fig-2a-2b.md) and [canonical gate/signal inventory](../architecture/processor-control.md). The four G? labels are repository identifiers, not printed Plessey gate labels.
[P2]: [US3814919A patent transcription](../transcriptions/US3814919A.rtf), processor-module description, particularly paragraphs 13–14 and 32–43; selected memory-control coding in paragraph 48.
[P3]: [US3879712A diagnostic patent](../transcriptions/US-3879712-A_transcription.txt), μPROG/SM/MBM/UPA/UPB mechanism, clocked microbits and diagnostic controls.
[P4]: [Processor Self-Test Program](../transcriptions/processor-self-test-program.md), sections 2 and 4: 83 rows by 150 control-signal columns; independent table-driven next-slot selection; simultaneous controls and set/reset fault-simulation example.
[R1]: [Pocket Reference pp. 3–4](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt), instruction forms/functions/indicator side effects.
[R2]: [Pocket Reference pp. 5–7](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt), special registers/internal addressing.
[R3]: [Pocket Reference p. 8](../transcriptions/System%20250%20Pocket%20Reference%20-%20pg8.txt), named MIP/MIF/MIS bits.
[R4]: [Pocket Reference pp. 9–10](../transcriptions/System%20250%20Pocket%20Reference%20pg9-pg10.txt), UPB/UPA display and single-slot/decoder controls.
[P5]: [US3771146A patent transcription](../transcriptions/US3771146A.rtf), TD/TI trap indication, register selectors, generated offsets and microprogram-controlled trap state.
[H]: [Processor hardware description](../architecture/processor-hardware-design-description.txt), detailed explanation and source qualifications.

**Status:** Ledger establishes destinations, control actions and exclusions with traceable sources. No original 150-column bit-name assignment or individual microbit truth table has been reconstructed; the 83×150 matrix remains a measured historical constraint, not a table completed by assumption.
