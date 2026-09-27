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

---

## 18. September 2026 update — split the genesis problem

Subsequent reconstruction of processor startup, the Special Fault/Start-Up Block, `C(S)`, `SPECIAL`, and `C(N)` substantially narrows the original genesis question. This section updates the interpretation above **without deleting the earlier reasoning**. Sections 1–17 remain as the research trail and continue to contain useful resource-lifecycle questions, but the PRA should no longer be treated as the leading explanation of *bootstrap* capability genesis.

The key distinction now is between **bootstrap genesis** and **resource/capability genesis during an already running system**.

### 18.1 Bootstrap genesis is now substantially understood

The startup reconstruction provides a concrete architectural root of authority.

The processor establishes `C(S)`, the Fault Start-Up Block capability, as exceptional hardware capability state at power-up. `C(S)` does not depend on an already functioning ordinary SCT lookup. It gives the processor access to the Special Fault/Start-Up Block. The surviving patent material and current reconstruction show that this block supplies the parameters from which the startup/fault microcode establishes a restricted special capability-table environment. A reserved segment pointer, `RSPC-0`, is then interpreted through that environment to identify the Dump Stack of the process to be entered. Automatic `CHANGE PROCESS` restores a legitimate executable process context.

The current reconstructed chain is therefore:

```
processor hardware
       |
       v
      C(S)
       |
       v
Special Fault / Start-Up Block
       |
       v
restricted special SCT environment
       |
       v
     RSPC-0
       |
       v
process Dump Stack capability
       |
       v
automatic CHANGE PROCESS
       |
       v
first legitimate ordinary process
```

This breaks the circularity that motivated the original question "where does the first capability come from?" An ordinary PP250 instruction does not have to manufacture the authority needed to fetch and execute itself. Exceptional processor state and microcode establish the protected authority chain and then cross into ordinary process execution through `CHANGE PROCESS`.

Accordingly, **the existence of the first executable capability state is no longer the principal unresolved capability-genesis problem**. The exact whole-system cold-load mechanism — how the necessary startup structures first get into otherwise empty or invalid memory — remains a separate bootstrap/loading question, but it should not be conflated with capability forgery by ordinary software.

### 18.2 Establishing C(N) does not require capability manufacture

The normal-interrupt investigation closes another part of the chain.

`C(N)` is the Normal Interrupt Block capability. The running system must establish it before normal automatic interrupt entry can operate. The Pocket Reference places MIP in the Dump Stack process image, and patent material identifies `SPECIAL` as a one-instruction Primary Indicator state in which `LC` can address the special-purpose capability-register bank.

The direction of `LC` is crucial:

```
legitimate stored capability --LC under SPECIAL--> special C register
```

Thus the initial process reached through the `C(S)` path can install a legitimate stored capability into a special register such as `C(N)` without constructing capability bits from ordinary data.

The reconstructed ancestry is:

```
C(S)
  |
  v
startup / checkout
  |
  v
legitimate initial process
  |
  v
MIP / SPECIAL + LC
  |
  v
C(N)
  |
  v
Normal Interrupt Block
```

This strengthens rather than weakens the anti-forgery model: a special processor register can be populated from already legitimate capability authority without providing an unrestricted capability-construction operation.

### 18.3 Internal Mode and SPECIAL are distinct

Section 14 remains correct but can now be stated more precisely.

Internal Mode is not a hidden supervisor mode and is not the discovered capability-genesis mechanism. It exposes selected processor-internal state under constrained architectural rules; the evidence examined so far does not turn it into an arbitrary data-to-capability conversion path.

`SPECIAL` is a different mechanism. It changes the register selection of one `LC`, permitting a **stored capability** to be loaded into a special-purpose capability register. It therefore explains installation of special capability state such as `C(N)`, but it still does **not** explain how a new stored capability comes into existence in the first place.

Neither mechanism, as presently understood, gives ordinary software a general capability-forging operation.

### 18.4 The remaining capability-genesis hole

The central unresolved problem is now much narrower:

> **Once legitimate capability-controlled execution exists, by what architectural mechanism can authority over a previously unrepresented physical resource first enter the ordinary SCT-backed capability universe?**

For storage, the unresolved transition can be represented as:

```
physical store / resource
        ?
        |
        v
      SCT entry
        ?
        |
        v
first legitimate stored capability
        |
        v
       LC
        |
        v
expanded C-register capability
```

Published descriptions associate the Store Allocator with allocating physical store, obtaining/populating an SCT entry, and returning a capability. What is still missing is the exact architectural operation that authorises the crucial first step. If ordinary data cannot simply be loaded into a C register as a capability, then "the Store Allocator creates a capability" is a description of the required result, not yet an explanation of the mechanism.

This is the **general resource/capability genesis problem** and should now be the principal target of this research note.

### 18.5 The PRA hypothesis must be retained but reclassified

The Primordial Resource Allocator model in the earlier sections was developed when bootstrap genesis and resource genesis were still entangled. It remains useful because it asks important questions about provenance, resource discovery, monotonic authority, allocation and relinquishment.

However, it should now be read as an **earlier architectural hypothesis about resource admission and authority management**, not as the leading explanation for how the processor obtains its first capability.

In particular, the discovery of hardware-established `C(S)` removes the need to posit a PRA merely to explain the first transition into capability-controlled execution. It does **not** by itself answer whether some PRA-like software object, resource allocator, preconstructed root resource capability/set, or another mechanism subsequently accounts for authority over allocatable physical resources.

The unresolved question is therefore not simply whether a PRA existed. It is what concrete representation and operation supplied the Store Allocator or equivalent software with legitimate authority from which newly allocated resource capabilities could be produced.

### 18.6 Revised two-level model

The current architecture should be investigated as two related but distinct chains:

```
A. BOOTSTRAP GENESIS — substantially reconstructed

hardware
   |
  C(S)
   |
Start-Up Block
   |
special SCT / RSPC-0
   |
Dump Stack
   |
automatic CHP
   |
legitimate process


B. RESOURCE / CAPABILITY GENESIS — still unresolved

physical resource
   |
   ?     <--- principal hole
   |
SCT representation
   |
   ?     <--- exact first-capability construction mechanism
   |
stored capability
   |
  LC
   |
C-register capability
   |
derivation / transfer / ordinary capability operation
```

This distinction prevents evidence about processor bootstrap from being asked to solve a different problem: how a running System 250 admits and represents new resource authority.

### 18.7 Updated research questions

The highest-priority questions are now:

1. What exact operation does the Store Allocator use when it "creates" the first capability for newly allocated store?
2. How is a new or free SCT entry populated, and what architectural checks distinguish legitimate population from capability forgery?
3. What authority must the process performing that operation already possess?
4. Is there a special capability, SCT-management operation, microcode transition, reserved representation, or protected package mechanism specifically for this purpose?
5. Does the capability originate first as an SCT identity/stored capability, or first as some exceptional loaded capability representation?
6. How is the initial pool of free physical store represented as authority after the first process has been established?
7. How is the initial/free SCT-entry pool represented and protected?
8. Do the early PP250 design and later Andrews capability-derivation mechanisms differ materially here?
9. Can evidence about the `666` mixed data/capability Dump Stack segment constrain the mechanism by which such an object was originally created?
10. Which surviving Store Allocator, SCT, garbage-collection, processor self-test, ROS/PDOS, patent, or microprogram descriptions expose this transition directly or indirectly?

### 18.8 Current research position

The research position is therefore:

- **Hardware bootstrap root:** substantially explained by `C(S)` and the startup/fault transition.
- **Transition to an executable process:** substantially explained by special SCT state, `RSPC-0`, the Dump Stack and automatic `CHANGE PROCESS`.
- **Installation of running special capability state such as `C(N)`:** substantially explained by legitimate stored capabilities plus MIP/`SPECIAL`/`LC`.
- **Arbitrary data-to-capability conversion:** no evidence found; the architecture continues to argue against it.
- **General creation of the first ordinary capability representing newly admitted/allocated resource authority:** **still unresolved and now the central capability-genesis hole.**

Future work on capability genesis should begin from that narrowed question rather than reopening the already substantially reconstructed `C(S)` bootstrap chain, unless new primary evidence contradicts the reconstruction.
