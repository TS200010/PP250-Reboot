# PP250 Reboot

**Reconstructing the Plessey System 250...**

The Plessey System 250 was an early commercial capability-based computer developed in the United Kingdom by Plessey. It was a groundbreaking computer architecture and which has perhaps more relevance today than in the 1970's.

**PP250 Reboot** is an attempt to reconstruct the System 250 from the surviving technical documentation, software, patents, publications, hardware information, and the recollections of people who worked on the project.

The immediate objective is to gather, preserve, digitise and cross-reference as much reliable information about the machine as possible. Once that body of evidence has been assembled, we can reconstruct the architecture and determine what can be established about the machine's behaviour.

That reconstruction can then become executable: first as a reference emulator, then potentially as a hardware implementation on an FPGA. Beyond that lies a further objective—to explore what a modern computer derived from PP250's capability architecture could become.

**Why?**

PP250 occupies a distinctive place in the history of (capability-based) computing. It was not merely a theoretical architecture: it became an operational commercial system, and those who worked closely with it understood that its err... capabilities went well beyond conventional computer architectures of its time.

Yet PP250 has received surprisingly little recognition for the power of the ideas embodied in its design. There is an important distinction between understanding PP250 as a collection of hardware and software features and understanding the computational model that lay behind it. The former can be learned from documentation; the latter requires seeing what the architecture makes possible.

Much of the history of computing is described through the lens of the conventional von Neumann model, in which computation operates on data and addresses in a shared memory. PP250 approached computing from a different foundation: the ideas associated with capability systems and, ultimately, with the more general computational principles explored by Church and others. That difference is easy to miss if PP250 is viewed simply as an unusual computer of its period.

For the people who worked deeply with the system, the implications could be apparent in the architecture itself. For those who encountered PP250 only through descriptions of it, much of that significance was easier to overlook. This project is an attempt to recover not only what PP250 did, but to understand the architectural ideas that made it possible—and to make those ideas accessible and relavent again.

**Materials and Purpose**

Much of the original technical material is now difficult to find, and some aspects of the architecture are incompletely documented in the surviving sources.

This project therefore has three connected purposes:

* Preservation — collect and make accessible surviving PP250 technical and historical material.
* Reconstruction — determine, as accurately as possible, how the system actually worked.
* Experimentation — make the reconstructed architecture executable and use it to investigate the continuing value of its capability model.

A working emulator and hardware implementations would be consequences of that work, not substitutes for establishing the evidence.

**Evidence before assumptions**

A central principle of the project is to distinguish between:

* **Documented** — explicitly stated in a surviving source.
* **Corroborated** — supported independently by more than one source.
* **Inferred** — a technically strong conclusion derived from available evidence.
* **Uncertain** — an interpretation for which the evidence is incomplete.
* **Speculative** — a possibility requiring further evidence.
* **Gaps** — a known gap in the record requiring further research. 

The project should not just silently fill gaps in the historical record.

Where two sources disagree, the disagreement itself is recorded.

**Primary reconstruction: from inert hardware to the first real program**

The main technical objective of PP250 Reboot is not, in the first instance, to recreate ROS, POS, or the complete historical System 250 software environment. It is to reconstruct the **PP250 hardware architecture and its initial state**, starting with an inert machine and continuing until the point at which the first legitimate ordinary PP250 program can execute.

That boundary gives the historical investigation a concrete engineering target:

> **Starting from an inert machine, what hardware state and microcode mechanisms are required to reach the first legitimate execution of ordinary PP250 software?**

### Below the boundary — the machine we must reconstruct

Everything needed to reach that first instruction is part of the primary reconstruction. This includes, insofar as the surviving evidence allows us to establish it:

* processors, store modules and the shared bus;
* the 24-bit data registers and 48-bit capability registers;
* special-purpose processor registers;
* capability representation, validation and access checking;
* physical/module addressing;
* the System Capability Table (SCT) mechanism;
* dump-stack representation and manipulation;
* faults and the hardware/microcode fault sequence;
* Change Process (CHP);
* watchdog and interval-timer mechanisms;
* processor initialisation and admission;
* cold-start behaviour;
* any state held outside the ordinary programmer-visible architecture during start-up;
* the mechanism by which the first valid capabilities come into existence;
* and whatever initial memory/SCT/dump-stack structures must already exist before ordinary software can run.

The reconstruction must not hide the difficult bootstrap problem behind an assumed operating system. If the first program needs a capability, an SCT entry, C6/C7, a dump stack or some other protected state, the project must explain **where it came from and how the machine could legitimately have constructed it**.

### The boundary — the first legitimate process

Immediately before the first ordinary software instruction executes, we should eventually be able to describe the complete relevant processor state.

That means answering questions such as:

* What is in C0-C7, particularly C6 and C7?
* What is in D0-D7?
* What are the IAR, indicators and other control state?
* What are the watchdog and timer states?
* What points to the SCT and dump stack?
* Which SCT entries already exist and what do they describe?
* Which capabilities already exist?
* Where did those capabilities come from?
* Where is the first executable code and how was it loaded?
* What exact event — cold start, fault sequence, CHP, or combination — causes the first instruction to be fetched?

The desired result is not merely a plausible register dump. Every element of that initial state should have a provenance through documented hardware behaviour or an explicitly identified reconstruction where documentation is missing.

### Above the boundary — proof that the reconstructed machine is sufficient

The complete historical operating system is outside this initial reconstruction boundary. However, the architecture immediately above the boundary is still important because it provides a **proof of sufficiency**.

The first real program must receive enough legitimate authority to build a functioning capability system from the state provided by the machine. It should be possible, using normal PP250 mechanisms, to:

```
FIRST REAL SOFTWARE INSTRUCTION
            |
            v
legitimate initial capabilities
            |
            +-- manage/allocate physical resources
            +-- establish and manage SCT entries
            +-- derive restricted capabilities
            +-- construct process/dump-stack state
            +-- create another process
            +-- CHP to that process
                         |
                         v
              SELF-SUSTAINING SYSTEM
```

We do not need to reproduce the historical Store Allocator or the whole historical operating system merely to prove the hardware reconstruction. A small demonstrator could eventually establish the same architectural fact: **the machine has supplied enough legitimate initial state for software to construct everything above it using the ordinary capability mechanisms.**

This gives us a powerful test for every bootstrap hypothesis.

If the proposed initial state requires the first program to manufacture a capability from nothing, something is missing.

If it requires an undocumented supervisor mode or a permanent escape from capability protection, something is probably wrong.

If it supplies sufficient ancestral authority for the first software to derive the required resources and then irreversibly reduce or transfer that authority, it is architecturally viable and can be tested against the surviving evidence.

### Definition of success

The primary reconstruction reaches its target when:

> **Hardware/microcode can transition from power-on state, through the documented fault/CHP and capability mechanisms, into a valid first process possessing sufficient legitimate capability authority to construct all subsequent software-managed resources, without relying on an undocumented privileged mode or arbitrary capability fabrication.**

At that point we have reconstructed a machine capable of becoming a working System 250, rather than merely implementing isolated PP250 instructions.

Current work on capability genesis, the primordial resource allocator, SCT creation and reclamation, dynamic resource admission/removal, processor admission through the fault machinery, and the origin of C6/C7 should therefore be judged against this objective. These investigations are not attempts to reconstruct the whole operating system. They are ways of determining whether the hardware and initial state we reconstruct are **sufficient to support a hardware system that can actually work**.

**The technical reconstruction**

The eventual reconstruction is expected to cover areas including:

* processor architecture
* instruction set
* registers and indicators
* capability representation and protection
* addressing and segmentation
* C-register usage
* memory and the system bus
* multiprocessor operation
* interrupts
* I/O
* assembler and compiler
* linker and loader
* operating software
* CORAL and other languages
* microcode, where evidence exists
* hardware implementation
* programming conventions and system software

**Executable reconstruction**

A reference emulator is a later objective. It should implement the architecture that the evidence supports, rather than an architecture invented to make an emulator convenient to write.

Where historical behaviour cannot be established, the emulator should make assumptions explicit. In that sense it will be an executable reconstruction: a way to test whether the individual conclusions form a complete and coherent machine.

A possible next stage is an RTL implementation on an FPGA. The FPGA would implement PP250 architectural behaviour as digital hardware—registers, instruction execution, capability checks, process changes, memory access and faults—without necessarily reproducing the original gates, timing or microcode.

The intended progression is:

1. evidence-backed architectural model;
2. reference emulator and assembler;
3. test suite comparing documented and reconstructed behaviour;
4. FPGA implementation checked against the reference model;
5. experimental multiprocessor, memory and peripheral systems;
6. only if useful, consideration of an ASIC.

The distinction between architecture and implementation must remain explicit. An FPGA implementation could legitimately be a reconstructed PP250 while using a completely different modern microarchitecture internally.

**A modern descendant**

Historical reconstruction is one machine. A modern PP250-derived capability computer is another. The project should not silently modernise uncertain parts of the original and present the result as history.

The descendant would preserve and test the central ideas that make PP250 distinctive:

* capabilities as the source of authority;
* hardware-enforced access to segments and objects;
* capability registers that cannot be manipulated as ordinary data;
* protected execution domains and C6/C7-style context;
* process switching and multiprocessor shared memory;
* protection without relying on a conventional privileged supervisor escape hatch.

It need not preserve features that existed because of 1970s technology. Word size, physical bus signalling, memory technology, storage and peripheral interfaces may all be redesigned.

The outside of the machine should be modern. USB, SPI, SD storage, displays, networking and contemporary debugging interfaces could sit behind a modern interconnect such as Wishbone, AXI-Lite or a simpler purpose-built fabric. Memory-mapped I/O remains useful where it fits, but reproducing the original PP250 bus is not an objective in itself.

The essential rule is that a modern transaction must not bypass the capability model. Access to RAM or a device should be derived from authority held by the initiating process. A process given access to an SD controller should not thereby acquire access to USB or unrelated memory. DMA is a particularly valuable experiment: a device could be given a bounded capability describing exactly which memory it may access instead of receiving an unrestricted physical address.

This gives the project a longer-term research question:

> What would the Plessey System 250 capability architecture look like if the machine were designed today?

The historical work provides a rigorous starting point for answering that question. It prevents the modern machine from becoming merely a new capability design with a PP250 label attached to it.

**Sources**

The repository will contain both machine-readable transcriptions and references to the original material.

Original documents should be preserved separately from derived material. Transcriptions and interpretations must never replace the original evidence.

**Contributions**

Information from former PP250 engineers, programmers, users and maintainers is particularly valuable.

If you have PP250 documentation, software, photographs, technical notes, personal recollections or information about people involved in the project, please get in touch or open an issue.

**PP250 Reboot** is an historical and technical reconstruction project.

The goal is simple:

**Find out what PP250 really was, preserve what can still be recovered, make it possible to run again, and discover what its capability architecture can still become.**

## Repository layout

- `documentation/` — project documentation and guidance.
- `sources/` — records of original or reproduced source material.
- `transcriptions/` — working transcriptions derived from source material.
- `architecture/` — evidence-based notes on system architecture.
- `instruction-set/` — material concerning the instruction set.
- `software/` — preserved software-related material and its documentation.
- `people/` — research notes on people connected with the subject.
- `patents/` — patents and related research material.
- `research/` — research notes, leads, and bibliography.
- `emulator/` — reference implementation and executable architectural experiments.
- `hardware/` — future RTL, FPGA, interconnect and physical implementation work.

## Working principles

- Preserve provenance: identify where material came from and its known status.
- Separate source images or copies from transcriptions and interpretation.
- Do not present unverified claims as historical fact.
- Prefer small, well-described additions over speculative reconstruction.

## Contributing

Please describe the source, date of access or acquisition where known, any transformations made, and remaining uncertainties when adding archival or research material.

## About me...

As you might have guessed I worked on the PP250 in Taplow Berkshire. A programmer on the CORAL 250 Compiler team and later its project leader.
