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

In **Direct Mode**, no operand is fetched from store. The operand is the signed literal contained in the instruction. When a modifier Data register is specified, the contents of the selected Data register are added to the signed literal to form the operand.

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
ADD D1, D2, 5
```

`D1 ← D1 + D2 + 5`

With no modifier:

```text
ADD D1, 5
```

`D1 ← D1 + 5`

With a zero signed literal:

```text
ADD D1, D2, 0
```

`D1 ← D1 + D2`

**Store Mode**

```text
ADD D1, 10, D2, C3
```

`D1 ← D1 + C3[D2 + 10]`

The contents of D2 are added to 10 to form the offset into the segment identified by C3.

With no modifier:

```text
ADD D1, 10, C3
```

`D1 ← D1 + C3[10]`

### 3.3 Shift Instructions

### 3.4 Data Transfer and Store Instructions

### 3.5 Conditional and Unconditional Control Transfer

### 3.6 Capability Operations

### 3.7 Subroutine and Process Control
