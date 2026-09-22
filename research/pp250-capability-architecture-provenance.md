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
