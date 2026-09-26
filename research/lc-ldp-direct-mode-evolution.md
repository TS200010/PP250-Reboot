# LC/LDP Direct-Mode Evolution — Work in Progress

**Status:** Research reconstruction; unresolved.  
**Date:** 26 September 2026

## Purpose

Record the source discrepancy and current interpretation concerning the `LC` (Load Capability) and `LDP` (Load Pointer / Load Capability Pointer) instructions between the early PP250 described in the Checkout paper and the May 1976 System 250 Pocket Reference.

This note deliberately distinguishes documented evidence from reconstruction. It should not yet be treated as a settled instruction-set specification.

## 1. Primary evidence: early Checkout description

Leaman, Lloyd and Repton, *The development and testing of a processor self-test program* (received May 1972), describes the PP250 instruction set in an appendix.

For `LC` it gives:

```text
Load capability | LC | Store: YES | Direct: NO |
Capability Register D: loaded with the capability specified by A.
```

For `LDP` it gives:

```text
Load capability pointer | LDP | Store: YES | Direct: NO |
D := Pointer associated with A.
```

Thus the early documented machine has **no direct form of either LC or LDP**.

The same paper states that most instructions may be executed either in store mode, where operands are a register and store address, or direct mode, where operands are a register and a register plus literal. It also states that store addressing is capability-qualified and that base, limit and access-code violations cause a fault.

The phrase `D := Pointer associated with A` is currently the strongest surviving concise description of LDP semantics. It also strongly indicates that the result is held in a D register; a reconstruction in which LDP directly manufactures a new 48-bit capability in a capability register is therefore disfavoured unless later primary documentation explicitly establishes changed semantics.

## 2. Primary evidence: May 1976 Pocket Reference

The System 250 Pocket Reference, Issue 1, May 1976, lists octal instruction codes including:

```text
                     Store   Direct
Load Capability LC     30      70
Load Pointer    LDP    31      71
```

The same table records a register field for both instructions. `LC` does not change LT/EQ; `LDP` does.

The Pocket Reference gives the general instruction layouts as:

```text
Store mode:
FUNCTION | REG | MOD | CAP | ADDRESS

Direct mode:
FUNCTION | REG | MOD | SIGNED LITERAL
```

Therefore, by May 1976, the Pocket Reference unequivocally assigns direct-mode opcodes to **both LC and LDP**.

## 3. The discrepancy

The two primary sources therefore disagree in a systematic way:

| Source | Approx. architecture date | LC store | LC direct | LDP store | LDP direct |
|---|---:|---:|---:|---:|---:|
| Checkout paper | 1972 | YES | NO | YES | NO |
| Pocket Reference Issue 1 | May 1976 | 30 | 70 | 31 | 71 |

This is unlikely to be explained merely by a generic misunderstanding of direct mode because **both related capability/pointer instructions change together**.

### Working interpretation

The strongest current hypothesis is that direct forms of LC and LDP were introduced during architectural evolution between the early PP250 described by Checkout and the System 250 represented by the 1976 Pocket Reference.

This is a hypothesis, not yet a demonstrated historical fact. Possible alternatives include differences between processor/system variants, errors or simplifications in the Checkout appendix, or semantics of the 1976 opcode table not yet understood.

## 4. Security constraint on direct LC

A direct `LC` cannot safely mean simply:

```text
arbitrary literal bits -> capability register
```

if those bits are interpreted as unrestricted capability authority. Such an operation would apparently permit ordinary software to forge capabilities and defeat the architecture's controlled capability mechanism.

Therefore the existence of opcode `70` creates an important reconstruction problem:

> **What does the direct operand of LC denote, and what machinery makes that operation safe?**

The general direct-mode layout alone is insufficient to answer this. Capability-related instructions may give instruction-specific meaning to the fields.

Do not infer from the word `literal` that arbitrary data is thereby converted into authority.

## 5. Current interpretation of LDP

An earlier reconstruction treated the LDP result as a possible index or offset into an array/object. That is not established and should not be used as the primary interpretation.

The stronger historical clue is the Checkout terminology:

```text
Load capability pointer
D := Pointer associated with A.
```

Together with System 250 literature distinguishing capability pointers from the expanded base/limit/access form held operationally in capability registers, this suggests that `pointer` may denote a **compact identity/reference for a capability** rather than an index within the data object protected by that capability.

A useful conceptual model to investigate is therefore:

```text
capability pointer / protected capability identity
                 |
                 | capability machinery
                 v
      operational capability register
      [base | limit | access rights]
```

In this interpretation an LDP result may be analogous to an object/capability identifier: it identifies a capability without itself being the expanded authority held in a capability register.

This remains a reconstruction. In particular, we have not yet established exactly what `Pointer associated with A` means.

## 6. Why LC and LDP should now be investigated together

The paired change is the most interesting observation produced by comparing the sources:

```text
1972: LC  store only       LDP store only
1976: LC  store + direct   LDP store + direct
```

Rather than reconstructing the two direct forms independently, research should ask whether a **single architectural change in capability-pointer handling** made both operations meaningful.

Questions include:

1. Did the representation or interpretation of capability pointers change between the early and later machines?
2. Does direct LC identify an already-authorised logical capability rather than manufacture one from arbitrary bits?
3. Does direct LDP operate on that same capability-pointer namespace/representation?
4. Are the direct operands indices, capability-pointer values, offsets, or instruction-specific encodings despite occupying the general signed-literal field?
5. Why does LDP change LT/EQ while LC does not?
6. Is there a relationship to later pointer-register, SCT, capability propagation or access-reduction mechanisms described in patents?
7. Can code from Checkout, ROS/COS/POS, the processor self-test material, or surviving Plessey listings demonstrate actual LC/LDP operand usage?

## 7. Falsification / evidence targets

The architectural-evolution hypothesis should be revised or rejected if primary evidence shows, for example:

- the Checkout table incorrectly marked existing direct forms as unavailable;
- the 1976 `70/71` opcodes have meanings other than direct LC/LDP as understood from the table;
- early hardware already implemented these opcodes despite the Checkout documentation;
- different PP250/System 250 processor variants account for the difference rather than chronological evolution.

Highest-value evidence would be the missing **Plessey Telecommunications System 250 Processor Instruction Set Manual**, microprogram listings, assembler definitions, or real program listings using opcode `70` or `71`.

## 8. Current state

### Established

- Early Checkout documentation says LC and LDP are store-mode only.
- Checkout calls LDP **Load capability pointer** and defines it as `D := Pointer associated with A`.
- The May 1976 Pocket Reference assigns LC `30/70` and LDP `31/71`, providing store and direct opcodes for both.
- The 1976 general direct format contains REG, MOD and signed-literal fields.

### Strong working hypothesis

LC and LDP acquired direct forms as part of an architectural development in capability-pointer handling between the early PP250 and the 1976 System 250 documentation.

### Unresolved

- Exact semantics of direct LC.
- Exact semantics of direct LDP.
- Exact meaning and representation of `Pointer associated with A`.
- Why LDP changes LT/EQ.
- Whether the source difference is chronological evolution, variant difference, or documentation discrepancy.

Until those points are resolved, this document remains **work in progress** and its interpretations must not be promoted to documented PP250 architectural facts.