# Kenneth J. Hamer-Hodges's Statements About the PP250 in the Church Machine Repository

**Source repository:** `khhodges/cloomc-project`  
**Prepared for:** PP250-Reboot  
**Purpose:** Source summary, not an interpretation of the PP250-Reboot reconstruction  
**Date reviewed:** 2026-09-22

## Scope and method

This note records what Kenneth J. Hamer-Hodges's Church Machine repository says about the Plessey PP250 / System 250. It deliberately separates those statements from the PP250-Reboot project's own reconstruction hypotheses.

The Church Machine repository is a modern design project. Much of it describes CLOOMC, Golden Tokens, LUMPs, the Namespace, modern FPGA implementations, and other mechanisms that did not exist in the PP250. This document therefore records both:

1. statements explicitly made about the historical PP250; and
2. statements explicitly describing Church Machine features as inherited from, descended from, or continuing PP250 ideas.

It does **not** assume that a modern Church Machine mechanism existed in the PP250 merely because it appears in a section bearing a PP250 heading.

The Church Machine repository itself identifies `docs/cloomc-foundation.md` as the authoritative architectural overview covering “the PP250 heritage” as well as the modern architecture.

---

## The objective of the Church Machine / CLOOMC project

The Church Machine project is not an attempt to reconstruct the PP250. Hamer-Hodges's stated objective is substantially broader: to take what he regards as the fundamental architectural principle demonstrated by PP250 — capabilities as the machine's only authority mechanism — and generalize it into a modern general-purpose computer architecture.

In the project's own framing, PP250 demonstrated that a fully capability-mediated machine could operate successfully in the telecommunications domain. Church Machine is intended to carry that approach into general computing.

Its principal objectives are to:

- eliminate raw-address and ambient-authority escape paths so that access to resources is capability-mediated;
- separate **authority** (the “Church” domain) from ordinary **data computation** (the “Turing” domain);
- make software components capability-confined abstractions whose accessible world is determined by the capabilities supplied to them;
- enforce those boundaries in hardware rather than relying upon a conventional privileged operating-system or cybersecurity layer;
- support revocation, fault containment, recovery and long-lived replaceable abstractions;
- extend capability authority beyond local memory to storage and remote/network resources;
- make the capability graph and system structure visible through an integrated development environment; and
- demonstrate the architecture in practical hardware as well as in the simulator and IDE.

The repository's broader thesis is that conventional computers place a Turing/von-Neumann machine at the centre and subsequently add protection mechanisms around it. Church Machine reverses that relationship: the capability/abstraction system is intended to define the computer, while conventional mutable computation operates inside capability-defined boundaries.

`docs/prologue.md` summarizes the intended division by saying that **the Turing domain handles computation while the Church domain handles authority**, and describes this as an insight pioneered by PP250 and completed by Church Machine.

A concise statement of the project's objective is therefore:

> **To build a practical modern general-purpose computer in which authority is a first-class hardware concept, derived only from capabilities, with conventional computation permanently confined inside capability-defined boundaries — generalizing the architectural approach first demonstrated by PP250.**

This objective provides the context for the repository's Golden Tokens, Namespace, LUMPs, CLOOMC instruction set, IDE and FPGA implementation. They are modern mechanisms intended to pursue that objective, rather than claims about the detailed implementation of the historical PP250.

---

## 1. Hamer-Hodges's overall characterization of PP250

### 1.1 “First immersive capability computer”

In `docs/cloomc-foundation.md`, Hamer-Hodges describes the PP250 as:

> “The First Immersive Capability Computer (PP250)”

He states that the PP250 was the first immersive capability computer successfully fielded commercially and says that it operated in production for approximately two decades.

The same document says that every PP250 object was accessed through a hardware-validated capability key, which it also calls a **descriptor**, and that a program could not reach memory for which it did not hold such a descriptor.

This is the central historical characterization repeated throughout the repository: PP250 was not a conventional computer with capability protection added to selected operations. Capability addressing was the machine's basic access model.

### 1.2 Full immersion rather than capability “patching”

`docs/prologue.md` says that PP250:

- had no raw-memory-address escape mechanism;
- required a capability for every memory access;
- did not place a conventional unrestricted machine beneath the capability system;
- had no “God-mode” account or superuser able to bypass the capability structure.

The repository repeatedly contrasts this with later architectures that retain conventional addressing and overlay capability protection on top.

In Hamer-Hodges's terminology, PP250 is therefore a **full-immersion capability architecture**.

### 1.3 Commercial and operational use

The foundation document says PP250 was successfully fielded commercially, remained in production for about two decades, and served during the first Gulf War.

It also states that there was no reported security breach of the capability model during that operational life.

These are historical claims made by the repository; this note does not independently verify them.

---

## 2. The PP250 capability model

### 2.1 Capabilities as the only route to objects

Across the foundation, prologue, manifesto and architectural comparison material, Hamer-Hodges characterizes the PP250 principle as:

**No capability → no path → no access.**

The machine is described as enforcing access in hardware rather than depending upon software policy.

### 2.2 No raw memory addresses

`docs/prologue.md` explicitly states:

> “No raw memory addresses.”

It explains this as every memory access requiring an unforgeable capability specifying the accessible object and permitted operations.

This is presented as one of the defining differences between PP250 and conventional von Neumann machines.

### 2.3 No privileged-mode bypass

`docs/cloomc-foundation.md` identifies as inherited from PP250 the rule that every memory access is capability-mediated and that there is no privileged mode which simply bypasses the capability check.

The modern Church Machine contains its own special hardware mechanisms, but Hamer-Hodges presents **absence of a software privilege bypass around capability enforcement** as PP250 heritage.

### 2.4 Enter capability and the continuing distinction between Enter and Execute

The Church Machine retains **Enter** as a distinct capability permission.

Its modern permission model separates the Turing-domain permissions **R/W/X** (Read, Write, Execute) from the Church-domain permissions **L/S/E** (Load, Save, Enter). The two domains are deliberately distinct.

An **E-GT** (a Golden Token carrying Enter authority) gives its holder authority to enter or call an abstraction. This is the modern Church Machine descendant of the PP250 **Enter capability** concept: authority to invoke a protected domain without thereby granting ordinary read, write, or execute access to its underlying representation.

The important continuity is that **Enter and Execute remain different kinds of authority**. Execute belongs to ordinary code execution in the Turing domain; Enter belongs to protected capability-mediated transfer of control in the Church domain.

In simplified form:

```text
PP250                         Church Machine

Enter capability       →      E-GT / E permission
       │                           │
       └──── controlled entry ─────┘
                    │
                   CALL
```

This is one of the clearest examples in the repository of a specific PP250 capability concept surviving into the Church Machine rather than merely influencing its general philosophy.

### 2.5 Capability hardware

The foundation document calls PP250 capabilities **hardware-enforced capability keys** and says the original descriptors were “wide hardware words.”

It contrasts those with the modern Church Machine's new 32-bit Golden Token representation.

The important historical claim is therefore not that PP250 used Golden Tokens—it did not—but that capability information had a representation recognized and enforced by the hardware.

---

## 3. Descriptor table, segment table and the SCT

Hamer-Hodges uses the terms **descriptor table** and **segment table** when discussing the PP250.

`docs/cloomc-foundation.md` says:

- the PP250 descriptor table was the **system map**;
- the Church Machine Namespace table is the **direct descendant of the PP250's segment table**.

In PP250 terminology the structure performing this role is the **System Capability Table (SCT)**. Accordingly, this note treats Hamer-Hodges's “PP250 segment table” and “descriptor table” as references to the SCT unless evidence emerges that he intended a different PP250 structure.

This is one of the repository's clearest statements of architectural lineage:

**PP250 SCT / descriptor table / segment table → Church Machine Namespace table.**

Hamer-Hodges therefore sees the central authoritative table of the modern machine not as a new concept but as a continuation of the PP250 system map.

---

## 4. PP250 and the Church/Turing distinction

The Church Machine project divides computation into a protected **Church** capability/authority domain and a **Turing** data-computation domain.

The repository presents this as a development and generalization of the PP250 architectural idea rather than claiming that the historical PP250 literally implemented the modern CLOOMC ISA.

`docs/prologue.md` says:

> “The Turing domain handles computation. The Church domain handles authority.”

It then says:

> “This is the insight that the PP250 pioneered and the Church Machine completes.”

The modern implementation has separate data registers and capability registers, and ordinary data computation cannot manufacture capabilities. Hamer-Hodges uses this modern architecture to express what he regards as the fundamental PP250 idea: authority is a distinct machine concern, enforced by hardware rather than represented merely as arbitrary binary data.

The modern instruction names and exact Church/Turing division are new and must not be attributed to PP250 without historical evidence.

---

## 5. Operating system and superuser statements

`docs/prologue.md` describes PP250 as having:

> “No operating system.”

and:

> “No superuser.”

The literal “no operating system” wording requires contextual reading because System 250 documentation describes COS, POS, ROS and PDOS. Hamer-Hodges's surrounding argument is directed against a **central privileged operating-system authority** that can bypass protection.

His substantive claim is that authority in PP250 was distributed through capabilities rather than derived from an all-powerful supervisor or user identity.

The “no superuser” claim is consistent with the same architectural argument: there was no account whose identity alone conferred unrestricted machine-wide access.

This section records Hamer-Hodges's wording without converting it into a claim that no PP250 operating-system software existed.

---

## 6. Garbage collection and resource lifetime

`docs/prologue.md` attributes **deterministic garbage collection** to PP250.

It says resources belonging to terminated calls were reclaimed using a **four-phase hardware GC cycle**, and describes the purpose as ensuring that capabilities referring to deallocated resources were invalidated atomically.

Other Church Machine documentation contains sections labelled “Garbage Collection (PP250)” while describing modern Namespace entries, Golden Token sequence numbers and Church Machine implementation details. Those modern details must not be read as historical PP250 mechanisms.

The historical claim made by Hamer-Hodges is therefore:

- PP250 performed deterministic resource reclamation;
- it used a four-phase collection process;
- stale capability authority had to be invalidated as resources were reclaimed.

The precise original PP250 mechanism should be checked against contemporary documentation.

---

## 7. Fault handling and recovery

Fault recovery is one of the strongest recurring PP250 themes in the Church Machine repository.

### 7.1 Recovery as a hardware architectural property

`docs/pp250-fast-boot.md` says that PP250 responded to faults with hardware-enforced recovery rather than waiting for firmware or a conventional software supervisor.

The document presents rapid automatic recovery as a PP250 principle which the Church Machine deliberately inherits.

### 7.2 Claimed three-instruction recovery

The same document makes the specific historical claim that PP250 responded to a fault with a **three-instruction boot sequence**, with no countdown, firmware handshake or polling loop, and says recovery was extremely fast.

This is a specific retrospective statement and should be checked against Hamer-Hodges's contemporary fault-recovery paper and other original PP250 sources.

### 7.3 Cold boot and fault recovery

`docs/cloomc-foundation.md` contains a section titled “PP250 Fault Recovery” and states that the PP250 design makes no distinction between cold boot and fault recovery.

However, the section then describes Golden Tokens, FPGA bitstreams, BRAM, Church Machine firmware and other modern mechanisms.

Consequently there are two different things in this passage:

1. a stated PP250 principle: recovery and startup follow the same fundamental restoration path;
2. a modern Church Machine implementation of that principle.

The latter is not historical PP250 evidence.

### 7.4 State is not trusted after a fault

The modern recovery design discards processor state and reconstructs valid capability state from trusted initial information. The repository explicitly presents this as being in the PP250 recovery tradition.

Whether the original PP250 performed the same individual steps must be established from contemporary sources.

---

## 8. Boot and initial authority

The Church Machine foundation document says that the capability laws apply from the instant power is applied and explicitly presents this as part of the PP250-derived capability philosophy.

The modern Church Machine realizes this with a trusted boot image, Namespace, Thread state and a hardware boot sequence.

Those particular mechanisms are modern.

What Hamer-Hodges attributes to PP250 heritage is the stronger architectural rule that **there must not be an early boot interval in which normal capability enforcement is simply absent**.

The Church Machine's modern transient Machine/Microcode authority (“M”) is likewise a modern mechanism. It demonstrates Hamer-Hodges's current distinction between powers internal to protected machine implementation and powers available to executing software, but the repository does not by itself establish that historical PP250 used an identical M mechanism.

---

## 9. Segments and secondary storage

`docs/cloomc-foundation.md` explicitly distinguishes Church Machine LUMPs from PP250 storage management.

It says:

> “They did not exist in the PP250. The PP250 loaded segments from disk…”

This is useful because it is an explicit negative as well as positive historical statement:

- LUMP architecture is new;
- PP250 used segments;
- those segments could be loaded from disk.

The modern Church Machine's fetch/load mechanisms must therefore not be projected backwards onto PP250.

---

## 10. Inform and Outform terminology

The modern Church Machine uses Golden Token types including **Inform** and **Outform**, especially in its network-transparency architecture.

PP250 documentation also uses Inform/Outform terminology, but the modern Golden Token representation and network machinery are explicitly new.

The Church Machine repository therefore demonstrates continuity of terminology and concept in Hamer-Hodges's thinking, but its modern HTTPS/RPC, seals, token formats and Namespace representation are not historical PP250 mechanisms.

Any claim about the precise PP250 Inform/Outform operation should come from the original PP250 sources rather than be reconstructed from the modern implementation.

---

## 11. What Hamer-Hodges says was new after PP250

The foundation document is particularly valuable because it explicitly lists things that **did not exist in PP250**.

### 11.1 LUMPs

The Church Machine LUMP architecture is new. PP250 instead loaded segments from disk.

### 11.2 CLOOMC ISA

The CLOOMC instruction set is a new Church Machine design.

Modern instructions such as LOAD, SAVE, CHANGE, SWITCH, TPERM and LAMBDA therefore cannot simply be assigned to PP250 because Hamer-Hodges uses them to express related capability ideas.

### 11.3 Golden Tokens

The modern 32-bit Golden Token encoding is new.

The foundation contrasts it with PP250's wider hardware descriptors.

Consequently the following are modern rather than PP250 features unless separately documented historically:

- 32-bit GT layout;
- modern token type fields;
- GT sequence/version fields;
- modern integrity/seal mechanisms;
- Mint's exact implementation;
- the modern Namespace representation.

---

## 12. Direct architectural lineage claimed by Hamer-Hodges

The repository does not present Church Machine as merely inspired by PP250. `docs/cloomc-foundation.md` calls it PP250's **direct architectural successor** and says the lineage is architectural rather than metaphorical.

The explicit lines of continuity claimed across the repository are:

1. **Capability-only access** — authority to memory/resources is represented by hardware-enforced capabilities.
2. **No raw-address escape path** — capability protection is immersive rather than optional.
3. **No privileged software bypass** — ordinary privilege does not override capability checks.
4. **Separate treatment of authority and ordinary computation** — developed explicitly in Church Machine as Church versus Turing domains.
5. **Authoritative system map** — PP250 SCT/descriptor/segment table develops into the Church Machine Namespace.
6. **Hardware participation in protection** — capability validity is a machine property rather than an OS convention.
7. **Fault containment and automatic recovery** — rapid hardware recovery is treated as PP250 heritage.
8. **Capability-defined confinement** — a computation can act only within authority explicitly available to it.

---

## 13. Claims about PP250's historical significance

The repository makes several broad historical judgments.

Hamer-Hodges describes PP250 as:

- the first commercially fielded immersive capability computer;
- proof that a capability-only architecture could work in production;
- a successful rather than failed capability-machine experiment;
- a “road not taken” when mainstream computing adopted conventional operating-system and privilege models;
- the practical predecessor which Church Machine is intended to complete and generalize.

`docs/fulcrum-of-computer-science.md` says System 250 implemented capabilities in hardware in 1969.

`docs/prologue.md` uses 1972 for PP250's entry into production.

These dates describe different milestones in the repository's narrative rather than necessarily constituting a contradiction, but they should be reconciled against primary historical sources before being used as a formal chronology.

---

## 14. Reliability and security claims

The foundation document says PP250 accumulated operational evidence demonstrating that the capability model was practical.

The repository attributes to PP250:

- long production service;
- mission-critical telecommunications use;
- capability-mediated isolation;
- rapid fault recovery;
- deterministic resource reclamation;
- absence of reported breaches of the capability model.

These should be understood as Hamer-Hodges's retrospective claims. They are important historical testimony but should be independently corroborated where possible.

---

## 15. Comparison with later capability architectures

The Church Machine material groups PP250 and Church Machine together as **full-immersion** capability machines and contrasts them with architectures it characterizes as capability “patching.”

The comparison says full-immersion systems have:

- no conventional memory model available alongside capabilities;
- no raw pointer escape;
- capabilities as the only machine interface to protected resources;
- a total rather than partial capability boundary.

The repository contrasts this with CAP, iAPX 432 and CHERI. Those comparisons are Hamer-Hodges's architectural argument and should not be treated as a neutral historical assessment of those other machines without separate review.

For the purposes of PP250 history, the relevant point is how he classifies PP250: **capabilities were the machine model, not an optional protection facility.**

---

## 16. Source-quality notes

The repository mixes four kinds of material:

### A. Explicit PP250 historical statements

Examples:

- PP250 used hardware capabilities/descriptors.
- every memory access was capability-mediated;
- PP250 used a segment/descriptor table as the system map;
- PP250 loaded segments from disk;
- PP250 used deterministic garbage collection;
- PP250 had hardware fault recovery.

These are direct retrospective testimony and should be recorded as such.

### B. Explicit lineage statements

Examples:

- Church Machine Namespace is a direct descendant of the PP250 segment table/SCT;
- Church Machine inherits PP250 capability enforcement;
- Church Machine inherits the PP250 fault-recovery principle;
- Church Machine is described as PP250's direct architectural successor.

These tell us how Hamer-Hodges himself understands the relationship between the two machines.

### C. Modern mechanisms implementing an inherited principle

Examples:

- Golden Tokens;
- LUMPs;
- Mint;
- modern Namespace versioning;
- modern three-instruction Church Machine boot ROM;
- modern M-elevation;
- FPGA BRAM recovery;
- modern Inform/Outform network tunnels.

These may illuminate the lineage but are not themselves evidence of PP250 implementation.

### D. Broad historical or argumentative claims

Examples include the “road not taken” narrative, comparisons with Unix/CHERI/CAP, cybersecurity claims and claims about why the industry selected other architectures.

These are useful for understanding Hamer-Hodges's position but should not be mixed with technical PP250 evidence.

---

## 17. Files in Ken's repository containing important PP250 material

The following files were particularly significant in this review:

- `docs/cloomc-foundation.md` — principal source for PP250 heritage and explicit old/new distinctions.
- `docs/prologue.md` — historical narrative and strongest concise description of PP250 as a full-immersion capability machine.
- `docs/pp250-fast-boot.md` — explicit PP250 fault-recovery claims and their influence on modern recovery.
- `church_machine_final_design.md` — modern architecture and several PP250-labelled/inherited concepts; useful but requires separation of modern implementation from historical claims.
- `docs/fulcrum-of-computer-science.md` — historical positioning of System 250 and capability hardware.
- `attached_assets/CLOOMC_Manifesto.md` — philosophical statement of unforgeable capability authority, with PP250 used as historical precedent.
- `docs/overview.md` — concise current Church Machine architectural rules useful for distinguishing the modern design from PP250.
- `replit.md` — repository guide identifying `cloomc-foundation.md` as the authoritative document for PP250 heritage.

Other Church Machine documents discuss mechanisms whose terminology overlaps PP250—CALL/RETURN, capability registers, Inform/Outform, garbage collection and fault recovery—but should be treated primarily as Church Machine documentation unless they explicitly make a PP250 claim.

---

## 18. Concise statement of Ken Hamer-Hodges's PP250 account

Taken on its own terms, Hamer-Hodges's 2026 repository describes PP250 as a commercially successful, fully immersive hardware capability machine in which capability authority—not raw addressing, user identity or supervisor privilege—was the fundamental means of reaching resources.

Its descriptor/System Capability Table was the machine's authoritative system map. Programs operated within capability-defined authority; there was no ordinary raw-memory escape path or superuser bypass. Segments were loaded from disk. Resource lifetime was managed with deterministic garbage collection, and fault recovery was designed as a rapid hardware-supported operation rather than conventional software-supervisor recovery.

Hamer-Hodges regards the Church Machine as a direct architectural successor to PP250, particularly through capability-only authority, the SCT-to-Namespace lineage, hardware enforcement, confinement and fault recovery. At the same time, he explicitly identifies LUMPs, CLOOMC and the modern 32-bit Golden Token representation as new designs rather than PP250 mechanisms.

That distinction—**PP250 principle versus modern Church Machine implementation**—is essential when using this repository as historical evidence.

---

## 19. Recommended use in PP250-Reboot

This document should be treated as a **secondary/retrospective source index**.

Where a Hamer-Hodges statement agrees with a contemporary PP250 paper, manual, patent or pocket reference, the contemporary source should normally be cited as the primary architectural evidence and Hamer-Hodges's 2026 statement as useful corroboration.

Where his repository contains a PP250 statement not yet found in contemporary material, it should be retained explicitly as **Hamer-Hodges retrospective testimony** and investigated rather than discarded or silently promoted to established fact.

This is especially important because Hamer-Hodges was directly involved with the original architecture: his modern repository is unusually valuable testimony, but it is also separated from the original work by more than fifty years and is embedded in the design of a new machine.

---

## Repository references

- Kenneth J. Hamer-Hodges, `khhodges/cloomc-project`, GitHub.
- `docs/cloomc-foundation.md`, v1.1, 2026-05-15.
- `docs/prologue.md`.
- `docs/pp250-fast-boot.md`.
- `docs/fulcrum-of-computer-science.md`.
- `church_machine_final_design.md`.
- `attached_assets/CLOOMC_Manifesto.md`.
- `docs/overview.md`.
- `replit.md`.

