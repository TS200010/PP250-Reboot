# Capability Genesis and Resource Lifecycle in System 250

## Status

This note records the current reconstruction developed from the published System 250 architecture, patents, and discussion of the PP250-Reboot project. It deliberately distinguishes **documented behaviour** from **architectural inference/hypothesis**.

The central method is: **give me the data structures and I will give you the algorithms**. The known hardware structures constrain quite strongly how the missing operating-system mechanisms could have worked.

## 1. The missing architectural layer

Published descriptions normally begin with a functioning capability system: processes exist, the System Capability Table (SCT) exists, capability registers can be loaded, and Store Allocator and other protected packages are operating.

That leaves a genesis question:

> Where does the first authority come from from which all subsequent capabilities are derived?

Ordinary capability derivation cannot provide the answer indefinitely. There must be a root of authority established by the machine/start-up environment.

### Working hypothesis: primordial resource allocator

A particularly simple model is a persistent **Primordial Resource Allocator (PRA)**.

At initialisation it receives maximal authority over the allocatable physical resource namespace. Conceptually this is the "whole machine" capability: all six relevant permissions over resources not reserved for architectural machinery such as the SCT/recovery structures.

The PRA remains alive, but its authority monotonically decreases. As resources are admitted it derives appropriate authority, transfers it to specialised resource handlers, and relinquishes its own authority over that resource.

```
                         Primordial Resource Allocator
                                     |
                         initial physical authority
                                     |
          +--------------------------+-----------------------+
          |                          |                       |
       memory                     disk                  communications
          |                          |                       |
          v                          v                       v
   Store Allocator             Disk handler           Comms handler
```

## 2. Everything is an addressable resource

At the lowest architectural level there need not be a strong distinction between store, high-speed peripherals, communications interfaces, controllers, or other resources exposed through the system addressing mechanism.

The architectural principle becomes:

> **Everything controllable through the address space is an addressable resource; authority over it is represented by capabilities.**

Software interpretation comes later.

## 3. One primordial capability versus a root capability set

The aesthetically simplest model is one capability covering the entire physical resource address space with all permissions:

```
RD WD X RC WC EC
```

The System 250 physical address includes a module component in addition to the address within a module. However, the documented loaded capability representation appears to contain a Store Module address separately from base and limit. It remains open whether one historical PP250 capability could span module boundaries.

Two representations therefore remain possible:

1. **Literal single root capability** spanning the physical resource namespace.
2. **Root capability set**, with maximal authority for each module/resource.

These are equivalent at the authority-model level; the historical representation must decide which PP250 permitted.

## 4. Resource discovery is not authority creation

A running system could acquire new hardware. Discovery of a new module does not by itself have to confer authority over it.

```
new module appears
       |
configuration/idle/system process discovers it
       |
Primordial Resource Allocator invoked
       |
authority for that resource is isolated/derived
       |
specialised resource handler receives it
       |
PRA relinquishes that authority
```

Possible discovery opportunities include an idle process, periodic interval-timer activity, scheduler activity, configuration management, or operator action. Which historical mechanism was used remains to be established.

## 5. Processor admission is different

A new processor introduces another execution engine, not necessarily new authority.

There is a project recollection that a processor could be brought online simply by **faulting it**. This fits the fault model:

```
processor offline/new
       |
fault
       |
hardware/microcode fault sequence
       |
valid System 250 process established
       |
processor joins existing work system
```

The processor acquires authority from the process state it executes rather than permanent processor-specific privilege. The original wording/document still needs to be located.

## 6. Scheduling, watchdog and interval activity

Known/recalled structures suggest that processes execute until a scheduling event such as blocking/waiting, yielding/change-process activity, interval timer activity, or watchdog expiry/fault.

The Watchdog Timer is part of process state and provides runaway-process containment. Interval timing provides a separate source of normal system activity. System 250 processors share work through a common work list.

Periodic resource discovery therefore need not require conventional device interrupts.

## 7. Peripheral removal

If a peripheral is physically removed, existing capabilities and its SCT entry do not magically disappear.

An attempted access can still:

1. pass the capability access check;
2. resolve through the SCT;
3. issue a transaction to the module address;
4. receive no valid response from the physical module;
5. cause a hardware/bus fault.

The normal fault machinery can therefore detect disappearance. Recovery/resource management can mark the resource unavailable and begin revocation/reclamation.

**Physical disappearance and logical capability invalidation are separate events.**

## 8. SCT entries are resources

The SCT is not merely a passive lookup table. SCT entries themselves form a finite managed resource.

For a new segment, the Store Allocator conceptually obtains:

```
physical resource + free SCT entry
                  |
                  v
             system object
                  |
                  v
        capability (rights + SCT reference)
```

Published descriptions associate the Store Allocator with allocating storage, obtaining an SCT entry, populating it with the physical segment description, and producing the corresponding capability.

Thus there are two resource pools: **physical resources** and **naming resources (SCT entries)**. The origin and representation of the initial free-SCT pool is part of the genesis problem.

## 9. Why SCT entries cannot simply be reused

An SCT index may be present in capabilities distributed throughout the system.

```
capability -> SCT[147] -> object A
```

If object A is destroyed and SCT[147] immediately reused:

```
capability -> SCT[147] -> object B
```

an old capability could accidentally acquire authority over an unrelated new object.

Therefore SCT-entry reclamation cannot be ordinary immediate deallocation. System 250 garbage-collection mechanisms and SCT status/marking facilities are relevant. Reuse must occur only when stale references can no longer confer unintended authority.

This is also important for hot-unplug: removing a peripheral cannot safely mean simply putting its SCT slot straight back on the free list.

## 10. SCT temporary invalidation

Published patent material describes SCT checksum behaviour that permits an SCT entry to be made temporarily unusable, including during relocation. This distinguishes:

- making an object temporarily inaccessible;
- establishing that no live capability references remain;
- finally recycling its SCT identity.

The exact historical lifecycle should be documented separately from the current architectural inference.

## 11. Capability authority lifecycle

The emerging model has four operations:

**Genesis:** physical/machine authority enters the capability universe.

**Derivation:** existing authority creates equal or lesser authority; authority must not increase.

**Transfer:** authority is passed between protection domains/processes.

**Destruction/reclamation:** authority becomes unusable and eventually its naming resources may safely be reused.

This is useful for both historical investigation and PP250-Reboot design.

### Hypothesis: SC attenuates capability rights at store time

A candidate mechanism for **derivation** is that the access-bit field associated with `SC` (SAVE/store capability) acts as a mask when a capability register is saved into a capability block.

The proposed rule is conceptually:

```
stored rights = capability-register rights AND SC access mask
```

On this model, a resource creator can hold a capability with broader legitimate authority and save a deliberately restricted version without first requiring a separate capability-reduction operation. The same operation both stores the capability and attenuates the authority propagated through it.

This would satisfy the required monotonicity property: `SC` could remove rights but could not create a right absent from the source capability register.

**Status: HYPOTHESIS.** The instruction encoding and exact `SC` semantics must be checked to establish whether the access bits are in fact used this way. In particular, the reconstruction predicts that no `SC` mask can cause the stored capability to acquire an access right not already present in the source capability register.

## 12. Security property of the PRA

The PRA need not be trusted merely by convention. After handing a resource to its specialised allocator/handler, it should no longer possess capability authority over that resource.

```
PRA owns authority for module 6
        |
module 6 discovered as disk interface
        |
Disk handler receives appropriate capability
        |
PRA's authority for module 6 is destroyed/removed
```

The PRA cannot subsequently access the disk simply because it admitted it historically.

This follows the System 250 principle that what a process can do is determined by the capabilities it currently possesses, not a permanent privileged identity.

## 13. Unresolved representation problem

If primordial authority is literally one base/limit capability, removing an interior resource creates two disjoint ranges:

```
[---------------- ROOT ----------------]

allocate:
          [--- R ---]

remaining:
[--------]         [--------------------]
```

A single ordinary base/limit capability cannot normally represent both remaining ranges. A persistent PRA therefore probably needs a set/tree/list of remaining resource capabilities, a resource-space indirection mechanism, module-granular primordial capabilities, or another representation yet to be found.

This is a key data-structure question.

## 14. Internal Mode is not the missing privilege layer

Internal Mode should not be treated as a conventional supervisor mode. Current understanding is that it provides capability-controlled access/inspection of processor-internal registers, with important restrictions on modification. It is not an arbitrary capability-manufacturing backdoor.

Capability genesis remains a separate problem.

## 15. Provisional architectural picture

```
PHYSICAL MACHINE
       |
       v
machine/fault/start-up mechanisms
       |
       v
PRIMORDIAL RESOURCE AUTHORITY
       |
       v
Primordial Resource Allocator
       |
       +----------+-----------+-----------+
       |          |           |           |
       v          v           v           v
    Store       Disk       Comms       other
  Allocator    Handler     Handler     handlers
       |
       v
physical segment + SCT entry
       |
       v
restricted capabilities
       |
       v
ordinary processes/packages
```

There is no requirement here for a conventional permanent supervisor mode.

## 16. Questions to resolve from primary documentation

1. What exactly represents the initial/free physical-resource pool?
2. Could a single capability span multiple module addresses?
3. What is the exact loaded 48-bit capability layout and how does module addressing interact with base/limit?
4. What data structure represents the Store Allocator's free resources?
5. Where does the Store Allocator obtain its initial free SCT entries?
6. What exact operation creates/populates an SCT entry?
7. What exact operation invalidates and ultimately frees an SCT entry?
8. How is a newly installed store/peripheral detected and admitted?
9. What mechanism gives initial authority over a newly appearing module?
10. Can primordial authority cover module addresses at which no hardware was present when the authority was created?
11. Find the original statement that a processor is brought online by faulting it.
12. Determine the roles of watchdog expiry, interval timer, scheduler and idle processing in configuration/recovery activity.
13. Determine what happens at bus level when an addressed module fails to respond.
14. Identify exactly which structures are established below the first CHP/fault-created process.

## 17. Research discipline

Until primary documentation settles these points, maintain a strict distinction between:

- **documented** System 250 behaviour;
- **strong inference forced by known data structures**;
- **candidate architecture for PP250-Reboot**.

The primordial-resource-allocator model is currently a reconstruction/hypothesis, not a claim that historical PP250 definitely implemented it in this form.

Nevertheless, it provides a coherent explanation of capability genesis, dynamic resource admission, permanent absence of a privileged kernel, resource-specific allocators, hot removal/fault handling, and the provenance of authority throughout the running system.
