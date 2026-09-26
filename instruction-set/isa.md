# PP250 Instruction Set Architecture

## 1. Architectural State

The PP250 is a **24-bit machine**.

### Registers

| Register class | Number | Width | Designation |
|---|---:|---:|---|
| Data registers | 8 | 24 bits | D0–D7 |
| Capability registers | 8 | 48 bits | C0–C7 |

D0 is the **mask register**. It is not used as a modifier register. D1–D7 may be used as modifier registers.

### Register Notation

Where an instruction field selects a register, a subscript identifies the selected register.

| Notation | Meaning |
|---|---|
| D_REG | Contents of the Data register selected by `REG` |
| D_MOD | Contents of the Data register selected by `MOD` |
| C_CAP | Capability register selected by `CAP` |

Thus, for example, D2 denotes the contents of Data register D2.

### Indicators

The PP250 has three indicators.

| Indicator | Meaning |
|---|---|
| LT | Less Than |
| EQ | Equal |
| OV | Overflow |

LT and EQ describe the result of instructions which update them. LT is set when the result is less than zero and cleared otherwise. EQ is set when the result is zero and cleared otherwise.

The conditional jump instructions test these indicators. `JEQ` and `JNE` test the EQ condition. `JLT`, `JLE`, `JGT`, and `JGE` use the LT and EQ conditions. `JOV` tests the OV condition.

---

## 2. Instruction Formats

Each instruction occupies one **24-bit word**.

The PP250 provides two instruction modes: **Direct Mode** and **Store Mode**.

In **Direct Mode**, no operand is fetched from store. The operand is formed from the signed literal contained in the instruction and, when a modifier Data register is specified, the contents of that selected Data register. Thus the register contribution and literal contribution are independently optional.

In **Store Mode**, the operand is fetched from store. The instruction identifies a Capability register, an address, and optionally a modifier Data register. When a modifier is specified, the contents of the selected Data register are added to the address to form an offset. The offset is used to access the segment identified by the Capability register and fetch the operand.

The two modes use different 24-bit instruction formats.

The instruction mode is encoded by the most significant bit of the 6-bit `FUNCTION` field, which is also the most significant bit of the 24-bit instruction word. A clear bit specifies Store Mode and a set bit specifies Direct Mode. The mode bit is nevertheless part of the instruction function code; it is not a separate instruction field. Corresponding Store- and Direct-mode function codes therefore differ by `8r40`.

### MOD Field

`MOD` is a 3-bit field.

| MOD | Meaning |
|---:|---|
| 0 | No modifier |
| 1–7 | Selects Data register D1–D7 respectively |

D0 is not selected by the `MOD` field.

### Direct Mode

```text
+----------+-------+-------+----------------+
| FUNCTION |  REG  |  MOD  | SIGNED LITERAL |
+----------+-------+-------+----------------+
     6        3       3            12
```

| Field | Width | Meaning |
|---|---:|---|
| FUNCTION | 6 | Instruction code |
| REG | 3 | Selects the register used by the instruction |
| MOD | 3 | `0` for no modifier; `1–7` select D1–D7 |
| SIGNED LITERAL | 12 | Signed literal contained in the instruction |

The Direct-mode operand is:

`operand = D_MOD + SIGNED LITERAL`

When `MOD = 0`, there is no modifier and the operand is the signed literal.

This is the adopted ISA interpretation. England Figure 7 independently demonstrates the register-only form (`OR D1 D3`, `EOR D1 D3`) and the literal-only form (`LD D1 0`, `LSH D3 -1`). A surviving historical example in which both `MOD` and `SIGNED LITERAL` are non-zero simultaneously is still sought.

### Store Mode

```text
+----------+-------+-------+-------+---------+
| FUNCTION |  REG  |  MOD  |  CAP  | ADDRESS |
+----------+-------+-------+-------+---------+
     6        3       3       3        9
```

| Field | Width | Meaning |
|---|---:|---|
| FUNCTION | 6 | Instruction code |
| REG | 3 | Selects the register used by the instruction |
| MOD | 3 | `0` for no modifier; `1–7` select D1–D7 |
| CAP | 3 | Selects Capability register C0–C7 |
| ADDRESS | 9 | Address |

The Store-mode operand is:

`operand = C_CAP[D_MOD + ADDRESS]`

When `MOD = 0`, there is no modifier and the operand is:

`C_CAP[ADDRESS]`

The store reference is subject to the bounds and access rights of C_CAP.

### Historical Assembler Source Notation

The binary instruction-field layout and assembler source operand order are not the same thing.

For Store mode, the machine encoding is:

```text
FUNCTION | REG | MOD | CAP | ADDRESS
```

England Figure 7 demonstrates historical assembler source order as:

```text
OP REG ADDRESS MOD CAP
```

For example:

```text
CMP D2 TBL D1 C1
```

maps to:

```text
REG     = D2
ADDRESS = TBL
MOD     = D1
CAP     = C1
```

and addresses the Store operand at:

```text
C1.base + TBL + D1
```

subject to C1's normal bounds and access checks.

Figure 7 also establishes colon-terminated labels (`LOOP:`, `UPPER:`, `FINISH:`), symbolic branch operands, symbolic Store addresses, register-only Direct forms, literal-only Direct forms, and the zero-operand source form `RET`.

Where an omitted field would contribute zero, the historical source can use an abbreviated form. The exact historical spelling of every possible abbreviation is not yet established; in particular, a Store-mode source form with `CAP` present but `MOD` omitted is not demonstrated by Figure 7 and should not be treated as established assembler syntax merely from the machine semantics.

### England Figure 7 — Complete Code Example

The following complete example is transcribed from D. M. England, *Architectural Features of System 250*, Figure 7, "Examples of Commands". It is retained here as a primary validation specimen for reconstruction of the assembler and command language. The comments are included because they directly document the intended register contents and instruction behaviour.

```text
------
LOGIN SMITH JOB15
ASM BINSEARCH (
    .. BINARY SEARCH SUBROUTINE
    INPUT:  C1 CONTAINS CAPABILITY FOR BLOCK
            D2 CONTAINS PATTERN TO BE MATCHED
            D3 CONTAINS ADDRESS OF MIDDLE ELEMENT
    INDICATORS: ZERO SET IF MATCHED
                NONZERO IF NOT MATCHED

        LD  D1  0               .. SET START ADDRESS
LOOP:   OR  D1  D3              .. OR IN TRIAL INCREMENT
        CMP D2  TBL D1  C1      .. COMPARE WITH ELEMENT
        JEQ FINISH               .. JUMP OUT IF MATCH
        JGT UPPER                .. JUMP IF UPPER HALF
        EOR D1  D3              .. LOWER HALF - REMOVE INCREMENT
UPPER:  LSH D3  -1              .. SHIFT INCREMENT RIGHT ONE PLACE
        JNE LOOP                 .. REPEAT IF NOT FINISHED
        CMP D2  TBL D1  C1      .. SET INDICATORS
FINISH: RET
        )

PRINT OLDFILE

BLOCK FRED 615 RWD

PROCESS ANALYSER CPX CODEX

AT JOE 3 ( PRINT FRED (2*P)
    N:=N-1)
IF N<0 THEN ( REMOVE JOE 3) )

LOGOUT
```

This specimen should ultimately be used as an assembler acceptance/validation case. The assembly portion in particular must be accepted according to historical source syntax rather than a modernised notation invented by the reconstruction project.

---

## 3. Instruction Set

### 3.1 Instruction Summary

The instruction summary will be populated from the Pocket Reference instruction table.

### 3.2 Arithmetic and Logical Instructions

#### 3.2.1 ADD — Add

##### 3.2.1.1 Encoding

For `ADD`, `REG` selects a Data register.

**Direct Mode**

```text
8r46 | REG | MOD | SIGNED LITERAL
```

**Store Mode**

```text
8r06 | REG | MOD | CAP | ADDRESS
```

##### 3.2.1.2 Operation

`D_REG ← D_REG + operand`

##### 3.2.1.3 Modes

| Direct Mode | Store Mode |
|---|---|
| Yes | Yes |

##### 3.2.1.4 Indicators

| Indicator | Effect |
|---|---|
| LT | Set if the result is less than zero; otherwise cleared |
| EQ | Set if the result is zero; otherwise cleared |
| OV | Set if the addition overflows; otherwise cleared |

##### 3.2.1.5 Examples

**Direct Mode**

```text
ADD D1 D2 5
```

`D1 ← D1 + D2 + 5`

With no modifier:

```text
ADD D1 5
```

`D1 ← D1 + 5`

With a zero signed literal, historical source syntax can omit the zero literal:

```text
ADD D1 D2
```

`D1 ← D1 + D2`

The corresponding machine encoding has `SIGNED LITERAL = 0`.

**Store Mode**

```text
ADD D1 10 D2 C3
```

`D1 ← D1 + C3[D2 + 10]`

The contents of D2 are added to 10 to form the offset into the segment identified by C3.

With no modifier, the machine semantics are:

```text
D1 ← D1 + C3[10]
```

The exact abbreviated historical assembler spelling for a Store-mode instruction with `CAP` present and `MOD` omitted is not yet established by Figure 7, so no unevidenced source spelling is asserted here.

### 3.3 Shift Instructions

### 3.4 Data Transfer and Store Instructions

### 3.5 Conditional and Unconditional Control Transfer

### 3.6 Capability Operations

### 3.7 Subroutine and Process Control
