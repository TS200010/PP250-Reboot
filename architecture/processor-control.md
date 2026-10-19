# System 250 Architecture — Processor Control

## Special-purpose CPU registers

Page 7 identifies:

| Register | Name / description |
|---|---|
| C10 | C(D) DUMPSTACK |
| C11 | C(I) INTERVAL TIMER |
| C12 | C(C) SCT |
| C13 | C(N) NORMAL INTERRUPT BLOCK |
| C14–C17 | not described |

A separate `C(S)` capability identifies the FAULT START-UP BLOCK and is not assigned one of C10–C17 in the Pocket Reference table.

Named special data registers include D10 (absolute D/S pushdown pointer), D11 (watchdog timer), D12 (first-fault MIF copy), D15 (interrupt accept register), and D17 (IAR).

## Indicator and fault registers

The source identifies MIP (Primary Indicator Register) and MIF (CPU Fault Indicator Register). MIP is saved at Dump Stack offset octal `20`; the dump-stack description says its MIF entry is a copy of the CPU Fault Indicator Register in the OS-specific portion where present.

## Processor naming and source aliases

Use stable **functional names** for components, paths and signals in this document. Patent lettering and gate numbers are source aliases, not global architecture identifiers. Qualify a reference as, for example, **US3814919A, Fig. 2, G6**. Gate G6 in another patent is not presumed to be the same gate.

When another source is examined:

- Add an alias to an existing functional entry only where the evidence establishes the same function and connections.
- Keep similar but unconfirmed components or paths separate, recording the proposed correspondence as inference.
- Record differences of arrangement or generation explicitly. Matching lettering alone does not establish matching state, bit assignments or implementation.
- Preserve source errors as printed; record the supported reading separately.
- Keep detailed drawing inventories in their figure captures and link them here.

The entries below establish the first source mapping. Their functional names are editorial naming conventions, not claims that these exact names appeared in Plessey documentation.

## Processor components and acronyms — US3814919A arrangement

**PRIMARY EVIDENCE / DOCUMENTED OBSERVATION:** [US 3,814,919 A, Figures 2a and 2b](../patents/US-3814919-A.pdf), PDF pages 3–4, printed drawing sheets 2–3. See the [combined figure capture](../transcriptions/figures/US-3814919-A-fig-2a-2b.md) and [patent transcription](../transcriptions/US3814919A.rtf), “PROCESSOR MODULE DESCRIPTION”.

This is the processor arrangement shown in the early patent, published in 1974 with a 1971 priority date. It supplements the later Pocket Reference account above; it does not replace it. In particular, this patent's MIP/MIS contents must not be assigned the mature machine's MIP/MIF bit layout. The [Generation 2 reconstruction](generation-2-system-250.md), especially its processor-control sections, remains the integrated account of the mature machine; [Process Model](process-model.md) owns programmer-visible process state.

The table gives expansions where supplied and descriptive meanings otherwise; it does not invent formal expansions for labels such as SSCR or IP.

| Stable functional name | Patent alias | Expansion or documented meaning |
|---|---|---|
| Processor module | CPU | Processor module / central processing unit |
| Accumulator stack | ACC STK | Accumulator/working-register stack |
| Capability base stack | BASE STK | Base-address half of the capability-register stack |
| Capability type-and-limit stack | TC/LMT STK | Type code and limit-address half of the capability-register stack |
| Result register | RES REG | Result register |
| Operand register | OP REG; OPREG | Operand register |
| Instruction register | IR | Instruction register |
| Special fault-block capability register | SSCR | Described as the special block capability register |
| Fixed fault-block address strapping | SF | Hard-wired strapping field feeding SSCR |
| Store-interface output register | SDIREG | Described as the memory data input register: holds processor-to-memory address/data |
| Historic register stack | HIS STK | Historic register stack |
| Primary indicator register, patent arrangement | MIP | Primary machine indicator register |
| Secondary indicator register, patent arrangement | MIS | Secondary machine indicator register |
| Arithmetic unit | MILL | Arithmetic unit; MILL is a name, not an expanded acronym |
| Address/access comparator | COMP | Comparator, including capability address/parity and access checks described in prose |
| Parity generation/checking circuit | PGC; PCG in one text passage | Parity generation and comparison/checking circuit |
| Store-parity inversion circuit | IP | Switchable inversion circuit on incoming store parity |
| Micro-program control unit | μPROG | Micro-program control unit |
| Accumulator-stack selector | SELA | ACC STK line-selection circuit |
| Capability-base-stack selector | SELB | BASE STK line-selection circuit |
| Capability-limit-stack selector | SELL | TC/LMT STK line-selection circuit |
| Historic-stack selector | SELH | HIS STK line-selection circuit |

## Data paths and gates — US3814919A arrangement

The patent describes parallel operation and typically 24-bit highways. A single drawn path represents a parallel connection; the two-arrow groupings do not specify two-bit buses.

Circles containing **2** are micro-program-controlled AND gating; circles containing **1** are OR isolation. The internal numeral is a symbol classification, not a gate reference. “MILL first input” and “MILL second input” mean its left and right top inputs as drawn, not documented port names.

All aliases in this table are scoped to **US3814919A, Figures 2a–2b**. **G?1–G?4** are repository-assigned identifiers agreed during manual review, not printed patent labels.

| Stable functional gate name | Source alias | Source → destination |
|---|---|---|
| Primary-indicators-to-main-highway gate | G1 | MIP → MHW |
| Main-highway-to-accumulator gate | G2 | MHW → ACC STK |
| Special-capability-to-arithmetic gate | G3 | SSCR → MILL first-input OR |
| Main-highway-to-store-output-register gate | G4 | MHW → SDIREG |
| Store-output-register-to-memory gate | G5 | SDIREG → SIH |
| Store-data-to-operand gate | G6 | SOH through GS → operand-input OR |
| Store-data-to-capability-base gate | G7 | SOH through GS → BASE STK |
| Store-parity-to-capability-base gate | G8 | SPB → BASE STK parity input |
| Store-data-to-capability-limit gate | G9 | SOH through GS → TC/LMT STK |
| Store-parity-to-capability-limit gate | G10 | SPB → limit-stack parity input |
| Capability-base-to-arithmetic/comparator gate | G11 | BASE STK → MILL first-input OR and COMP |
| Capability-limit-to-arithmetic/comparator gate | G12 | TC/LMT STK → MILL second-input OR and COMP; output shown entering COMP twice |
| Main-highway-to-result gate | G13 | MHW → RES REG |
| Result-to-arithmetic gate | G14 | RES REG → MILL first-input OR |
| Operand-to-arithmetic gate | G15 | OP REG → MILL second-input OR |
| Accumulator-to-arithmetic gate | G16 | ACC STK → MILL first-input OR |
| Main-highway-to-operand gate | G17 | MHW → operand-input OR |
| Operand-to-history gate | G18 | OP REG → history-input OR |
| Main-highway-to-special-capability gate | G19 | MHW → SSCR |
| Incoming-store-data gate | GS | SOH → common distribution to G6, G7 and G9 |
| Store-data-to-instruction gate | G?1 | SOH → IR |
| Secondary-indicators-to-main-highway gate | G?2 | MIS → MHW |
| Parity-circuit-to-history gate | G?3 | PGC → history-input OR |
| History-to-arithmetic gate | G?4 | HIS STK → MILL second-input OR |

The four OR junctions have no separate printed gate references:

| Stable functional junction name | Gated inputs | Destination |
|---|---|---|
| Arithmetic first-input OR | G3, G11, G14, G16 | MILL first input |
| Arithmetic second-input OR | G12, G15, G?4 | MILL second input |
| Operand-input OR | G6, G17 | OP REG |
| History-input OR | G18, G?3 | HIS STK |

MILL's output feeds MHW, whose return path crosses both halves of the drawing and supplies the destination gates. The two sheets describe one continuous circuit. The figure capture retains the detailed join ledger.

**Owner-confirmed drawing reading:** COMP receives BPB and the G11 output, and LBP and the G12 output. G12's output is drawn entering COMP twice; these are two drawn entries from that output, not two separate G12 gates. “LBP” retains the owner's manually confirmed lettering; the original figure capture read it as “LPB”. This document records that correction explicitly without silently changing the source capture.

## Control and condition signals — US3814919A arrangement

The stable names describe function. Expansions/meanings combine drawing labels with the patent's processor description, particularly its micro-program control and parity-circuit descriptions. A descriptive expansion does not establish individual wires or timing within a signal group.

| Stable functional signal/path name | Source alias | Expansion or meaning; direction |
|---|---|---|
| Main data highway | MHW | Main highway; MILL and gated indicator outputs → register-loading paths |
| Incoming store-data highway | SOH | Store output highway; memory → processor |
| Outgoing store-address/data highway | SIH | Store input highway; SDIREG through G5 → memory |
| Incoming store-control group | SOHCS | Store output highway control signals; memory → μPROG and parity/checking paths |
| Outgoing store-control group | SIHCS | Store input highway control signals; μPROG → memory access controls |
| Micro-program control group | μPGCS | Micro-program control signals, including gate controls |
| Accumulator selection | RSEL | Register selection; μPROG → SELA |
| Capability-register selection | CRSEL | Capability-register selection; μPROG → SELB and SELL |
| Arithmetic operation controls | AUμS | Arithmetic-unit micro-program control signals; μPROG → MILL |
| Arithmetic conditions | AUCS | Arithmetic-unit condition signals; MILL → μPROG |
| Comparator operation controls | CμS | Comparator micro-program control signals; μPROG → COMP |
| Comparator conditions | CIS | Comparator condition-indicating signals; COMP → μPROG; comparator indications also feed MIP |
| Primary fault-indicator controls | FIS | Fault-indicator control signals; μPROG → MIP |
| Machine-indicator conditions | ICS | Indicator condition signals; MIP/MIS → μPROG |
| Primary-to-secondary fault indication | F | Fault indication; MIP → MIS |
| Secondary-indicator controls | SIμCS | Secondary-indicator micro-program control signals; μPROG → MIS |
| Parity-check conditions | PCS | Parity-check condition signals; PGC → μPROG |
| Internal parity state | PS | Parity state from MIS → IP, PGC and COMP |
| Incoming store parity | SPB | Store parity bit; through IP → G8, G10 and PGC |
| Stored capability-base parity | BPB | Base parity bit; BASE STK → COMP |
| Stored capability-limit parity | LBP | Limit parity bit; limit stack → COMP; owner-confirmed label (capture originally read LPB) |
| Returned memory parity check | PCW | Parity check wire in SOHCS; memory-returned parity → PGC |
| Historic-selector increment | INC | Increment historic-register selector; μPROG → SELH |
| Store-output-register increment | +1S | Increment SDIREG by one; μPROG → SDIREG |

SF, SELA, SELB, SELL and SELH denote the strapping field and selection circuitry listed above, rather than additional data highways.

## μPROG condition inputs and control outputs — cross-check against US3771146A

**DOCUMENTED OBSERVATION:** [US3771146A](../transcriptions/US3771146A.rtf), “Processor Module Description”, distinguishes the comparator **COMP** from the micro-program control unit **μPROG**. COMP performs comparisons/checks and returns condition signals **CIS**; μPROG receives those and other processor conditions and issues timed control outputs. The following labels belong to the **μPROG interface**; they are not eight inputs and four outputs of COMP.

| Signal | Direction relative to μPROG | Source-supported meaning |
|---|---|---|
| OP=0 (printed also as OP=O) | Input | Operand-register zero indication from detector ZD |
| AC | Input / protection information | Access code monitored by the trap-detection circuitry TD; precise connection and bit-level interpretation at μPROG not established by this inventory |
| SOHCS | Input | Store Output Highway Control Signals from the memory interface, including timing/response information |
| FICS | Input | Fault-indicator condition signals from fault register FIR |
| ICS | Input | Indicator condition signals from primary/secondary indicators PIR/SIR |
| AUCS | Input | Arithmetic-unit condition signals from MILL, including overflow |
| CIS | Input | Comparator condition/indication signals from COMP |
| TI | Listed input; meaning unknown | No source-supported expansion or specific function established in the passages reviewed |
| RSEL | Output | Accumulator/working-register selection (SELA) |
| CRSEL | Output | Capability-register stack selection (SELB/SELL) |
| SIHCS | Output | Store Input Highway Control Signals: outgoing store-operation control and timing |
| GOS | Output | Micro-program-generated offset value for internal address calculations; documented, for example, when addressing a Dump Stack word |

**Supporting descriptions:** US3771146A's “Micro-Program Control Unit” explicitly identifies AUCS, CIS, ICS, FICS and OP=0 as incoming conditions and RSEL/CRSEL/SIHCS as controlled outputs; its “Step S7 — Dump RCP” describes GOS providing a generated Dump Stack offset. The store-highway description establishes SOHCS/SIHCS direction. Access-code monitoring by TD is described separately. This table preserves the observed AC and TI labels without asserting undocumented individual connections.

**WORKING RECONSTRUCTION:** these input conditions can qualify clocked slot selection as well as the generated datapath controls. The existence and broad roles of the signal groups are evidenced; their exact routing into the slot matrix, micro-bit matrix and other μPROG circuitry, and the Boolean equations or individual micro-bit assignments, remain **UNKNOWN**. This inventory does not assume that COMP is the next-slot decision unit or that the mode bit is routed through COMP.

## Source discrepancies and limits of this inventory

The owner manually checked the following against the drawing/text during review of the combined capture:

- The incoming-memory-to-operand path is **G6**; the main-highway-to-operand path is **G17**. The S16 passage beginning “When the memory produces the word RSPC-O” nevertheless prints **G17** after GS. **DOCUMENTED OBSERVATION:** G17 is printed. **WORKING RECONSTRUCTION:** G6 is the intended reference for that incoming-memory transfer, following the drawn path. The passage is on PDF page 16, printed column 18, just above S17.
- The S9 passage beginning “This operation is performed by activating gates” prints **G3, G4 and G6**. **DOCUMENTED OBSERVATION:** that is the printed list. **WORKING RECONSTRUCTION:** G6 is extraneous to the described SSCR → MILL → SDIREG transfer; G3/G4 provide that route, and G5 subsequently sends the address to memory. No replacement printed text is invented.
- The S16 passage prints **PCG**, while the drawing labels the parity circuit **PGC**. Use the functional name above and retain both source spellings as aliases.
- The annotation split beneath μPROG across the sheets reads **μPGCS**.
- The four unnumbered AND gates receive the local editorial identifiers G?1–G?4 listed above.

Individual control wires, microinstruction encoding, enable timing and every bit-level parity/comparator pin assignment are not specified by this inventory. This is a limit of the source-level gate/signal description, not a claim that an established architectural mechanism is missing. No new implementation mechanism or design is proposed.
