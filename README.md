# PP250 Reboot

**Reconstructing the Plessey System 250...**

The Plessey System 250 (incorporating the PP250 processor) was an early commercial capability-based computer developed in the United Kingdom by Plessey. It was a groundbreaking computer architecture which perhaps has more relevance today than it did in the 1970s.

**PP250 Reboot is, first of all, a personal project.** I worked on the System 250 in the 1970s and, fifty years later, I would like to find out how much of the machine can still be recovered, understood and ultimately made to run again. Much of the documentation has disappeared, memories are imperfect, and there are some wonderfully awkward gaps in what survives. Working out what must have been there is a large part of the fun.

The repository is public because this is much more likely to succeed with help. If you worked on System 250, have documentation or software tucked away somewhere, recognise something we have misunderstood, or simply find the architecture interesting and want to contribute, that would be enormously welcome.

The work is intended to be careful even though the motivation is personal. Surviving technical documentation, software, patents, publications, hardware information and recollections are being gathered, preserved and cross-referenced so that reconstruction can be distinguished from guesswork.

The reconstruction can then become executable as a reference emulator and simulator: a workbench for testing whether it is coherent, for recreating as much of the original System 250 as the evidence permits, and for exploring what its architectural ideas might mean for modern capability-native hardware and software.

Along the way the project has already begun to raise some deeper questions about capability machines, programming models and the computational structure of System 250. Those are worth pursuing seriously, and useful research may come out of them. That would be a welcome consequence of the project rather than its justification. The primary aim remains to reconstruct this remarkable old machine and see where that takes us.

This is not being done in ignorance of later capability work. In particular, Ken Hamer-Hodges' **CLOOMC** work explores ideas closely related to the natural mapping between capabilities, protected objects and software structure. That is important context for this project: the aim is not to reinvent that mapping, but to reconstruct the System 250 beneath it, understand what the original architecture actually provided, and see what follows from that understanding.

This is roughly where the project is heading:

1. **System 250 Architectural Reconstruction.** Reconstruct the original hardware and behaviour as faithfully as the surviving evidence permits, explicitly recording what is documented, inferred, or still unknown. The historical reconstruction remains the foundation for everything that follows.

2. **Executable reconstruction and emulator workbench.** Once the architecture is sufficiently reconstructed, make it executable as a reference emulator/simulator. The workbench gives us a way to test the coherence of the reconstruction, recreate as much of the original System 250 as the evidence permits, and experiment with historically distinct derivatives and new ideas without contaminating the faithful baseline.

3. **Architectural and capability-native exploration.** Use the reconstructed machine to explore what its architectural ideas might mean beyond historical emulation. This includes the natural relationship between capabilities, protected objects and software abstractions, and how contemporary languages and compiler infrastructure might map onto capability-native hardware rather than merely being protected by it.

   One particularly interesting line of enquiry has emerged during the reconstruction itself: **M⟨H,T⟩**. The surviving System 250 material identifies two aspects of the architecture as the **Church machine** and the **Turing machine**. In this project we use **H** for the Church machine, since **C** is already used for capability registers, and **T** for the Turing machine.

   The surviving architectural description appears to include within the Church machine two conceptually different things: the capability machinery itself, and machinery capable of acting on the state of the Church and Turing machines together. **Change Process (CHP)** provides a concrete example: through the Dump Stack it can replace a complete process state containing both Turing-machine state and Church-machine capability state.

   We provisionally call this higher-level concept **M**. M is not a third machine parallel to H and T. It may be a meta-machine which manipulates H and T, or perhaps the underlying machinery which implements them. The **M⟨H,T⟩** model is therefore a developing idea arising from the reconstruction, to be tested against it rather than imposed upon it.

4. **Modern FPGA Capability Machine.** If the preceding work gets far enough, it would be fascinating to build a real computer informed by what survives the reconstruction and subsequent experiments. This would not be an attempt to reproduce System 250 hardware. It could use contemporary FPGA technology, RAM, buses, peripherals, DMA, storage and networking, and might eventually point toward an ASIC if the architecture proved interesting enough.

5. **Inter-computer capability exploration.** Another question worth thinking about is whether capability authority can meaningfully extend between independently protected machines without reducing a capability to forgeable/copyable data or merely replacing the capability model with cryptographic credentials. There are some interesting ideas here, particularly around the distinction between representation and authority to reconstruct, but this remains an unsolved question rather than an implementation objective.

So the broad direction is from **historical evidence**, through **architectural reconstruction** and an **executable workbench**, into whatever interesting computational and capability-native questions the reconstructed machine exposes, and perhaps ultimately into experiments with new hardware. M⟨H,T⟩ is one important emerging line of enquiry within that journey, not a prerequisite that the rest of the project depends upon.

**Relationship to other capability work.** PP250 Reboot is not an attempt to rediscover the correspondence between capabilities and protected objects. Ken Hamer-Hodges' **CLOOMC** work is particularly relevant here, and **CHERI** and **CHERIoT** provide important modern capability architectures and comparison points. Our starting point is the reconstruction of System 250 itself: recovering the architecture from the surviving evidence, making that reconstruction executable, and then seeing what can be learned from it. This includes questions such as how contemporary language and object abstractions might map naturally onto hardware capabilities rather than merely being protected by them. CLOOMC, CHERI and CHERIoT are not work to be reinvented, but related work from which this project can learn and against which its experiments and observations can be compared.

**Why?**

Because System 250 was an extraordinary machine, I spent part of my early career programming it, and I would like to understand the rest of it.

It was not merely a theoretical architecture: it became an operational commercial system. Yet much of its technical history is now difficult to recover, and there is an important difference between knowing a list of PP250 features and understanding how the machine fitted together and what those features made possible.

Much of computing developed around the conventional von Neumann model, while System 250 approached protection, authority and computation in a rather different way. That makes it interesting both historically and technically. If reconstructing it also turns up ideas that remain useful today, so much the better.

### On a more serious note…

I have also become increasingly convinced that something important in the System 250 architecture has been overlooked. PP250 is usually remembered, when it is remembered at all, as an early capability machine. That is true, but I suspect it is an incomplete description which has encouraged later discussion to concentrate on capabilities primarily as a mechanism for protection and controlled access to memory.

To those of us who programmed the machine, capabilities were also part of a remarkably natural way of structuring software. Protected objects, the authority to use them, and the interfaces through which they were entered were reflected directly in the machine architecture rather than being abstractions constructed entirely in software above it. Ken Hamer-Hodges' later **CLOOMC** work is important evidence that I am not alone in seeing significance in that relationship.

Much subsequent academic work on capability machines has understandably concentrated on security and protection. Those are important properties, but I wonder whether classifying System 250 simply as an early member of that lineage has caused another part of its architecture to receive much less attention than it deserves.

I don't yet know whether that suspicion will survive a complete reconstruction. That is one reason for doing the reconstruction carefully. Rather than starting by claiming that System 250 contained a forgotten answer, I would like to rebuild enough of it to find out exactly what was there, how the hardware and software fitted together, and whether there really is an architectural idea worth recovering.

**Materials and Purpose**

Much of the original technical material is now lost, and some aspects of the architecture are incompletely documented in the surviving sources.

There are therefore three practical strands to the project:

* Preservation — collect and make accessible surviving PP250 technical and historical material.
* Reconstruction — work out, as accurately as possible, how the system actually worked.
* Experimentation — make the reconstructed architecture executable and see what we can learn from it.

A working emulator and any eventual hardware implementation are consequences of that work, not substitutes for establishing the evidence.

**Evidence before assumptions**

A central principle of the project is to distinguish between:

* **Documented** — explicitly stated in a surviving source.
* **Corroborated** — supported independently by more than one source.
* **Inferred** — a technically strong conclusion derived from available evidence.
* **Uncertain** — an interpretation for which the evidence is incomplete.
* **Speculative** — a possibility requiring further evidence.
* **Gaps** — a known gap in the record requiring further research.

The project should not just silently fill gaps in the historical record. Where two sources disagree, the disagreement itself is worth recording.

**Primary reconstruction: from inert hardware to the first real program**

The first substantial technical milestone I have set myself is not to recreate ROS, POS, or the complete historical System 250 software environments. It is to reconstruct the **PP250 hardware architecture and its initial state**, starting with an inert machine and continuing until the point at which the first legitimate ordinary PP250 program can execute.

That gives the historical investigation a wonderfully concrete engineering question:

> **Starting from an inert machine, what hardware state and microcode mechanisms are required to reach the first legitimate execution of ordinary PP250 software?**

Everything needed to cross that boundary belongs to the reconstruction. We need to account for the provenance of the protected state required by the first legitimate process rather than hiding the bootstrap problem behind an assumed operating system. We then need to establish that the resulting process has sufficient legitimate authority for ordinary PP250 mechanisms to construct a continuing capability-managed system.

The detailed reconstruction boundary, proof-of-sufficiency criterion, emulator contract and later-project boundaries are preserved in `research/reconstruction-sufficiency-and-workbench-boundaries.md`. Detailed startup reconstruction is in `research/pp250-boot-and-processor-startup.md`; capability genesis and resource lifecycle work is in `research/capability-genesis-and-resource-lifecycle.md`.

The emulator/workbench will be the executable test of the reconstruction and a playground for subsequent experiments, including the developing M⟨H,T⟩ idea and capability-native software work. It must preserve a faithful historical baseline while keeping later patents and new experiments explicitly separate. Any FPGA work would follow from what survives that process and would be a new architecture, not evidence about the historical machine. Inter-computer capability authority remains a separate unsolved problem to think about.

**Sources**

The repository will contain both machine-readable transcriptions and references to the original material.

Original documents should be preserved separately from derived material. Transcriptions and interpretations must never replace the original evidence.

**Contributions**

Information from former PP250 engineers, programmers, users and maintainers would be particularly valuable.

If you have PP250 documentation, software, photographs, technical notes, personal recollections, information about people involved in the project, or simply spot something here that looks wrong, please get in touch or open an issue. Help reconstructing the machine would be very welcome.

**PP250 Reboot** is an historical and technical reconstruction project, but above all it is something I am doing because I want to see whether this machine can be understood and brought back to life.

The goal is simple:

**Find out what PP250 really was, preserve what can still be recovered, make it possible to run again, and see where its ideas lead.**

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

I worked on the PP250 at Plessey in Taplow, Berkshire, first as a programmer on the CORAL 250 compiler team and later as its project leader. Fifty years later, I'd quite like to understand the rest of the machine.