# CHP process creation and resumption: reconstruction hypothesis

Status: **research reconstruction / hypothesis**, 25 September 2026. This note preserves the useful reasoning chain that led to the current model. It deliberately omits discarded intermediate ideas and should not be read as a documented instruction specification.

## The problem

The Process Dump Stack contains enough architectural state to resume an existing process. The difficult case is therefore not resumption but **first creation**: before a process has ever run, no Dump Stack exists, yet after first activation the system must retain a protected capability to the newly created Dump Stack so that the process can subsequently be resumed.

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

Thus direct CHP is naturally suited to an **already existing process**. Its Dump Stack already contains the state needed for restoration, including its saved capability/data state and execution frame.

## New-process case: store CHP

A new process is different: there is no Dump Stack to designate yet. What can exist is a capability-protected **process construction/management structure** containing the material required to establish the initial process state, notably the initial C6, C7 and IAR information and any other required process parameters.

The current hypothesis is that **store-mode CHP is the first-activation / process-creation form**. Its capability-mediated store operand designates this construction structure.

Before first activation the structure contains an empty Dump Stack slot:

```text
Process construction / management structure

    ...
    Dump Stack slot: EMPTY
    ...
    C6 Initial
    C7 Code
    IAR Initial
    ... other initial process information ...
```

Store-mode CHP can then perform an M-level operation that ordinary software cannot safely perform:

1. consume the legitimate capability-protected initial process information;
2. create/initialise a valid Process Dump Stack;
3. mint the protected Dump Stack capability required by the architecture;
4. write that capability into the previously empty Dump Stack slot in the supplied structure;
5. install the new Dump Stack as the active process state and start the process from its initial C6/C7/IAR context.

Conceptually:

```text
process structure with empty DS slot
              |
              | CHP store
              v
      create valid Dump Stack
              |
              +--------------------------+
              |                          |
              v                          v
write DS capability into          activate initial
process structure                 process state
```

## The crucial persistent return path

CHP changes process, so process creation cannot conveniently depend on returning the newly minted Dump Stack capability in an ordinary register to the creator.

The supplied store structure itself provides the persistent return path.

Before creation:

```text
structure[DS-slot] = EMPTY
```

After store-mode CHP:

```text
structure[DS-slot] = capability -> newly created Dump Stack
```

The creator/scheduler therefore does not need an ordinary return value. When it later regains execution, the same persistent management structure now contains the authority required to resume the created process.

This closes the process lifecycle:

```text
FIRST ACTIVATION

process construction structure
DS slot empty
C6/C7/IAR initial state
        |
        | CHP store
        v
new Dump Stack created
DS capability written back into structure
        |
        v
new process executes


SUBSEQUENT ACTIVATION

scheduler / management process
        |
        | capability-pointer-table entry now reaches DS
        v
CHP direct <entry offset>
        |
        v
existing Dump Stack
        |
        v
process resumes
```

## Relationship to the ROS/PDOS Pocket Reference diagram

The Pocket Reference's ROS/PDOS process structure is especially suggestive but must remain **OS-specific evidence**, not be promoted into the universal hardware definition.

It shows a Process Base with a capability at offset 3, access code `666`, leading to the Process Dump Stack, while the Dump Stack contains a `760` link back toward the Process Base. Under the present hypothesis, the forward Dump Stack link is exactly the kind of persistent result that first activation must establish.

A possible ROS/PDOS realisation is therefore:

```text
Process Base before first activation

offset 3 : EMPTY
... initial process material ...

             CHP store
                 |
                 v
Process Base after first activation

                 666
 offset 3 -----------------> Dump Stack
                               |
                               | 760 backlink
                               v
                          Process Base
```

The important reconstruction is **not** that offset 3 is universally architectural. It is that a process-management structure can contain an initially empty Dump Stack-capability slot which CHP fills when creating the process. ROS/PDOS may choose offset 3 for that role.

## Why this hypothesis is attractive

This model explains several otherwise disconnected features with one mechanism:

- why CHP has materially different direct and store forms;
- how a process can be created when no incoming Dump Stack yet exists;
- how the newly created Dump Stack capability gets back to the creator/scheduler despite CHP transferring execution away from it;
- why later resumption needs only a numerical offset into an already-authorised capability-pointer environment;
- why ordinary software need not possess a general mechanism for manufacturing the unusually powerful Dump Stack capability;
- why the Dump Stack has a processor-defined representation capable of preserving complete resumable process state;
- why the Pocket Reference can show an OS process-management object with a persistent capability link to its Dump Stack.

In the M/H/T reconstruction, this would place Dump Stack creation in **M**: ordinary T-level computation supplies authorised construction material through H's capability mechanisms, while M creates the special valid process-state object and its capability.

## Evidence boundary and open questions

The following are established independently of this hypothesis:

- CHP exists in both store and direct forms.
- Direct mode does not contain a CAP field; store mode does.
- CHP changes the active Dump Stack/process context.
- an existing Dump Stack contains the architectural state needed to resume its process.
- patent material describes normal CHANGE PROCESS using an instruction-supplied offset through reserved capability/segment-pointer machinery to obtain an incoming dump area.
- the ROS/PDOS Pocket Reference diagram shows a Process Base link at offset 3 to the Dump Stack and a backlink from Dump Stack to Process Base.

The following remain **hypotheses requiring corroboration**:

- store-mode CHP is specifically the new-process/first-activation form;
- CHP itself allocates or otherwise creates the Dump Stack storage rather than being supplied preallocated storage in the construction structure;
- CHP itself creates the special Dump Stack capability and writes it into an initially empty slot;
- the ROS/PDOS offset-3 `666` link is the concrete result of that store-mode CHP operation;
- the exact contents and layout of the store-mode process-construction operand;
- how storage allocation, failure, rollback and resource exhaustion are handled.

These uncertainties should be resolved without assuming that the ROS/PDOS Process Base layout is universal across COS, POS, ROS and PDOS.

## Provenance correction

An earlier research note used the recalled example `CHP 3 0 C6`. That recollection has been withdrawn as unreliable and **must not be used as evidence**. The present hypothesis does not depend on that recalled instruction. Any appearance of offset 3 here comes solely from the documented ROS/PDOS Process Base diagram and is explicitly treated as an OS-specific structural observation, not as universal CHP syntax.
