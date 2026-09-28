# Capability Genesis via Incomplete Outform — Working Reconstruction

## Status

**WORKING RECONSTRUCTION — NOT DOCUMENTED ARCHITECTURE.**

This note records the reasoning reached on 28 September 2026 about how the early System 250 Store Allocator might have manufactured the first capability for a newly created segment without requiring a general data-to-capability or capability-attenuation instruction.

It should be read alongside `capability-genesis-and-resource-lifecycle.md`. The purpose of this note is to preserve the argument chain so that it can be tested and, if necessary, falsified against primary material.

## 1. Facts and constraints already established

The reconstruction starts from the following documented observations or strong conclusions from the existing corpus:

1. England describes the **Store Allocator** as allocating a new segment and delivering a capability for it, and as the one place in the system where capabilities can be manufactured.
2. An Inform capability in primary store contains **ACCESS + SCT reference**.
3. An Outform capability uses the persistent/backing-store identity of the object rather than its current SCT reference.
4. An SCT entry is **not a capability**. It is object-side data containing such information as BASE, LIMIT, SUMCHECK and state/flag information.
5. ACCESS belongs to the individual capability. Different capabilities with different ACCESS may refer to the same SCT entry/object.
6. The SCT itself is below the ordinary H capability hierarchy and is defined through special processor capability state. Its entries are nevertheless data structures.
7. The machine already requires an ordinary virtual-store mechanism: a valid segment may have no primary-store allocation; an attempt to reference it traps, the system allocates/fetches primary store, updates the SCT, and execution continues.
8. Levy's description states that when a new segment is created, a disk address is assigned, an SCT entry is allocated, and the SCT initially indicates that no primary memory is allocated. The first reference traps and primary memory is then allocated.
9. The early architecture does not appear to provide a general ordinary-program access-reduction operation. Andrews/Wheatley's later work explicitly introduces access reduction as an enhancement. Therefore later LCM-style attenuation must not be projected backwards to explain early capability genesis.
10. Mixed data/capability storage later permits the same protected segment to be accessed by data operations under `WD` authority and by capability operations under capability authority. The Dump Stack is an important known example.

## 2. The problem to explain

The unexplained transition has been:

```text
ordinary/system data describing a new object
                |
                ?
                v
first legitimate capability
```

A general operation of the form

```text
arbitrary ACCESS + arbitrary existing object -> capability
```

cannot be available to ordinary H execution without destroying the security model: it would allow a process to encapsulate an existing object to which it had no authority.

The key insight is that **the security-critical operation is not choosing the ACCESS bits**. ACCESS to a genuinely new object can safely be maximal. The security-critical operation is binding those rights to storage containing pre-existing information.

Therefore a safe genesis mechanism need only guarantee:

> A newly manufactured capability can initially become bound only to a newly created, empty object, never to an arbitrary existing object.

## 3. Proposed mechanism: incomplete Outform as the creation request

The Store Allocator may not need a special `MAKECAP` instruction at all.

Instead it can construct, using ordinary data writes into storage for which it has `WD`, a **skeleton/incomplete Outform representation** describing the requested new capability.

Conceptually:

```text
ACCESS = desired initial authority
FORM   = Outform
OBJECT/BACKING IDENTITY = UNASSIGNED
SIZE   = requested size (wherever the architecture records it)
```

The exact encoding of `UNASSIGNED` is **unknown**. Zero is an obvious candidate for a backing-store address sentinel, but there is currently no documentary evidence that zero is the historical encoding. The reconstruction requires only a distinguishable unassigned state.

At this point the bits can have been constructed as **data**. They do not yet need to constitute usable authority over any existing object because no object identity has been assigned.

## 4. LC deliberately crosses into the normal Outform/virtual-store path

The Store Allocator then attempts to `LC` the skeleton representation.

The proposed sequence is:

```text
Store Allocator / H

construct skeleton Outform as data
        |
        | LC
        v
processor cannot complete normal capability load
because represented object is not yet backed/resident
        |
        v
ordinary virtual-store / not-present fault
```

**SUMCHECK is explicitly not part of this hypothesis.** A SUMCHECK failure represents SCT integrity failure and may take a much more severe recovery path. The required mechanism is analogous to a page/segment-not-present fault: a valid request whose represented object is not currently backed or resident in the required form.

## 5. M completes the binding

The fault transition leaves ordinary H execution and enters the M authority domain.

In the M/H/T reconstruction, M is not restricted to literal microcode. A fault handler reached only through M-controlled special processor state belongs to the M authority domain even if parts of the handler execute PP250 instructions as a process.

The handler can distinguish two conceptually different Outform cases:

```text
Outform + existing backing identity
        -> existing object; ordinary fetch/Inform path

Outform + UNASSIGNED backing identity
        -> new object; allocation path
```

For the second case M performs the security-critical operation:

```text
UNASSIGNED
     |
     v
allocate NEW backing-store object/identity
     |
     v
associate requested size/state
     |
     v
allocate/complete SCT representation as required
     |
     v
produce/complete the Inform representation
```

The essential invariant is that M never interprets `UNASSIGNED` as authority to an arbitrary existing object. It binds it only to fresh storage.

## 6. Retry completes ordinary capability loading

After the M-side allocation/completion, the original operation can be resumed or retried through the normal mechanism:

```text
skeleton/new Outform
        |
        | fault + M completion
        v
legitimate object representation
        |
        | retry LC / complete Inform
        v
C register
ACCESS + BASE + LIMIT
```

Once a legitimate loaded capability exists, normal `SC`/capability-storage mechanisms can deliver/store the capability for the requester.

From the caller's point of view the Store Allocator has simply:

```text
allocate segment -> return capability
```

The internal fault/representation protocol can remain invisible.

## 7. Why this would be safe

The Store Allocator may be able to choose any initial ACCESS for a **new** object without violating authority monotonicity, because there is no pre-existing authority over that object to exceed.

Safe case:

```text
arbitrary ACCESS
      +
UNASSIGNED / NEW object
      |
      v
M allocates fresh empty storage
```

Forbidden case:

```text
arbitrary ACCESS
      +
existing object X
      |
      v
new authority to X
```

The reconstruction therefore relocates the unforgeability requirement. The ACCESS bits themselves need not be secret or impossible to construct as data. What must be protected is the transition that binds them to an object identity.

## 8. Why mixed data/capability storage matters

A mixed segment provides a plausible place in which the Store Allocator can construct the provisional representation using data instructions under `WD` and then cause it to be interpreted by a capability operation.

This is analogous to the later Dump Stack problem: the underlying storage can contain both data state and capability state under different access disciplines.

This does **not** yet prove that the historical Store Allocator used a mixed segment in exactly this way. Chronology matters: England's 1974 description indicates mixed segments were an extension not yet implemented at that point. Therefore an earlier implementation may have used an equivalent protected construction structure or a different mechanism.

The reconstruction should consequently distinguish:

- the **semantic protocol**: incomplete new-object representation -> protected fault path -> fresh object binding -> legitimate capability;
- the **physical storage mechanism** used by a particular System 250 revision to hold the provisional representation.

## 9. Relationship to Inform and Outform

This reconstruction suggests that capability genesis may be better understood as creation of the **persistent object relationship**, with Inform merely being the active representation needed while the object participates in the current SCT.

Conceptually:

```text
GENESIS

ACCESS + NEW/UNASSIGNED persistent identity
              |
              | M binds fresh object
              v
ACCESS + persistent object identity
          (Outform semantics)


ACTIVE REPRESENTATION

ACCESS + persistent object identity
              |
              | Inform machinery
              v
ACCESS + SCT[n]
              |
              | LC
              v
ACCESS + BASE + LIMIT
```

The exact ordering may differ in the historical implementation: Levy says segment creation assigns a disk address and allocates an SCT entry as part of creation. That may mean backing identity and SCT allocation occur within the same Store Allocator/fault protocol rather than as separate externally visible phases. This does not invalidate the central idea; it constrains where the M-side completion occurs.

## 10. What this explains if correct

The reconstruction would explain several otherwise awkward facts together:

- why England can say that the Store Allocator is the one place capabilities are manufactured;
- why no general H-level `data -> capability` instruction need exist;
- why arbitrary ACCESS selection for a new object is safe;
- why capability manufacture cannot be used to steal an existing object's contents;
- why the normal Outform/Inform virtual-store machinery is relevant to capability genesis;
- why later Andrews access-reduction machinery solves a different problem: deriving reduced authority to an **existing** object rather than creating initial authority to a **new** object;
- why process creation can sit above Store Allocation: the Process Allocator can request a Dump Stack segment/capability from the Store Allocator rather than manufacture that capability itself.

## 11. Points that could falsify or materially alter the reconstruction

The hypothesis should be attacked rather than assumed. In particular, it is weakened or falsified if primary evidence shows any of the following:

1. The Store Allocator directly writes a legitimate Inform capability by a documented special instruction that already explains genesis.
2. Outform capability recognition requires an already legitimate capability representation and cannot begin from a provisional data-written representation.
3. The virtual-store/not-present fault path cannot be invoked during capability loading or cannot resume/retry the relevant operation.
4. No distinguishable unassigned/new-object state exists in the representation or allocation protocol.
5. The fault handler lacks sufficient protected authority to allocate/bind a fresh backing object and establish the SCT representation.
6. Early System 250, before mixed segments, used a capability-manufacturing mechanism incompatible with the semantic protocol above.

## 12. Immediate research questions

The next documentary work should concentrate narrowly on:

1. What exact SCT state triggers the System 250 equivalent of a page/segment-not-present fault?
2. Does `LC` itself encounter that state while resolving a capability, or is the fault delayed until a later data/code reference through the loaded capability?
3. What MIP/MIF indication identifies that fault?
4. Which special capability/register (`C(N)`, `C(S)`, or another path) is used to enter its handler?
5. What information about the faulting capability, SCT entry and destination C register is preserved for the handler?
6. How does the handler distinguish an existing outformed object from a newly created object requiring first backing-store allocation?
7. At what point is the unique disk/backing-store identity assigned during Store Allocator operation?
8. What exact mechanism allows the early pre-mixed-segment implementation to construct the first capability, if mixed data/capability storage was not yet available?

Until these are resolved, this note remains a **working reconstruction**, not a claim about documented PP250 implementation.