# Levy Chapter 4 evidence: PP250 object/resource architecture

**Status:** research evidence addition, 26 September 2026  
**Evidence class:** SECONDARY EVIDENCE  
**Source:** Henry M. Levy, *Capability-Based Computer Systems* (Digital Press, 1984), Chapter 4, “The Plessey System 250”.  
**Local working source:** `transcriptions/levy-capability-based-computer-systems-chapter-4-pasted-extract.txt` (chat-derived text; original PDF still to be acquired and checked).

## Why this matters to the top-down PP250 research programme

The PP250-Reboot top-down work has independently been exploring whether System 250's capability structure can be understood as more than protected memory: in particular, whether an Enter capability plus a Central Capability Block (CCB) and operation offset forms a hardware-supported object/service interface resembling, in modern terms, an object reference plus protected dispatch table.

Levy's Chapter 4 provides important secondary evidence that the actual Plessey operating-system resource model was organised in substantially this way.

## Levy's resource model

Levy describes the operating system as a collection of protected subsystems managing resources. He distinguishes ordinary segment capabilities, on which hardware can operate directly, from logical-resource capabilities. A logical resource is addressed by an **Enter capability** through which the holder requests operations on that resource.

The resources he lists include storage segments, processes, users, jobs, text files, symbol directories, data streams and synchronising flags.

Users obtain operating-system services through a **Central Facilities Block** containing Enter capabilities for resource-allocation routines. Creating a resource returns an entry/Enter capability through which that resource can subsequently be manipulated.

For each individual resource instance, Levy says that the resource's representation is defined by the **Central Capability Block addressed by the returned Enter capability**. That CCB contains:

- execute capabilities for procedures that implement the resource's operations; and
- capabilities for segments containing the resource's private/state data.

A distinct CCB exists for each resource instance, while resources of the same type can share the same code segments.

Conceptually:

```text
holder
  |
  | Enter capability
  v
resource-instance CCB
  |
  +-- offset n --> execute capability --> shared resource-type code
  +-- offset m --> execute capability --> shared resource-type code
  +-- ...
  |
  +------------> capability --> private state of this resource instance
  +------------> capability --> other implementation resources
```

The protected CALL mechanism supplies the operation selector as an offset into the entered CCB. The caller can therefore possess authority to invoke the resource interface without thereby receiving capability-read authority over the implementation capabilities inside the CCB.

## Relationship to the PP250-Reboot object/vtable interpretation

This substantially strengthens, but does not by itself prove every part of, the project's modern object-oriented interpretation.

**SECONDARY EVIDENCE from Levy:**

- logical resources are represented by per-instance CCBs;
- clients receive Enter capabilities for those resources;
- resource operations are represented by execute capabilities in the CCB;
- CALL selects an operation by an offset within the entered CCB;
- resource state is reached through capabilities held inside the CCB;
- instances of the same resource type can share implementation code.

**PP250-REBOOT INTERPRETATION / INFERENCE:** this is structurally analogous to an object reference plus protected dispatch/interface table, with hardware-enforced encapsulation. In modern terms the CCB can resemble a protected vtable combined with an authority environment, but Levy does not claim that the PP250 designers conceived it in modern C++/Swift object-oriented terminology.

This evidence should therefore be read alongside `research/source-language-semantics-as-hardware-authority.md` and the object/interface discussion in `research/pp250-capability-architecture-provenance.md`.

## Architectural consequence

The important point is stronger than “capabilities protected memory.” In the described operating system, possession of an Enter capability can denote **authority over a logical resource interface**. The capability identifies not merely storage but the right to invoke a controlled set of operations on a particular resource instance whose implementation state remains behind the capability boundary.

That is direct historical evidence relevant to the PP250-Reboot top-down programme: the original System 250 software model actually used the architecture to represent higher-level resources in a form closely related to the object/authority structure we have been reconstructing.

## Verification boundary

This note is based on Levy, a retrospective secondary source. The corresponding claims should now be traced into England 1974, Cosserat 1974 and other contemporary Plessey material. The local Levy text is itself a chat-derived working copy and must be checked against the original Chapter 4 PDF when acquired.
