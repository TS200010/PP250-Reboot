# Capability-Native Process Environment and Linking

**Status:** PP250-Reboot research hypothesis  
**Date:** 2026-09-23

## 1. Observation

A conventional linker is required largely because separately compiled code contains symbolic references which must ultimately be bound to implementation addresses.

The PP250 capability model suggests that this may not be necessary for calls between protected domains.

An interface can instead be defined by fixed offsets within a capability block. For example:

```text
ACCOUNT_SERVICE = 1
OPEN_ACCOUNT    = 4
```

The caller needs to know only these interface constants. It need not know the physical address, SCT entry, code segment, or implementation identity of the service.

Conceptually:

```text
LC    ACCOUNT_SERVICE,0,C6
CALL  OPEN_ACCOUNT,0,C6
```

The first operation obtains an authority; the second invokes an interface entry relative to that authority. The symbolic names can therefore be resolved to small constants at compile/assembly time rather than linked to addresses.

This raises the hypothesis that conventional linking between such components may be unnecessary. The remaining construction problem is not symbol-to-address binding but the construction of the correct capability graph.

## 2. Process environment hypothesis

The idea can be simplified further.

Rather than the operating system automatically giving every process a globally known system-services capability, adopt a convention that a process's initial C6 capability block contains, at a fixed offset, the environment authority supplied by its creator. For illustration:

```text
C6[0] -> parent-supplied environment/system-services capability
```

The exact offset and instruction sequence are hypotheses until verified against primary PP250 documentation. The architectural point does not depend on offset zero specifically.

If a creator wants a child process to have access to system services, it must place an appropriate capability at this conventional entry. If it does not, the child has no route to those services.

This is not a denial implemented by consulting an access-control list. It is absence of authority.

## 3. Authority propagates through creation

A process therefore need not receive universal ambient operating-system authority.

```text
creator
   |
   | creates process and supplies selected capabilities
   v
new process C6
   |
   +-- [0] -> environment capability
                |
                +-- service A
                +-- service B
                +-- object/service C
```

The creator can omit the environment capability entirely, or potentially supply a reduced environment exposing only selected services.

Thus process creation is also authority delegation.

## 4. Reachability defines the process's world

This leads to a stronger formulation:

> A process's accessible world is the transitive closure of the capabilities reachable from its initial capability environment.

The executable code alone does not determine what resources or services the process can access.

Two instances of the same binary can execute in different authority environments because their creators supply different capability graphs.

There need be no global name such as `AccountService` which the process can resolve independently. The program can only traverse capabilities reachable from authority it already possesses.

## 5. Interface constants rather than linked addresses

A service interface may therefore look conceptually like:

```text
ACCOUNT_SERVICE = 1
OPEN_ACCOUNT    = 4
CLOSE_ACCOUNT   = 5
GET_BALANCE     = 6
```

These constants are part of the interface ABI. They contain no implementation address.

Compare:

```text
Conventional model

symbol
  -> linker
  -> implementation address
```

with the proposed capability-native model:

```text
symbol
  -> compile-time interface offset

creator
  -> supplies authority/capability graph

capability + interface offset
  -> protected invocation
```

Implementation placement can therefore be independent of the caller. Code can move, SCT mappings can change, and an implementation may potentially be replaced without rewriting callers, provided the published capability interface remains compatible.

## 6. Operating-system services become ordinary protected services

An important consequence is that there may be no architectural requirement for a universally reachable privileged operating-system interface.

What we conventionally call operating-system services can instead be protected domains reachable through capabilities deliberately supplied to a process.

The process does not acquire authority because it is a process or because an OS recognises its identity. It possesses authority because another authority deliberately gave it capabilities.

This fits the broader PP250-Reboot proposition:

> Authority should be represented by possession of unforgeable capability paths, not by ambient access followed by permission checks.

## 7. Relationship to inter-capabilities

An Enter capability can be understood as an edge between independently protected domains.

The target domain's C6 block provides its protected environment and interface structure. The caller need know only the interface-relative selector. CALL performs the protected transition without exposing the implementation's internal authority.

Thus an inter-domain interface can be represented as:

```text
authority to domain + interface offset
              |
              v
             CALL
              |
              v
protected execution in target capability environment
```

This is more than a procedure pointer. It is an authority-bearing connection between protection domains.

## 8. Consequence for software construction

If this model is historically correct, the apparent absence so far of a conventional PP250 linker becomes less surprising.

The system-building problem changes from:

> Where is the implementation of this external symbol?

to:

> What capabilities must this process receive when it is constructed?

That resembles dependency injection structurally, but with a crucial difference: the dependencies are hardware-enforced authorities rather than ordinary software references.

## 9. Evidence status and questions

This note records an architectural hypothesis emerging from PP250-Reboot discussion. It must not yet be presented as established historical PP250 behaviour.

Primary-source work should now test:

- how initial C6 capability blocks were constructed;
- which capabilities were conventionally present when a process was created;
- whether system-service capabilities occupied conventional CCB offsets;
- exact LC semantics and destination-register behaviour;
- how CALL selected entries relative to a supplied capability;
- whether CORAL/System 250 separately compiled modules used external symbols;
- whether a linker, linkage editor, binder, loader, or equivalent existed;
- how published service/procedure interfaces were represented;
- whether interface offsets were defined as compile-time constants;
- how capability graphs were assembled when applications/processes were created.

The key distinction to preserve during that investigation is:

> **Linking code locations and constructing authority relationships are different operations. PP250 may substantially replace the former with the latter.**
