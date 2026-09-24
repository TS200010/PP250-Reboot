# System 250 Architecture — Working Reconstruction

## Status

This document is the current working reconstruction of the Plessey System 250 architecture.

It is **not primary evidence**. Statements below are derived from source material held or transcribed in this repository. Where the available evidence does not establish semantics, this document records the fact without filling the gap by assumption.

This first pass is deliberately limited primarily to evidence in the *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7, with additional contemporary Plessey papers and patent material where explicitly identified below.

## Source basis

Primary sources used for this revision include:

- *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7.
- D. Halton, *Hardware of the System 250 for Communication Control* (1972).
- Contemporary Plessey capability-register patent material transcribed in this repository.
- Repository transcriptions under `transcriptions/`.

The transcriptions themselves warn that ambiguous characters should be checked against the scans before being treated as definitive.

## Architectural word size

The instruction-format diagrams number bits 23 through 0. This establishes a 24-bit instruction/data word representation in the material covered here.

## Instruction formats

The Pocket Reference distinguishes two instruction formats:

### Store mode

The store-mode instruction contains fields labelled FUNCTION, REG, MOD, CAP and ADDRESS.

### Direct mode

The direct-mode instruction contains fields labelled FUNCTION, REG, MOD and SIGNED LITERAL.

## Programmer-visible instruction set

Page 3 lists:

`ADD AND ASH CALL CHP CMP COR CSH DIV EOR JMP JEQ JGT JGE JLT JLE JNE JOV LC LD LDM LDN LDP LSH MOVE MPY OR RET SC SD SDM SUB SWP SWPM`

The jump family shares store/direct function codes 36/76 and uses the register field to select the condition.

### Capability-related instructions

The instruction table explicitly includes `LC`, `LDP`, `SWPM`, `SC`, `CALL`, and `RET`. Full semantics must be established from the wider source corpus rather than inferred from their names alone.

## Church/Turing interpretation

The current research model distinguishes:

```text
H = Church/authority machine
T = Turing/general computational machine
M = governing transition machine

capability = protected authority mechanism/representation,
             orthogonal to the H/T/M decomposition
```

A strong current hypothesis identifies the six special Church instructions as `LC`, `SC`, `LDP`, `CALL`, `RET`, and `CHP`. This remains a historical-source verification item.

## Capability access rights

Page 4 identifies six named access rights: `EC`, `WC`, `RC`, `ED`, `WD`, `RD`.

The diagrams are transcribed as:

- COS capability pointer: `1 1 EC WC RC ED WD RD 0`
- POS access codes: `0 1 1 EC WC RC ED WD RD`

The exact relationship between these OS-specific diagrams and the more general capability-form/type encoding below must be kept version- and source-sensitive.

## Stored capability type/form field

Contemporary Plessey patent material describes the high-order two-bit classification of a stored capability/reference. These two bits determine what kind of protected reference the remaining word represents and therefore what processing is appropriate.

| Two-bit value | Meaning |
|---|---|
| `11` | Active store-segment capability — refers through the System Capability Table (SCT) to a segment currently represented in main-store capability machinery |
| `10` | Passive/backing-store segment capability — represents a segment in backing store rather than an immediately usable active SCT reference |
| `01` | Resource capability — represents a non-store logical/system resource |
| `00` | Null capability — no capability/authority |

These values are architecturally important: the two-bit field is not merely another pair of access permissions. It classifies the form of capability and determines the interpretation of the rest of the stored representation.

The six access rights (`EC WC RC ED WD RD`) are conceptually distinct from this form/type classification. A capability therefore carries both a statement of **what kind of protected reference it is** and, where applicable, **what operations are permitted through it**.

## System Capability Table (SCT)

### Role

The SCT is the system indirection structure used when an active stored capability is expanded into a capability register. A stored active capability carries an SCT reference/index rather than a raw physical base address. The processor uses the SCT reference relative to the special SCT capability register `C(C)` / `C12` to obtain the physical segment bounds.

This gives a useful separation:

```text
stored active capability
    = capability form/type + access rights + SCT identity/reference

SCT entry
    = physical realisation of that segment identity

loaded capability register
    = physical base/bounds + access authority
```

Consequently, relocating a segment need not require rewriting every stored capability that designates it: its SCT identity can remain stable while the SCT entry is changed.

### Entry structure

The current primary-source reconstruction is that a normal SCT entry occupies **three 24-bit words**:

| Entry word | Contents |
|---:|---|
| 0 | Sum-check / validity word |
| 1 | Base |
| 2 | Limit (or segment extent, according to source terminology) |

Halton states that capability loading uses the System Capability Table and that the table contains a sum-check formed from the base and limit values. The patent material describes the corresponding three-word descriptor access and validation sequence.

The access rights are **not supplied by the SCT entry**. They originate in the stored capability and are combined with the base/limit information obtained through the SCT to form the loaded capability-register representation.

Conceptually:

```text
       STORED ACTIVE CAPABILITY
 +-------------------------------+
 | form=11 | rights | SCT ref    |
 +----+--------+----------+-------+
      |        |          |
      |        |          v
      |        |      SCT[reference]
      |        |      +----------------+
      |        |      | sum-check      |
      |        |      | base           |
      |        |      | limit          |
      |        |      +-------+--------+
      |        |              |
      |        +-------+      |
      |                |      |
      v                v      v
   active form     LOADED CAPABILITY REGISTER
                   +-------------------------+
                   | base                    |
                   | access rights + limit   |
                   +-------------------------+
```

### Sum-check and temporary invalidity

The sum-check protects the integrity of the base/limit descriptor during capability loading. Patent material further describes zeroing the check word as a mechanism for making an SCT entry temporarily unavailable while its descriptor is being changed, such as during relocation. A capability load encountering that state does not simply obtain an unchecked descriptor.

Thus word 0 is more than passive error-detection data: at least one distinguished value participates in the segment-state protocol.

### Important caution about “flag bits”

A later secondary description refers to special flag bits associated with this area. At present the stronger contemporary evidence does **not** justify adding unidentified flag fields to the three-word SCT entry. The clearly established classification flags are the two high-order form/type bits of the **stored capability** (`11`, `10`, `01`, `00`) described above. The zero sum-check condition provides a separate SCT-entry state mechanism.

Until a primary-source SCT figure or description establishes additional embedded bits, do not invent extra SCT flag fields in the emulator or architecture specification.

## Data and capability registers

The process dump-stack format explicitly saves C0–C5 and D0–D7, with C6 and C7 separately represented as execution state.

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

The source identifies MIP (Primary Indicator Register) and MIF (CPU Fault Indicator Register). The dump-stack description says its MIF entry is a copy of the CPU Fault Indicator Register.
