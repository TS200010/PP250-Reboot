# PP250 / System 250 Architectural Generations

**New evidence, 2 October 2026:** the dated evidence update at the end records which earlier claims are superseded; the original reasoning is preserved.

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

The same rule applies specifically to the SCT. Each generation may use a different SCT entry representation and different state/flag bits. Reconstruction should establish the semantics required **within that generation**; it should not treat cross-generation bit positions, flag continuity, or a precise evolutionary mapping as an unresolved architectural problem. A missing Generation-A encoding detail remains worth recording only where it prevents reconstruction of Generation-A behaviour.

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

#### A→B: from operation × type to semantic access rights

**DOCUMENTED FACT:** Figure 3 represents permitted access by combining a permitted store operation with a data type. `PS` distinguishes store read, store write and store read/write; `DT` distinguishes data, program/instruction words and PRSP. `RTE` separately supplies routing/administrative interpretation.

**WORKING ARCHITECTURAL INTERPRETATION:** this means Generation A expresses ordinary authority primarily as **what store operation may be performed on what type of segment**:

```text
PS:  READ | WRITE | READ/WRITE
              ×
DT:  DATA | PROGRAM | PRSP
```

The important limitation of this representation is what it does **not** express as first-class access semantics. Program-ness is a segment type; there is no separately named `EXECUTE DATA` access in Figure 3. Likewise PRSP identifies the reserved-segment-pointer-table type; Figure 3 does not contain a separately named `ENTER CAPABILITY` access.

By Generation B the six named rights are instead:

```text
EC  WC  RC   ED  WD  RD
```

This is therefore not merely a different packing of the same fields. The later representation makes operation-on-kind semantics themselves explicit authorities: read/write capability, read/write/execute data, and enter capability. In particular, **execute** and **enter** have emerged as access rights in their own right rather than being implicit in a typed-segment model.

This is evidence for an architectural evolution from an **operation × type** model in A toward a more explicit **semantic-authority** model in B.

#### Architectural significance of A→B

**WORKING ARCHITECTURAL INTERPRETATION:** A→B is best treated as an **architectural redesign**, particularly in the representation of authority, rather than merely as maturation or repacking of the Generation A mechanism.

Generation B retains the fundamental capability objective, but changes the organising abstraction. Generation A asks, in effect, what store operation may be performed on a segment of a particular type, with routing/administrative interpretation alongside it. Generation B instead makes the semantically meaningful operations themselves the rights:

```text
Generation A                         Generation B

PS × DT × RTE                        EC WC RC | ED WD RD
operation × type × routing     ->    explicit semantic authorities
```

The appearance of `EC` and `ED` is especially significant. ENTER and EXECUTE are no longer consequences implicit in the treatment of PRSP/program-typed segments; they are independently represented authorities. READ and WRITE are similarly divided according to whether they apply to capability or data content.

The other A→B changes — including the movement from `WCR` terminology to `C` registers and the reorganisation of the special-register architecture — should therefore be examined as parts of this broader redesign rather than assumed to be isolated renamings.

This contrasts with B→C. Generation C preserves the six-right semantic-authority structure introduced in B and refines the architecture around it: FORM, propagation and represented-object state become more explicitly separated, and the reserved-register/system structures are extended. Thus the present reconstruction is:

```text
A -> B    substantial architectural redesign,
          especially of capability/access semantics

B -> C    refinement and decomposition of the B architecture,
          preserving its basic semantic-authority model
```

This distinction is about the *kind* of architectural change, not its historical cause. The surviving sources do not yet establish that the designers themselves described A→B as a redesign, nor that particular operational experience caused the changes.

**CAUTION:** this interpretation is based on the documented contrast between the Figure 3 `PS/DT/RTE` scheme and the later six named rights. It does not assert a particular intermediate bit mapping or that every possible `PS × DT × RTE` combination was valid.

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
| Six `EC WC RC ED WD RD` rights | documented in US3814919A; distinct from the Figure 3 generation/version representation | documented | retained/reworked in later representation |
| Nine-bit COS/POS layouts | no evidence | documented | not assumed |
| Inform/Outform virtual-store representation | early conceptual/characteristic evidence; exact encoding unresolved | documented; residence/form represented through ACCESS | passive/FORM machinery revised |
| GARBAGE SCT bit | not yet established | documented | documented |
| VISITED SCT bit | not yet established | documented | not in currently reconstructed C layout |
| PRESENCE SCT bit | no evidence | **do not assume** | documented |
| FORM discriminator | no | no evidence for later meaning | documented |
| Propagation/access reduction | no evidence | no evidence for later mechanism | documented |
| Zero-SUMCHECK temporary SCT unavailability | documented in US3771146A | documented | continuity not assumed without source check |

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

The change is also semantic, not merely bit-level. Generation A's Figure 3 factors permitted access into operation (`PS`) and segment type (`DT`), whereas Generation B exposes the six operation-on-kind rights `EC WC RC ED WD RD`. The later architecture therefore contains explicit **execute** and **enter** authorities that are absent as named access semantics from the Figure 3 scheme.

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

### B–C as architectural refinement

**WORKING ARCHITECTURAL INTERPRETATION:** the B→C transition appears to be more than a change of encoding or software convention. It looks like a refinement of the Generation B architecture in which concepts that B mixes together are separated according to their semantics. Unlike A→B, the six-right semantic-authority model is retained rather than replaced.

In Generation B, the ACCESS representation carries two different kinds of information:

```text
ACCESS
 ├── authority
 │    EC WC RC ED WD RD
 │
 └── representation / system state
      Inform / Outform
      memory / backing-store interpretation
```

The six named rights are capability semantics: they describe what operations the holder of the capability is authorised to perform. Inform/Outform or resident/backing-store interpretation is different in kind. It concerns the representation and management state through which the referenced object is reached; it is not itself authority granted to the holder.

Generation C appears to separate these concerns:

```text
CAPABILITY
    │
    ├── FORM
    │     what kind of capability/reference is this?
    │
    ├── ACCESS
    │     what operations does this authority permit?
    │
    └── PROPAGATION
          how may this authority be transmitted/reduced?

SCT
    │
    └── PRESENCE
          is the represented segment presently available?
```

This division is architecturally coherent because presence is a property of the represented segment, not of each individual authority referring to that segment. Multiple capabilities may designate the same segment while sharing one residence/presence state in its SCT entry.

Conversely, propagation is naturally a property of a capability: it constrains what may be done with that particular authority when it is transmitted or derived.

On this interpretation, Generation C separates **capability-local semantic properties** from **object/segment-management state**:

```text
Capability-local semantics        SCT / represented-object state

FORM                              GARBAGE
ACCESS                            PRESENCE
PROPAGATION                       BASE
                                  LIMIT
```

The `VISITED → PRESENCE` change should therefore not be regarded merely as reuse of SCT bit 22. Together with the revised capability representation, it is evidence of a broader architectural reorganisation in which the SCT becomes a clearer locus for state of the represented segment, while the capability representation becomes a clearer locus for the semantics of authority.

This interpretation also cautions against describing COS/POS-era ACCESS layouts as mere conventions. Their mixed semantics may instead represent an earlier stage in the architectural development that Generation C subsequently disentangles.

**STATUS:** the individual B and C bit layouts and mechanisms are documented; the interpretation of their relationship as deliberate semantic separation and architectural maturation is a reconstruction from those documented changes, not an explicit historical statement by the designers.

### A→B: from the Wilkes capability-register model toward explicit authority

**DOCUMENTED SOURCE CONNECTION:** the early Plessey capability patent explicitly cites M. V. Wilkes, *Time-Sharing Computer Systems* (1968), Chapter 4, and describes Wilkes's capability registers as holding segment descriptors consisting of **base, limit and type code**, with the type code specifying the permitted mode of access. The patent then states that the Plessey invention contemplates the use of such capability registers.

The surviving evidence therefore supports a direct intellectual connection from the Wilkes capability-register model to the earliest Plessey architecture. It does **not** by itself establish the designers' reasons for the subsequent A→B redesign.

A useful conceptual comparison is:

| Concept | Wilkes model as described by the early Plessey patent | Generation A | Generation B |
|---|---|---|---|
| Designation | base + limit | base + limit | base + limit |
| Permitted access | type/access code | `PS × DT × RTE` | `EC WC RC ED WD RD` |
| Data read/write | permitted mode of access | `PS` applied with data type | explicit `RD`, `WD` |
| Execute | program is represented through segment/type information | `DT=P`; no separately named execute right | explicit `ED` |
| Capability read/write | not separately identified in the cited Wilkes formulation | capability structures represented through type/mechanism | explicit `RC`, `WC` |
| **Enter capability** | **no equivalent yet identified in the cited Wilkes formulation** | **`DT=PRSP`, but no independent ENTER authority** | **`EC`: ENTER CAPABILITY is a first-class authority** |

**WORKING HISTORICAL HYPOTHESIS:** Generation A may be a relatively direct Plessey realisation and extension of the capability-register model described by Wilkes: protected segment descriptors combining designation with type/permitted access. Experience building and using that machine may then have exposed a more fundamental abstraction: **the capability should express the semantic authority held by its possessor, rather than primarily classify a segment and the store operations applicable to that type.**

On this reading, the A→B transition is:

```text
Wilkes capability-register model
base + limit + type/permitted access
        |
        v
Generation A
protected typed segments
PS × DT × RTE
        |
        |  possible conceptual re-evaluation
        v
Generation B
explicit semantic authority
EC WC RC | ED WD RD
```

The six Generation B rights have an important conceptual symmetry:

```text
DATA / CODE              CAPABILITY

RD  read                 RC  read capability
WD  write                WC  write capability
ED  execute              EC  enter capability
```

**WORKING ARCHITECTURAL INTERPRETATION:** the `ED ↔ EC` pair is especially significant. Read and write are ordinary access operations; **execute** and **enter** are controlled transitions. `ED` authorises transition into execution of code. `EC` authorises entry through a capability structure into another protected authority context. Thus `EC` is difficult to explain merely as improved segmented-memory protection: it expresses an operation on the capability/authority structure itself.

This suggests that Generation B may be the point at which the distinctive System 250 authority architecture crystallised. In A, PRSP is a **type of protected object**. In B, ENTER is an **authority possessed by the holder**. Similarly, program-ness in A is expressed through type, whereas B exposes EXECUTE as authority.

That interpretation also offers a possible motivation for the redesign: once a substantial system is constructed from capabilities, **type is not the same thing as authority**. Different holders may need different permitted relationships with the same underlying object; capability manipulation itself needs controlled operations; and crossing a protection boundary is better represented as an authorised transition than as a special consequence of accessing an object of a particular type.

#### Why might Generation A have become inadequate?

The endpoint comparison above does not by itself explain the redesign. The following argument records the architectural pressures that could plausibly have led from A to B. These are **reconstructed motivations**, not statements presently attributed to the designers.

**1. PRSP may expose the weakness of type-based authority most clearly.**

In Generation A, `PRSP` is a data/segment type. Yet the interesting property of such a structure is not merely *what kind of storage object it is*. Possession of the appropriate relationship to it permits a controlled transition through capability structure into another protection/authority environment.

The two formulations are therefore conceptually different:

```text
Generation A
this is a PRSP-type object and particular operations on that type
have special architectural consequences

Generation B
the holder possesses authority to ENTER
```

Once the operation is understood as a security-relevant transition, representing **ENTER** directly as `EC` is cleaner than deriving it from the type of object being accessed. PRSP may therefore have been one of the cases that made the limitations of the A model especially visible.

**2. Program type and execute authority expose the same distinction.**

`DT=P` says something about **what the referenced segment is**. `ED` says something different: **what this particular holder is authorised to do with it**.

That distinction matters because object identity/type and authority need not coincide. Conceptually, different capabilities can designate the same underlying object while conferring different operations on it. B's explicit rights make this relationship natural; A's operation × type formulation couples it more tightly to classification.

The same conceptual change therefore appears twice:

```text
A: P     is a type          B: ED is authority to execute
A: PRSP  is a type          B: EC is authority to enter
```

This parallel is one of the strongest reasons to treat A→B as a redesign of the authority abstraction rather than merely a revised encoding.

**3. Independent rights make attenuation and delegation cleaner.**

A capability system becomes substantially more useful when one holder can be given less authority than another over the same underlying object. With independently represented semantic rights, the intended reduction is straightforward to express: for example, retain read authority while withholding write, execute, capability-write or enter authority.

In A, permitted behaviour is more tightly entangled with the object's `DT` classification and the `PS` operation field. B's six-right model instead makes authority resemble a set of independently meaningful permissions. That gives the architecture a much cleaner basis for **delegation with reduced authority**.

This does not establish the exact B propagation rules; it identifies a design pressure that the B representation is intrinsically better able to express.

**4. RTE suggests a possible conflation of authority, mechanism and policy in A.**

Generation A's access structure includes `RTE` distinctions such as normal-store operation, queue, dump and internal-register segment. These are not all naturally descriptions of authority held by the capability possessor. Some describe special architectural treatment or system-management role.

Thus A's access/type code appears to carry several kinds of information at once:

```text
what store operation is permitted?       PS
what kind of information/object is it?   DT
how is this special segment treated?     RTE
```

As the system became more complex, this mixture could have become difficult to extend coherently. B's explicit semantic access rights can be read as movement toward separating **what authority the holder possesses** from **what the referenced system object is and how the machine/OS manages it**. Generation C's later separation of ACCESS, FORM, PROPAGATION and SCT state would continue that direction.

**5. Building a substantial operating system may have changed what a capability was understood to be for.**

The Wilkes-derived starting point is readily understood as protected addressing: a capability defines a segment and constrains access to it. But an operating system constructed extensively from capabilities encounters relationships that go beyond ordinary memory protection:

```text
authority to read data
authority to modify data
authority to execute code
authority to read or write capability structures
authority to enter a service/protection environment
authority to pass reduced authority elsewhere
```

If practical System 250 software increasingly used capabilities to construct protected services and relationships between components, a descriptor organised primarily around storage operation and segment type would become a poor description of what the architecture was actually enforcing.

On this hypothesis, experience with A did not show that the **capability idea** was wrong. It showed that **protected typed-segment access was too narrow a realisation of it**.

**6. The changing register vocabulary is parallel evidence of abstraction maturing.**

The A→B change in access semantics does not occur in isolation. The register vocabulary also moves away from implementation-oriented names such as Workspace Capability Register, Dump Area Capability Register, Master Capability Register and Local Start-up Capability Register toward architectural objects and roles such as `C(D)`, `C(C)`, `C(N)`, `C(S)`, with ordinary capability registers becoming `C0`–`C7`.

Some of those changes are more than renaming: the early Local Start-up structure appears to develop into more differentiated normal-interrupt and start-up/fault structures, while the capability-table and process/code roles become increasingly explicit.

This parallel does not prove a common design motivation, but it is consistent with the same broad transition: **from an implementation-oriented realisation of protected capability registers toward a machine whose architectural vocabulary directly describes authority-bearing objects and operations.**

#### Reconstructed motivation

Taken together, these observations suggest the following possible history:

```text
Wilkes model
capability register = protected segment descriptor
base + limit + type/permitted access
        |
        v
Generation A
a practical hardware realisation
PS × DT × RTE
        |
        | experience constructing a real capability-based system may reveal:
        |
        |-- type is not authority
        |-- program type is not execute authority
        |-- PRSP type is not enter authority
        |-- capability manipulation needs its own controlled rights
        |-- delegation benefits from independently reducible rights
        |-- system-object treatment should not be conflated with holder authority
        |
        v
Generation B
authority-centred capability semantics
EC WC RC | ED WD RD
```

The central historical hypothesis is therefore stronger than “the access encoding was improved”:

> **Generation A may have demonstrated the Wilkes capability-register idea in a real machine, while experience with that machine led Plessey to recast the architectural boundary around explicit semantic authority. Generation B may be the result of that conceptual step.**

**STATUS:** the Wilkes→early-Plessey documentary connection and the A/B endpoint encodings are documented. The proposition that operational experience with Generation A caused or motivated the authority-centred Generation B redesign is **HYPOTHESIS**, not presently established by documentary evidence. The exact Wilkes pages cited by the patent should be recovered and compared directly before attributing finer semantic distinctions to Wilkes himself.

### B→C: possible high-level-language / compiler influence

**DOCUMENTED GENERATION C EVIDENCE:** the later architecture adds a cluster of local-store machinery rather than merely a single stack pointer:

- `C(L)` designates the **Local Store Stack Block**;
- `LNR` is the **Level Number and Local Store Stack Pointer Register**;
- `LCCR` is the **Local Capability Count and Local Store Clear Count Register**.

These names are documented in the later internal-register-addressing patent material.

**FIRST-HAND RECOLLECTION (Anthony Stanners):** the Generation-B-era CORAL compiler had to solve conventional high-level-language calling and activation problems in software/compiler convention. Parameters could be passed in registers or on the stack; register preservation across calls therefore required an ABI/compiler convention; and the compiler implementation maintained its own local-store/stack mechanism rather than relying on an architectural local-store facility.

This recollection is recorded as historical evidence from a participant in the CORAL compiler project, not as a claim established by the surviving processor documentation.

**WORKING HISTORICAL/ARCHITECTURAL HYPOTHESIS:** some Generation C additions may have been influenced by practical experience compiling high-level procedural languages for Generation B. This is particularly worth investigating because Martyn Andrews, associated with the CORAL compiler project, is also associated with the later architectural work.

The compiler problem can be framed as:

```text
Generation B

processor CALL/process machinery
        +
compiler/runtime convention
        ├── register parameter convention
        ├── register preservation across calls
        ├── stacked parameters
        ├── local-variable storage/stack
        └── activation-record management

Generation C

processor CALL/process machinery
        +
architectural local-store machinery
        ├── C(L)
        ├── LNR: level + local-store-stack pointer
        └── LCCR: local-capability / local-store-clear counts
        +
compiler/runtime convention
```

On this hypothesis, B→C is not simply processor optimisation. At least some of its refinements may move mechanisms that a Generation B compiler/runtime had to construct by convention into an architecturally defined facility.

There is a potentially deeper capability issue. A conventional compiler-maintained local stack defines language-level activation and lifetime in software. Architectural local-store levels, particularly in combination with a **Local Capability Count**, may have allowed local capability state to track procedure activation more directly. This possibility is **UNRESOLVED**: the name `Local Capability Count` is not sufficient evidence for its precise semantics, and no claim about capability lifetime or automatic revocation should be made until the patent description is examined in detail.

This gives a specific research programme for the Generation C material: for each new C mechanism, ask not only **what changed from B?**, but also **what compiler/runtime problem present on B would this mechanism remove or simplify?** The Local Store Stack is the strongest current candidate.

**STATUS:** the Generation C register machinery is documented; the Generation-B CORAL implementation experience above is first-hand recollection; the causal connection between compiler experience and the Generation C design is a research hypothesis and is not yet established by documentary evidence.

### A–C

Any apparent A–C similarity must be checked through B rather than treated as proof of uninterrupted implementation. Later patents frequently describe an evolved machine while retaining old architectural names.

## 9. Special-register terminology and genealogy

The names of the processor's reserved/special capability registers change across the surviving generations. These changes must not be normalised away: in several cases they expose a change in architectural decomposition rather than a simple renaming.

### 9.1 Generation A — early hidden capability registers

US3757307A (priority 1970) uses the following early register terminology:

| Generation A register | Source description |
|---|---|
| `DCR` | Dump Area Capability Register |
| `ICR` | capability defining the storage area containing the System Interrupt Word (SIW) |
| `MCR` | capability defining the Master Capability Table |
| `LSCR` | capability defining the processor's Dedicated Local Start-up Area |

The programmer-addressable set is called the **workspace capability registers**, `WCR0`–`WCR7`. `WCR6` conventionally defines the main reserved-segment-pointer table and `WCR7` the current instruction segment.

### 9.2 Generation B — named special-purpose registers

Halton's 1972 System 250 description uses:

| Register | Function |
|---|---|
| `C(D)` | Process Dump Stack |
| `C(I)` | System Interrupt Word |
| `C(C)` | System Capability Table |
| `C(N)` | Normal Interrupt Block |
| `C(S)` | Start-up / check-out block |

The May 1976 Pocket Reference calls these **Special Purpose CPU Registers** and gives:

| Pocket Reference register | Name |
|---|---|
| `C10` | `C(D)` DUMPSTACK |
| `C11` | `C(I)` INTERVAL TIMER |
| `C12` | `C(C)` SCT |
| `C13` | `C(N)` NORMAL INTERRUPT BLOCK |
| separate | `C(S)` FAULT START-UP BLOCK |

This exposes an important version-sensitive point: Halton's 1972 description associates `C(I)` with the System Interrupt Word, while the 1976 Pocket Reference labels `C(I)` INTERVAL TIMER. The name `C(I)` therefore cannot be assigned one timeless function without a source/version qualifier.

### 9.3 Generation C — expanded special-purpose set

US4383297 gives the later special-purpose capability-register set:

| Generation C register | Function |
|---|---|
| `C(D)` | Process Dumpstack |
| `C(I)` | Interval Timer Word |
| `C(C1)` | System Capability Table 1 |
| `C(N)` | Normal Interrupt Block |
| `C(L)` | Local Store Stack Block |
| `C(P)` | present in the special-register set; function must be taken from the patent text before assigning one here |
| `C(C2)` | System Capability Table 2 |
| `C(S)` | Special Start Up Block |

Thus the later architecture expands the named set and splits the System Capability Table register function into `C(C1)` and `C(C2)`.

### 9.4 Working genealogy

| Generation A | Generation B | Generation C | Status / interpretation |
|---|---|---|---|
| `DCR` Dump Area Capability Register | `C(D)` Process Dump Stack | `C(D)` Process Dumpstack | **Strong functional continuity** |
| `MCR` Master Capability Table | `C(C)` System Capability Table | `C(C1)`, `C(C2)` System Capability Tables | **Strong functional lineage**, with MCT→SCT terminology change and later split |
| `ICR` System Interrupt Word area | `C(I)` SIW in Halton 1972; `C(I)` Interval Timer in 1976 | `C(I)` Interval Timer Word | **Function/name evolves**; not safe to state simply `ICR = C(I)` across versions |
| `LSCR` Dedicated Local Start-up Area | `C(N)` Normal Interrupt Block plus distinct `C(S)` Fault Start-up Block | `C(N)` Normal Interrupt Block and `C(S)` Special Start Up Block | **Probable architectural decomposition**, not a proved one-to-one rename |
| — | — | `C(L)` Local Store Stack | **Later addition evidenced** |
| — | — | `C(P)` | **Later addition evidenced; exact function separately to be verified** |
| `WCR0`–`WCR7` workspace capability registers | `C0`–`C7` general-purpose capability registers | `C0`–`C7` retained | **Terminology simplification/standardisation**; detailed role continuity remains source-sensitive |
| `WCR6` reserved-segment-pointer-table capability | `C6` process capability-pointer block/context role | `C6` retained | **Likely lineage with terminology/mechanism evolution** |
| `WCR7` current instruction segment | `C7` current code block | `C7` retained | **Strong functional continuity** |

### 9.5 Architectural significance

The register vocabulary appears to mature in parallel with the access vocabulary.

Generation A describes implementation-oriented register roles — Dump Area Capability Register, Master Capability Register, Local Start-up Capability Register — and workspace registers `WCRn`. Generation B increasingly names reserved registers by the architectural object or service they designate: `C(D)`, `C(C)`, `C(N)`, `C(S)`, while the ordinary set becomes `C0`–`C7`.

More importantly, the change is not wholly cosmetic. The early Dedicated Local Start-up Area participates directly in normal interrupt entry, whereas later evidence has separate `C(N)` and `C(S)` authorities for normal-interrupt and fault/start-up structures. Likewise the single capability-table register lineage eventually becomes `C(C1)` and `C(C2)`.

The special-register genealogy should therefore be used as an additional discriminator when assigning an otherwise ambiguous source to an architectural generation. A familiar function under an early register name is evidence of lineage, but not by itself proof that the surrounding register architecture is identical.

**CAUTION:** register names and functions must be quoted with source/version provenance. In particular, `C(I)` demonstrably changes description between the 1972 Halton material and the 1976 Pocket Reference, so later meanings must not be projected backwards.

## 10. Current research questions

1. What exact mechanism detects ordinary non-residence in Generation A?
2. What are Generation B SCT bits 21..16?
3. What do the three non-right positions in the COS and POS nine-bit ACCESS diagrams mean, including the exact encoding of the active/passive distinction?
4. Why are the six named rights displaced by one position between COS and POS?
5. Can one processor decode both COS and POS representations, and if so how is the convention selected or recognised?
6. What is the exact relationship between Generation A's Figure 3 `PS/DT/RTE` encoding and the separate early characteristic/class-code evidence?
7. At what exact revision were the Generation C FORM, propagation and PRESENCE mechanisms introduced?
8. Which mechanisms currently attributed to B can be proved to have existed already in A?
9. Which B mechanisms survived unchanged into C?

## 11. Research consequence

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


## Addendum — C(S) fault-address evolution as a Generation B→C discriminator

The fault/start-up register provides another concrete difference between the earlier and later architectural evidence.

**DOCUMENTED ENDPOINTS:** the early SSCR fault mechanism uses an alterable store-module selector together with a hard-wired corresponding within-store Fault Block address; on retry the store-module-number field is incremented. In the later US4383297 architecture, C(S) defines the Special Start Up Block and the **twelve most significant bits of its Base** are the fault-sequence increment field. Those same twelve Base bits, and only those bits of C(S), are alterable through Internal Mode; the module number itself remains an eight-bit Base-address field.

Schematically:

```text
earlier:
[ module : 8 ][ fixed corresponding within-store address ]

later:
[ module : 8 ][ high within-module : 4 ][ fixed low offset : 12 ]
|<----------- twelve alterable / incremented bits ----------->|
```

**RECONSTRUCTION / HYPOTHESIS:** the extra four variable address bits may reflect the transition from an early homogeneous 32K-store implementation to an architecture accommodating larger and potentially heterogeneous store modules. They would permit a recovery structure to occupy a store-size-appropriate high region rather than forcing a historical 32K-era reserved location to remain embedded within a larger module. Internal Mode would then give trusted configuration/reconfiguration software a controlled way to establish the preferred C(S) recovery address while the machine is healthy, leaving the fault microsequence able to operate autonomously later.

This explanation is not yet documented as designer intent. The exact original Fault Block address, the software that writes C(S)[23:12], and the exact arithmetic meaning of the later twelve-bit increment remain unresolved.

Alternative explanations evaluated during reconstruction — replicated 4K-spaced Start-Up Blocks, a 4K physical store/SAU unit, blind 4K search as the primary mechanism, and a fixed-address per-store record that first loads C(S) — are unsupported or weakened by the evidence inspected so far. The detailed reasoning and status of each hypothesis are preserved in [pp250-boot-and-processor-startup.md](pp250-boot-and-processor-startup.md).

## Evidence update — 2 October 2026: early six rights and documented local lifetime

**DOCUMENTED OBSERVATION:** US3814919A, Description 48, explicitly assigns RD, WD, ED, RC, WC and EC to bits 16–21, leaving bits 22–23 spare. The patent has 4 March 1971 priority and a 1 March 1972 US filing. This places the six-right representation in the early patent evidence; it does not date its first delivered implementation.

This **supersedes the chronology assumed in the A→B interpretations above**, including the suggestion that named execute/enter authority first crystallised in Generation B. The Figure 3 `PS/DT/RTE` representation remains documented and structurally distinct. The earlier comparison and its causal hypotheses are preserved as the reasoning that led to the question; they can no longer establish a clean A→B redesign date or attribute its motivation to experience with Generation A. For reconstruction purposes the two schemes are now treated as **generation/version-specific representations**. Their exact mapping and implementation ordering are not active architectural questions unless evidence later shows that such a mapping is required. Later LOU, propagation and local-store machinery still require their own later evidence.

US3771146A, Description 111–121, also establishes zero-SUMCHECK temporary unavailability and process-restoration refresh in the early corpus, correcting the matrix's former uncertainty for A.

**DOCUMENTED OBSERVATION, Generation C:** US4486831A, Background/Summary 16–18 and Description 291–297, expressly documents procedure-associated local descriptors, automatic deallocation on return, invalidation of expired local capabilities and restrictions on storage into lower levels. This answers the earlier local-lifetime question: the evidence is now the description, not the name “Local Capability Count”. Whether compiler experience motivated these features remains **HYPOTHESIS**.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.
