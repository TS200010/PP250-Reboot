# SCT Failure, Process Ephemerality and System Recovery

## Status

**Working reconstruction / research note.**

This note records the reasoning around catastrophic loss of the normal System Capability Table (SCT). It deliberately separates:

- facts directly supported by the surviving System 250 sources;
- architectural inferences that follow from those facts;
- a working reconstruction of catastrophic SCT failure;
- implications for a modern descendant.

The central recovery model described here is **not yet directly documented in a primary source**. No source has yet been found saying explicitly that loss of the SCT causes the current process universe to be abandoned and a new SCT/process epoch to be created. The reconstruction is important because it resolves an apparent contradiction without requiring undocumented SCT replication.

The later speculative design discussion about replicated SCT entries, generations, persistent-object identifiers, dirty-block mirroring, etc. is intentionally **not** included here. That belongs to a separate modern-design investigation.

---

## 1. The apparent SCT single-point-of-failure problem

The investigation began with an apparent contradiction.

The normal System Capability Table is fundamental to conversion of stored main-store capabilities into the base/limit form used by the processor. A stored capability contains access authority together with an SCT reference. Loading the capability uses the corresponding SCT entry to obtain the physical descriptor of the target block.

Yet the normal SCT appears to reside in ordinary modular store.

System 250 was deliberately designed to tolerate processor and storage-module failures. In particular, the SPECIAL/check-out machinery has replicated entry structures in different storage modules so that check-out can continue even if the store module being used during check-out is itself faulty.

This raises a basic question:

> **What happens if the failed storage module is the one containing the normal SCT?**

So far, no primary evidence has been found that the normal runtime SCT was mirrored, reconstructed from another copy, written to disk as a recoverable image, or restored with its previous SCT indices intact.

If C(C) defines the normal SCT and the SCT disappears, essentially every normal stored main-store capability appears to lose its meaning.

At first sight this looks like a major architectural hole.

---

## 2. What the sources establish about SCT use

The early patents establish the essential form of capability expansion.

A stored main-store capability contains an SCT/MCT reference and access rights. The referenced table entry supplies the base and limit used to construct the expanded capability held in a processor capability register.

The early MCT/SCT entry is protected by checking information. The first word is a sum-check related to the base and limit. The hardware checks this information while loading a capability.

The hardware paper describes the protection more explicitly: the SCT sum-check is designed to detect store faults, data-path corruption and addressing faults. The three-word packet arrangement also makes certain store-addressing failures detectable.

Thus failure of the store containing the SCT should result in a detected fault rather than silent corruption of authority.

### 2.1 Expanded capabilities do not continuously consult the SCT

Once a capability has been expanded into a processor capability register, ordinary memory references use the base/limit information in that register. They do not reconsult the SCT on every access.

The relocation patent makes the consequence explicit. A process may already have loaded capabilities before relocation begins. Changing the corresponding table entry therefore does not retroactively change those processor registers.

The relocation mechanism deals with this by making the table entry unusable, interrupting processors/processes as necessary, moving the block, changing the table entry, and causing capability state to be reloaded through process transition.

Therefore:

> **Changing an SCT entry does not retroactively alter already-expanded capability registers.**

This is relevant to SCT failure because a processor might execute briefly using already-expanded capabilities, but normal execution cannot continue indefinitely once further capability loads or crossings require the missing SCT.

---

## 3. The SPECIAL/check-out environment is different

System 250 contains an independent fault/check-out mechanism.

The fault-checkout patent describes multiple check-out program entry segments located in different storage modules. If a further store fault occurs during check-out, another entry segment can be selected.

The hardware description likewise identifies special capability registers including C(S), the start-up block capability, and describes the mechanism by which the store-module selection associated with the special environment can be changed.

This establishes an important architectural distinction:

> **Failure of the normal SCT need not prevent the processor from entering a trustworthy SPECIAL/check-out environment.**

The replicated check-out structures solve the problem of reaching recovery code when ordinary storage is faulty.

They do **not**, by themselves, demonstrate replication of the normal runtime SCT.

The hardware paper also states that, after satisfactory check-out, the action taken is system dependent. The hardware establishes that the processor and its accessible environment are trustworthy; it does not specify one universal policy for preservation of the interrupted computation.

---

## 4. Inform/outform capability representation and object identity

The England architecture paper establishes an important distinction between capability representation and object residency.

A capability resident in main store is represented using an SCT reference.

A capability resident on disk is represented differently: disk identity/address information replaces the SCT reference.

At the same time, England states that a capability resident in main store requires a corresponding SCT entry even when its target block currently exists only on disk.

Therefore two separate questions must not be conflated:

1. Is the **capability representation** in main-store form or disk/outform form?
2. Is the **target object** resident or non-resident?

A non-resident object can still have a main-store capability referring to an SCT entry.

Conversely, when the capability itself is represented on disk, the SCT reference is replaced by disk identity/address information.

This has a major consequence:

> **The SCT index is not the ultimate persistent identity of the object.**

No evidence has been found that an SCT index must survive an inform -> outform -> later inform cycle when the capability representation itself has gone to disk.

---

## 5. Virtual store already spans the main-store/disk boundary

The System 250 operating-system paper strengthens this interpretation.

When a process requests new storage, the Store Allocator first allocates the appropriate backing-store space and delivers a capability. Main-store allocation or transfer is performed when the block is actually used.

For an unchanged block that has been brought into main store, the main-store copy can later be discarded without performing a transfer back to disk.

Most importantly, the paper describes capability blocks as being held on disk and says that the capability network extends from the address space of main store into the address space of disk regardless of the physical boundary.

Thus the logical object/capability world is not synonymous with the current SCT/main-store manifestation.

---

## 6. Program and process are fundamentally different objects

The key conceptual step came from reconsidering England's distinction between a **program** and a **process**.

England describes a program as a static structure of code blocks, constant data blocks and a network of constant capability blocks, with a defined starting point.

Applying processing power at that starting point creates an execution. During execution the program requests data and capability blocks and constructs:

> a private sub-network of capabilities, a private data structure unique to this run of the program.

That execution is called a **process**.

This distinction was historically significant.

Contemporary System 250 programming explicitly emphasised **re-entrant** and **serially reusable** software. Earlier programming practice often treated a loaded program, its working data and its execution as a much more monolithic entity. If two independent analyses were required, two loaded instances of the program might be used.

System 250 instead makes the distinction structurally explicit:

```text
PROGRAM
  reusable code
  constants
  relatively static capability structure
          |
          +----------+----------+
          |          |          |
       PROCESS 1  PROCESS 2  PROCESS 3
       private    private    private
       state      state      state
       and        and        and
       capability capability capability
       network    network    network
```

The code is not synonymous with one execution of the code.

The operating-system paper goes further. Processes themselves are logical resources represented by data structures. A process structure contains such things as its starting point, priority and space for preserving register values during normal suspension/interruption.

The process allocator/manager and store allocator/manager form the lowest software layers from which higher-level facilities are built.

A processor is also not permanently associated with a process. A process may execute on different CPUs at different times.

Thus:

> **program != process != processor**

This distinction is central to the SCT recovery problem.

---

## 7. The one-process thought experiment

The recovery problem becomes much clearer if all unnecessary complexity is removed.

Consider a System 250 containing one user program and one process.

The machine has completed startup and has a valid basic architectural environment:

- a normal SCT;
- interrupt handling;
- interval-timer structures;
- store management;
- process management;
- whatever other basic structures are required for normal execution.

The user program is then started and creates one process.

Now assume the storage module containing the SCT fails.

Because normal capability loading can no longer proceed, all processors ultimately fault. In capability terms the effect is global even though only one physical store module has failed.

The machine enters its independent SPECIAL/check-out environment.

Suppose the normal startup/recovery mechanisms can again establish a clean basic architectural environment:

- a new SCT;
- valid interrupt machinery;
- valid timer machinery;
- valid store management;
- valid process management.

At this point there is no architectural necessity to reconstruct the old user process.

The user program can simply be started again.

That creates a **new process**.

Conceptually:

```text
PROGRAM
   |
old process
   |
private capability network
   |
old SCT epoch

       SCT FAILURE
            X

SPECIAL / CHECK-OUT
   |
clean basic machine
   |
new SCT epoch
   |
start PROGRAM
   |
new process
   |
new private capability network
```

The old process has not been recovered. It has been replaced by a new execution of the same program.

---

## 8. The SCT as an execution-epoch structure

This leads to a much simpler interpretation of the SCT.

The SCT need not be a persistent directory whose numbering must survive catastrophic failure.

Instead it can be understood as the current main-store manifestation table for the presently active capability universe.

During an execution epoch:

```text
program
   |
process
   |
private capability network
   |
inform capabilities
   |
SCT
```

If the SCT is catastrophically lost, the current inform capability universe can be abandoned.

The loss is therefore catastrophic to the **execution epoch**, but not necessarily catastrophic to the **system**.

A possible base recovery sequence is:

```text
catastrophic SCT loss
        |
current inform capability epoch invalid
        |
current processes may be abandoned
        |
SPECIAL/check-out survives
        |
basic system environment re-established
        |
programs restarted
        |
new processes construct new capability networks
```

This eliminates any requirement to recover the previous SCT numbering.

It also eliminates the need for an undocumented persistent mapping whose purpose is merely to remember which old SCT slot belonged to which backing-store object.

---

## 9. Abandoned process state and garbage collection

The private capability network constructed by a process may contain temporary blocks and other process-specific resources.

If the process disappears and nothing in the surviving persistent capability world refers to those objects, they have become unreachable.

They do not need to be explicitly reconstructed or individually torn down as part of SCT recovery.

They can simply become garbage.

The System 250 capability/storage model already includes reclamation of blocks that are no longer reachable through capabilities. Thus remnants of a destroyed process can eventually be reclaimed through the normal garbage-collection mechanisms.

Conceptually:

```text
old process
    X
    |
old private capability network
    |
no surviving capability path
    |
garbage
    |
eventual reclamation
```

This makes process ephemerality and capability garbage collection complementary concepts.

---

## 10. Persistent application data is a different problem

Destroying a process is not the same thing as destroying all data that the program may use.

For example, consider an account-processing application.

```text
ACCOUNT PROGRAM
     |
     +---- persistent account information
     |       Smith
     |       Jones
     |       Brown
     |
     +---- process: update Smith
     +---- process: enquire Jones
     +---- process: update Brown
```

The Smith account is not intrinsically part of the temporary process that happens to be updating it.

The process merely has authority to access and modify that persistent resource.

Similarly, in telephone switching many processes can execute the same re-entrant call-processing algorithm, each servicing a different call. A call process is naturally ephemeral. If a catastrophic machine fault destroys the call, the subscriber can initiate another call and a new process can service it.

This gives a clean separation:

> **Program** = reusable/static algorithm and relatively persistent structure.  
> **Process** = one temporary execution plus its private working capability network.  
> **Persistent application state** = resources whose lifetime may exceed any individual process.

---

## 11. Consistency of persistent state belongs above basic process recovery

Some applications cannot tolerate arbitrary interruption of updates to persistent state.

If a process can fail halfway through an operation and leave persistent data inconsistent, additional recovery semantics are required.

Those semantics may use techniques such as:

- checkpoints;
- before/after records;
- duplicated records;
- sequence numbers;
- journals or transaction logs;
- rollback/replay;
- application-specific reconciliation.

Such techniques existed in various forms in contemporary data-processing systems, particularly databases, but they are conceptually distinct from the basic System 250 process mechanism.

The base machine does not have to infer that, for example, two account updates constitute one indivisible business transaction.

This produces a useful division of responsibility:

```text
BASE SYSTEM
-----------
restore a trustworthy machine
restore SCT/store/process machinery
restart programs

APPLICATION / HIGHER-LEVEL SERVICE
----------------------------------
decide what data must persist
define semantic atomicity
maintain consistency across catastrophic restart
recover, replay or roll back incomplete work
```

The base architecture can therefore provide service recovery without guaranteeing transparent continuation of every process.

---

## 12. Why the check-out machinery can be extremely robust without preserving every process

This reconstruction explains an otherwise puzzling asymmetry in System 250.

The machine contains elaborate mechanisms to:

- detect processor and store faults;
- enter a restricted SPECIAL environment;
- execute check-out code independently of the normal capability environment;
- retry check-out from another store module if the first check-out environment encounters a store fault;
- prevent a faulty processor from returning uncontrolled to the system.

Yet the surviving sources do not show an equally elaborate mechanism for reconstructing every interrupted process after catastrophic loss of the normal SCT.

That may not be an omission.

The purpose of check-out may fundamentally be:

> **establish that the machine is trustworthy again**

rather than:

> **preserve every computation that existed before the fault.**

Normal virtual-memory faults are explicitly described differently: the faulting process is suspended, the required block is brought into main store, the process is returned to the ready list and the instruction is retried.

The designers therefore knew how to specify transparent continuation when it was required.

Catastrophic hardware check-out does not appear to make the same universal promise.

---

## 13. Working reconstruction

The resulting working reconstruction is:

> **Loss of the storage module containing the normal SCT may invalidate the entire current inform capability epoch. All processors consequently enter the independent fault/check-out environment. The system can then establish a fresh basic capability environment and SCT, after which store/process management can create new processes by restarting programs. Preservation of old SCT indices or current processes is not required by the base architecture. Process-private objects that are no longer reachable become garbage and can subsequently be reclaimed. Applications requiring persistent state to remain consistent across such a restart must provide, or use a higher-level service providing, the necessary persistence and recovery semantics.**

This model explains the apparent SCT single point of failure without postulating undocumented SCT mirroring.

It is also consistent with:

- SCT-protected capability loading;
- the non-retroactive nature of SCT changes to expanded registers;
- independent replicated SPECIAL/check-out structures;
- disk-form capability identity being different from SCT reference;
- disk-first allocation of store;
- the capability network spanning disk and main store;
- the program/process distinction;
- dynamically constructed private process capability networks;
- process/resource creation by allocators;
- capability-based garbage collection.

---

## 14. Evidence status and unresolved questions

This reconstruction must not be promoted to established historical fact without further evidence.

No primary source has yet been found stating explicitly:

> "If the SCT is lost, abandon all current processes, create a new SCT and restart the programs."

Equally, no source has yet been found requiring:

- replication of the normal SCT;
- preservation of old SCT indices across catastrophic restart;
- reconstruction of the previous SCT from backing store;
- survival of all existing processes after loss of the SCT-bearing module.

The most important remaining architectural gap is the exact transition from SPECIAL/check-out or genuine cold start into the first normal capability/process environment.

In particular, the following remain unresolved:

1. the exact protected/special-register state at power-up;
2. whether cold power-up follows the same C(S) -> special environment -> automatic process-change path as fault recovery;
3. how the Start-Up Block, special capability structures, initial Dump Stack and first normal code are populated in a completely cold machine;
4. how the first normal C(C)/SCT is established;
5. how store management and process management are first made available;
6. the exact state after the automatic process change before the first ordinary instruction;
7. where the recollection that "three instructions booted the machine" fits into this sequence.

These questions belong with the existing startup reconstruction and should remain explicitly unresolved.

---

## 15. Modern implication — separate from the historical reconstruction

From a modern perspective, accepting loss of the SCT as destruction of the entire current execution epoch would probably not be an acceptable availability boundary.

More importantly, the historical model places a potentially large burden on user/application software. Any application whose persistent state must remain consistent across catastrophic system restart has to provide its own recovery mechanism or use a higher-level service that does.

A modern descendant can reasonably move more of the consequences of physical hardware failure back beneath the system abstraction.

The architectural requirement need not be "replicate the SCT". A more fundamental requirement would be:

> **An inform capability should continue to denote the same logical object across faults within the machine's specified fault-containment envelope.**

That would allow physical failures in the implementation of the capability/object machinery to be hidden below the abstraction, while applications would still remain responsible for their own semantic transactions.

In M<H,T> terms, a modern design should investigate whether the H-level authority/object relationships can survive failures in their physical M-level representation.

How that is implemented is a separate research problem. Possible replicated SCT structures, generations, persistent-object mappings, dirty-block redundancy and related mechanisms should not be projected backwards onto the historical PP250 without evidence.

---

## 16. Current conclusion

The SCT may indeed have been a physical single point whose destruction brought the current normal capability environment to its knees.

That need not imply that System 250 had no coherent recovery model.

The crucial conceptual step is that **processes are ephemeral executions of reusable programs**. Once that distinction is taken seriously, catastrophic SCT loss need not require reconstruction of the old process universe at all.

The base system can recover the machine.

Programs can run again.

New processes can be created.

Unreachable remnants of the old processes can become garbage.

Applications whose persistent state requires stronger consistency guarantees can add those guarantees at the level where the required semantics are actually known.

This is presently the simplest reconstruction that fits the evidence without inventing an undocumented mechanism for preserving the old SCT namespace.
