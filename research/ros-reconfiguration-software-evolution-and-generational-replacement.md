# ROS Reconfiguration, Software Evolution, and Generational Replacement

**Status:** Research note — evidence and working hypotheses, not an established reconstruction  
**Date:** 2026-10-01

## 1. Purpose

This note preserves an investigation into normal System 250 reconfiguration and software evolution. The investigation deliberately set aside the separate question of the exact C(S) bootstrap/recovery sequence and did not attempt to reconstruct a particular ROS implementation beyond what surviving sources support.

The starting question was whether System 250's strong hardware recovery mechanisms were matched by mechanisms for ordinary, deliberate reconfiguration: adding and removing modules, maintaining a continuously operating system, and changing software during the life of the installation.

The discussion did not reach a final historical mechanism. It did, however, expose a coherent set of architectural and software ideas that substantially narrow the next research question.

## 2. Design requirement: continuity during evolution

The operational-requirements material makes clear that reconfiguration was not merely a response to faults. The complete failure of the control system was intended to have a probability no greater than roughly once in fifty years, requiring redundancy and facilities for control reconfiguration.

The same requirements material anticipates:

- program change or replacement as the means of changing facilities;
- major expansion of processing power during the life of an installation without reprogramming the system;
- evolution of both the network and the control computer;
- software subsystems written by different teams at different times throughout the system's life; and
- subsystem interfaces intended to permit freedom of design and evolution within the interface envelope.

A particularly important requirement states that continuity is to be maintained despite addition, subtraction, modification, evolution, or maintenance of both hardware and software modules.

This makes online evolution a design requirement, not merely a modern interpretation of the architecture.

## 3. Hardware reconfiguration is explicitly online

The ROS Pocket Reference maintenance commands provide direct evidence of deliberate online hardware reconfiguration.

```
REMOVE <Device Type> <No.>
    Removes the device from the online system.

RESTORE <Device Type> <No.>
    Tests and if OK returns device (or port) to the online system.
```

Related commands include STATUS, DIAGNOSE, TEST, SUSPEND and CANCEL.

The significant pattern is:

```
ONLINE
   |
REMOVE / ISOLATE
   |
TEST / MODIFY
   |
VALIDATE
   |
RESTORE
   |
ONLINE
```

RESTORE is not simply reinsertion: the device is tested and returned only if satisfactory. There is therefore an explicit validation barrier before readmission.

The same Pocket Reference also exposes low-level live maintenance of stored material:

```
PRBLK  <Disc No.> <SCT Offset> ...
PABLK  <Disc No.> <SCT Offset> ...
PRINTS ...
PATCH ...
PRDISC ...
PADISC ...
```

PABLK modifies the contents of a block on a specified disc. These commands demonstrate that ROS maintenance included direct inspection and modification of the system's stored representation, although they do not themselves define a high-level software-upgrade protocol.

## 4. SCT indirection and live modification

Stored Inform capabilities identify an object through an SCT reference plus ACCESS rights. Loading a capability resolves the SCT entry to the current base and limit.

This gives the system an important indirection:

```
stored capability
ACCESS + SCT[n]
        |
        v
      SCT[n]
        |
        v
 current BASE / LIMIT / object state
```

Surviving descriptions also indicate that modification/relocation of an SCT entry is synchronized so that other processors can be prevented from using the entry while it is changed.

This is strong evidence for live transformation of an object's current system realization without rewriting every stored capability referring to it.

However, this must not be overextended into an assumed software-update mechanism. A capability already expanded into a capability register contains resolved BASE/LIMIT information. It has not yet been established how a live processor holding such a capability is affected by a contemporaneous SCT change, nor whether SCT retargeting was used for semantic replacement of software rather than relocation of the same object.

## 5. The ROS logical-resource model

D. M. England's 1972 *Operating System of System 250* provides the most important evidence for the software side.

England defines a logical resource as a data structure composed of store blocks. A resource-type allocator:

1. creates the appropriate resource/data structure;
2. obtains a capability block;
3. inserts execute capabilities for the standard operations on that resource; and
4. inserts capabilities for the constituent blocks of that particular resource's data structure.

The caller receives an Enter capability representing the resource.

Conceptually:

```
                 ENTER capability
                       |
                       v
              resource capability block
              +-----------------------+
              | EX -> operation 1 ----+----> shared code
              | EX -> operation 2 ----+----> shared code
              | EX -> operation 3 ----+----> shared code
              |                       |
              | cap -> data block A --+----> instance state
              | cap -> data block B --+----> instance state
              +-----------------------+
```

The capability block therefore joins protected operations to the private data structure of a resource instance.

This gives System 250 a clean separation between:

- external resource identity and authority — the Enter capability;
- the protected interface — execute capabilities in the resource capability block;
- implementation code, potentially shared by many instances; and
- per-instance mutable state.

England calls the resulting user interface standard, dynamic and adaptive.

## 6. Program versus process

England makes a further distinction that became central to this investigation.

A program is a static structure of code blocks, constant data blocks and constant capability blocks. Applying a CPU at its start point creates an execution which allocates its own blocks and constructs its own private capability/data structure. That execution is a **process**.

Multiple processes may execute the same reentrant program simultaneously.

There is one process-ready list for the whole system. A process can run on any CPU, and separate portions of one process's execution may run on different CPUs.

Consequently, CPUs are deliberately abstracted away from software identity. They provide processing power to the shared process/resource world.

This weakens the earlier hypothesis that software evolution would fundamentally be achieved by rebooting processors one at a time into different operating-system versions. Processor REMOVE/RESTORE is clearly useful for hardware reconfiguration, but the natural software unit is a process, package or resource rather than a CPU.

## 7. Application package structure and recovery

The telephone-switching material gives unusually concrete evidence of application structure.

A package is described as a functionally independent collection of code or data blocks sharing a common Capability Pointer Table. A typical package contains:

- Normal Operation Code
- Capability Pointers
- Fault Messages
- Package Restart Code
- File Audit Code
- Device Test Code
- Private Files

Recovery is therefore explicitly designed into the package structure.

The fault-security routines first reload application read-only areas from a maintained second copy and then instruct the application to **reconstruct or re-validate its read-write areas**.

If faults persist, progressively stronger actions include system testing/restart, switching faulty hardware out, application revalidation/reconstruction, system reload and trial reconfiguration.

The important conceptual separation is:

```
replaceable/reloadable code and read-only material
                    +
mutable application state requiring semantic reconstruction or validation
```

ROS/application recovery does not assume that mutable state can simply be replaced with program code.

## 8. Concrete example: reconstructing the switch map

The telephone application provides a useful worked example of semantic state recovery.

The switch-network map is a mutable software representation of physical switch state. Normal traffic continuously provides information allowing discrepancies between the map and hardware to converge back toward consistency.

If the complete map is lost, a new map can be constructed from the individual call records for calls in progress.

An audit routine constructs and compares map information a small section at a time. The paper explicitly notes that calls cannot safely be set up or cleared in the region being audited without complicating the audit, so the audit is deliberately localized to minimize disturbance.

This demonstrates several important ideas:

- mutable derived state can be reconstructed from more authoritative state;
- validation can be performed before trusting reconstructed state;
- only a limited region need be prevented from changing during consistency work; and
- recovery need not imply stopping the whole installation.

This is evidence for local quiescence/consistency barriers at application level, though not yet evidence of a generic ROS software-upgrade primitive.

## 9. The data-structure problem

Changing code is only part of software evolution. If an interface changes the representation of data passed across it, or if a resource changes its private persistent representation, old and new code may not understand one another.

Three distinct compatibility questions should therefore be separated:

### 9.1 Operation/interface compatibility

An Enter capability and offsets into its protected interface effectively define operations available to callers. If old and new software coexist, the meaning of those operations must either remain compatible or be explicitly versioned.

### 9.2 Transient interchange representation

Messages, argument blocks, returned results and shared buffers must have representations understood by the communicating generations.

Changing:

```
Request-A = account, amount, flags
```

to:

```
Request-B = account, currency, amount, flags, timestamp
```

cannot be solved merely by retargeting code.

### 9.3 Private resource state

A resource's internal data structure is protected behind its Enter interface. This is much easier to evolve because callers need not know the representation. Nevertheless, changing the representation of a long-lived resource still requires conversion, reconstruction, compatibility, or replacement of that resource.

The System 250 resource abstraction therefore reduces the data-structure evolution problem but does not abolish it.

## 10. Initial replacement hypothesis: migrate a resource

One possible model considered during the investigation was explicit resource migration:

```
old code + old state
        |
     quiesce
        |
convert/reconstruct state
        |
install new code/state
        |
validate
        |
resume
```

The SCT exclusion mechanism, package restart/audit facilities and application reconstruction mechanisms make such a design architecturally plausible.

However, no source found so far says that ROS performed general software evolution in this way.

A related possibility is constructing a complete replacement resource alongside the old one and then redirecting a higher-level capability reference to it. Again, the architecture appears capable of supporting this, but the historical rebinding operation has not been identified.

## 11. Stronger working hypothesis: no migration of existing instances

A simpler possibility emerged late in the discussion.

**Existing instances may never migrate at all.**

Suppose program/package generation A is currently used to create call-processing instances. A new generation B is installed. The changeover need only affect the creation of *new* processes/resources:

```
                         time ->

Program A available
       |
       +---- A1 ------------------------------> terminates
       +--------- A2 ------------------------------> terminates
       +-------------- A3 --------------------> terminates
                         |
                         | CHANGEOVER
                         |
Program B available      +-- B1 ------------------------>
                         +------- B2 -------------------->
```

A1, A2 and A3 retain their A code, A data representations and A capability structures for their entire lifetimes.

B1 and B2 are born using B and need never understand A's private representation.

Once the final A instance terminates, generation A has drained and its program/resources can eventually become unreachable and reclaimable.

In modern terminology this resembles a **draining or generational deployment**, but that terminology must not be projected back onto the historical system without evidence.

### Why this hypothesis fits the known architecture

It is consistent with:

- England's explicit distinction between a singular reentrant program and multiple independent processes executing it;
- private per-process capability/data structures;
- finite-lived telephone-call processes;
- standard subsystem interfaces;
- capability-protected resource boundaries;
- coexistence of independent processes in a common multiprocessor system;
- processor independence; and
- capability-based lifetime/reachability.

It also avoids the hardest form of live data migration. A call record belonging to an A call can remain in A format until that call ends; B calls can use B format from birth.

## 12. Where generational replacement does not solve the problem

The hypothesis is not a universal answer.

A resource that outlives both generations and is shared by A and B still requires a compatible interface:

```
A process ----+
              +----> long-lived shared resource
B process ----+
```

The resource's private representation can remain hidden behind its Enter capability, but the operation protocol used by A and B must be compatible, versioned, adapted, or separately exposed.

Similarly, permanently persistent data may require explicit conversion or an implementation capable of understanding more than one representation.

Generational replacement therefore moves the difficult compatibility boundary outward; it does not eliminate it.

## 13. Reconsidering the missing "middle arrow"

Earlier reasoning assumed an update required:

```
OLD RESOURCE
     |
     | migrate/rebind
     v
NEW RESOURCE
```

The generational hypothesis suggests that this middle arrow may not exist for finite-lived instances.

Instead, the significant binding may be at **instance/process creation**:

```
process/resource type X
          |
          +-- before changeover --> generation A start/interface capability
          |
          +-- after changeover ---> generation B start/interface capability
```

Existing instances are untouched. The old generation disappears by attrition as its instances terminate.

If historically correct, the software-changeover primitive would therefore not be "replace this running process/resource." It would be closer to:

**change the program/package/interface used to create future instances of this type.**

This is presently a working hypothesis, not a documented ROS mechanism.

## 14. Relationship to recovery

A useful recurring pattern appears across the evidence:

```
isolate / cease creating or using
          |
reconstruct / reload / modify
          |
audit / test / validate
          |
readmit / resume
```

Hardware REMOVE/RESTORE follows it explicitly.

Application fault recovery follows it through reload plus reconstruction/revalidation.

Map auditing follows it locally.

It is therefore plausible that planned software evolution reused related principles. But the investigation has **not** established that ROS had one universal recovery/update transaction or that fault recovery and software upgrade were implemented by the same mechanism.

## 15. What has NOT been established

The following must remain unresolved:

1. Whether ROS supported general online replacement of a software package as a first-class operation.
2. Whether major software changes used SCT retargeting.
3. Whether a live resource capability block/CCB was modified to point to new execute code.
4. How already-loaded capability registers were handled during SCT modification.
5. Whether ROS deliberately quiesced running processes for software updates.
6. Whether old and new package generations could coexist.
7. Whether new process creation could be rebound from one program generation to another while existing processes continued.
8. How long-lived shared resources handled interface/data-format evolution across software generations.
9. How persistent data formats were upgraded.
10. What exact ROS structures represented a process/program/package type and selected its start capability.
11. Whether the package Capability Pointer Table is directly related to the resource capability-block/CCB structures discussed by England and later by Levy.
12. Whether the absence of an obvious high-level UPDATE/REPLACE command in the surviving Pocket Reference is architecturally meaningful or merely a limitation of the surviving material.

## 16. Next research target

The most discriminating next question is now:

> **How did ROS select the program/package and start capability used when creating a new process or resource instance, and could that selection be changed while existing instances continued to run?**

Evidence should be sought for:

- process creation;
- process-type descriptors;
- program/package descriptors;
- start capabilities/start points;
- program or command binding;
- Capability Pointer Tables;
- resource-type allocators;
- replacement or redefinition of allocator/code capabilities;
- command generation and symbolic capability bindings;
- package loading/unloading;
- old/new versions or generations;
- coexistence of program versions;
- removal of code once no processes use it; and
- compatibility/versioning of standard subsystem interfaces.

A positive result here would strongly support the generational-draining model. A negative result would return attention to live resource mutation, restart boundaries, or a more global ROS changeover mechanism.

## 17. Current research position

The investigation began with a concern that System 250's hardware recovery might be stronger than its ability to perform deliberate software reconfiguration.

The surviving material instead shows that continuous hardware and software evolution was an explicit requirement and that the architecture/software stack contained several relevant mechanisms: dynamic resources, protected interfaces, shared reentrant programs, private process state, online hardware removal/restoration, virtual-store indirection, application restart/audit code, and semantic reconstruction of mutable state.

What remains missing is the historical **changeover mechanism**.

The strongest current hypothesis is that for finite-lived activities such as telephone calls there may have been no migration of live instances at all: old instances could continue under their original software generation while new instances were created under a replacement generation, allowing the old generation to drain naturally.

This is an attractive architectural explanation, but it is **not yet the endpoint** and must not be recorded as established System 250 behaviour without further primary evidence.
