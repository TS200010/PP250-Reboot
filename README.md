# PP250 Reboot

**Reconstructing the Plessey System 250...**

The Plessey System 250 (incorporating the PP250 processor) was an early commercial capability-based computer developed in the United Kingdom by Plessey. It was a groundbreaking computer architecture and which has perhaps more relevance today than in the 1970's.

**PP250 Reboot** is an attempt to reconstruct the System 250 from the surviving technical documentation, software, patents, publications, hardware information, and the recollections of people who worked on the project.

The immediate objective is to gather, preserve, digitise and cross-reference as much reliable information about the machine as possible. Once that body of evidence has been assembled, we can reconstruct the architecture and determine what can be established about the machine's behaviour.

That reconstruction can then become executable as a reference emulator and simulator: a research bench for testing whether our reconstruction is coherent, for recreating as much of the original System 250 as the evidence permits, and for experimenting with historically distinct derivatives such as later patented developments and clearly identified new extensions.

The project is defined this way:

1. **System 250 Reconstruction Research Bench.** A software emulation/simulation environment whose primary purpose is historical and architectural reconstruction. We implement the original hardware and behaviour as faithfully as the surviving evidence permits, explicitly recording what is documented, inferred, or still unknown. Once we have a faithful baseline, the same simulator becomes an experimental platform: later Plessey patents can be implemented as variants, and we can explore our own extensions without contaminating the baseline reconstruction.

2. **Modern FPGA Capability Machine.** A new real computer inspired by the System 250 capability principles—closer in spirit to CHERIoT and CLOOMC as a modern small capability machine, but not an attempt to reproduce System 250 hardware. It uses contemporary FPGA technology, RAM, buses and peripherals. We are free to choose an existing modern interconnect, use DMA, modern storage/networking and redesign things where appropriate. The research question becomes: what does a clean hardware capability machine look like if we take the important System 250 ideas and build it today? This could eventually lead toward an ASIC if the architecture proves interesting. It is entirely possible that this work is a fork or contribution to the aforementioned projects rather than a separate ground up reconstruction. 

3. **Top-Down Capability-Native Design.** A complementary research programme that starts with the software system we would like a capability-native computer to express and works downward toward the required hardware abstractions. Rather than beginning with PP250 instructions and asking what software can be built on them, this track begins with objects, services, private state, interfaces and authority relationships and asks what capability graph, invocation mechanisms, protected structures and processor support make those abstractions natural. The objective is to determine where this independently derived top-down machine converges with the faithfully reconstructed PP250 architecture. Repeated convergence—on mechanisms such as ENTER, CALL, capability closure, authoritative capability tables and the inability of ordinary data to manufacture authority—would help distinguish fundamental architectural invariants from implementation choices imposed by 1970s technology.

**Top-down design principle — hardware protects authority; software defines meaning.** Software may define what an object represents and what its operations mean, and those semantics may be faulty without thereby creating new authority. Ordinary software state must not be able to manufacture, amplify or escape the authority legitimately granted to it. The trusted boundary should contain only the mechanisms required to preserve this authority integrity; application meaning should remain above it. The full argument is recorded in `research/authority-integrity-and-semantic-boundary.md`.

4. **Inter-computer capability research.** For now this is deliberately thinking and documentation only. The problem is whether capability authority can meaningfully extend between independently protected machines without reducing a capability to forgeable/copyable data or merely replacing the capability model with cryptographic credentials. We have interesting ideas, particularly around the distinction between representation and authority to reconstruct, but we have not solved it.

There is also an important relationship between the reconstruction and the top-down design programme. They deliberately approach the same question from opposite directions:

```
BOTTOM-UP                               TOP-DOWN

Historical evidence                     Desired software system
       |                                           |
       v                                           v
System 250 reconstruction               Objects / services
       |                                           |
       v                                           v
instructions and capability             Interfaces and authority
mechanisms                              relationships
       |                                           |
       v                                           v
SCT / CCB / processes                   Capability graph
       |                                           |
       +-------------------+   +-------------------+
                           |   |
                           v   v
                        CONVERGENCE
                             |
                             v
                Architectural invariants
                             |
                             v
             Modern FPGA Capability Machine
```

The reconstruction gives us an evidence-based understanding of what System 250 actually did. The top-down programme asks a different question: what would computing look like if capability architecture, rather than the conventional address-space/process/operating-system model, were the foundation of software design?

The two tracks are intended to meet in the middle. If a mechanism appears only because the original PP250 hardware required it, the top-down derivation may not need it. If mechanisms such as Enter, capability closure, the SCT or non-forgeable authority reappear when the machine is independently derived from software requirements, that is evidence that they may be architectural invariants rather than historical baggage.

This also changes the role of the modern FPGA machine. It need not simply be a modernised PP250. It can become the experimental implementation of the architecture that survives this two-direction analysis: historically grounded from below and independently justified from above. Experimental changes in the reconstruction research bench remain explicitly identified as variants so that they do not contaminate the historical baseline.

**Why?**

System 250 occupies a distinctive place in the history of (capability-based) computing. It was not merely a theoretical architecture: it became an operational commercial system, and those who worked closely with it understood that its err... capabilities went well beyond conventional computer architectures of its time.

Yet System 250 has received surprisingly little recognition for the power of the ideas embodied in its design. There is an important distinction between understanding PP250 as a collection of hardware and software features and understanding the computational model that lay behind it. The former can be learned from documentation; the latter requires seeing what the architecture makes possible.

Much of the history of computing is described through the lens of the conventional von Neumann model, in which computation operates on data and addresses in a shared memory. System 250 approached computing from a different foundation: the ideas associated with capability systems and, ultimately, with the more general computational principles explored by Church and others. That difference is easy to miss if System250 is viewed simply as an unusual computer of its period.

For the people who worked deeply with the system, the implications could be apparent in the architecture itself. For those who encountered described only through descriptions of it, much of that significance was easier to overlook. This project is an attempt to recover not only what described did, but to understand the architectural ideas that made it possible—and to make those ideas accessible and relavent again.

**Materials and Purpose**

Much of the original technical material is now lost, and some aspects of the architecture are incompletely documented in the surviving sources.

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

The main technical objective of PP250 Reboot is not, in the first instance, to recreate ROS, POS, or the complete historical System 250 software environments. It is to reconstruct the **PP250 hardware architecture and its initial state**, starting with an inert machine and continuing until the point at which the first legitimate ordinary PP250 program can execute.

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

**Executable reconstruction and simulation research bench**

The executable System 250 work is intended first and foremost as a **software research bench**. Its reference emulator/simulator should implement the architecture that the evidence supports rather than an architecture invented for convenience.

Its first responsibility is to provide a faithful baseline against which our historical hypotheses can be tested. Where historical behaviour cannot be established, assumptions must be explicit and traceable to their evidence status.

Once that baseline exists, the simulator can also support deliberately separate experimental variants. These may include mechanisms described by later patents, alternative historical developments, and our own extensions. Such variants must never silently alter the faithful System 250 model.

The intended progression is:

1. evidence-backed architectural model;
2. reference emulator, simulator and assembler;
3. test suite comparing documented and reconstructed behaviour;
4. increasingly complete reconstruction of the original multiprocessor System 250 environment;
5. separate experimental branches/models implementing later patents and clearly labelled new extensions.

The objective is therefore broader than simply making PP250 instructions execute. The simulator should become an experimental laboratory in which the reconstructed architecture can be observed, challenged and extended while the historical baseline remains intact.

**Modern FPGA capability machine — a separate project**

A modern FPGA machine is a second and distinct undertaking. It is **not the faithful System 250 reconstruction in hardware**.

Its purpose is to take the architectural ideas that make System 250 interesting—hardware-protected capabilities, non-forgeable authority, protected execution domains, controlled derivation of authority, and protection without an ordinary privileged escape path—and ask what a small real capability computer built with contemporary technology should look like.

This machine may therefore use:

* a modern FPGA implementation;
* contemporary RAM;
* a standard modern system interconnect;
* modern high-speed storage and peripherals;
* conventional modern DMA mechanisms where appropriate;
* contemporary debugging and development interfaces;
* and architectural simplifications or extensions justified by the new design.

It need not reproduce the System 250 word width, electrical bus, memory modules, physical packaging, peripheral interfaces or other implementation choices dictated by 1970s technology.

The historical reconstruction remains valuable to this project because it gives us a rigorously understood capability architecture from which to draw ideas. But the FPGA machine must be described as a **new System-250-inspired capability computer**, not as evidence about how the original machine worked.

This separation is deliberate:

```
surviving evidence
       |
       v
System 250 reconstruction
research bench
       |
       +---- faithful historical baseline
       |
       +---- later patents / experimental derivatives
       |
       +---- architectural lessons
                    |
                    v
          modern FPGA capability machine
          (separate new architecture)
```

**Inter-computer capabilities — separate unsolved research**

A third line of work concerns capability authority between independently protected computers.

This is currently a **thinking and research problem**, not an implementation objective of the System 250 reconstruction and not a settled feature of the FPGA machine.

The central difficulty is that a capability protected inside one machine cannot simply be transmitted over an ordinary communication channel without becoming data. Data can be copied, modified and manufactured. The unresolved architectural question is therefore not merely how to serialise a capability, but what legitimate authority permits a receiving machine to turn received information into local capability authority.

Ideas explored in this area must be treated as research hypotheses rather than established architecture. In particular, the project should not imply that the inter-computer capability problem has been solved simply because representations can be authenticated, encrypted or transported securely.

Until a satisfactory authority model is established, this work remains conceptually separate from both the faithful System 250 research bench and the modern FPGA capability machine.

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
