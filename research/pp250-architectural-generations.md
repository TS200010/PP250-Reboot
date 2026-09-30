# PP250 / System 250 Architectural Generations

## Status

**RESEARCH NOTE — WORKING RECONSTRUCTION**

This note records evidence for architectural features across successive PP250 / System 250 generations. It is deliberately not an `architecture/` specification: the boundaries between generations, the continuity of individual mechanisms, and some bit-level interpretations remain research questions.

The purpose is to prevent evidence from one generation being silently projected into another.

## 1. Working generational model

Current evidence is best explained by at least three architectural generations.

| Generation | Approximate evidence period | Working description |
|---|---|---|
| **A** | c. 1970–72 | Original PP250 / early System 250 architecture represented by the foundational capability and trap/fault patents and early papers |
| **B** | c. 1975–76 | Mature operational System 250 represented by later Plessey patents and the May 1976 Pocket Reference, including COS/POS-era encodings |
| **C** | c. 1979–80 onward | Andrews/Wheatley architectural development represented by the later capability-register, process-suspension and internal-register-addressing patents |

These labels are research conveniences, not historical Plessey names.

The dates describe the evidence, not necessarily exact hardware release dates. A patent priority date establishes that a mechanism existed in the design history by that date; it does not by itself establish the first or last machine on which it appeared.

## 2. Reconstruction rule

A feature documented in one generation must not automatically be used to fill an unexplained field or mechanism in another.

In particular:

- do not use the later Andrews/Wheatley `PRESENCE` bit to explain Generation B virtual store without independent evidence;
- do not assume Generation B COS/POS nine-bit ACCESS positions existed in the original Generation A processor;
- do not reinterpret the Generation A eight-bit `PS/DT/RTE` type code using later `EC WC RC ED WD RD` meanings;
- do not assume a mechanism disappeared merely because a later source describes it differently;
- distinguish **documented continuity** from **plausible continuity**.

## 3. Features evidenced across all three generations

The following broad architectural ideas appear to persist, although their representation and implementation can change.

| Feature | A | B | C | Status |
|---|:---:|:---:|:---:|---|
| Capability-register addressing | ✓ | ✓ | ✓ | Documented |
| Base/bounds protection for store capabilities | ✓ | ✓ | ✓ | Documented |
| Access authority associated with capabilities | ✓ | ✓ | ✓ | Documented; encoding changes |
| Protected loading/use of capabilities | ✓ | ✓ | ✓ | Documented; mechanisms evolve |
| System/master capability table concept | ✓ | ✓ | ✓ | Documented, but entry state semantics evolve |
| Three-word SCT/MCT descriptor family | ✓ | ✓ | ✓ | Strong continuity; individual fields/state meanings evolve |
| Capability-mediated process/system structure rather than conventional unrestricted supervisor authority | ✓ | ✓ | ✓ | Broad architectural continuity; exact mechanisms are version-sensitive |

This table records conceptual continuity only. It does not imply bit-compatible representations.

## 4. Generation A — original PP250 / early System 250

### 4.1 Eight-bit permitted-access type code

US3787813 / GB1329721 documents an eight-bit permitted-access type code:

```text
       PS        DT              RTE
     2 bits    2 bits          4 bits
                         NSO | Q | DUMP | IR
```

The patent describes:

- `PS`: permitted store operation;
- `DT`: data/program/PRSP-type interpretation;
- `RTE`: routing/administrative classification including normal store operation, Queue, Dump and internal-register segment.

This is structurally different from the later six named `EC WC RC ED WD RD` rights.

### 4.2 Early capability class / characteristic evidence

Other early patent evidence describes characteristic/class codes associated with capability use, including distinctions reconstructed as active store, passive/backing-store, resource and null/special classes.

**UNRESOLVED:** the exact relationship between this evidence and the Figure 3 `PS/DT/RTE` eight-bit format has not yet been established. They must not be forced into a single bit map merely because they belong to the same broad period.

### 4.3 Early SCT/MCT

US3787813 establishes a three-word MCT entry:

```text
word 0   CHECK / SUMCHECK
word 1   BASE
word 2   LIMIT plus type/state information where applicable
```

LC reads the descriptor and independently forms a local check from BASE and LIMIT before accepting the loaded capability.

The original patent also allows some type-code information common to all references to a segment to reside in the MCT and be merged with information from the stored capability pointer.

### 4.4 Virtual store

Contemporary System 250 operating-system material establishes virtual-store behaviour by 1972: blocks move between main store and disk, attempted use of a nonresident block is detected, software brings the block into main store, and the trapped instruction can subsequently be reattempted.

**UNRESOLVED:** the exact Generation A hardware representation of ordinary non-residence remains under investigation.

## 5. Generation B — mature System 250, c. 1975–76

### 5.1 Nine-bit ACCESS representation

The May 1976 Pocket Reference records six named access semantics:

`EC WC RC ED WD RD`

but gives different nine-position diagrams for COS and POS:

```text
COS   1 1 EC WC RC ED WD RD 0
POS   0 1 1 EC WC RC ED WD RD
```

These are evidence for Generation B representation, but their relationship is not yet explained.

**Important:** because the same Pocket Reference presents both, COS versus POS must not currently be treated as a separate processor generation. They may be alternative system conventions supported by the same underlying processor machinery.

### 5.2 Generation B SCT state

The 1975-priority allocation/deallocation patent establishes that the third SCT word contains a 16-bit LIMIT offset with eight higher positions available for additional state.

For the garbage-collection mechanism it identifies:

```text
bit 23      GARBAGE
bit 22      VISITED
bits 21..16 unresolved/spare/state
bits 15..0  LIMIT
```

This is direct evidence for Generation B and must not be overwritten by the later Andrews/Wheatley interpretation.

### 5.3 SUMCHECK zero

Generation B patent evidence uses a zeroed SUMCHECK to make an SCT entry temporarily unavailable while it is being changed, including relocation.

This is a documented synchronization/unavailability mechanism.

It is **not yet established** that zero SUMCHECK is the normal representation of a segment paged out to backing store.

### 5.4 Inform / Outform and virtual store

Generation B evidence supports the mature Inform/Outform model:

- active/Inform capabilities in primary store identify an SCT entry;
- passive/Outform capabilities in secondary store carry persistent backing-store identity;
- capability-containing blocks are converted as they move between primary and secondary storage;
- an SCT slot cannot be reclaimed merely because one capability becomes Outform; it remains required while Inform/active references to that SCT entry survive.

In Generation B, residence/form is represented in the capability ACCESS encoding rather than by a `PRESENCE` state in the SCT. An Inform/active capability uses the active-store form and an SCT index; an Outform/passive capability uses the backing-store form and persistent backing-store identity. The processor therefore distinguishes the nonresident/passive representation from the ACCESS/type information and does not interpret its pointer as an ordinary usable SCT reference.

This is a defining difference from Generation C. Generation B does **not** require the later SCT `PRESENCE` bit to encode this distinction.

## 6. Generation C — Andrews/Wheatley development

Generation C introduces a substantially revised capability representation.

### 6.1 FORM and propagation/access reduction

The later patents explicitly describe capability FORM discrimination and propagation/access-reduction machinery. This is a later architectural mechanism and must not be used to decode the Generation B COS/POS diagrams.

### 6.2 Revised SCT state

The later memory-protection patent gives the SCT LIMIT/state word with:

```text
bit 23      GARBAGE
bit 22      PRESENCE
bits 21..16 spare/other
bits 15..0  LIMIT
```

`PRESENCE = 0` is explicitly associated with a trapped/noninterpreted SCT entry.

The important historical observation is therefore:

```text
Generation B                 Generation C

23 GARBAGE                   23 GARBAGE
22 VISITED                   22 PRESENCE
21..16 unresolved            21..16 spare/other
15..0 LIMIT                  15..0 LIMIT
```

The change from `VISITED` to `PRESENCE` occurs in the same broad later architectural development in which capability FORM/propagation semantics are revised. Whether these changes were introduced in one indivisible hardware revision remains to be proved, but they belong to the same later evidence family.

## 7. Cross-generation feature matrix

| Mechanism / representation | A | B | C |
|---|---|---|---|
| Capability registers | documented | documented | documented |
| Three-word SCT/MCT | documented | documented | documented |
| CHECK/SUMCHECK | documented | documented | documented/retained |
| BASE | documented | documented | documented |
| 16-bit LIMIT concept | documented in loaded CR; SCT details source-sensitive | documented | documented |
| 8-bit `PS/DT/RTE` access/type | documented | not assumed | not assumed |
| Six `EC WC RC ED WD RD` rights | not backdated | documented | retained/reworked in later representation |
| Nine-bit COS/POS layouts | no evidence | documented | not assumed |
| Inform/Outform virtual-store representation | early conceptual/characteristic evidence; exact encoding unresolved | documented; residence/form represented through ACCESS | passive/FORM machinery revised |
| GARBAGE SCT bit | not yet established | documented | documented |
| VISITED SCT bit | not yet established | documented | not in currently reconstructed C layout |
| PRESENCE SCT bit | no evidence | **do not assume** | documented |
| FORM discriminator | no | no evidence for later meaning | documented |
| Propagation/access reduction | no evidence | no evidence for later mechanism | documented |
| Zero-SUMCHECK temporary SCT unavailability | not yet established | documented | continuity not assumed without source check |

## 8. Features shared by groups of generations

### A–B

A and B clearly share the fundamental PP250 capability/SCT architecture, but B must be treated as an evolved implementation rather than a bit-compatible restatement of A.

Likely or documented continuities include:

- capability-register addressing;
- SCT-mediated segment identity;
- protected capability loading;
- virtual-store architecture;
- hardware-supported trapping into capability-constrained software.

The exact access-code encoding is **not** continuous in the evidence.

### B–C

B and C share much mature System 250 machinery and terminology, but C revises capability representation and SCT state semantics.

Particularly visible continuities/changes are:

- the three-word SCT remains;
- GARBAGE remains in bit 23;
- the 16-bit LIMIT field remains;
- bit 22 changes from documented B use as VISITED to documented C use as PRESENCE;
- capability access/form semantics are redesigned;
- in B, the active/passive (Inform/Outform) distinction is represented through ACCESS; in C, the revised representation includes explicit FORM semantics while SCT `PRESENCE` carries segment-presence state.

This pairing is architecturally significant: the change in capability representation and the change in SCT state interpretation should be studied together rather than treating `PRESENCE` as a field that can be projected backwards into B.

### A–C

Any apparent A–C similarity must be checked through B rather than treated as proof of uninterrupted implementation. Later patents frequently describe an evolved machine while retaining old architectural names.

## 9. Current research questions

1. What exact mechanism detects ordinary non-residence in Generation A?
2. What are Generation B SCT bits 21..16?
3. What do the three non-right positions in the COS and POS nine-bit ACCESS diagrams mean, including the exact encoding of the active/passive distinction?
4. Why are the six named rights displaced by one position between COS and POS?
5. Can one processor decode both COS and POS representations, and if so how is the convention selected or recognised?
6. What is the exact relationship between Generation A's Figure 3 `PS/DT/RTE` encoding and the separate early characteristic/class-code evidence?
7. At what exact revision were the Generation C FORM, propagation and PRESENCE mechanisms introduced?
8. Which mechanisms currently attributed to B can be proved to have existed already in A?
9. Which B mechanisms survived unchanged into C?

## 10. Research consequence

The PP250-Reboot reconstruction should no longer speak casually of **the** PP250 capability encoding or **the** SCT layout without a generation qualifier when the distinction matters.

The current working model is:

```text
Generation A
original 8-bit/type-code architecture
        |
        v
Generation B
mature 9-bit COS/POS-era System 250
        |
        v
Generation C
Andrews/Wheatley FORM/propagation/PRESENCE architecture
```

This is a framework for organising evidence, not a claim that only three physical processor revisions existed. Additional intermediate revisions may emerge as the patent and software corpus is reconstructed.
