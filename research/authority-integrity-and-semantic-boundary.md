# Authority Integrity and the Semantic Boundary

Status: Working architectural hypothesis
Project: PP250-Reboot

## Core principle

> **Hardware protects authority; software defines meaning.**

Software may decide what an abstraction represents and what its operations mean. Those semantics may be wrong, buggy, or compromised without thereby granting new authority.

The trusted mechanism must ensure that ordinary computation cannot manufacture, amplify, or escape the authority legitimately granted to it.

## Argument chain

This principle emerged through a sequence of progressively narrower questions.

### 1. PP250 looked like more than memory protection

The CCB, Enter, and CALL suggest structures resembling objects: a protected capability environment, private state, callable entry points, references to other protected abstractions, and a graph of reachable authority.

This suggested that capability architecture might be capable of being the structural model of software rather than merely protecting software built using another model.

### 2. The first historical hypothesis was too strong

An initial possibility was that capability researchers had simply failed to recognize the object connection.

Comparison with Hydra, KeyKOS, and EROS showed that this was wrong. Researchers explicitly understood capabilities as protected references to objects and interfaces.

### 3. The second hypothesis was also too strong

It was then possible to ask whether researchers understood objects but had failed to move the model into hardware.

The iAPX 432, System/38, and PSOS showed that this was also too strong. Rich object concepts had been pushed toward or into machine architecture.

### 4. The question became one of semantic level

Architectures appear to protect authority at different semantic levels:

```text
lower semantic content                         higher semantic content

memory -> resource -> domain -> object/interface -> typed object
```

Software capability systems can place rich object/interface authority above conventional hardware. Hardware object machines can make object types and operations architectural concepts.

PP250 appears potentially interesting because its hardware authority may sit between those extremes: richer than raw bounded memory, because it includes domains and protected Enter/CALL, but below application object meaning.

### 5. An apparent PP250 limitation became potentially important

A capability does not inherently know the meaning of the protected bits. The machine need not know that a structure is a BankAccount, that an entry means Withdraw, or that a segment contains an invoice.

Embedding such meaning in hardware couples the machine to particular software abstractions. The absence of semantic meaning may instead be what makes the authority mechanism general.

### 6. The critical objection: software can fail

If meaning moves upward into software, a runaway or faulty process can corrupt that meaning. This requires distinguishing two different forms of integrity.

## Semantic integrity versus authority integrity

### Semantic integrity

Software owns meaning: what an object represents, what an entry means, and what a service is intended to do.

A program can contain a semantic bug and be wrong without acquiring authority over anything outside its legitimate capability closure.

### Authority integrity

The trusted capability mechanism owns facts such as:

- a capability is genuine;
- it designates a particular protected resource or domain;
- it carries defined rights;
- ordinary data cannot become a capability;
- authority cannot be amplified by ordinary derivation;
- CALL may enter only an authorised target;
- the entered domain receives only its legitimate capability environment.

These facts must not depend on ordinary writable software data.

A process may be completely faulty and still be confined if it cannot manufacture, amplify, or escape its authority.

## Three possible boundary placements

### Boundary too low

```text
HARDWARE
    memory bounds

SOFTWARE
    object identity
    authority
    permissions
    protected invocation
    authority transitions
```

If software must enforce the authority model itself, failure of the enforcing software may compromise authority.

### Boundary too high

```text
HARDWARE
    application object types
    method semantics
    system abstractions
    authority
```

Authority is protected, but the processor becomes coupled to high-level software abstractions and programming models.

### Candidate minimal boundary

```text
SOFTWARE MEANING
    application abstractions
    object types
    operation meanings
    business semantics
--------------------------------
HARDWARE AUTHORITY
    unforgeable identity
    legitimate possession
    permissions
    controlled derivation
    protected invocation
    authority transition
    protected capability environment
```

The hardware need not know what an object means. It must ensure that software cannot obtain authority it was never granted.

## PP250 example

Consider a software abstraction interpreted as a bank account:

```text
C6 -> protected capability environment

        C0 -> private balance data
        C1 -> transaction log
        C2 -> banking service
        C3 -> another legitimate resource
        C7 -> implementation code
```

The term BankAccount need not exist anywhere in the processor.

An external caller may possess only an Enter capability. A faulty caller still should not thereby be able to manufacture data-access authority to the balance, acquire capability-write authority to the protected environment, fabricate a capability to the transaction log, convert arbitrary data into one of those capabilities, or acquire unrelated authority absent from its legitimate capability graph.

Capability architecture does not make programs correct. It constrains the authority available to a faulty component.

## Fault containment by construction

A component can be logically wrong, corrupted, stuck in a runaway loop, or fed hostile input without necessarily gaining additional authority.

The architectural guarantee sought is not:

> software cannot fail.

It is:

> **software failure does not itself create authority.**

This separates program correctness from authority correctness.

## Why richer hardware semantics do not automatically provide stronger authority

Putting an application concept such as a BankAccount, File, or Semaphore into the architecture does not necessarily make the authority model stronger.

The essential hardware question is not what the protected thing means. It is:

> **What authority exists over it, how was that authority obtained, and can ordinary computation exceed it?**

If authority integrity is preserved independently of semantic meaning, software can introduce abstractions never anticipated by the processor designer without requiring a new protection mechanism.

## Candidate design rule

For every mechanism proposed above the trusted authority boundary, ask:

> **If the software implementing this mechanism fails completely, can it manufacture, amplify, or escape its legitimate authority?**

If no, the mechanism may remain above the boundary.

If yes, some invariant on which that mechanism relies must be enforced below the boundary.

The proposed rule is:

> **Put only the mechanisms required to preserve authority integrity below the trusted boundary. Leave meaning above it.**

## Consequences for top-down design

The Top-Down Capability-Native Design programme should therefore avoid beginning with "How do we implement objects in hardware?"

Instead it should ask:

> **What is the lowest-semantic-level authority substrate from which the required software abstractions can be constructed while remaining confined even when their implementation fails?**

Candidate required properties currently include non-forgeable capability identity, inability of ordinary data computation to create authority, controlled capability derivation and attenuation, protected capability storage, protected invocation, controlled authority transition on invocation, confinement to reachable authority, lifecycle/revocation mechanisms, and a legitimate bootstrap path for the first authority.

Whether PP250 supplies exactly this set is a research question, not an assumption.

## Comparative questions

For each relevant capability architecture ask:

1. What authority does hardware itself recognize?
2. What authority is enforced by trusted software?
3. What semantic meaning does hardware need to understand?
4. Can faulty ordinary software manufacture or amplify authority?
5. What mechanism mediates protected invocation?
6. What authority becomes available after invocation?
7. Is there any privileged escape path outside the capability model?

These questions should be applied to PP250, CAP, Hydra, KeyKOS, EROS, System/38, iAPX 432, PSOS, CHERI, and relevant later systems.

## Relationship to the Authority Machine thesis

The Authority Machine thesis is therefore not simply an investigation of whether capabilities can support objects.

It asks whether a general-purpose computer can place its trusted architectural boundary at authority integrity while leaving semantic meaning above that boundary.

```text
TOP-DOWN SOFTWARE REQUIREMENTS
             |
             v
what must remain true even if
the software fails?
             |
             v
AUTHORITY INVARIANTS
             |
             v
minimum trusted mechanisms
             |
             v
candidate hardware boundary
             |
             v
compare with reconstructed PP250
```

If that independent derivation converges on PP250-like mechanisms, the convergence is evidence that they may be architectural invariants rather than historical artifacts.

If it does not, the divergence identifies what a modern authority machine requires beyond PP250.

## Working proposition

> **Meaning may be mutable, buggy, and application-defined. Authority must remain unforgeable.**

The architectural principle derived from it is:

> **Hardware protects authority; software defines meaning.**

The purpose of the research is to determine the minimum mechanism required to make that statement true for a useful general-purpose computer.
