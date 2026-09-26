# System 250 Architecture — Working Reconstruction

## Status

This document is the current working reconstruction of the Plessey System 250 architecture.

It is **not primary evidence**. Statements below are derived from source material held or transcribed in this repository. Where the available evidence does not establish semantics, this document records the fact without filling the gap by assumption.

This first pass is deliberately limited primarily to evidence in the *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7, with additional contemporary Plessey papers and patent material where explicitly identified below.

## Source basis

Primary sources used for this revision include:

- *System 250 Pocket Reference Book*, Issue 1, May 1976, pages 0–7.
- D. Halton, *Hardware of the System 250 for Communication Control* (1972).
- D. M. England, *Architectural Features of System 250*, Figure 7, "Examples of Commands".
- Contemporary Plessey capability-register, interrupt, and store-allocation patent material.
- Repository transcriptions under `transcriptions/`.

The transcriptions themselves warn that ambiguous characters should be checked against the scans before being treated as definitive.

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
| `11` | Active store-segment capability — refers through the System Capability Table (SCT) to a segment represented by an SCT identity |
| `10` | Passive/backing-store segment capability — represents a segment in backing store rather than an immediately usable active SCT reference |
| `01` | Resource capability — represents a non-store logical/system resource |
| `00` | Null capability — no usable capability/authority |

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
    = physical realisation and current state of that segment identity

loaded capability register
    = physical base/bounds + access authority,
      or a distinguished unusable/trap representation
```

Consequently, relocating a segment need not require rewriting every stored capability that designates it: its SCT identity can remain stable while the SCT entry is changed.

### Entry structure

A normal SCT entry occupies **three 24-bit words**:

| Entry word | Contents |
|---:|---|
| 0 | Sum-check / validity word |
| 1 | Base |
| 2 | Limit/extent plus additional access/spare-bit capacity used by later system mechanisms |

Halton states that capability loading uses the System Capability Table and that the table contains a sum-check formed from the base and limit values. The patent material describes the corresponding three-word descriptor access and validation sequence.

Later Plessey store-allocation/deallocation patent material establishes that the third SCT word has spare capacity in its access-code portion. Two such bits are used by the described garbage-collection mechanism as **GARBAGE** and **VISITED** bits. This is concrete primary-source evidence for additional SCT flag/state bits and is likely the basis of later descriptions referring to special SCT flags.

The ordinary capability access rights are **not supplied by the SCT entry**. They originate in the stored capability and are combined with the base/limit information obtained through the SCT to form the loaded capability-register representation.

### Sum-check, relocation and unavailable segments

The sum-check protects the integrity of the base/limit descriptor during capability loading. Patent material describes zeroing the check word as a mechanism for making an SCT entry temporarily unavailable while its descriptor is being changed, including relocation.

The interrupt patent further establishes an important distinction: encountering such an unavailable SCT state during capability loading need not immediately execute the whole software recovery action. The capability-loading machinery can establish a distinguished **unusable/trap representation** in the destination capability register. Hardware detects that state when the capability register is subsequently used and enters the normal interrupt/change-process machinery.

Thus the mechanism is approximately:

```text
stored active capability (11)
        |
        v
      LC / SCT lookup
        |
        +-- valid descriptor --> normal expanded C register
        |
        +-- unavailable state --> distinguished unusable C-register state
                                      |
                                      v
                              attempted use of C register
                                      |
                                      v
                              hardware trap detection
                                      |
                                      v
                                   C(N)
                                      |
                                      v
                           Normal Interrupt Block
                                      |
                                      v
                         target Dump Stack capability
                                      |
                                      v
                              automatic CHP
                                      |
                                      v
                         Normal Interrupt process
```

This is more precise than saying simply that "LC page-faults": capability loading establishes the protected state that causes the later attempted use to trap.

### C(N), the Normal Interrupt Block, and automatic CHP

The normal operational trap path is distinct from the fault/start-up path through `C(S)`. `C(N)` / `C13` designates the **Normal Interrupt Block (NIB)**. The NIB contains the capability pointer used to identify the Dump Stack of the Normal Interrupt process. For an automatic change process there is no explicit CHP instruction supplying an incoming-process operand, so this NIB pointer supplies the target required by the CHP machinery.

The architectural chain is therefore:

```text
normal interrupt / trap condition
        |
        v
      C(N)
        |
        v
Normal Interrupt Block
        |
        v
incoming Dump Stack capability/pointer
        |
        v
automatic CHP
        |
        v
Normal Interrupt process
```

The Dump Stack then supplies the ordinary process state restored by CHP, including the capability and data registers and the C6/C7/IAR execution context. `C(N)` does **not** itself contain C6/C7; it supplies the route to the process Dump Stack from which the process context is restored.

### Establishing C(N): SPECIAL and the C(S)-rooted startup chain

The Pocket Reference places MIP (Primary Indicator Register) at Dump Stack offset octal `20`, in the common hardware process-state area. Patent material establishes that CHP saves/restores the primary indicator state and identifies **SPECIAL** as a one-instruction primary-indicator state which permits an `LC` instruction to address the corresponding special-purpose capability register rather than the ordinary C-register bank. `LC` still has its ordinary direction: a stored capability is read and loaded into the selected capability register.

This yields the current reconstructed bootstrap:

```text
C(S) hardware-rooted fault/start-up state
        |
        v
checkout / startup machinery
        |
        v
first legitimate process image
(including MIP at Dump Stack offset 20)
        |
        v
CHP restores MIP with SPECIAL available
        |
        v
one LC loads C(N)
        |
        v
normal interrupt callback path established
```

This is recorded as **reconstructed architecture** under the repository's observation → constraint → reconstruction method. The surviving material establishes the components independently: C(S) roots fault/start-up; CHP restores process indicator state; MIP is in the Dump Stack; SPECIAL redirects one LC to the special capability-register bank; and C(N) is required for normal automatic interrupt entry. Taken together they make C(S) the ancestry of the authority/state by which the running system establishes C(N), rather than requiring an unexplained second root of processor authority.

This does not mean ordinary normal interrupts traverse the destructive C(S)/checkout path. C(S) establishes the initial trusted running state; C(N), once established, is the normal operational entry path.

### Trap discrimination and storage management

The Normal Interrupt process can recover information about the capability/reference responsible for the suspended operation from the saved process state/dump stack and use its form/type to discriminate the required software action.

The contemporary patent descriptions distinguish the broad cases:

- active (`11`) capability whose SCT representation is unavailable: segment/store-management handling;
- passive/backing-store (`10`) capability: page-changing/disc handling is required;
- resource (`01`) capability: resource/I/O handling;
- null (`00`) capability: null/trap handling rather than usable authority.

This provides a capability-native virtual-store path rather than a conventional privileged page-fault handler. Hardware detects an unusable authority state and performs the protected process transition; ordinary capability-constrained system processes determine the reason and perform storage management.

A reconstructed page-in path is therefore:

```text
faulting process
      |
      | attempts to use unavailable/passive authority
      v
hardware detects trap representation
      |
      v
C(N) -> NIB -> incoming Dump Stack
      |
      v
automatic CHP to Normal Interrupt process
      |
      | inspect saved offending reference/state
      v
store/page-management process
      |
      +-- determine required segment
      +-- obtain/allocate main-store space
      +-- arrange disk-to-store transfer
      +-- install/update SCT physical descriptor
      +-- establish valid SCT check/state
      v
I/O proceeds / completes
      |
      v
scheduler can make waiting process runnable
      |
      v
original operation can be retried/resumed
```

England's system description identifies the store-management package as responsible for moving blocks between backing store and main store, and Plessey allocation patent material describes disk-to-main-store transfer being initiated by an I/O-handler process, proceeding asynchronously, and completion feeding back into scheduling of waiting processes.

The important architectural point is that this does **not** require a permanently privileged supervisor execution mode. The hardware supplies protected state detection and process transition; the higher-level storage policy is implemented by capability-controlled software.

### Active versus passive representation

Do not equate "segment is not currently resident" with "every reference to it has type `10`." An active (`11`) reference identifies an SCT entry and can remain meaningful while the SCT entry is unavailable. A passive (`10`) representation instead carries backing-store identity/address information and is used when authority itself has been converted to its backing-store/outform representation.

When capability-containing blocks are moved between main store and backing store, embedded capability representations may therefore require inform/outform conversion. The SCT identity mechanism allows active references elsewhere in main store to remain stable across relocation/page movement.

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

The source identifies MIP (Primary Indicator Register) and MIF (CPU Fault Indicator Register). MIP is saved at Dump Stack offset octal `20`; the dump-stack description says its MIF entry is a copy of the CPU Fault Indicator Register in the OS-specific portion where present.