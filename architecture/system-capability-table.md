# System 250 Architecture — System Capability Table

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

### Already-expanded capabilities during relocation

**DOCUMENTED OBSERVATION:** US3771146A, Description 121, identifies the hazard of a processor retaining a capability expanded before the relocation decision. The relocating process interrupts the affected processors. Entering the interrupt handler and returning to the interrupted process reloads the workspace capability registers through the reserved pointers in its Dump Stack. While the MCT sumcheck is zero, those loads produce unusable capability registers; attempted use then traps.

Thus an SCT update alone does not invalidate previously expanded registers. The described relocation mechanism coordinates process transitions to refresh them. The complete acknowledgement/barrier protocol, active-channel coordination, software-version replacement and safe SCT identity reuse are not established by this passage. See the [batch source review](../research/pp250-patent-transcriptions-review-2026-10-02.md).

### Active versus passive representation

Do not equate "segment is not currently resident" with "every reference to it has type `10`." An active (`11`) reference identifies an SCT entry and can remain meaningful while the SCT entry is unavailable. A passive (`10`) representation instead carries backing-store identity/address information and is used when authority itself has been converted to its backing-store/outform representation.

When capability-containing blocks are moved between main store and backing store, embedded capability representations may therefore require inform/outform conversion. The SCT identity mechanism allows active references elsewhere in main store to remain stable across relocation/page movement.
