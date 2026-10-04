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

Thus there are two resource pools: **physical resources** and **naming resources (SCT entries)**. How a running allocator manages those pools is ordinary resource-management architecture; only their provenance at genuine cold start belongs to the remaining bootstrap question.

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

Ordinary runtime capability creation is supplied by the protected allocator/resource mechanism. The remaining genesis issue is confined to cold-start provenance of the initial authority.

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

### 18.4 Runtime capability creation versus the remaining cold-start question

Subsequent evidence and reconstruction close the earlier runtime-genesis question: once legitimate capability-controlled execution exists, protected resource allocators create resources, establish the required SCT/resource state, and return capabilities for them. This is ordinary system resource management, not an unexplained capability-forging operation.

The remaining genesis question is therefore **cold-start provenance**: how the initial valid authority and structures required to start that already-legitimate allocator/process environment are established from inert state.

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

---

## 19. September 2026 update — capability representations, object identity and authority

This section records the representation distinctions established while working bottom-up from the SCT, Inform/Outform capabilities and loaded capability registers. It refines the resource-genesis question without replacing the earlier reasoning.

### 19.1 Capability and object are distinct

**DOCUMENTED OBSERVATION / NECESSARY INFERENCE:** an SCT entry is not itself an individual capability. Multiple Inform capabilities can refer to the same SCT entry while carrying different ACCESS values. They therefore confer different authority over the same object.

```text
Capability A:  ACCESS A + SCT[42]
Capability B:  ACCESS B + SCT[42]
                         |
                         v
                       SCT[42]
                         |
                         v
                       Object X
```

Consequently ACCESS belongs to the individual capability, not simply to the SCT entry or object.

A useful semantic decomposition is therefore:

```text
CAPABILITY
    |
    +-- ACCESS             what authority this capability confers
    |
    +-- OBJECT REFERENCE   what that authority applies to
```

This is a **WORKING RECONSTRUCTION of the semantics**, not a claim about a common physical encoding.

The ACCESS field contains the six documented capability permissions `EC WC RC ED WD RD`. Additional bits exist in the ACCESS field, but they must be described according to the particular documented format/version. They must not all be labelled collectively as `FORM`; in at least one interpretation only two of the additional bits constitute form information.

### 19.2 Inform representation

**DOCUMENTED OBSERVATION:** the primary-store Inform/active representation is a 24-bit stored capability whose semantic content is:

```text
ACCESS + SCT reference
```

It does not directly encode the target object's BASE and LIMIT. The SCT reference selects the SCT entry through which the target object is resolved.

Thus:

```text
ACCESS A + SCT[42]
ACCESS B + SCT[42]
```

are distinct capabilities over the same object.

The exact subdivision of the ACCESS field and SCT-reference field must be taken from the relevant architecture/OS/version rather than imposed from a later summary.

### 19.3 The SCT is object-side state, not another capability representation

**DOCUMENTED OBSERVATION:** material examined so far describes an SCT entry as three 24-bit words containing BASE, LIMIT, a 24-bit checksum and additional flag bits. Later patent material identifies examples including `GARBAGE`, `VISITED` and `PRESENCE`. These are **generation/version-specific SCT representations**: later fields must not be projected backwards, and no cross-generation bit-for-bit continuity is assumed or required.

`LIMIT` is retained as the architectural term. It must not be casually renamed `LENGTH`: evidence describing it as a limiting offset must be reconciled with sources using looser descriptions such as size/length when the exact comparison semantics are reconstructed.

**NECESSARY INFERENCE:** because different Inform capabilities with different ACCESS values can share the same SCT entry, the SCT centralises the current mapping/state of the object while authority remains in the individual capabilities.

```text
Inform A                         Register A
ACCESS A + SCT[42] ----+------> ACCESS A + BASE42 + LIMIT42
                       |
                       +-- SCT[42]
                       |
Inform B               +------> Register B
ACCESS B + SCT[42] ------------> ACCESS B + BASE42 + LIMIT42
```

### 19.4 Outform representation

**DOCUMENTED OBSERVATION:** when a capability itself is represented in secondary storage, the Outform/passive representation substitutes a persistent object reference for the active SCT reference. Available descriptions identify this persistent reference with the object's unique disk identity/address assigned when the segment is created.

Its established semantic content is therefore:

```text
ACCESS + persistent object reference
```

The physical width and detailed bit-level encoding of Outform remain **UNKNOWN** and are deliberately not asserted here.

**NECESSARY INFERENCE:** ACCESS must survive Inform-to-Outform conversion independently of object identity. Two capabilities with different authority over the same object cannot collapse into a bare persistent object identifier:

```text
Inform:
    ACCESS A + SCT[42]
    ACCESS B + SCT[42]

              |
              | outform
              v

Outform:
    ACCESS A + persistent-object-X
    ACCESS B + persistent-object-X
```

### 19.5 Loaded capability-register representation

**DOCUMENTED OBSERVATION / WORKING RECONSTRUCTION:** loading an Inform capability resolves its SCT reference through the SCT and produces the execution-time capability-register representation:

```text
Inform capability
ACCESS + SCT[n]
          |
          v
        SCT[n]
       BASE + LIMIT
          |
          v
Capability register
ACCESS + BASE + LIMIT
```

The loaded capability register is 48 bits. BASE denotes the System 250 module/address starting point established in the addressing reconstruction; LIMIT bounds access relative to that base; ACCESS remains the authority of the particular capability being loaded.

The SCT reference needed to reconstruct the stored Inform form is retained separately in process state (the Process Dump Stack), rather than requiring it to be encoded in the 48-bit register representation itself.

### 19.6 Capability representation and target-object residence are independent

This distinction corrects an earlier overstatement made during the investigation.

**NECESSARY INFERENCE:** `Inform` and `Outform` describe the representation/location of the **capability itself**. They do not by themselves state whether the target object is currently resident in primary store.

It is therefore incorrect to equate:

```text
Inform  = resident target object
Outform = non-resident target object
```

An Outform capability stored on disk may designate an object that is currently resident in primary store. Conversely, an Inform capability in primary store may refer through an SCT entry whose target object is currently non-present.

Capability representation and target-object residence are separate dimensions.

### 19.7 What is preserved and what changes

The current semantic summary is:

```text
                         AUTHORITY             OBJECT DENOTATION

INFORM                   ACCESS                SCT reference

OUTFORM                  ACCESS                persistent object identity

CAPABILITY REGISTER      ACCESS                BASE + LIMIT
```

This is a semantic table, not a claim that the three representations share a physical width or field layout.

The common element is the authority carried by ACCESS. What changes is how the target object is denoted in the context in which the capability is represented.

### 19.8 Role of the SCT

The SCT is the indirection mechanism connecting an active Inform object reference to the object's current system realisation:

```text
individual capability                    object-side state

ACCESS -------------------+
                           |
SCT reference -----> SCT entry -----> BASE / LIMIT / flags / ...
                           |
                           v
                         object
```

It is not the authority itself and can be shared by capabilities carrying different ACCESS values.

This also explains why physical relocation need not require every Inform capability to be rewritten: the SCT entry can change its current mapping while Inform capabilities continue to refer to that SCT entry.

### 19.9 Persistent object identity and SCT identity are distinct

**DOCUMENTED OBSERVATION / NECESSARY INFERENCE:** an SCT index is an active-system reference, not necessarily the persistent identity of the object. Converting an individual capability from Inform to Outform does **not** itself permit the corresponding SCT entry to be reclaimed: the entry must remain allocated while any Inform/active capabilities still refer to it. Only when no Inform/active references to that SCT entry remain may the slot be reclaimed/reallocated. Outform capabilities can survive such reclamation because they use the persistent object identity rather than the SCT index; if one is later Inform'ed, the object may therefore be assigned an SCT entry with a different index.

```text
persistent object X
       |
       +--> at one time: SCT[27]
       |
       +--> later:       SCT[103]
```

Capabilities to X retain their individual ACCESS across this change of active SCT representation.

### 19.10 Consequence for the genesis investigation

The earlier genesis model must now distinguish several operations that had sometimes been conflated:

1. creation/existence of an object;
2. allocation/assignment of an SCT entry representing its current active mapping;
3. creation of a capability referring to that object;
4. selection of the ACCESS carried by that particular capability;
5. expansion of an Inform capability into BASE/LIMIT/ACCESS in a capability register.

**NECESSARY INFERENCE:** a new capability to an already represented object need not imply creation of a new SCT entry. It may be another capability carrying a different ACCESS while referring to the existing object/SCT entry. Conversely, allocating or populating an SCT entry is not by itself equivalent to creating authority to use the object.

This substantially sharpens the remaining genesis question. Rather than asking only how hardware is 'turned into a capability', the investigation must identify separately how System 250 establishes an object representation and how it creates a legitimate, unforgeable stored capability carrying a particular ACCESS to that object.

### 19.11 Remaining documentary/representation details

The following remain open:

- exact Outform physical width and encoding where useful for a particular generation;
- exact bit layout of the Inform ACCESS field and SCT reference for a particular generation/OS where needed for faithful emulation;
- exact LIMIT comparison semantics, including inclusive/exclusive boundary behaviour;
- exact meaning of SCT state/flag bits **within any generation where that meaning affects reconstruction**; cross-generation bit continuity or chronology is not itself an architectural requirement;
- exact architectural mechanism that creates a new legitimate stored capability or changes/attenuates ACCESS associated with an existing object reference;
- implementation details of a particular generation's VM/storage-manager handling, where needed for faithful emulation. Inform/Outform conversion itself belongs to the established VM/storage-management and VM-trap mechanism.

These are documentary or generation-specific implementation details, not automatically current architectural open questions. Promote one only if its absence prevents reconstruction of observed behaviour; see `pp250-open-questions.md`.


---

## 20. September 2026 update — mixed-access construction as the leading genesis reconstruction

This section records the **currently most plausible implementation hypothesis** for the remaining ordinary resource/capability-genesis problem. It is not yet established by surviving primary documentation.

### 20.1 Constraints the mechanism must satisfy

The reconstruction is attractive because it uses only mechanisms already known to exist in System 250:

- mixed data/capability access exists (the Process Dump Stack is the clearest example);
- stored Inform capabilities contain ACCESS plus an SCT reference;
- `LC` loads a stored capability through the SCT into a capability register;
- the Store Allocator is described as allocating store and returning a capability;
- segment creation establishes a new object/SCT representation before ordinary use of that object;
- primary-store residence is separable from capability representation and may be established later;
- no general early data-to-capability instruction comparable to Andrews' later capability-manipulation mechanisms has yet been found.

There is no per-word capability/data metadata in the mixed-store model currently reconstructed. Whether a word is being treated as data or as a stored capability is determined by the operation and architectural convention; in the Dump Stack that convention is enforced by M.

### 20.2 Candidate implementation

**STRONG WORKING HYPOTHESIS:** the Store Allocator possesses private mixed-access construction storage. It can write the bit pattern of an Inform capability using data access and then load that location using capability access.

Conceptually:

```
caller requests:
    size/limit L
    initial ACCESS A

Store Allocator
    |
    +-- requests creation of a new storage object
    |
    +-- a fresh SCT entry n is established for that new object
    |
    +-- in private mixed construction storage, write as DATA:
    |       [ A | SCT[n] ]
    |
    +-- load the same location as a CAPABILITY:
    |       LC -> Cx
    |
    +-- return/pass Cx (or a stored form of it) to the caller
```

The exact Store Allocator-to-SCT/store-management interface is **UNKNOWN**. In particular, surviving evidence has not yet established the exact calls, ordering, or which layer allocates the SCT entry versus secondary/backing storage.

### 20.3 Important strengthening: genesis is for a new object, not arbitrary existing primary store

The important security distinction is that this reconstruction does **not** require an operation of the form:

```
existing primary-store address + chosen ACCESS -> capability
```

The stored Inform capability being constructed contains an SCT reference, not an arbitrary primary-store BASE. The proposed genesis path starts by establishing a **new object** and its SCT identity. The resulting first capability is authority to that newly created object.

Thus the candidate primitive is better characterised as:

```
create new object + choose initial ACCESS -> first capability
```

rather than:

```
choose existing memory + choose ACCESS -> forge capability
```

A newly created object may initially have no primary-store residence; later reference can cause primary store to be allocated/loaded independently. This is consistent with the distinction already established in Section 19 between capability representation and target-object residence.

### 20.4 Mixed access and the unresolved safety question

Mixed access deserves explicit attention because, in the absence of per-word type metadata, it appears superficially capable of bridging the data and capability interpretations of the same storage.

The proposed construction depends on the following operation being legitimate:

```
WD writes [ACCESS | SCT reference] as data
                 |
                 v
LC subsequently treats that location as a stored capability
```

**THIS STEP IS NOT YET DOCUMENTED.** It is an implementation hypothesis that must be checked against the exact early `WD`, `RC`, `LC`, access-code and mixed-segment semantics.

Likewise, this note does **not** conclude that possession of arbitrary mixed access necessarily constitutes a general capability-forging primitive. That question requires the exact rules governing `LC` from a word written as data and any checks associated with the SCT reference.

The Dump Stack demonstrates that mixed data/capability storage existed and that M could impose a convention on how particular locations were interpreted. It does not by itself prove the proposed Store Allocator construction sequence.

### 20.5 Why this is currently the leading reconstruction

Compared with alternatives examined so far, this mechanism requires fewer unsupported architectural additions. It does not require:

- a hidden MAKECAP instruction;
- an undocumented general data-to-capability conversion instruction;
- Andrews' later derivation mechanism to have existed in the early machine;
- a deliberately invalid SCT checksum;
- a deliberately faulting `LC`;
- page-fault handling to manufacture the first capability; or
- construction of an incomplete Outform capability followed by fault-driven completion.

It instead combines already established architectural pieces in a direct way: **new object/SCT allocation, mixed storage, the normal Inform representation, and ordinary capability load**.

For that reason it should presently be treated as the **most plausible reconstruction examined**, while remaining explicitly below the status of documented System 250 behaviour.

### 20.6 What would confirm or falsify it

The most valuable evidence would establish any of the following:

1. the Store Allocator's actual access to mixed data/capability storage;
2. the exact instruction sequence by which it returned a newly created capability;
3. whether `LC` can load a capability representation whose bits were placed in mixed storage by an ordinary data write;
4. the exact mechanism by which a fresh SCT entry becomes associated with a newly allocated backing-store object;
5. whether any architectural check prevents an arbitrary existing SCT reference from being substituted during such construction.

Until such evidence is found, this mechanism is a **leading plausible genesis implementation**, not an asserted historical fact.


### 20.7 Alternatives examined and downgraded or rejected

For completeness, the following subsidiary explanations have been considered during the genesis investigation. They are retained here as **negative research results** so that they are not inadvertently reintroduced later without new evidence.

#### Hidden capability-creation instruction

**DOWNGRADED — NO EVIDENCE FOUND.**

One possibility was an undocumented or special instruction capable of constructing a capability directly from data or from an address plus ACCESS.

This would solve genesis mechanically, but it is a poor fit with the evidence examined so far. No such early instruction has been found, and a general data-to-capability operation would require substantial protection if it were not to undermine the capability model. The later Andrews/Wheatley work also makes it less attractive to postulate an earlier general capability-manipulation primitive without documentary evidence.

This possibility is not logically impossible, but it should not be used as the working explanation unless primary evidence for such an instruction appears.

#### Projecting Andrews/LCM-style derivation backwards

**REJECTED AS AN EXPLANATION OF THE EARLY GENESIS MECHANISM.**

Later Andrews/Wheatley mechanisms provide explicit capability manipulation/derivation facilities. These are important evidence about later development of the architecture, but they must not be projected backwards into an earlier PP250 revision merely because they would make genesis easy to explain.

More importantly, derivation from an existing capability addresses a different problem from creation of the first authority to a genuinely new object. The early genesis mechanism must therefore be reconstructed independently unless evidence establishes that the later mechanism already existed.

#### Deliberate SUMCHECK corruption or invalid SCT state

**REJECTED AS THE NORMAL GENESIS PATH.**

A hypothesis considered during the investigation was that the Store Allocator might deliberately construct an invalid SCT entry or SUMCHECK condition in order to force an M-side transition that would complete capability creation.

This was downgraded because SUMCHECK belongs to SCT/processor integrity checking and fault recovery. Deliberately invoking an integrity-failure path to perform routine object creation is both architecturally awkward and unsupported by the evidence examined. It also conflates fault checkout with ordinary virtual-store/resource allocation.

SUMCHECK remains relevant to SCT integrity and recovery, but it should not presently be treated as part of normal capability genesis.

#### Ordinary first-reference primary-store allocation as genesis

**REJECTED AS THE CAPABILITY-GENESIS EVENT.**

The system requires a page/segment-not-present-like mechanism because an established object may have no current primary-store allocation. On first actual reference, primary store can be allocated or the object fetched and its SCT state updated.

However, Levy's ordering places creation of the segment/object, assignment of secondary/backing storage, allocation of its SCT representation, and return of a capability before that later first-reference event.

Therefore:

```
first reference -> allocate/load primary store
```

must not be confused with:

```
create new object -> create its first capability
```

The former is a residence/virtual-store operation on an already represented object. It does not by itself explain how the first legitimate capability was manufactured.

#### Status of the incomplete-Outform/fault hypothesis

**RETAINED AS A PLAUSIBLE ALTERNATIVE, BUT NO LONGER THE LEADING RECONSTRUCTION.**

The separately recorded incomplete-Outform hypothesis proposed that the Store Allocator constructs a provisional new-object representation and uses a protected M-side fault/completion path to bind it to fresh storage and complete capability creation.

It remains more coherent than the rejected SUMCHECK variant, but it requires several mechanisms for which direct evidence has not yet been found: an incomplete/unassigned Outform state, capability-load recognition of that state, and a protected completion/retry protocol.

The simpler mixed-access Inform construction in Sections 20.2–20.5 currently requires fewer unsupported additions and is therefore the leading working hypothesis.

The fault-driven branch was examined in more detail than this summary implies. Three distinct mechanisms were considered: deliberate SUMCHECK failure; a normal access-fault/page-fault-like `LC` path with automatic retry; and use of MIP FIRST ATTEMPT/second-fault state to distinguish a later execution. They were rejected for different reasons. SUMCHECK is an integrity/check-out mechanism and too heavyweight; normal access-fault retry completes through the ordinary Inform/SCT path and therefore does not match the documented genesis sequence; and FIRST ATTEMPT describes fault-checkout state rather than an ordinary instruction retry with changed semantics. The full reasoning is preserved in `capability-genesis-outform-working-reconstruction.md`, Section 13.

### 20.8 Current comparison

The genesis candidates should presently be classified as follows:

```
ordinary resource/capability genesis
|
+-- hidden MAKECAP/data-to-capability instruction
|      status: no evidence; downgraded
|
+-- later Andrews/LCM-style derivation projected backwards
|      status: rejected for early genesis without evidence
|
+-- deliberate SUMCHECK/integrity fault
|      status: rejected as normal genesis path
|
+-- ordinary first-reference primary-store allocation
|      status: real mechanism, but rejected as the genesis event
|
+-- incomplete Outform + protected M completion
|      status: retained plausible alternative; no longer leading
|
+-- mixed-access Inform construction
       WD [ACCESS | fresh SCT] -> LC
       status: current leading plausible reconstruction
```

These classifications are provisional research judgements. Any should be reopened if primary evidence materially changes the constraints.

## Evidence update — 2 October 2026: relocation and later local lifetimes

**DOCUMENTED OBSERVATION:** US3771146A, Description 121, refreshes already-expanded capabilities during relocation by interrupting affected processors and restoring their processes through the table. This is evidence for relocation control, not general revocation, safe SCT identity reuse or the creation of primordial authority. See [canonical SCT mechanism](../architecture/system-capability-table.md#already-expanded-capabilities-during-relocation).

**DOCUMENTED OBSERVATION, later architecture:** US4486831A, Background/Summary 16–18 and Description 291–297, ties local descriptors to nesting levels, automatically deallocates RLS allocations on return, invalidates registers designating expired local storage and restricts storing capabilities into a lower associated level. Protected Return's D(0) null mask is separate from the saved call descriptor used for restoration (249–251). These rules support procedure-local lifetime enforcement, not a claim that every system capability uses that lifecycle or that the 1976 processor implemented it.

The supplied US4121286A.rtf duplicates US4050059A.rtf. It adds no allocation/deallocation evidence and cannot independently confirm the existing GARBAGE/VISITED or table-reuse reconstruction. Existing original-source findings remain separate evidence. The batch review records the provenance issue without altering either supplied file.

See [2 October 2026 patent-transcription review](pp250-patent-transcriptions-review-2026-10-02.md) for the complete evidence record and textual limitations.
