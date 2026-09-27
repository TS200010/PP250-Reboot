# System 250 Architecture — Instruction Set

## Architectural word size

The instruction-format diagrams number bits 23 through 0. This establishes a 24-bit instruction/data word representation in the material covered here.

## Instruction formats

The Pocket Reference distinguishes two instruction formats.

### Data register D0 and the modifier field

The processor self-test appendix states that the programmer-visible register set contains eight 24-bit Data registers, that **seven** of them can be used as modifiers in Store mode or as the second register in Direct mode, and that the eighth is the **mask register**. The instruction descriptions in the same appendix identify that mask register explicitly as `D(0)`.

The same source describes the modifier register in Store-mode address formation as **optional**.

Taken together, these statements establish the interpretation of the three-bit `MOD` field:

- `MOD = 0` specifies **no modifier**;
- `MOD = 1` through `7` select Data registers D1 through D7;
- D0 is the mask register and is not used as a modifier register.

Thus an unmodified Store-mode reference requires no Data register to contain zero. Similarly, where Direct mode uses the modifier/second-register field, D1–D7 are the available Data registers and the zero field value supplies no register contribution.

### Store mode

The Store-mode instruction contains the fields:

```text
FUNCTION | REG | MOD | CAP | ADDRESS
   6        3     3     3       9
```

`REG` selects the register used by the instruction. Depending on the instruction, this may be a Data register or a Capability register.

`MOD` is optional: zero specifies no modification; values 1–7 select Data registers D1–D7.

`CAP` selects one of the Capability registers C0–C7.

`ADDRESS` is the 9-bit address field.

#### Store-mode address formation

The role of `MOD` in Store mode is established independently of the Pocket Reference format diagram. Levy identifies it as the Data register used as the address modifier or index, and the processor self-test appendix explicitly describes the specified modifier register as optional.

When `MOD` is non-zero, the contents of the selected Data register are added to `ADDRESS`:

```text
offset = ADDRESS + D[MOD]
```

When `MOD` is zero:

```text
offset = ADDRESS
```

The resulting offset is used through the Capability register selected by `CAP`:

```text
memory address = C[CAP].base + offset
```

The store reference is subject to the bounds and access rights of the selected Capability register.

Thus the significant Store-mode relationship is:

```text
ADDRESS + optional D[MOD]
        |
        v
offset within the segment designated by C[CAP]
        |
        v
store operand
```

This is established architecture.

### Direct mode

The Direct-mode instruction contains the fields:

```text
FUNCTION | REG | MOD | SIGNED LITERAL
   6        3     3          12
```

There is no `CAP` or `ADDRESS` field. Direct mode therefore does not make the ordinary Store-mode operand reference through a Capability register.

`REG` selects the register used by the instruction.

`SIGNED LITERAL` is a 12-bit signed literal.

The remaining question is the complete meaning of `MOD`.

#### Established zero-literal behaviour

Levy provides an important additional fact about Direct mode: when `SIGNED LITERAL` is zero, `MOD` identifies the second register of a two-register instruction.

For Direct-mode `LD`, therefore:

```text
LD D1 D2 0
```

means:

```text
D1 = D2
```

This register-to-register behaviour is established evidence, not reconstruction.

We therefore know:

```text
Direct mode, SIGNED LITERAL = 0:

    operand = D[MOD]
```

for an ordinary Data-register operation such as `LD` and a non-zero `MOD` field.

#### Non-zero-literal reconstruction hypothesis

England's Figure 7 now supplies a real executable-source example of a non-zero signed literal in Direct mode:

```text
LSH D3 -1
```

This establishes that the assembler accepts the literal-only form with no modifier contribution. Figure 7 also supplies register-only Direct-mode examples:

```text
OR  D1 D3
EOR D1 D3
```

These establish the complementary assembler form in which the second Data register is supplied and the literal contribution is zero.

The general interpretation remains:

```text
operand = D[MOD] + SIGNED LITERAL
```

with either contribution omitted in source syntax when zero/not required. The established zero-literal case follows naturally:

```text
SIGNED LITERAL = 0

operand = D[MOD] + 0
        = D[MOD]
```

For Direct-mode `LD`, the general rule is therefore:

```text
D[REG] = D[MOD] + SIGNED LITERAL
```

For example:

```text
LD D1 D2 0
```

gives:

```text
D1 = D2
```

while:

```text
LD D1 D1 1
```

gives:

```text
D1 = D1 + 1
```

The remaining evidential gap is narrower than before: Figure 7 independently demonstrates a non-zero modifier/register contribution and a non-zero literal contribution, but does not itself contain an instruction in which both are non-zero simultaneously.

#### Consistency of MOD across Store and Direct modes

The Direct-mode interpretation is entirely consistent with the established use of `MOD` elsewhere in the instruction set.

In Store mode:

```text
modified value = ADDRESS + D[MOD]
```

In Direct mode:

```text
modified value = SIGNED LITERAL + D[MOD]
```

Thus `MOD` has the same underlying role in both instruction formats: when non-zero it selects the Data register whose contents are added to the value supplied by the instruction. A zero `MOD` value specifies no register contribution.

What differs is the use made of the result:

```text
STORE MODE

ADDRESS + optional D[MOD]
        |
        v
offset for store access through C[CAP]
        |
        v
store operand


DIRECT MODE

SIGNED LITERAL + optional D[MOD]
        |
        v
operand supplied directly to the instruction
```

This also provides a straightforward hardware interpretation: the same modification/addition mechanism could be used by both instruction formats, with its result directed either to capability-relative store addressing or directly to the instruction's operand path. That hardware interpretation is inferential, but it is consistent with the instruction formats.

The terminology `MOD` is likewise consistent with this interpretation: in both modes it identifies the Data register used to modify the value contained in the instruction when modification is specified.

### Evidential status

The present position is therefore:

```text
D0:
    mask register; not a modifier register
        ESTABLISHED

MOD = 0:
    no modifier contribution
        ESTABLISHED

Store mode:
    ADDRESS + optional D[MOD]
        ESTABLISHED

Direct mode, literal = 0 and MOD != 0:
    MOD supplies the second register
        ESTABLISHED

Direct LD D1 D2 0:
    D1 = D2
        ESTABLISHED

Direct mode, MOD = 0 and non-zero literal:
    literal supplies the operand contribution
        ESTABLISHED BY ENGLAND FIGURE 7

Direct mode, MOD != 0 and non-zero literal simultaneously:
    operand = D[MOD] + SIGNED LITERAL
        ARCHITECTURALLY CONSISTENT; NO FIGURE 7 EXAMPLE OF BOTH NON-ZERO
```

Figure 7 therefore materially strengthens the Direct-mode reconstruction: real System 250 source demonstrates both the register-only and signed-literal-only forms. The only remaining source-evidence question is whether a surviving example can be found with both a non-zero modifier and a non-zero literal in the same Direct-mode instruction.

### Historical assembler source syntax

England's Figure 7, "Examples of Commands", is currently the strongest primary example in the repository of actual System 250 assembly source. It establishes that assembler source notation must be distinguished from binary instruction-field order.

The Figure 7 binary-search example includes:

```text
        LD   D1  0
LOOP:   OR   D1  D3
        CMP  D2  TBL  D1  C1
        JEQ  FINISH
        JGT  UPPER
        EOR  D1  D3
UPPER:  LSH  D3  -1
        JNE  LOOP
        CMP  D2  TBL  D1  C1
FINISH: RET
```

For the demonstrated Store-mode `CMP`, the machine encoding fields are:

```text
FUNCTION | REG | MOD | CAP | ADDRESS
```

but the historical assembler writes the operands as:

```text
CMP REG ADDRESS MOD CAP
```

Thus:

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

and denotes a comparison of D2 with the store operand addressed at:

```text
C1.base + TBL + D1
```

subject to the normal C1 bounds and access checks.

Figure 7 also establishes:

- colon-terminated symbolic labels, e.g. `LOOP:`, `UPPER:`, `FINISH:`;
- symbolic branch operands, e.g. `JEQ FINISH`, `JGT UPPER`, `JNE LOOP`;
- symbolic Store-mode address operands such as `TBL`;
- register-only Direct-mode forms such as `OR D1 D3` and `EOR D1 D3`;
- literal-only Direct-mode forms such as `LD D1 0` and `LSH D3 -1`;
- zero-operand source form `RET`.

The assembler therefore permits fields whose contribution is absent/zero to be omitted in the source notation. This does not imply that the underlying machine fields cease to exist. In particular, the full Direct-mode semantic form remains useful and valid for describing instructions such as:

```text
LD D1 D1 1
```

meaning:

```text
D1 = D1 + 1
```

No change is made here to CALL or CHP source syntax because Figure 7 does not contain examples of those instructions.

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
