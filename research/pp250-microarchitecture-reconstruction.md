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
2. LC capability read and expansion;
3. SC capability store;
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

## 13. Immediate research method

When a new internal name or control appears:

1. record the literal source wording and provenance;
2. search all existing transcriptions for the same name and plausible variants;
3. distinguish engineering-interface state from software-visible/Internal Mode state;
4. correlate it with architectural effects without assuming causation;
5. add a flow only when ordering has evidence;
6. retain contradictions and version differences explicitly.

The aim is eventually to recover enough of the internal machine to explain *how* architectural operations and protected transitions are implemented, while preserving the boundary between documented hardware, reconstruction and hypothesis.
