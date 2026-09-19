# System 250 Architecture — Working Reconstruction

## Status

This document is the current working reconstruction of the Plessey System 250 architecture.

It is **not primary evidence**. Statements below are derived from source material held or transcribed in this repository. Where the available evidence does not establish semantics, this document records the fact without filling the gap by assumption.

This first pass is deliberately limited primarily to evidence in the *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7.

## Source basis

Primary source used for this revision:

- *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7.
- Repository transcriptions:
  - `transcriptions/System 250 Pocket Reference pg0-pg2 transcription.txt`
  - `transcriptions/System 250 Pocket Reference pg3-pg4 transcription.txt`
  - `transcriptions/System 250 Pocket Reference pg5-pg7 transcription.txt`

The transcriptions themselves warn that ambiguous characters should be checked against the scans before being treated as definitive.

## Architectural word size

The instruction-format diagrams number bits 23 through 0. This establishes a 24-bit instruction/data word representation in the material covered here.

## Instruction formats

The Pocket Reference distinguishes two instruction formats:

### Store mode

The store-mode instruction contains fields labelled:

- FUNCTION
- REG
- MOD
- CAP
- ADDRESS

The exact bit widths and semantics of these fields are not yet stated here because the current ASCII transcription does not preserve the graphical field boundaries with sufficient confidence.

### Direct mode

The direct-mode instruction contains fields labelled:

- FUNCTION
- REG
- MOD
- SIGNED LITERAL

Again, exact field widths should be established from the source image or additional documentation before being specified here.

## Programmer-visible instruction set

Page 3 lists the following assembly commands:

`ADD AND ASH CALL CHP CMP COR CSH DIV EOR JMP JEQ JGT JGE JLT JLE JNE JOV LC LD LDM LDN LDP LSH MOVE MPY OR RET SC SD SDM SUB SWP SWPM`

The table distinguishes store-mode and direct-mode opcodes where applicable and identifies whether an instruction uses a register field, changes LT/EQ indicators, or can set overflow.

The jump family shares store/direct function codes 36/76 and uses the register field to select the condition:

| Register field | Assembly | Condition |
|---:|---|---|
| 0 | JEQ | Equal |
| 1 | JNE | Not equal |
| 2 | JGT | Greater than |
| 3 | JLT | Less than |
| 4 | JGE | Greater or equal |
| 5 | JLE | Less or equal |
| 6 | JOV | Overflow |
| 7 | JMP | Unconditional |

The Pocket Reference records that JOV clears overflow.

### Capability-related instructions

The instruction table explicitly includes:

- `LC` — Load Capability — store 30, direct 70
- `LDP` — Load Pointer — store 31, direct 71
- `SWPM` — Swap Masked — store 32
- `SC` — Store Capability — store 34
- `CALL` — Call — store 10
- `RET` — Return — direct 75

The presence, names and opcodes of these instructions are established by the Pocket Reference. Their full capability semantics are **not established by pages 0–7 alone** and must not be inferred merely from their names.

## Capability access rights

Page 4 contains capability/access-code diagrams for COS and POS.

Both identify six named access rights:

- `EC`
- `WC`
- `RC`
- `ED`
- `WD`
- `RD`

The current evidence therefore uses **ED**, not the generic abbreviation `X`, for the execute-related data access code.

The exact meanings and enforcement rules of all six codes should be documented only when supported by source material. Their names and ordering in the page-4 diagrams are primary evidence.

The diagrams are transcribed as:

- COS capability pointer: `1 1 EC WC RC ED WD RD 0`
- POS access codes: `0 1 1 EC WC RC ED WD RD`

No interpretation of the fixed bits is made in this revision.

## Data and capability registers

The process dump-stack format explicitly saves:

### Capability registers

- C0
- C1
- C2
- C3
- C4
- C5

### Data registers

- D0
- D1
- D2
- D3
- D4
- D5
- D6
- D7

The same dump-stack format separately records C6 and C7 as part of saved execution state. This establishes that C6 and C7 exist, but their architectural roles are not derived here solely from this table.

## Special-purpose CPU registers

Page 7 lists the following special-purpose data registers:

| Register | Description |
|---|---|
| D10 | ABSOLUTE D/S PUSHDOWN POINTER |
| D11 | WATCHDOG TIMER |
| D12 | FIRST FAULT MIF COPY |
| D13 | not described |
| D14 | not described |
| D15 | INTERRUPT ACCEPT REGISTER |
| D16 | not described |
| D17 | INSTRUCTION ADDRESS REGISTER (IAR) |

It also lists special-purpose capability registers:

| Register | Name / description |
|---|---|
| C10 | C(D) DUMPSTACK |
| C11 | C(I) INTERVAL TIMER |
| C12 | C(C) SCT |
| C13 | C(N) NORMAL INTERRUPT BLOCK |
| C14 | not described |
| C15 | not described |
| C16 | not described |
| C17 | not described |

A separate special capability register is shown as:

- `C(S)` — FAULT START-UP BLOCK

The Pocket Reference does not assign C(S) one of the C10–C17 numbers in this table.

## Indicator and fault registers

The source identifies:

- MIP — Primary Indicator Register
- MIF — CPU Fault Indicator Register

The dump-stack description says that its MIF entry is a copy of the CPU Fault Indicator Register.

Page 7 gives internal-mode examples:

- `MIP = 8R400`
- `MIF = 8R100`
- `D1 = 8R62`

The exact notation and decoding of these examples remain to be documented.

## Internal mode addressing

Page 7 provides a 12-bit Internal Mode Addressing diagram, bits 11 through 0.

The diagram provides selections for:

- lower half (limit) / upper half (base) of a capability register;
- one of sixteen capability or data registers;
- data registers;
- capability registers;
- historical registers;
- Primary Indicator Register;
- Fault Indicator Register;
- special capability register C(S);
- Secondary Indicator Register.

Because this is a layout-sensitive diagram, exact bit assignments should be checked against the scan before they are made normative in the architecture specification.

## Process state evidence

The Pocket Reference contains ROS/PDOS process structures and dump-stack layouts. These are operating-system-specific structures, but they expose architectural state.

The common saved state includes C0–C5, D0–D7, a pushdown pointer, watchdog timer register and MIP. Additional saved state varies between COS, POS, ROS and PDOS.

The dump-stack table contains repeated C6, C7 and IAR entries associated with initial, code, block and subroutine state. Their precise CALL/RETURN interpretation is not specified in this revision.

## ROS/PDOS state and internal priority word

Page 5 defines a 24-bit ROS/PDOS state and internal priority word containing fixed bits and fields labelled `x`, `q`, `p`, `c`, `r`, `w`, `f`, `s` and `b`.

The accompanying definitions establish:

- `r = 0`: process currently running on a CPU
- `r = 1`: process not running
- `cccc`: CPU number when `r = 0`
- `ppppp`: 23 minus current priority
- `qqqqq`: 23 minus standard priority
- `x = 1`: process is on the Ready List
- `s = 1`: suspended on WAITFOR
- `f = 1`: process has faulted
- `bb = 00`: double unblocked
- `bb = 01`: unblocked
- `bb = 10`: blocked
- `bb = 11`: double blocked
- `w`: may be 0 or 1

These are ROS/PDOS software/process-state definitions and should not automatically be generalized into processor architecture.

## MOVE instruction

Page 4 gives a specific MOVE example, `MOVE D2 0 D5`, and identifies:

- source capability C2
- source offset D2
- destination capability C3
- destination offset D3
- sumcheck D5
- count D0

It further states that overflow is to be reset before MOVE, that overflow is set on exit by the STATUS wire, that an indicator is set on count, and that a non-zero count means the device is BUSY.

This is retained as established instruction-level evidence, but the complete MOVE mechanism and its relationship to devices/bus transfers require additional documentation.

## Areas deliberately unresolved

The following questions are **not answered by the pages used for this first pass**:

- exact semantics of LDP;
- how capabilities are created;
- how capability rights are reduced or masked;
- exact LC, SC and SWPM semantics;
- full CALL and RET semantics;
- architectural roles of C6 and C7;
- meaning and structure of the SCT referenced by C12/C(C);
- processor cold-start/bootstrap behaviour;
- how C(S), C(N), C(D) and C(I) are populated;
- exact internal-mode address bit assignments;
- exact interpretation of the fixed COS/POS capability/access-code bits;
- relationship between the phrase "privileged system facilities" in the dump-stack notes and the processor protection model.

These should remain open until supported by primary or reliable secondary evidence.

## Evidence discipline

Future revisions should preserve the distinction between:

1. what a primary source explicitly states;
2. conclusions that follow directly from multiple pieces of evidence;
3. historical recollection;
4. architectural inference;
5. unresolved hypothesis.

Where new evidence conflicts with this reconstruction, the conflict should be recorded and investigated rather than silently reconciled.
