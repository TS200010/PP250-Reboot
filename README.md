# PP250 Reboot

**Reconstructing the Plessey System 250...**

The Plessey System 250 (incorporating the PP250 processor) was an early commercial capability-based computer developed in the United Kingdom by Plessey. It was a groundbreaking computer architecture and which has perhaps more relevance today than in the 1970's.

**PP250 Reboot** is an attempt to reconstruct the System 250 from the surviving technical documentation, software, patents, publications, hardware information, and the recollections of people who worked on the project.

The immediate objective is to gather, preserve, digitise and cross-reference as much reliable information about the machine as possible. Once that body of evidence has been assembled, we can reconstruct the architecture and determine what can be established about the machine's behaviour.

That reconstruction can then become executable as a reference emulator and simulator: a research bench for testing whether our reconstruction is coherent, for recreating as much of the original System 250 as the evidence permits, and for exploring what its architectural ideas might mean for modern capability-native hardware and software.

The project is defined this way:

1. **System 250 Architectural Reconstruction.** Reconstruct the original hardware and behaviour as faithfully as the surviving evidence permits, explicitly recording what is documented, inferred, or still unknown. The historical reconstruction remains the foundation for everything that follows.

2. **Executable reconstruction and emulator workbench.** Once the architecture is sufficiently reconstructed, it becomes executable as a reference emulator/simulator. The workbench provides a way to test the coherence of the reconstruction, recreate as much of the original System 250 as the evidence permits, and experiment with historically distinct derivatives and new ideas without contaminating the faithful baseline.

3. **Architectural and capability-native research.** Use the reconstructed machine as a basis for investigating what its architectural ideas mean beyond historical emulation. This includes the natural relationship between capabilities, protected objects and software abstractions, and how contemporary languages and compiler infrastructure might map onto capability-native hardware rather than merely being protected by it.

   One research direction has emerged during the reconstruction itself: **M⟨H,T⟩**. The surviving System 250 material identifies two aspects of the architecture as the **Church machine** and the **Turing machine**. In this project we use **H** for the Church machine, since **C** is already used for capability registers, and **T** for the Turing machine.

   The surviving architectural description appears to include within the Church machine two conceptually different things: the capability machinery itself, and machinery capable of acting on the state of the Church and Turing machines together. **Change Process (CHP)** provides a concrete example: through the Dump Stack it can replace a complete process state containing both Turing-machine state and Church-machine capability state.

   We provisionally call this higher-level concept **M**. M is not a third machine parallel to H and T. It may be a meta-machine which manipulates H and T, or perhaps the underlying machinery which implements them. The **M⟨H,T⟩** model is therefore a developing theory arising from the reconstruction, to be tested against it rather than imposed upon it.

4. **Modern FPGA Capability Machine.** A new real computer informed by what survives the reconstruction and subsequent research. It is not an attempt to reproduce System 250 hardware. It may use contemporary FPGA technology, RAM, buses, peripherals, DMA, storage and networking, and may eventually lead toward an ASIC if the architecture proves interesting.

5. **Inter-computer capability research.** For now this is deliberately thinking and documentation only. The problem is whether capability authority can meaningfully extend between independently protected machines without reducing a capability to forgeable/copyable data or merely replacing the capability model with cryptographic credentials. We have interesting ideas, particularly around the distinction between representation and authority to reconstruct, but we have not solved it.

The broad direction is therefore from **historical evidence**, through **architectural reconstruction** and an **executable workbench**, into research on the computational and capability-native ideas exposed by the reconstructed machine, and ultimately into experiments with new hardware. M⟨H,T⟩ is an important emerging line of enquiry within that programme, not a prerequisite that the rest of the project depends upon.

**Relationship to other capability work.** PP250 Reboot is not an attempt to rediscover the correspondence between capabilities and protected objects. Ken Hamer-Hodges' **CLOOMC** work is particularly relevant here, and **CHERI** and **CHERIoT** provide important modern capability architectures and comparison points. Our starting point is the reconstruction of System 250 itself: recovering the architecture from the surviving evidence, making that reconstruction executable, and using it to investigate which of its ideas remain useful in a modern machine and software environment. This includes questions such as how contemporary language and object abstractions might map naturally onto hardware capabilities rather than merely being protected by them. During that reconstruction the M⟨H,T⟩ line of enquiry has emerged; it may reveal an aspect of the original architecture that is obscured when System 250 is described simply as a capability machine. CLOOMC, CHERI and CHERIoT are therefore not work to be reinvented, but related work against which both the reconstruction and the new questions it raises can be compared.

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

Everything needed to cross that boundary belongs to the reconstruction. The project must account for the provenance of the protected state required by the first legitimate process rather than hiding the bootstrap problem behind an assumed operating system. It must then establish that the resulting process has sufficient legitimate authority for ordinary PP250 mechanisms to construct a continuing capability-managed system.

The detailed reconstruction boundary, proof-of-sufficiency criterion, emulator contract and later-project boundaries are preserved in `research/reconstruction-sufficiency-and-workbench-boundaries.md`. Detailed startup reconstruction is in `research/pp250-boot-and-processor-startup.md`; capability genesis and resource lifecycle work is in `research/capability-genesis-and-resource-lifecycle.md`.

The emulator/workbench is the executable test of the reconstruction and a platform for subsequent research, including the developing M⟨H,T⟩ theory and capability-native software experiments. It must preserve a faithful historical baseline while keeping later patents and new experiments explicitly separate. The FPGA work follows from what survives that process and is a new architecture, not evidence about the historical machine. Inter-computer capability authority remains a separate unsolved research problem.

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