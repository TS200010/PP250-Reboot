# System 250 Architecture — Faults, Interrupts and Startup

### C(N), the Normal Interrupt Block, and automatic CHP

The normal operational trap path is distinct from the fault/start-up path through `C(S)`. `C(N)` / `C13` designates the **Normal Interrupt Block (NIB)**. The NIB contains the capability pointer used to identify the Dump Stack of the Normal Interrupt process. For an automatic change process there is no explicit CHP instruction supplying an incoming-process operand, so this NIB pointer supplies the target required by the CHP machinery.

The architectural chain is therefore:

```text
normal interrupt / trap condition
        |
        v
      C(N)
        |
        v
Normal Interrupt Block
        |
        v
incoming Dump Stack capability/pointer
        |
        v
automatic CHP
        |
        v
Normal Interrupt process
```

The Dump Stack then supplies the ordinary process state restored by CHP, including the capability and data registers and the C6/C7/IAR execution context. `C(N)` does **not** itself contain C6/C7; it supplies the route to the process Dump Stack from which the process context is restored.

### Establishing C(N): SPECIAL and the C(S)-rooted startup chain

The Pocket Reference places MIP (Primary Indicator Register) at Dump Stack offset octal `20`, in the common hardware process-state area. Patent material establishes that CHP saves/restores the primary indicator state and identifies **SPECIAL** as a one-instruction primary-indicator state which permits an `LC` instruction to address the corresponding special-purpose capability register rather than the ordinary C-register bank. `LC` still has its ordinary direction: a stored capability is read and loaded into the selected capability register.

This yields the current reconstructed bootstrap:

```text
C(S) hardware-rooted fault/start-up state
        |
        v
checkout / startup machinery
        |
        v
first legitimate process image
(including MIP at Dump Stack offset 20)
        |
        v
CHP restores MIP with SPECIAL available
        |
        v
one LC loads C(N)
        |
        v
normal interrupt callback path established
```

This is recorded as **reconstructed architecture** under the repository's observation → constraint → reconstruction method. The surviving material establishes the components independently: C(S) roots fault/start-up; CHP restores process indicator state; MIP is in the Dump Stack; SPECIAL redirects one LC to the special capability-register bank; and C(N) is required for normal automatic interrupt entry. Taken together they make C(S) the ancestry of the authority/state by which the running system establishes C(N), rather than requiring an unexplained second root of processor authority.

This does not mean ordinary normal interrupts traverse the destructive C(S)/checkout path. C(S) establishes the initial trusted running state; C(N), once established, is the normal operational entry path.

**Evidence qualification, 2 October 2026:** US3814919A confirms the early fault-block/root and automatic CHP sequence; its subsequent-fault branch bypasses the outgoing dump (Description 132). This supplies a documented recovery precedent for restoration without a further outgoing dump, but does not establish the virgin cold-start path. US3771146A saves early SECOND GROUP bit 4 with primary state; US4486831A documents saved/restored PIR and one-instruction SPECIAL selection (53, 58). The prepared-process/SPECIAL bootstrap above remains a reconstruction combining version-sensitive observations, not a directly documented universal microsequence. Automatic detection of incomplete special registers and repeated CHP grants is a competing speculative explanation. Initial memory/SCT population and primordial authority provenance remain unresolved. See the [startup evidence update](../research/pp250-boot-and-processor-startup.md#evidence-update--2-october-2026-verified-fault-roots-and-startup-alternatives).

### Trap discrimination and storage management

The Normal Interrupt process can recover information about the capability/reference responsible for the suspended operation from the saved process state/dump stack and use its form/type to discriminate the required software action.

The contemporary patent descriptions distinguish the broad cases:

- active (`11`) capability whose SCT representation is unavailable: segment/store-management handling;
- passive/backing-store (`10`) capability: page-changing/disc handling is required;
- resource (`01`) capability: resource/I/O handling;
- null (`00`) capability: null/trap handling rather than usable authority.

This provides a capability-native virtual-store path rather than a conventional privileged page-fault handler. Hardware detects an unusable authority state and performs the protected process transition; ordinary capability-constrained system processes determine the reason and perform storage management.

A reconstructed page-in path is therefore:

```text
faulting process
      |
      | attempts to use unavailable/passive authority
      v
hardware detects trap representation
      |
      v
C(N) -> NIB -> incoming Dump Stack
      |
      v
automatic CHP to Normal Interrupt process
      |
      | inspect saved offending reference/state
      v
store/page-management process
      |
      +-- determine required segment
      +-- obtain/allocate main-store space
      +-- arrange disk-to-store transfer
      +-- install/update SCT physical descriptor
      +-- establish valid SCT check/state
      v
I/O proceeds / completes
      |
      v
scheduler can make waiting process runnable
      |
      v
original operation can be retried/resumed
```

England's system description identifies the store-management package as responsible for moving blocks between backing store and main store, and Plessey allocation patent material describes disk-to-main-store transfer being initiated by an I/O-handler process, proceeding asynchronously, and completion feeding back into scheduling of waiting processes.

The important architectural point is that this does **not** require a permanently privileged supervisor execution mode. The hardware supplies protected state detection and process transition; the higher-level storage policy is implemented by capability-controlled software.
