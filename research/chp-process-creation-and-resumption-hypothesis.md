# CHP process creation and resumption: reconstruction hypothesis

Status: **research reconstruction / hypothesis**, 25 September 2026.

## Core reconstruction

The Process Dump Stack contains enough architectural state to resume an existing process. The difficult case is first activation: storage for a Dump Stack can already have been allocated, but no valid resumable process state yet exists.

The Pocket Reference documents both store-mode and direct-mode CHP. Direct mode supplies numerical addressing information but no CAP field; store mode supplies capability-mediated store addressing. This suggests two stages of the process lifecycle.

### Existing process: direct CHP

Patent material describes normal CHANGE PROCESS as using an instruction-supplied offset down the current process's reserved segment-pointer/capability-pointer table to obtain the incoming Dump Stack through the capability machinery.

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
restore target process state
```

Thus direct CHP is naturally suited to resuming an already established process. The numerical operand is not itself authority; the authority resides in the protected capability-pointer environment.

### New process: store CHP

Before first activation, normal storage/resource-allocation machinery can allocate the block intended to become the Dump Stack. A capability-protected process construction/management structure can contain the initial C6, C7, IAR and other required startup information, together with a capability to that allocated block.

The current hypothesis is that store-mode CHP is the first-activation/process-establishment form. CHP need not allocate storage; it establishes valid processor Dump-Stack state in storage already supplied to it and starts the new process.

## The unresolved `666` transition: two live options

The ROS/PDOS Pocket Reference diagram shows the Process Base at offset 3 holding a `666` capability to the Dump Stack. If the reconstruction above is correct, the allocated storage block must acquire the required `666` Dump-Stack capability form no later than first activation.

At present the repository evidence does not determine who establishes `666`. There are two live possibilities.

### Option 1 — `666` exists before CHP

The storage/process allocation machinery creates the empty storage block and, through whatever protected capability-creation mechanism is available to it, establishes a `666` capability before CHP is executed.

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

Under this model CHP does not increase capability authority. It establishes the processor-defined Dump Stack state in storage for which the required capability already exists.

The unresolved question is then: **what K/H/T-side protected mechanism permits the allocator to establish the initial `666` capability?**

### Option 2 — M changes `660` to `666` during CHP

The allocation machinery creates the empty storage block and supplies a more ordinary capability, provisionally represented as `660`.

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

The storage object remains the same; M establishes the special Dump-Stack capability form as part of first activation. Under this model ordinary K/H/T software does not require a general mechanism for manufacturing `666`.

`660` is a current candidate for the pre-CHP access form, **not a documented fact**. The exact pre-CHP access code remains to be established.

### Current position

Neither option is presently preferred as historical fact. The uncertainty has been reduced to a precise question:

> **Who establishes the Dump Stack access field `666`: protected allocation/process-construction machinery before CHP, or M during store-mode CHP?**

The downstream lifecycle is compatible with either option.

## Persistent process state and subsequent resumption

By the time first activation is complete, the process-management structure contains the capability required to reach the established Dump Stack:

```text
structure[DS-slot] = 666 capability -> Dump Stack
```

When the creator/scheduler later regains execution, this persistent capability provides the authority required for subsequent resumption. Direct CHP needs only the numerical offset that resolves through the current process's protected capability-pointer environment to the existing Dump Stack.

```text
FIRST ACTIVATION

allocated DS storage + initial C6/C7/IAR
        |
        | CHP store
        v
valid initial Dump Stack established
DS capability is/has become 666
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

## Relationship to the ROS/PDOS Pocket Reference diagram

The Pocket Reference's ROS/PDOS process structure is suggestive but remains OS-specific evidence. It shows a Process Base with a capability at offset 3, access code `666`, leading to the Process Dump Stack, while the Dump Stack contains a `760` link back toward the Process Base.

```text
PROCESS BASE

                 666
 offset 3 -----------------> Dump Stack
                               |
                               | 760 backlink
                               v
                          Process Base
```

This is evidence of the post-establishment relationship. It does not tell us the state of offset 3 immediately before first CHP. Under Option 1 it already contains the `666` capability to empty allocated storage. Under Option 2 it contains a less powerful storage capability, provisionally `660`, which M transforms to `666` during CHP.

Offset 3 must not be treated as universally architectural; it is documented here for ROS/PDOS.

## Evidence boundary

Established independently of this hypothesis:

- CHP exists in both store and direct forms.
- Direct mode has no CAP field; store mode does.
- CHP changes process context using Dump Stack state.
- an existing Dump Stack contains the state needed to resume its process.
- patent material describes normal CHANGE PROCESS using an instruction-supplied offset through reserved capability/segment-pointer machinery to obtain an incoming dump area.
- storage/resource allocation machinery can create blocks and deliver capabilities for them.
- the ROS/PDOS Pocket Reference shows a Process Base link at offset 3, access `666`, to the Dump Stack and a `760` backlink.

Still requiring corroboration:

- store-mode CHP is specifically the first-activation/process-establishment form;
- the exact store-mode construction structure;
- whether `666` exists before CHP (Option 1) or is established by M during CHP (Option 2);
- the exact pre-CHP access code if Option 2 is correct;
- the detailed initialisation/validation performed by CHP.

## Provenance correction

The earlier recalled example `CHP 3 0 C6` has been withdrawn as unreliable and must not be used as evidence. This reconstruction does not depend on it. Any appearance of offset 3 here comes solely from the documented ROS/PDOS Process Base diagram and is treated as OS-specific, not as universal CHP syntax.
