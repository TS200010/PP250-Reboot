# PP250 Reboot

**Reconstructing the Plessey System 250...**

The Plessey System 250 was an early commercial capability-based computer developed in the United Kingdom by Plessey. It was a groundbreaking computer architecture and which has perhaps more relevance today than in the 1970's.

**PP250 Reboot** is an attempt to reconstruct the System 250 from the surviving technical documentation, software, patents, publications, hardware information, and the recollections of people who worked on the project.

The immediate objective is not to write an emulator ... but perhaps we might one day.

The first objective is to gather, preserve, digitise and cross-reference as much reliable information about the machine as possible. Once that body of evidence has been assembled, we can reconstruct the architecture and determine what can be established about the machine's behaviour. An emulator could perhaps then emerge from that reconstruction.

**Why?**

PP250 occupies a distinctive place in the history of (capability-based) computing. It was not merely a theoretical architecture: it became an operational commercial system, and those who worked closely with it understood that its err... capabilities went well beyond conventional computer architectures of its time.

Yet PP250 has received surprisingly little recognition for the power of the ideas embodied in its design. There is an important distinction between understanding PP250 as a collection of hardware and software features and understanding the computational model that lay behind it. The former can be learned from documentation; the latter requires seeing what the architecture makes possible.

Much of the history of computing is described through the lens of the conventional von Neumann model, in which computation operates on data and addresses in a shared memory. PP250 approached computing from a different foundation: the ideas associated with capability systems and, ultimately, with the more general computational principles explored by Church and others. That difference is easy to miss if PP250 is viewed simply as an unusual computer of its period.

For the people who worked deeply with the system, the implications could be apparent in the architecture itself. For those who encountered PP250 only through descriptions of it, much of that significance was easier to overlook. This project is an attempt to recover not only what PP250 did, but to understand the architectural ideas that made it possible—and to make those ideas accessible and relavent again.

**Materials and Purpose**

Much of the original technical material is now difficult to find, and some aspects of the architecture are incompletely documented in the surviving sources.

This project therefore has two purposes:

* Preservation — collect and make accessible surviving PP250 technical and historical material.
* Reconstruction — determine, as accurately as possible, how the system actually worked.

A working emulator would be a valuable consequence of that work.

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

**Emulator**

An emulator is a later objective.

It should implement the architecture that the evidence supports, rather than an architecture invented to make an emulator convenient to write.

Where the historical behaviour cannot be established, the emulator should make those assumptions explicit.

**Sources**

The repository will contain both machine-readable transcriptions and references to the original material.

Original documents should be preserved separately from derived material. Transcriptions and interpretations must never replace the original evidence.

**Contributions**

Information from former PP250 engineers, programmers, users and maintainers is particularly valuable.

If you have PP250 documentation, software, photographs, technical notes, personal recollections or information about people involved in the project, please get in touch or open an issue.

**PP250 Reboot** is an historical and technical reconstruction project.

The goal is simple:

**Find out what PP250 really was, preserve what can still be recovered, and make it possible to run again.**

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
- `emulator/` — reserved for future emulator work; no emulator development is planned here yet.

## Working principles

- Preserve provenance: identify where material came from and its known status.
- Separate source images or copies from transcriptions and interpretation.
- Do not present unverified claims as historical fact.
- Prefer small, well-described additions over speculative reconstruction.

## Contributing

Please describe the source, date of access or acquisition where known, any transformations made, and remaining uncertainties when adding archival or research material.

## About me...

As you might have guessed I worked on the PP250 in Taplow Berkshire. A programmer on the CORAL 250 Compiler team and later its project leader.
