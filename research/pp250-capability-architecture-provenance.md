# PP250 Capability Architecture Provenance

**Status:** Working research note  
**Project:** PP250-Reboot  
**Created:** 2026-09-22

## Purpose

This document tracks the intellectual and architectural provenance of the capability ideas surrounding Plessey System 250 (PP250), their treatment in later capability research, Kenneth J. Hamer-Hodges's modern Church Machine/CLOOMC work, and PP250-Reboot.

It is not intended to imply influence merely from architectural similarity. Each relationship should eventually be classified as one of:

- **documented influence** — a source explicitly records that one work influenced another;
- **explicit claimed descent** — a designer explicitly identifies a later architecture as descended from an earlier one;
- **architectural continuity/similarity** — a recognizable mechanism or invariant occurs in both, without proof of direct influence;
- **research inference** — a relationship proposed by PP250-Reboot and still requiring evidence.

## 1. Early provenance into System 250

Henry M. Levy's historical account records a specific route by which capability ideas entered System 250.

Maurice Wilkes learned about capabilities through visits to the University of Chicago and discussed the concept in his work on time-sharing systems. Wilkes sent a draft of his book to Plessey's Jack Cotton. Cotton incorporated capability concepts into System 250. Because of the resemblance between System 250 and the Chicago work, Bob Fabry, who had worked on the Chicago Magic Number Machine, later acted as a consultant to Plessey.

This gives a documented early provenance chain broadly of the form:

```text
Early capability research
        │
        ├── Dennis & Van Horn and related capability concepts
        │
University of Chicago capability work
        │
        ├── Maurice Wilkes encounters the ideas
        │
        └── Bob Fabry / Magic Number Machine
                    │
                    ▼
             Maurice Wilkes
                    │
              draft sent to
                    ▼
               Jack Cotton
                    │
                    ▼
          Plessey System 250
                 (PP250)
```

The exact relationship between Dennis & Van Horn, Chicago, Wilkes and the individual System 250 mechanisms should be documented more precisely as primary sources are reviewed. The important point is that the Wilkes/Cotton/Fabry connection is documented historical influence rather than an inference from similarity.

## 2. PP250's place in the later capability literature

PP250 did not disappear from the capability literature.

Levy's *Capability-Based Computer Systems* devotes a chapter to it and describes System 250 as the first operational capability hardware system and the first capability system sold commercially.

Modern CHERI literature also discusses PP250 technically. It records, among other things:

- eight general-purpose/data registers and eight capability registers C0–C7;
- C6 as the current security context;
- C7 as the current program capability;
- special-purpose capability registers;
- protected procedure calls based on an **Enter** capability;
- the distributed, non-hierarchical trust model;
- successful operation of that model on a multiprocessor.

Thus PP250 remains part of the recognized historical ancestry against which modern capability architectures are discussed.

A simplified mainstream research lineage can provisionally be drawn as:

```text
Early capability research
        │
        ├─────────────┬───────────────┐
        │             │               │
      PP250          CAP            Hydra
        │             │               │
        └────── capability-system research ──────┐
                                                  │
                                          later capability work
                                                  │
                                                CHERI
```

This diagram means **research lineage and historical context**, not that CHERI is a direct implementation descendant of PP250. The precise influence paths need to be supported individually.

## 3. Hamer-Hodges's continuation of the PP250 line

Kenneth J. Hamer-Hodges was part of the contemporary System 250 technical record; his 1972 paper *Fault Resistance and Recovery within System 250* is among the original PP250 literature.

More than fifty years later, his Church Machine/CLOOMC repository explicitly presents a modern architecture as a continuation of PP250.

His stated line of descent is approximately:

```text
                    PP250
                      │
          capability-only authority
          hardware enforcement
          descriptor / segment table
          Enter capability
          fault containment/recovery
                      │
                      ▼
          Church/Turing interpretation
                      │
                      ▼
               Church Machine
                      │
                      ▼
                   CLOOMC
```

Hamer-Hodges calls Church Machine a direct architectural successor to PP250. His modern design preserves or generalizes several principles he explicitly associates with PP250:

- capability-mediated authority;
- absence of a raw-address escape route;
- no ordinary privileged-software bypass of the capability model;
- separation of authority from ordinary data computation;
- Enter as distinct authority from Execute;
- an authoritative system map, with the Church Machine Namespace described as descended from the PP250 segment table/SCT;
- capability-defined confinement;
- hardware participation in protection and recovery.

At the same time, Hamer-Hodges explicitly identifies important Church Machine mechanisms as new, including CLOOMC, LUMPs and the modern Golden Token representation.

The lineage claim is therefore not that Church Machine reproduces PP250. It is that Church Machine develops what Hamer-Hodges considers PP250's fundamental architectural model into a modern general-purpose machine.

## 4. Recognition: PP250 versus the Hamer-Hodges continuation

There is an important split in the historical record.

**PP250 itself remains recognized** in mainstream capability research. Levy and later CHERI literature discuss it as an important early hardware capability machine and describe specific architectural mechanisms.

**Hamer-Hodges's modern Church Machine/CLOOMC continuation appears, at present, to have much less visible independent academic engagement.**

The current evidence therefore suggests two branches:

```text
                             PP250
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
   mainstream capability              Hamer-Hodges continuation
       research history
              │                                 │
        CAP / Hydra / etc.              Church/Turing interpretation
              │                                 │
      later capability work                 Church Machine
              │                                 │
            CHERI                              CLOOMC
```

The left branch continues to cite and analyse PP250 itself. The right branch is Hamer-Hodges's explicit attempt to continue its architectural ideas.

At present there is little evidence that these two modern conversations substantially engage with one another. This should be tested further through citation and literature searches rather than treated as a final conclusion.

## 5. PP250-Reboot: a second modern route from PP250

PP250-Reboot should not be represented as a descendant of Church Machine. Its route is different.

The project's current approach is:

```text
                         PP250
                           │
                           ▼
              reconstruct original machine
                           │
               identify architectural
                    invariants
                           │
                           ▼
              preserve capability model
                           │
                           ▼
            modernise implementation where
          this does not violate those invariants
```

This creates an interesting relationship:

```text
                              PP250
                         ┌──────┴──────┐
                         │             │
                         ▼             ▼
                 Hamer-Hodges      PP250-Reboot
                    route              route
                         │             │
              extract principles   reconstruct first
                         │             │
              modern successor     identify invariants
                         │             │
                         ▼             ▼
              Church Machine      modern PP250-derived
                  / CLOOMC              machine
```

The two projects therefore appear to begin from the same historical architecture but proceed in opposite directions.

Hamer-Hodges moves **forward from selected PP250 principles toward a new architecture**.

PP250-Reboot moves **backward first, reconstructing PP250 as accurately as possible, before deciding what can be modernised without losing the architecture's essential properties**.

The resulting convergence is potentially significant but must not be mistaken for evidence that PP250-Reboot ideas were derived from Church Machine. Where similar conclusions were reached independently, that fact should be preserved in the project history.

## 6. Areas of apparent convergence

The following subjects currently appear in both modern lines of investigation:

### Authority distinct from ordinary data

Both treat capability authority as fundamentally different from arbitrary binary data and resist architectures in which ordinary computation can simply manufacture authority.

### Enter distinct from Execute

PP250's Enter capability survives explicitly in Hamer-Hodges's modern E permission/E-GT. PP250-Reboot has independently identified Enter as central to protected domain crossing and CALL semantics.

### Authoritative system structure

Hamer-Hodges explicitly describes the Church Machine Namespace as descended from the PP250 segment table. PP250-Reboot identifies the System Capability Table as the authoritative structure that gives stored capability references their meaning.

### No unrestricted escape path

Both lines emphasize that capability-limited execution must not be able to turn itself into unrestricted physical-memory or device authority.

### Bootstrap authority

Both encounter the fundamental question of how a machine whose software cannot forge capabilities establishes legitimate capability state at startup.

### Resource lifecycle and revocation

Both must address creation, delegation, destruction/revocation and reuse of capability-protected resources.

### Extension beyond local memory

Both confront the problem of preserving authority when interaction extends to devices, storage or remote systems.

These similarities are research observations. Their exact provenance must be documented mechanism by mechanism.

## 7. Why provenance matters

The provenance question is not merely historical.

If a modern PP250-derived design contains an architectural idea, we need to know whether that idea is:

1. present in original PP250;
2. documented in later PP250 development;
3. introduced independently by Hamer-Hodges's Church Machine;
4. established in another capability architecture;
5. independently rediscovered by PP250-Reboot; or
6. newly proposed by PP250-Reboot.

That distinction matters for historical accuracy, technical understanding, attribution, and potential intellectual-property analysis.

Accordingly, future research notes should preserve dates and sources rather than silently folding later ideas back into the reconstructed 1970s architecture.

## 8. Provisional provenance map

```text
              Dennis & Van Horn / early capability research
                              │
                University of Chicago work
                              │
                ┌─────────────┴─────────────┐
                │                           │
          Maurice Wilkes                 Bob Fabry
                │                           │
                └──────────► Jack Cotton ◄──┘
                              │
                              ▼
                     PLESSEY SYSTEM 250
                            (PP250)
                              │
            ┌─────────────────┼──────────────────┐
            │                 │                  │
            ▼                 ▼                  ▼
       historical        mainstream        Hamer-Hodges
       PP250 record      capability         continuation
            │             research               │
            │                 │          Church/Turing
            │          CAP / Hydra /      interpretation
            │          later systems            │
            │                 │           Church Machine
            │               CHERI                │
            │                                    ▼
            │                                  CLOOMC
            │
            └──────────────────────┐
                                   │
                                   ▼
                             PP250-Reboot
                                   │
                        original reconstruction
                                   │
                       architectural invariants
                                   │
                                   ▼
                        modern PP250-derived
                              architecture
```

The arrows in this diagram do **not** all mean the same thing. Some represent documented influence, some explicit claims of descent, and some the organization of research literature. Each edge should ultimately be annotated with its evidence class.

## 9. Research questions to resolve

Future work should establish:

- the exact documentary path from Dennis & Van Horn and Chicago capability work to Wilkes and Cotton;
- which specific PP250 mechanisms can be traced to earlier capability proposals and which were Plessey innovations;
- whether Hamer-Hodges's modern Church/Turing interpretation appears in any contemporary PP250 material or is a later conceptualization;
- how modern CHERI authors understand PP250's influence and which PP250 properties they deliberately retained, altered or rejected;
- whether Church Machine/CLOOMC has received independent academic citation, implementation or critique under terminology not yet found;
- the chronology of Hamer-Hodges's modern work, including when the Church/Turing interpretation, Church Machine and CLOOMC concepts first appeared publicly;
- where PP250-Reboot independently converges with Church Machine and where the two architectures fundamentally diverge.

## 10. Initial sources

Primary and near-primary PP250 material in PP250-Reboot should remain the principal evidence for original System 250 architecture.

Important external provenance sources include:

- Henry M. Levy, *Capability-Based Computer Systems* (1984), Chapter 4, “The Plessey System 250”.
- D. M. England, contemporary System 250 architecture and capability papers.
- K. J. Hamer-Hodges, “Fault Resistance and Recovery within System 250” (1972).
- Kenneth J. Hamer-Hodges, modern Church Machine/CLOOMC repository.
- modern Cambridge/CHERI architecture reports discussing Plessey System 250.

The companion document `research/ken-hamer-hodges-pp250-statements.md` records Hamer-Hodges's modern statements about PP250 in greater detail.

---

## Evidence discipline

This document is deliberately a **provenance map**, not a claim that every similar idea has a single line of descent.

As research proceeds, each arrow should be backed by a source and labelled:

`DOCUMENTED INFLUENCE`, `CLAIMED DESCENT`, `ARCHITECTURAL SIMILARITY`, or `PP250-REBOOT INFERENCE`.

That will allow the project to preserve both the history of the original machine and the provenance of ideas emerging during its modern reconstruction.


---

## 11. Research hypothesis: was PP250 framed too narrowly as a protection architecture?

A potentially important explanation for PP250's limited influence on mainstream computer architecture is emerging from the historical material.

### 11.1 The way the architecture was presented

The contemporary and later literature overwhelmingly explains System 250 capabilities in terms of:

- protection;
- reliability;
- execution domains;
- resource ownership and control;
- protected procedure calls;
- operating-system resource management;
- fault containment.

These are genuine and important properties of the architecture. However, this framing may have obscured a more general consequence of the machine: **the capability architecture can represent software structure itself.**

The historical papers describe the mechanisms, but we have not yet found comparable contemporary emphasis on using those mechanisms as a general object/component model.

This remains a research hypothesis and should be tested against the complete contemporary literature before being stated as a historical conclusion.

### 11.2 The CCB as more than a security context

A Central Capability Block can be viewed conventionally as the collection of capabilities defining a process or domain's authority.

But structurally it can also resemble an object's protected environment:

```text
C6 → Central Capability Block

       +0   capability → operation / method A
       +1   capability → operation / method B
       +2   capability → operation / method C
       +3   capability → private state
       +4   capability → another protected object/domain
       ...
```

If another domain receives an **Enter capability** rather than ordinary capability-read access to this structure, it can be given authority to invoke permitted entry points without acquiring authority to inspect or manipulate the protected implementation.

This is strongly reminiscent of an object interface.

### 11.3 A hardware-protected analogue of a v-table

In a conventional object-oriented implementation, a reference to an object ultimately permits software to select an operation through some form of dispatch structure, commonly a v-table.

PP250's protected call mechanism suggests a stronger construction:

```text
object authority + entry selector
              │
              ▼
             CALL
              │
              ▼
hardware-mediated protected entry
              │
              ▼
callee executes in its own capability environment
```

This can be understood as analogous to a **hardware-enforced object dispatch table**, rather than merely a software v-table.

The distinction is important. A conventional v-table is data/code structure interpreted by software. A PP250 Enter capability can provide authority to invoke a protected entry without necessarily providing authority to inspect or rewrite the structure implementing that interface.

Thus:

```text
Conventional OO                  Possible PP250 interpretation

object reference          ↔      capability / Enter authority
v-table                    ↔      protected entry structure / CCB
method selector            ↔      entry displacement
method call                ↔      CALL
private object state       ↔      capabilities reachable inside domain
encapsulation              ↔      hardware-enforced capability boundary
object graph               ↔      graph of reachable capabilities
```

This table is an architectural interpretation, not a claim that the PP250 designers used modern object-oriented terminology.

### 11.4 Protected procedure versus object

The historical vocabulary matters.

System 250 was developed at the end of the 1960s and beginning of the 1970s. Although Simula had already introduced classes and objects, the later mainstream vocabulary of:

- object interfaces;
- encapsulation;
- dynamic dispatch;
- v-tables;
- dependency injection;
- component graphs;

was not yet the normal language in which computer architecture was described.

The PP250 designers therefore naturally described **protected procedures, execution domains and capabilities**.

A modern reading can see that substantially the same mechanisms may also describe:

**objects, interfaces, method invocation and object graphs.**

The possibility to investigate is that the architecture's software-structuring consequences were larger than the problem its designers were presenting it as solving.

### 11.5 Why this might have affected adoption

If PP250 is presented primarily as:

> a sophisticated hardware architecture for protecting resources,

then it competes against much cheaper and increasingly conventional mechanisms such as supervisor modes, memory-management units, rings, virtual memory, access-control lists and process isolation.

In that comparison, a capability machine can appear to be expensive specialized hardware solving a problem that conventional architectures can solve adequately in software plus simpler protection hardware.

The proposition changes considerably if the architecture is instead understood as:

> **hardware that directly represents and enforces the structure and authority relationships of software components.**

Under that interpretation:

- the CCB is not merely a security context; it can be an object's protected capability environment;
- Enter is not merely a protection key; it can be an invocation interface;
- CALL is not merely a protected procedure mechanism; it can be hardware-mediated method dispatch and domain transition;
- capability closure is not merely the set of resources a process may access; it can describe the reachable object graph;
- the SCT is not merely memory-protection metadata; it participates in maintaining authoritative object/resource identity independently of physical placement.

This would make PP250's capability machinery a **software architecture mechanism as well as a protection mechanism**.

### 11.6 Evidence that the broader interpretation emerged later

Kenneth Hamer-Hodges's modern Church Machine work explicitly moves toward the language of abstractions and separates authority from ordinary computation.

The Church Machine treats capability structure as the structure of the machine's software world rather than merely as an access-control layer.

This may indicate that at least one original System 250 designer subsequently came to interpret the architectural idea more broadly than the protection/resource-management framing prominent in the early literature.

It does not establish that the original Plessey team consciously intended PP250 as an object-oriented machine.

### 11.7 A possible historical explanation

The working hypothesis is therefore:

> **PP250 may have been understood and presented primarily as a capability protection and reliability architecture at a time when the vocabulary needed to describe its broader software-structuring potential was not yet mature.**

If so, part of the reason capability machines did not become mainstream may be that their proponents and their audience evaluated them primarily against alternative **protection mechanisms**, rather than against alternative **models for constructing software systems**.

This is not yet established historical fact. It is a hypothesis worth testing against:

1. the original System 250 papers;
2. Plessey manuals and training material;
3. patents;
4. contemporary conference discussions and reviews;
5. later recollections by Cotton, England, Cosserat, Hamer-Hodges and other designers;
6. early object-oriented literature, especially Simula and later Smalltalk;
7. contemporary and later descriptions of CAP and Hydra, where similar software-structuring effects may have been noticed.

### 11.8 Why this matters to PP250-Reboot

This hypothesis changes the question PP250-Reboot asks of the historical machine.

Instead of asking only:

> How did PP250 protect memory and resources?

we should also ask:

> **What software structures can the PP250 capability model express directly?**

That includes investigating whether the original architecture naturally supports:

- objects with private state;
- protected interfaces;
- method dispatch;
- polymorphic interfaces;
- composition through capability references;
- object graphs;
- service discovery through capability structures;
- dependency injection by capability transfer;
- replacement/revocation of implementations without exposing their representation.

If these emerge naturally from the original architecture, they are not merely security side effects. They may reveal that PP250 contained the basis of a substantially different general-purpose programming model whose significance was never fully articulated at the time.

The strongest form of the hypothesis can be stated as:

> **The capability system was not merely capable of protecting the program. It was capable of being the program's structural model.**

That proposition should now be treated as a research question and tested, not assumed.


---

## 12. Evolution of the hypothesis: from “object insight was missed” to “where should the hardware boundary be?”

The reasoning that led to the current research question is itself important and should be preserved. The conclusion changed as the historical comparison broadened.

### 12.1 Initial observation

The starting observation was that contemporary PP250 material appears to emphasize capabilities primarily as mechanisms for:

- protection;
- reliability;
- resource control;
- namespaces/domains;
- protected procedure invocation.

At the same time, the reconstructed mechanisms suggest a much broader software interpretation. A CCB can resemble a protected object environment; an Enter capability can resemble an object reference exposing an invocation interface; CALL can resemble protected method dispatch; and the reachable capability structure can resemble an object graph.

This led initially to the hypothesis that PP250's wider potential may simply never have been understood because it was presented as a protection architecture before modern object-oriented vocabulary became commonplace.

### 12.2 First correction: the object connection was not missed

That hypothesis proved too broad.

Hydra clearly treated capabilities as references to protected objects with permitted operations. KeyKOS explicitly developed a capability-based object-oriented operating system. EROS continued this tradition, treating capabilities as unforgeable object references coupled to authorized interfaces.

These systems demonstrate that researchers did understand the deeper relationship:

```text
capability
    │
    ├── designation
    ├── authority
    └── protected invocation
             │
             ▼
           object
```

Therefore the historical question cannot be:

> Why did nobody realize that capabilities could represent objects?

They did.

### 12.3 Second correction: researchers also put object models into hardware

A stronger version of the hypothesis also fails.

The Intel iAPX 432 was deliberately designed as an object-based computer architecture with hardware/microcode support for object-based programming and protected object references.

IBM System/38 likewise made objects central to the machine architecture and used protected/tagged capability-like pointers.

SRI's PSOS work also explored typed, unforgeable capabilities and a highly structured object-oriented capability architecture extending toward hardware.

Thus researchers did not merely build object-capability operating systems on conventional machines. Some explicitly attempted to move the object model into the processor architecture.

### 12.4 The question that survives

The comparison suggests that the interesting distinction may instead be **where the hardware/software semantic boundary is drawn**.

Three broad approaches can be distinguished:

```text
A. SOFTWARE OBJECT-CAPABILITY SYSTEM

    application objects
           │
    capability/object model
           │
    kernel / nanokernel
           │
    conventional processor

    Examples: Hydra, KeyKOS, EROS


B. HARDWARE OBJECT MACHINE

    application objects
           │
    architectural object types
           │
    object operations / descriptors
           │
    specialised processor architecture

    Examples to investigate:
    iAPX 432, System/38, PSOS


C. PP250 POSSIBILITY

    software-defined meaning
           │
    protected interface / domain
           │
    Enter / CALL
           │
    capability closure / CCB
           │
    capability registers
           │
    SCT / hardware authority enforcement
```

The potentially distinctive PP250 property is that the hardware need not know what an object **means**.

It need not know whether protected state represents a file, account, compiler object, telephone call, device, semaphore or application-defined abstraction.

The hardware may need to know only **authority**:

- what capability is possessed;
- what operations that capability permits;
- what protected domain may be entered;
- what capability environment becomes available after entry;
- and whether ordinary computation can manufacture or escape that authority.

### 12.5 Meaning above; authority below

This suggests a clean conceptual boundary:

```text
             SOFTWARE MEANING

          application abstraction
                   │
              interface
                   │
              operations
                   │
────────────────────────────────────
             HARDWARE AUTHORITY
                   │
            Enter capability
                   │
                 CALL
                   │
                 CCB
                   │
         capability closure
                   │
                 SCT
```

The upper layer decides **what something is**.

The lower layer decides **who has authority to do what with it**.

The capability hardware therefore does not have to implement an object-oriented language or understand application types. Objects can emerge from software structure while their encapsulation and authority relationships are enforced below the semantic boundary.

This is importantly different from concluding that PP250 contains a hardware v-table or that the processor itself understands objects. The v-table analogy helped reveal the possibility, but the deeper abstraction may be simpler than “object”.

### 12.6 Reinterpreting an apparent limitation

Earlier PP250-Reboot discussion noted that a capability protects an address space and authority over it but does not know the **meaning** of the bits within that space.

Initially this can look like a limitation.

The comparison with richer hardware object machines suggests the opposite interpretation may be worth testing:

> **Not knowing the meaning may be precisely what keeps the capability mechanism general.**

If the processor embeds particular object semantics, types or programming-language concepts, the architecture becomes coupled to those abstractions.

If the processor enforces only authority, software remains free to define new abstractions without changing the hardware protection model.

### 12.7 Revised historical trajectory

A useful research model is therefore not that PP250 discovered an object architecture which everybody subsequently overlooked.

A more plausible schematic history is:

```text
                    capability ideas
                          │
                          ▼
                        PP250
                          │
                 hardware authority
                          │
             ┌────────────┴────────────┐
             │                         │
             ▼                         ▼
    object-capability OSs       hardware object systems
             │                         │
          Hydra                    iAPX 432
             │                     System/38
          KeyKOS                       │
             │                  rich architectural
           EROS                  object semantics
             │                         │
             └────────────┬────────────┘
                          │
                          ▼
             later capability research
                          │
                        CHERI
```

This diagram is conceptual rather than a claim of direct historical descent. Each edge requires separate provenance evidence.

### 12.8 The resulting PP250-Reboot research question

The question has therefore evolved through three stages:

**Initial question**

> Did researchers fail to see that PP250 capabilities could provide the structural basis for object-oriented software?

**Corrected question**

> How far did capability researchers take the idea that capability graphs could define software structure?

**Current question**

> **Did PP250 place the hardware/software boundary at an unusually powerful minimal abstraction: hardware enforces authority, isolation and protected invocation, while software remains responsible for the meaning and type of the objects constructed above it?**

This is now one of the principal questions for the Top-Down Capability-Native Design track.

### 12.9 A falsifiable top-down experiment

The top-down programme provides a way to test the idea rather than merely admire it historically.

Begin without assuming PP250 mechanisms. Start with the requirements of a general-purpose software system:

```text
objects / services
       │
private state
       │
interfaces
       │
composition
       │
polymorphism
       │
controlled delegation
       │
revocation / lifecycle
       │
fault containment
```

Then derive the minimum machine mechanisms required to enforce the authority relationships between those abstractions.

If that independent derivation naturally converges on something resembling:

```text
capabilities
Enter
CALL
capability closure
CCB-like protected environments
SCT-like authoritative identity
non-forgeability
```

without requiring the hardware to understand application-level object types, that convergence would be significant.

It would suggest that these PP250 mechanisms are not merely historical implementation choices. They may represent a compact hardware substrate from which richer software abstractions naturally emerge.

Conversely, if the top-down derivation requires mechanisms PP250 cannot naturally express, that is equally valuable evidence and should identify where the original architecture's abstraction boundary is insufficient.

### 12.10 Working proposition

The resulting proposition is deliberately narrower than the original speculation:

> **The interesting possibility is not that PP250 uniquely discovered objects. It is that PP250 may have put the hardware boundary in an unusually clean place: the hardware understands authority without needing to understand the semantic meaning of the objects over which that authority operates.**

That proposition now needs comparison against Hydra, KeyKOS, EROS, iAPX 432, System/38, PSOS, CAP, CHERI and other relevant capability architectures.

The objective is not to establish that PP250 was “first” or “better”, but to determine precisely what abstraction boundary each architecture chose, why it chose it, and what consequences followed.
