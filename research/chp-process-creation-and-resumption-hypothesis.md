# CHP process creation and resumption: reconstruction hypothesis

Status: **research reconstruction / hypothesis**, 25 September 2026. This note preserves the useful reasoning chain that led to the current model. It deliberately omits discarded intermediate ideas and should not be read as a documented instruction specification.

## The problem

The Process Dump Stack contains enough architectural state to resume an existing process. The difficult case is therefore not resumption but **first creation**: before a process has ever run, no valid Dump Stack exists, yet after first activation the system must retain a protected capability to the Dump Stack so that the process can subsequently be resumed.

The Pocket Reference documents both **store-mode CHP** and **direct-mode CHP**. These forms provide fundamentally different inputs. Direct mode supplies numerical addressing information but no CAP field. Store mode supplies the normal capability-mediated store address. CHP's REG field is marked unused.

This suggests that the two forms may correspond to two different stages in a process lifecycle rather than merely two equivalent ways of naming the same operand.

## Existing-process case: direct CHP

Patent material describes normal CHANGE PROCESS as using an instruction-supplied offset down the current process's reserved segment-pointer/capability-pointer table to obtain, through the capability machinery, the Dump Stack of the process being entered.

The important point is that the direct-mode number is **not itself authority**. The authority resides in the current process's protected capability-pointer environment. Conceptually:

```text
CHP direct n
     |
     v
current process capability-pointer table [n]
     |
     v
capability for target Dump Stack
     |
     v
restore complete target process state
```

Thus direct CHP is naturally suited to an **already existing process**. Its Dump Stack already contains the state needed for restoration, including its saved capability/data state and execution frame. No C6, C7 or other startup material needs to be supplied afresh: it is already represented in the saved process state.

This also explains how the direct-mode operand can remain purely numerical without weakening capability protection. The number selects an entry in an already-authorised protected capability-pointer environment; it does not itself constitute or manufacture authority.

## New-process case: store CHP

A new process is fundamentally different: there is no valid Dump Stack to restore yet. What can exist is a capability-protected **process construction/management structure** containing the material required to establish the initial process state, notably initial C6, C7 and IAR information and any other required process parameters.

Storage for the future Dump Stack can already have been allocated by the normal storage/resource-allocation machinery. This is an important refinement of the original hypothesis: CHP need not, and probably cannot, allocate storage itself. The allocator can provide an existing empty block and a capability to it.

The current hypothesis is that **store-mode CHP is the first-activation / process-establishment form**. Its capability-mediated store operand designates the construction structure.

Conceptually, before first activation the structure can contain:

```text
Process construction / management structure

    ...
    Dump Stack slot -> capability to allocated empty storage
    ...
    C6 Initial
    C7 Code
    IAR Initial
    ... other initial process information ...
```

Store-mode CHP can then perform the M-level work required to turn this prepared state into a valid running process:

1. consume the legitimate capability-protected initial process information;
2. initialise the supplied storage as valid Process Dump Stack state;
3. ensure that the persistent Dump Stack reference has the special capability form required by the architecture;
4. install the resulting Dump Stack as the active process state;
5. start the process from its initial C6/C7/IAR context.

The unresolved point in step 3 is now sharply defined: whether the required `666` capability already exists before CHP, or whether M establishes it during CHP.

Conceptually:

```text
process construction structure
+ allocated DS storage
              |
              | CHP store
              v
      establish valid Dump Stack
              |
              +--------------------------+
              |                          |
              v                          v
persistent DS capability          activate initial
available in structure            process state
```

## The crucial persistent return path

CHP changes process. Therefore first activation cannot conveniently depend on returning a newly established Dump Stack capability in an ordinary register to the creator: execution has transferred to the new process.

The supplied process-management structure itself provides the persistent return path.

Before first activation, its Dump Stack slot refers to the storage being prepared. By the time first activation is complete, the same persistent structure must contain the capability required to reach the established Dump Stack:

```text
structure[DS-slot] = 666 capability -> established Dump Stack
```

The creator/scheduler therefore does not need an ordinary return value. When it later regains execution, the same persistent management structure contains the authority required to resume the created process.

This closes the process lifecycle:

```text
FIRST ACTIVATION

process construction structure
allocated DS storage
C6/C7/IAR initial state
        |
        | CHP store
        v
valid initial Dump Stack established
DS slot is/has become the required 666 capability
        |
        v
new process executes


SUBSEQUENT ACTIVATION

scheduler / management process
        |
        | capability-pointer-table entry reaches DS
        v
CHP direct <entry offset>
        |
        v
existing Dump Stack
        |
        v
process resumes
```

This is the central symmetry of the reconstruction: **store CHP establishes a process that does not yet possess resumable Dump Stack state; direct CHP subsequently resumes a process whose Dump Stack already exists.**

## The unresolved `666` transition: two live options

The ROS/PDOS Pocket Reference diagram shows the Process Base at offset 3 holding a `666` capability to the Dump Stack. If the present reconstruction is correct, the already allocated storage block must therefore acquire the required `666` Dump-Stack capability form no later than first activation.

At present the repository evidence does not determine **who establishes `666`**. There are two live possibilities.

### Option 1 — `666` exists before CHP

The storage/process allocation machinery creates the empty storage block and, through whatever protected capability-creation mechanism is available to that machinery, places a `666` capability to it in the process-construction structure before CHP is executed:

```text
allocator / process-construction machinery
            |
            v
allocated empty storage block
            |
            v
PB[DS-slot] = 666 capability -> empty block
            |
            | CHP store
            v
valid initial Dump Stack / process starts
```

Under this model CHP does **not** increase capability authority. Its job is to initialise/establish the processor-defined Dump Stack state in storage for which the required capability already exists.

The unresolved question moves earlier in the lifecycle:

> What K/H/T-side protected mechanism permits the allocator to establish the initial `666` capability?

### Option 2 — M changes `660` to `666` during CHP

The allocation machinery creates the empty storage block and supplies a more ordinary capability, provisionally represented here as `660`:

```text
PB[DS-slot] = 660 capability -> empty allocated block
```

Store-mode CHP then performs a narrowly defined M-level transformation while establishing the initial Dump Stack:

```text
(660, SCT[n])
      |
      | CHP store / M
      v
(666, SCT[n])
```

The physical/storage object remains the same; the special Dump-Stack capability form is established by M as part of the architecturally defined first-activation transition.

Under this model ordinary K/H/T software does **not** require a general mechanism for manufacturing `666`. The authority-changing operation is confined to CHP/M.

`660` is used here as the current candidate pre-CHP access form, **not as a documented fact**. The exact pre-CHP access code remains to be established.

### Current position

Neither option is presently preferred as historical fact. The reconstruction has been reduced to the precise unresolved question:

> **Who establishes the Dump Stack access field `666`: protected allocation/process-construction machinery before CHP, or M during store-mode CHP?**

The downstream lifecycle can be the same under either option.

## Relationship to the ROS/PDOS Pocket Reference diagram

The Pocket Reference's ROS/PDOS process structure is especially suggestive but must remain **OS-specific evidence**, not be promoted into the universal hardware definition.

It shows a Process Base with a capability at offset 3, access code `666`, leading to the Process Dump Stack, while the Dump Stack contains a `760` link back toward the Process Base:

```text
PROCESS BASE

                 666
 offset 3 -----------------> Dump Stack
                               |
                               | 760 backlink
                               v
                          Process Base
```

Under the present hypothesis, the forward Dump Stack link is exactly the kind of persistent relationship that first activation must leave behind so that the process can later be selected and resumed.

The important reconstruction is **not** that offset 3 is universally architectural. ROS/PDOS may choose offset 3 for this role; other operating systems need not use the same process-management layout.

Nor does the diagram tell us the state of offset 3 immediately before first CHP. Under Option 1 it already contains the `666` capability to empty allocated storage. Under Option 2 it contains a less powerful storage capability, provisionally `660`, which M transforms to `666` during CHP.

The `760` backlink from Dump Stack toward Process Base remains potentially significant because it gives the process state a route back to its OS-level management object, but its precise architectural role should not be assumed beyond the documented diagram.

## Why this hypothesis is attractive

This model explains several otherwise disconnected features with one mechanism:

- why CHP has materially different direct and store forms;
- how a process can be established when no valid incoming Dump Stack yet exists;
- why CHP does not need to allocate physical storage;
- how the persistent Dump Stack capability is available to the creator/scheduler despite CHP transferring execution away from it;
- why later resumption needs only a numerical offset into an already-authorised capability-pointer environment;
- why ordinary software need not necessarily possess a general mechanism for manufacturing the unusually powerful Dump Stack capability;
- why the Dump Stack has a processor-defined representation capable of preserving complete resumable process state;
- why the Pocket Reference can show an OS process-management object with a persistent `666` capability link to its Dump Stack.

In the M/H/T reconstruction, the two `666` alternatives have a useful architectural distinction. Under Option 1, protected K/H/T-side allocation machinery must already be capable of establishing `666`. Under Option 2, the exceptional authority transition is explicitly confined to **M** as part of CHP's defined process-establishment operation.

## Evidence boundary and open questions

The following are established independently of this hypothesis:

- CHP exists in both store and direct forms.
- Direct mode does not contain a CAP field; store mode does.
- CHP's REG field is documented as unused.
- CHP changes the active Dump Stack/process context.
- an existing Dump Stack contains the architectural state needed to resume its process.
- patent material describes normal CHANGE PROCESS using an instruction-supplied offset through reserved capability/segment-pointer machinery to obtain an incoming dump area.
- storage/resource allocation machinery can create blocks and deliver capabilities for them.
- the ROS/PDOS Pocket Reference diagram shows a Process Base link at offset 3, access `666`, to the Dump Stack and a `760` backlink from Dump Stack to Process Base.

The following remain **hypotheses requiring corroboration**:

- store-mode CHP is specifically the new-process/first-activation form;
- the exact contents and layout of the store-mode process-construction operand;
- whether the Dump Stack storage capability is already `666` before CHP (Option 1);
- whether M changes a pre-existing storage capability, provisionally `660`, to `666` during CHP (Option 2);
- the exact pre-CHP access code if Option 2 is correct;
- the detailed initialisation and validation performed by CHP;
- how failures, rollback and resource exhaustion are handled.

These uncertainties should be resolved without assuming that the ROS/PDOS Process Base layout is universal across COS, POS, ROS and PDOS.

## Provenance correction

An earlier research note used the recalled example `CHP 3 0 C6`. That recollection has been withdrawn as unreliable and **must not be used as evidence**. The present hypothesis does not depend on that recalled instruction. Any appearance of offset 3 here comes solely from the documented ROS/PDOS Process Base diagram and is explicitly treated as an OS-specific structural observation, not as universal CHP syntax.

Discarded exploratory paths, including the proposed LDP-to-CHP connection, are intentionally omitted from this note because they did not contribute to the surviving reconstruction.
