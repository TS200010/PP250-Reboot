# PP250 microarchitecture reconstruction

**Status:** Working reconstruction  
**Date:** 2026-09-30

## Purpose

Collect and correlate surviving evidence about the PP250 processor below the architectural instruction level: microprogram sequencing, slots, internal registers and highways, indicator/control state, capability handling, fault/process machinery, and the engineering interface.

This note is deliberately evidence-led. Names are retained exactly where their meanings are not yet established. It must not turn suggestive signal names into emulator behaviour without corroborating evidence.

This is also distinct from the M/H/T architectural reconstruction. **M is not synonymous with the microprogram.** The microprogram is an implementation mechanism. Some mechanisms reconstructed architecturally as M may be implemented by microprogram state, sequencing and gating, while ordinary H/T instructions are also implemented by the same lower-level machinery.

## 1. Evidence base

Current primary material in the repository includes:

- *System 250 Pocket Reference*, especially pages 8–10: MIF/MIP/MIS indicator names and CPU-display/engineering controls.
- *Processor Self-Test Program*: description of the MPS microprogram-level simulator, microinstruction/slot timing, control and data areas, fault injection, and slot/instruction tracing.
- Other processor patents, papers and transcriptions should be incorporated only as individual mechanisms are investigated.

Pocket-reference transcriptions retain their scan-verification caveat.

## 2. Established execution hierarchy

**DOCUMENTED OBSERVATION:** the Pocket Reference distinguishes `SINGLE SLOT` from `SINGLE INSTRUCTION`. Its CPU-display miscellaneous register also provides `STOP AFTER 'N' SLOTS`, `STOP ON SLOT 'N'`, and `INHIBIT MICROPROG. DECODE`. Page 8 names `Microprogram O/F` and `Inhibit Slot Decode`.

**DOCUMENTED OBSERVATION:** the processor self-test paper describes MPS as a CPU simulation at microprogram level in which every register, highway, microbit and control gate is accurately simulated. It reports approximately 250 microinstructions versus 70 PP250 instructions per second and says tracing may occur after every slot or every instruction.

**RECONSTRUCTION:** architectural instructions are implemented by a lower-level microprogrammed execution machine organised into slots. The exact relationship among the paper's terms *microinstruction*, *slot*, microbits and complete architectural instruction remains to be reconstructed carefully rather than assumed from modern terminology.

A useful investigation hierarchy is:

```text
architectural instruction
        |
        v
microprogram sequence
        |
        v
slot / microinstruction activity
        |
        v
control signals and internal transfers
        |
        v
register/highway state changes
        |
        v
architectural result or protected transition
```

The diagram is an investigation framework, not yet a recovered PP250 control-flow specification.

## 3. Slot timing and conditional sequencing

**DOCUMENTED:** the self-test paper describes a synchronous machine in which control signals are applied during a clock-high interval and new data values are fixed into memory elements at the negative-going edge.

For the paper's sequence around Slot `I`:

1. results from Slot `I-1` determine GOTO conditions and the identity of Slot `I+1`;
2. inputs to the Control Register are prepared for Slot `I+1`, with conditional inputs calculated from data-area state established at the end of Slot `I-1`;
3. current Control Register signals execute Slot `I` in the data area;
4. fault interrupt is forced if required; otherwise the Control Register takes its prepared inputs and sequencing advances.

The paper explicitly warns that a serial simulator must preserve this timing because parallel hardware transfers and conditional control otherwise produce different results.

This is valuable for future reconstruction: a named state bit observed during a slot cannot automatically be interpreted as a combinational consequence of the state being written during that same slot.

## 4. Internal names exposed by the CPU display

Pocket Reference pages 9–10 expose the following display-address names. Their presence is documented; their expansions and precise functions are **UNKNOWN unless independently established**.

| Display address | Name(s) |
|---|---|
| 401 | MISCELLANEOUS REG |
| 402 | IAR |
| 404 | OUT REG |
| 420 | UPAC, MOD, SAR, REG/SLOT 'N' |
| 440 | UPAL, AAD, CAD, CADCH |
| 500 | LS H0 |
| 600 | MS H0 |
| 1004 | LS M0 |
| 1010 | MS M0 |
| 1020 | LS M1 |
| 1040 | MS M1 |
| 1100 | MONITOR POINTS |
| 1200 | MONITOR POINTS |
| 2001–2020 | UPB0–UPB4 |
| 2040 | UPAN/UPA |
| 4001–4020 | UPB5–UPB9 |
| 4040 | FUN |

**Terminology warning:** historical `H0` in this engineering material must not be identified with the modern reconstruction symbol **H** (authority machine) merely because the letter is the same. Likewise `M0` and `M1` must not be identified with reconstructed **M** without evidence.

Other internal names already encountered elsewhere include `OPP` and `HAD`. Their meanings remain to be established.

## 5. Engineering control interface

Pocket Reference page 10 documents controls including:

- CLOCK;
- SINGLE SLOT;
- SINGLE INSTRUCTION;
- STOP conditions on portions of IAR;
- FORCE H0;
- INHIBIT MICROPROG. DECODE;
- REPEAT;
- STOP AFTER 'N' SLOTS;
- STOP AT FAULT;
- STOP ON SLOT 'N';
- TSCADCH and TSINITO controls/status;
- UPR;
- H. TOGGLE.

This is an engineering/display interface into the processor implementation. It is not presently evidence that normal software possessed equivalent authority, and it must be kept distinct from capability-controlled Internal Mode.

The interface is nevertheless architecturally useful evidence because it makes the instruction/slot distinction externally observable and gives names for internal state that may be correlated with logic diagrams, patents and microprogram descriptions.

## 6. MIF, MIP and MIS

Page 8 names three indicator groups:

- **MIF** — Fault Indicators;
- **MIP** — Primary Indicators;
- **MIS** — Secondary Indicators.

The complete transcription remains the authoritative source for the current names.

Particularly relevant microarchitectural observations include:

### Fault and protection state

Named MIF conditions include:

- Bus Corrupt;
- Interrupt T/O;
- Slave T/O;
- Cap. Parity Fault;
- Sumcheck Fault;
- Base/Limit Fault;
- Interface T/O;
- Parity Comparison;
- Read Data Parity;
- Invalid Operation;
- Power Failure;
- Invalid Control Code;
- Trap with MIP08;
- Hardware Fault 1/2;
- W.D.T. Expired;
- Access Violation;
- bits identifying the capability register on which failure occurred.

These names show that capability integrity, bounds/access checking and fault handling are represented explicitly in processor state. They do not by themselves establish the precise detection or recovery sequence.

### Primary state

Named MIP bits include arithmetic state and controls such as `Inh. Interface Flts.`, `Odd Data Parity`, `1st Attempt`, and `Inhibit Interrupts`. `MIP04 Second Group` remains unresolved; elsewhere in the repository a possible relationship to the second group of special-purpose registers is explicitly only a hypothesis.

MIP is saved in the common Process Dump Stack state, making at least part of it resumable architectural/process state rather than merely transient engineering state.

### Secondary/internal state

Named MIS bits include:

- Microprogram O/F;
- Inhibit Slot Decode;
- Interval Timer Matured;
- Multiply;
- Divide;
- Set Read Capability;
- IAR Decrement;
- Out = Limit;
- Trap;
- Time Up;
- Cycle Intercomplete;
- Move;
- Fault Toggle;
- Fault Link From B.P.W.;
- Dump Process Before Int;
- Internal Mode;
- Cap. Pointer in OPP;
- HAD Increment;
- Even Parity Internal;
- Busy;
- Status.

MIS is exposed through Internal Mode material but is not shown as a saved word in the documented Process Dump Stack layouts. Current repository work therefore treats at least some MIS bits as candidates for transient sequencing/semantic state. This is a **working reconstruction**, not a documented definition of MIS.

## 7. Capability-related microprogram state

Two MIS names are particularly important:

- `MIS08 Set Read Capability`
- `MIS19 Cap. Pointer in OPP`

Together with the self-test evidence that control conditions are carried through slot sequencing, these support investigation of explicit transient capability semantics within the microprogram.

A current **working interpretation** elsewhere in the repository is that `Set Read Capability` may condition a subsequent read as capability-related rather than ordinary data, while `Cap. Pointer in OPP` indicates that OPP may hold a capability pointer whose semantic status must be retained through an internal path.

The exact timing, OPP function, expansion mechanism and control sequence remain **UNKNOWN**. These names must not yet be converted directly into emulator behaviour.

Subsequent source analysis does, however, establish an architectural constraint on that unknown microsequence. For workspace capability registers C0–C5, US3771146A states that the corresponding reserved capability pointer is recorded in the Process Dump Stack whenever the capability register is loaded. The Pocket Reference shows each corresponding Dump Stack location as one 24-bit word. The loaded 48-bit capability register and its associated 24-bit Dump Stack pointer must therefore be treated as related processor/process state, although the exact microprogram sequence by which LC establishes both remains unknown.

## 8. Process, interrupt and fault machinery

The MIS names `Dump Process Before Int`, `Fault Toggle`, `Fault Link From B.P.W.`, `Internal Mode`, `Trap`, `Time Up` and `Cycle Intercomplete` expose candidate internal state associated with protected transitions.

`Dump Process Before Int` is particularly useful independent evidence that process dumping/interrupt entry has explicit processor-level machinery. It does not reveal the complete microprogram sequence.

Future work should correlate these indicators with:

- automatic and explicit CHP;
- normal interrupt entry through C(N);
- fault/start-up through C(S);
- Dump Stack save/restore;
- CALL/RET;
- watchdog and interval-timer behaviour.

Until such correlations are evidenced, this document should record observations rather than invent ordered flows.

## 9. Control area and data area

**DOCUMENTED:** the self-test paper explicitly distinguishes a processor **control area** and **data area**.

The simulator models registers, highways, microbits and control gates. Faults can be forced in both areas. The paper notes that some control-area faults operate at microprogram level but are observable only through instruction-level effects.

This distinction may provide a useful vocabulary for reconstructing internal flows:

```text
control area
   |
   | microbits / control signals
   v
data area
   |
   | register and highway transfers
   v
architectural state / result
```

This is a descriptive synthesis of the paper, not a claim that all PP250 governance maps neatly onto the control area.

## 10. Relationship to M/H/T

The microarchitecture evidence strengthens, but does not prove, the M/H/T reconstruction.

The safe relationship is:

```text
M = architectural/theoretical governing relation
microprogram = implementation machinery

some M mechanisms
       |
       v
may be realised by
microprogram sequencing/state/gating

but

microprogram != M
```

Ordinary arithmetic and instruction execution also use the microprogram. The research objective is therefore to identify which recovered microprogram mechanisms enforce capability semantics and legitimate state transitions, rather than relabelling the whole microprogram as M.

## 11. Reconstruction targets

As evidence accumulates, attempt evidence-qualified reconstructions of:

1. architectural instruction dispatch into microprogram sequences;
2. LC capability read, SCT expansion, and retention of the corresponding capability pointer in the Process Dump Stack;
3. SC capability store, including use of the retained capability identity where established by evidence;
4. LDP;
5. CALL;
6. RET;
7. explicit CHP;
8. automatic CHP / normal interrupt entry;
9. fault/start-up transition;
10. Internal Mode access;
11. base/limit and access-right checking;
12. capability-versus-data transfer through internal paths;
13. watchdog and interval-timer handling.

For each reconstructed flow record:

- source observations;
- named internal registers/highways/state;
- slot or timing constraints;
- architectural inputs and outputs;
- which steps are documented;
- which ordering is inferred;
- unresolved branches or names;
- predictions that could be tested against another source.

## 12. Open questions and vocabulary ledger

Maintain unresolved internal names rather than silently expanding them.

| Name | Current status |
|---|---|
| OPP | Internal path/register associated with `Cap. Pointer in OPP`; exact meaning unknown |
| HAD | `HAD Increment` documented; expansion/function unknown |
| UPAC | Displayed internal name; unknown |
| MOD | Displayed internal name; exact role to establish |
| SAR | Displayed internal name; exact role to establish |
| UPAL | Displayed internal name; unknown |
| AAD | Displayed internal name; unknown |
| CAD | Displayed internal name; unknown |
| CADCH | Displayed internal name; unknown |
| H0 | Displayed LS/MS state; must not be confused with reconstructed H |
| M0/M1 | Displayed LS/MS state; must not be confused with reconstructed M |
| UPB0–UPB9 | Displayed internal names; unknown |
| UPAN/UPA | Displayed internal name(s); unknown |
| FUN | Displayed internal name; likely function-related by name only; do not expand without evidence |
| B.P.W. | Appears in `Fault Link From B.P.W.`; unresolved |
| TSCADCH | Engineering toggle; exact meaning unknown |
| TSINITO | Engineering toggle/status; page 10 states set = CPU running, reset = CPU stopped |
| UPR | Engineering control/read name; unresolved |

This ledger should grow as patents, logic material, self-test descriptions and further manuals expose additional state.

## 13. Patent-derived implementation evidence

### 13.1 US3879712 / GB1422952 — microprogram and diagnostic interface

This early PP250 patent is unusually valuable because it describes implementation-level processor structures directly.

**DOCUMENTED:** the processor data area contains:

- register block `REGBLOCK`;
- instruction register `INSTREG`;
- arithmetic and logic unit `MILL`;
- processor-module bus-interface logic `BI/FL`;
- bus-interface/output register `OUTREG`.

The data area is controlled by **data-area manipulation signals (DAMS)** produced by microbits `UPB`. The microprogram area sets/resets the UPB microbits and microprogram-address toggles `UPA`.

**DOCUMENTED:** the microprogram area `μPROG` contains:

- a register of roughly 150 bits for UPA/UPB state;
- decoder `DEC`;
- slot matrix `SM`;
- microbit matrix `MBM`;
- combinational logic `CCL` producing data-area condition signals `DACS`;
- microprogram slot-control clock `CLK`.

The patent states that processor instructions are implemented by microprograms consisting of sequential microinstructions or **slots**. `CLK` advances slots; `UPA` selects the next slot through `SM`; `DACS` conditions slot selection; and the resulting microbits condition execution of the microinstruction.

This supplies a concrete first-order control model:

```text
architectural instruction in INSTREG
              |
              v
          microprogram
              |
       UPA --> SM <--- DACS
              |
              v
             MBM
              |
              v
          UPB microbits
              |
             DAMS
              |
              v
            data area
 REGBLOCK / MILL / BI/FL / OUTREG
```

The diagram is a synthesis of the patent description, not a reproduced patent figure.

#### H0 identification

The same patent's diagnostic mechanism refers to forcing diagnostic data onto **H0**. Together with the Pocket Reference display entries `LS H0`, `MS H0`, this is strong evidence that historical `H0` is an internal data highway rather than an unexplained architectural register.

This historical `H0` remains completely unrelated to the project's modern symbol **H** for the authority machine.

#### Diagnostic controls

The patent independently explains the Pocket Reference controls:

- single slot = one clock pulse;
- single instruction = run to completion of current instruction;
- inhibit microprogram decode = inhibit decode for the next slot from current UPA;
- stop after N slots;
- stop at fault when UPA reaches the fault-entry condition;
- stop at a specified slot by comparing current UPA with slot register `SR`;
- repeat by inhibiting sequencing of the instruction address/control register.

Additional diagnostic names include `MREG`, `REG1`, `REG2`, slot register `SR`, instruction-address comparator `IAC`, slot comparator `SC`, miscellaneous logic `ML`, and diagnostic interface `DI/F`.

This patent should be treated as a priority source for reconstructing the control unit and for interpreting Pocket Reference pages 9–10.

### 13.2 US3814919 — fault microprogram controls

The earlier fault-detection patent explicitly names a microprogram unit `μPROG`.

**DOCUMENTED:** it says the instruction register `IR` holds instruction control-bit fields and applies them to microprogram control. The microprogram unit issues timed/sequenced microprogram control signals `μPGCS` controlling:

- register input/output gates;
- arithmetic unit `MILL` through `AUμS`;
- comparator `COMP` through `CμS`;
- primary-indicator fault bits in `MIP` through `FIS`;
- secondary-indicator condition bits in `MIS` through `SIμCS`.

It can also select registers using `RSEL` and `CRSEL`, step a historical-register address selector using `INC`, increment memory-input register `SDIREG` using `+1S`, and generate memory-access control codes on `SIHCS` according to segment-descriptor type.

Elsewhere this patent describes fault-entry microsequence steps including `S2`, `S10`, `S16` and `S17`. These are potentially our first surviving named fragments of an actual PP250 microprogram flow and should be reconstructed separately against the patent rather than inferred from architectural CHP effects.

### 13.3 US4383297 — later processor datapath and Internal Mode

**VERSION WARNING:** US4383297 is later Wheatley/Andrews material. Its implementation names are valuable evidence about the evolved System 250 processor family but must not automatically be backdated to the May 1976 Pocket Reference processor.

The patent explicitly says the instruction function code `FC` accesses the microprogram unit and other instruction fields condition it.

Its documented instruction-input path includes:

```text
CPU input bus CBI
       |
     BR&T
       |
       +---- SI ----> BFI
       |
      DBIN
       |
      IDM
       |
      BII
       |
      IBL
       |
       +---- FC ----> microprogram unit
       |
     IREG  (address/offset information)
```

Names exposed here include:

- `CBI` — processor input bus;
- `BR&T` — bus receivers and terminators;
- `SI` — status information;
- `BFI` — bus fault indicators;
- `DBIN` — input-data leads/path;
- `IDM` — input data multiplexer;
- `BII` — internal input-bus path;
- `IBL` — instruction buffer;
- `IREG` — instruction register;
- `FC` — instruction function code.

The address/output side includes:

```text
capability/address formation
          |
         MAR
          |
      ACC / BCC
          |
         BIO
          |
         BIF
          |
        BS&C
          |
        BD&T
          |
         CBO
```

where the patent names `MAR` as the address register, `ACC` and `BCC` as capability-related comparators/checking logic, `BIO` as a highway, `BIF` as the bus interface, `BS&C` as bus sequence/control, `BD&T` as bus drivers/terminators, and `CBO` as the processor output bus.

For address formation the patent further names:

- `IMUX` — input multiplexer;
- `BM` — bit manipulator;
- `ALU` — arithmetic unit;
- `BCB` — capability base file;
- `CAPMUX` — capability multiplexer.

For Internal Mode the patent gives an especially useful documented sequence. After forming an Internal Mode address, the address offset is looped back and conveyed to microprogram control through arithmetic-unit condition signals `ALUCS`. The patent then describes register-read transfer paths using `MDOR`, `BIO`, `BII`, `MDIN`, `ALU`, and duplicated data files `ADF` and `BDF`.

This demonstrates that the later processor's Internal Mode is implemented through the ordinary protected address/data machinery plus a microprogram-conditioned internal loopback, rather than by an unrelated privileged instruction path.

### 13.4 Cross-source correlations

The patents now allow several previously isolated Pocket Reference/self-test names to be connected:

| Pocket Reference / self-test clue | Patent correlation | Status |
|---|---|---|
| H0 | US3879712 diagnostic data forced onto H0 | strong identification as internal highway |
| UPB0–UPB9 | US3879712 UPB microbits controlling data-area manipulation | strong functional correlation |
| UPAN/UPA | US3879712 UPA microprogram-address toggles | strong functional correlation; exact Pocket Reference suffixing still to check |
| SINGLE SLOT | US3879712 one clock pulse | documented |
| SINGLE INSTRUCTION | US3879712 run current instruction to completion | documented |
| INHIBIT MICROPROG. DECODE | US3879712 inhibits next-slot decode from current UPA | documented |
| STOP ON SLOT N | US3879712 compares UPA with slot register SR | documented |
| microprogram/slot distinction | US3879712 + processor self-test | independently corroborated |
| MIP/MIS control | US3814919 μPROG directly controls named MIP/MIS signals | documented for that processor generation |

This materially changes the research position: several entries previously retained as opaque engineering names now have direct implementation descriptions in primary patent material.

## 14. Expanded vocabulary ledger

The following patent names should be tracked in addition to the earlier unresolved ledger.

| Name | Evidence-qualified meaning |
|---|---|
| μPROG | microprogram unit/area |
| UPA | microprogram address toggles/state |
| UPB | microbits controlling data-area manipulation |
| SM | slot matrix |
| MBM | microbit matrix |
| CCL | combinational logic producing DACS |
| DACS | data-area condition signals |
| DAMS | data-area manipulation control signals |
| REGBLOCK | processor register block |
| INSTREG / IR | instruction register terminology in early patents; generation/context must be retained |
| MILL | arithmetic and logic unit in early patents |
| BI/FL | processor-module bus-interface logic |
| OUTREG | output/bus-interface register |
| RSEL / CRSEL | register-selection controls from μPROG |
| SDIREG | memory-input register named in US3814919 |
| SIHCS | memory-access control-signal highway |
| MAR | address register in later US4383297 processor |
| BIO | later internal highway |
| BIF | later bus interface |
| IDM | later input data multiplexer |
| IBL | later instruction buffer |
| IREG | later instruction register |
| IMUX | later input multiplexer |
| BM | later bit manipulator |
| CAPMUX | later capability multiplexer |
| BCB | later capability base file |
| ALUCS | later arithmetic-unit condition signals |
| MDIN / MDOR | later internal data registers/paths |
| ADF / BDF | later duplicated data files |
| ACC / BCC | later capability/access checking comparators; exact individual roles to retain from patent figures/text |

## 15. Immediate research method

When a new internal name or control appears:

1. record the literal source wording and provenance;
2. search all existing transcriptions for the same name and plausible variants;
3. distinguish engineering-interface state from software-visible/Internal Mode state;
4. correlate it with architectural effects without assuming causation;
5. add a flow only when ordering has evidence;
6. retain contradictions and version differences explicitly.

The aim is eventually to recover enough of the internal machine to explain *how* architectural operations and protected transitions are implemented, while preserving the boundary between documented hardware, reconstruction and hypothesis.

## Evidence update — 2 October 2026: transfer and bus microarchitecture

**DOCUMENTED OBSERVATION:** US3787818A, Description 48–50 and 63–90, describes channel transfer-stack pointer pairs, channel-owned source/destination capability registers, SCT sumcheck/bounds validation and repeated bounds checks. Special channel registers are processor-writable through a backdoor while offline. This extends the reconstruction to protected autonomous transfer; it does not settle live channel relocation or all permission checks. The canonical account is [I/O and interconnect](../architecture/io-and-interconnect.md).

US4050059A, Description 13 and 16–21, gives the whole-access-unit READ-AND-HOLD, same-bus WRITE/RESET release, 10-microsecond timeout and parity-based hold-integrity check. US4041460A, Description 17–19, adds received-address/inverse monitoring for bus diagnosis. These observations constrain atomic-transfer and diagnostic microsequences; they do not establish a lock covering a whole software relocation transaction. Reset-code wording differs between sources and must remain version-sensitive.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.


## Investigation update — 19 October 2026: instruction entry and clocked slot control

**Status: working reconstruction, not recovered gate wiring.** This section preserves the reasoning behind the current candidate, including rejected alternatives, tests and exceptions. Established processor hardware and its documented signals remain canonically inventoried in [processor control](../architecture/processor-control.md), including the US3771146A μPROG interface cross-check; instruction formats and semantics belong to the existing instruction-set account. Do not treat the proposed slot numbers below as measured PP250 slot assignments.

### A. Evidence and constraints

**DOCUMENTED OBSERVATION:** PP250 instruction execution and single-slot stepping are distinct. The self-test description identifies a clocked, parallel control/data machine, with state and next-slot conditions prepared in relation to clock transitions. US3879712 describes the slot matrix (SM), micro-bit matrix (MBM), combinational control logic (CCL), slot address (UPA), microbits and diagnostic control of slot decode. The engineering display identifies UPAL (previous), UPAC (current) and UPAN (next) slot-address positions; these labels are correlated with the patent mechanism, without a claimed full electrical assignment. The patent also describes a *select next instruction* micro-bit. The microprogram is **hardware control**, not software execution of slot instructions.

**CONSTRAINT:** a completed machine instruction must eventually cause another instruction fetch, with ordinary clocked selection of the next control state. No separate software instruction-fetch loop is implied. A provisional FETCH slot at octal `000` is an attractive working choice, **not documented as slot zero**. The evidence identifies at least 83 slots, so at least seven bits of slot address are required; it does not establish that exactly 128 slots were implemented.

**DOCUMENTED OBSERVATION:** the Pocket Reference gives a 24-bit instruction, with six FUNCTION bits, three REG bits and three MOD bits. The high FUNCTION bit distinguishes Store (0) from Direct (1); the lower five bits identify the base function. Store uses `CAP(3)|ADDRESS(9)`, with optional D[MOD] modification and a capability-relative store access. Direct uses a signed 12-bit literal with optional D[MOD] modification, normally without fetching the operand from store. Store and Direct function codes for the same base operation differ by octal `40`. The Pocket Reference opcode table is transcribed at [pages 3–4](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt); preserve its source-verification qualifications.

### B. How the instruction-operation entry might be selected

The investigation considered several simple mappings from opcode to the first operation-specific slot:

- Directly use five function bits for slots `000–037`: clashes with the proposed FETCH=`000`, since EOR has base function `00`.
- Use base function plus one: separates EOR from FETCH but introduces an increment/translation operation.
- Use a fixed high address bit/prefix: for example octal `040–077` or `100–137`, allowing five opcode bits to be routed directly into the low slot-address bits.
- Give Store and Direct their own 64 entry slots: straightforward, but duplicates entry addressing where both variants can share an operation.
- Use an arbitrary opcode-to-slot decoder: possible, but unnecessary to the simplest candidate; neither a decoder nor its absence is historically established.

**WORKING RECONSTRUCTION:** reserve `000` provisionally for FETCH and select an operation entry by `UPA_operation = 040₈ OR FUNCTION[4:0]`. This covers `040–077` (octal). Examples: EOR `00→040`; LD `03→043`; ADD `06→046`; CALL `10→050`; CHP `11→051`; SC/MOVE base `34→074`; conditional/unconditional jump family base `36→076`. The leading `040` is a **proposed fixed address prefix**, not a recovered microinstruction or proven gate signal.

**CONSTRAINT / PREDICTION:** if this arrangement approximates the original hardware, operation-entry slot selection can use the low five function bits, while Store/Direct remains available as a separate condition. Six base codes are unassigned in the Pocket Reference: octal `14, 20, 21, 25, 27, 37`, mapping under this proposal to `054, 060, 061, 065, 067, 077`. These are *logically unused operation entries*, not independently verified free physical slots or proven expansion provisions. Thirty-two entry addresses need not imply thirty-two different implemented operations or fully independent slot logic.

### C. Why operand preparation was moved ahead of function selection

An earlier candidate decoded the function immediately after FETCH and let each instruction's Store/Direct path acquire its own operand. The improved working candidate factors out the shared addressing-mode work:

```text
FETCH (provisionally slot 000)
  -> interpret Store/Direct mode
      Store:  address modification, capability selection, checks,
              and any required memory transaction
      Direct: form signed-literal operand and optional modifier
  -> select operation entry from the low five FUNCTION bits
  -> execute operation-specific clocked states
  -> final state selects FETCH again
```

This is a **functional sequencing hypothesis**, not a claim of one physical slot per box, a universal fixed-length cycle, or software performing the flowchart. A current slot asserts hardware controls; relevant registered/combinational conditions and the slot-selection mechanism prepare the next state for a clock transition. Micro-bit selection and slot sequencing must respect the self-test paper's timing distinction between data produced in one slot and conditions used in a subsequent selection.

**NECESSARY QUALIFICATION:** “prepare the operand” does **not** universally mean “fetch the operand word”. Store Data and Store Capability require a destination effective address and write controls; control/process operations may need capability/process state and multiple transactions. A better formulation is *prepare the addressing-mode-dependent inputs required by the operation*. Whether the historical machine did this **before** dispatch for every instruction remains unverified. Where a read is needed, its response and checking may extend across multiple slots; write-only instructions should not acquire a gratuitous data read.

### D. Mode dependence, exceptions and the SC/MOVE collision

Ordinary arithmetic/logical instructions provide the strongest reason to share Store/Direct operand preparation before common function execution. For special instructions — SD, SC, CALL, CHP, RET, MOVE and jumps — additional and sometimes fundamentally different control sequences are required. Their required operation and privilege/access effects remain defined by the ISA and capability/process reconstruction, not by this slot hypothesis.

**DOCUMENTED TABLE READING:** Store SC is opcode `34`, while Direct MOVE is opcode `74`: both have base function `34` and would select provisional operation slot `074`. They are not the same operation. A suspected table error and the idea of moving MOVE to an unassigned code were considered, but there is no evidence to justify silently correcting the original entry. Retain the printed codes and test the candidate control model against this collision. This requires either mode-conditioned control **at/after** the common operation entry, or different earlier paths that carry enough state to distinguish the operations. Merely preparing a uniform operand does not explain the collision.

**ALTERNATIVE / UNKNOWN:** Store/Direct might qualify micro-bit matrix outputs, participate in combinational next-slot selection, enter the slot matrix as a condition, or act in more than one of those places. The simplest candidate primarily uses mode to choose operand-preparation states, but exceptions establish that mode may remain relevant after preparation. No actual MBM, CCL or SM input pins, Boolean equations or next-slot address encoding have been recovered for this decision.

### E. Patent condition/control interface and its implications

The independent US3771146A patent description distinguishes comparator **COMP** from microprogram control **μPROG**: COMP performs checks and reports **CIS**; μPROG receives conditions and issues register, store and gate-control signals. The current [processor-control signal table](../architecture/processor-control.md#μprog-condition-inputs-and-control-outputs--cross-check-against-us3771146a) records the evidence for inputs **OP=0, AC, SOHCS, FICS, ICS, AUCS, CIS, TI** and outputs **RSEL, CRSEL, SIHCS, GOS**. It distinguishes direct patent expansions from qualified interpretations: notably **TI** is not expanded, and **AC** is access-code information monitored by trap-detection circuitry, without a proven per-wire route into μPROG.

**DOCUMENTED OBSERVATION:** AUCS carries arithmetic conditions, CIS comparator conditions, ICS indicator conditions, FICS fault conditions, OP=0 the operand-zero condition and SOHCS store-interface responses/controls; RSEL and CRSEL select working and capability registers; SIHCS commands store operations; GOS supplies a microprogram-generated offset in documented Dump Stack address formation. These groups explain how conditioned clocked control could respond to memory, comparison, arithmetic and fault results. **WORKING RECONSTRUCTION:** their information can qualify transitions and datapath actions. Exactly which conditions connect to SM, MBM, CCL or other μPROG subcircuits remains unestablished; do not relabel COMP as a control-decision block.

### F. Remaining tests and limitations

1. Reconcile the explicit select-next-instruction micro-bit, UPAL/UPAC/UPAN transitions and self-test timing with the proposed FETCH entry; do **not** claim slot `000` is observed.
2. Determine whether operand formation and capability checks can always precede operation selection, particularly for store writes, jumps, CALL/CHP/RET and fault/interrupt entry; test for instructions that must branch on function **before** operand acquisition.
3. Test the common `040–077` operation-entry assumption against any documented slot references. Do not treat illustrative Load Data `3,17,27` or Store Data `12,24,16,22` sequences from earlier conversation as verified original mappings.
4. Distinguish the mode-bit influence on operand formation, micro-bit enables and slot transitions using only corroborated conditions; preserve SC/MOVE `34/74` as a discriminating test.
5. Resolve, if possible within existing evidence, the precise meaning of TI and the per-signal AC/memory-response connections; do not invent expansions or circuitry.

The reconstructed hardware has **clocked slots and conditioned selection**, not a software microprogram interpreter. The compact operand-first/common-function-entry design is a **candidate explanation** of that hardware, not evidence that the historical PP250 used precisely these addresses or this factorisation.
