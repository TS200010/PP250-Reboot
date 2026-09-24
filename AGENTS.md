# PP250-Reboot Agent Instructions

## Purpose

This repository reconstructs and documents the architecture of the Plessey System 250 and provides experimental implementations of that architecture.

Historical accuracy takes precedence over filling gaps with plausible assumptions.

## Historical reconstruction method

The surviving PP250 documentation is fragmentary. No single surviving document should be assumed to contain a complete or definitive description of the machine. Different papers, patents, manuals, diagrams, operating-system descriptions and implementation accounts often describe different parts of the same underlying architecture from different viewpoints.

Research in this repository therefore uses **architectural reconstruction**, not merely documentary lookup. Treat the surviving sources as multiple partial observations of one underlying machine. A proposition does **not** require an explicit statement in a primary source before it can become a working architectural conclusion.

For significant reconstruction questions use the reasoning chain:

**Observation -> Constraint -> Reconstruction -> Prediction -> Falsification**

- **Observation** records what individual sources say, show diagrammatically, or necessarily demonstrate.
- **Constraint** identifies what must be true for multiple credible observations to coexist.
- **Reconstruction** proposes the simplest coherent architecture satisfying all known constraints.
- **Prediction** identifies consequences that should follow if the reconstruction is correct.
- **Falsification** actively looks for observations inconsistent with the reconstruction.

A reconstruction becomes stronger when it independently explains observations that were not used to construct it.

Do **not** repeatedly demand a source explicitly stating a reconstructed architectural conclusion. In many cases no such source exists; producing the missing whole from partial descriptions is a principal objective of PP250-Reboot. Do not respond to every architectural inference with a recommendation to search for documentary confirmation. First determine whether documentary confirmation should reasonably be expected to exist.

Distinguish clearly between:

- **DOCUMENTED OBSERVATION** — directly supported by a source;
- **NECESSARY INFERENCE** — required to reconcile documented observations;
- **WORKING RECONSTRUCTION** — the current simplest model explaining the evidence;
- **SPECULATION** — possible but not presently constrained by sufficient evidence;
- **UNKNOWN** — a question for which the available observations do not yet constrain an answer adequately.

Search the documentation when it can provide a **new observation, discriminate between competing reconstructions, test a prediction, or potentially falsify the current model**. Do not search merely to find explicit wording for a conclusion already reached by synthesis unless such explicit confirmation would materially change its evidential status.

When sources appear inconsistent, do not immediately choose one and discard the others. First consider differences of machine version, operating system, abstraction level, terminology, viewpoint, or implementation versus architecture. The objective is to find the underlying machine that explains all credible observations.

The principal test of a reconstruction is not simply **“where does a document say this?”** but:

> **Does this model explain the observations, contradict none of the reliable evidence, and introduce the fewest unsupported mechanisms?**

Documentary provenance remains essential: observations must remain traceable to their sources, and reconstructions must remain distinguishable from observations. The purpose of this method is not to weaken the evidence standard, but to permit the architecture to be reconstructed from evidence that was never assembled into a single definitive description by the original authors.

## Primary reconstruction objective

Treat the root `README.md` as the authoritative statement of project scope and direction. **Read it before doing architectural reconstruction, bootstrap research, emulator design, or hardware design**, and re-read the section **"Primary reconstruction: from inert hardware to the first real program"** whenever work touches cold start, faults, CHP, SCTs, dump stacks, capability genesis, C6/C7, processor initialisation, resource allocation, or initial machine state.

The immediate technical objective is to reconstruct the PP250 from an inert machine up to the first legitimate execution of ordinary PP250 software. Do not silently turn this into a project to recreate ROS/POS or the entire historical operating system.

Use the boundary defined in the README to guide research:

- **Below the boundary:** reconstruct the hardware, microcode-visible mechanisms, and initial state required before ordinary software can run.
- **At the boundary:** determine the complete relevant state of the first legitimate process and the provenance of every protected object/capability needed to establish it.
- **Above the boundary:** investigate only far enough to prove that the reconstructed initial state is sufficient for ordinary PP250 software to construct a self-sustaining capability system.

For every proposed bootstrap mechanism ask:

1. Where did the authority come from?
2. Which documented data structure holds it?
3. How was that structure created from the preceding machine state?
4. Does the proposal require capability fabrication that has not been explained?
5. Does it introduce an undocumented privileged/supervisor mechanism?
6. Can normal PP250 mechanisms take over once the first process is running?

The target is not merely to make an emulator boot by convenient assumptions. The target is an evidence-backed chain from power-on/inert state to the first real program.

Where documentation stops before that chain is complete, label the missing link explicitly as inference, hypothesis or unknown and investigate the data structures that constrain the possible algorithm. Prefer **data structures -> necessary algorithms** over inventing plausible operating-system behaviour.

The current completion criterion is the one stated in the README: hardware/microcode must be able to reach a valid first process with sufficient legitimate capability authority to construct subsequent software-managed resources without undocumented privilege or arbitrary capability fabrication.

## Evidence policy

Distinguish clearly between:

- PRIMARY EVIDENCE
- SECONDARY EVIDENCE
- INFERENCE
- HYPOTHESIS
- UNKNOWN

Never promote an inference or hypothesis to established architecture without supporting evidence. A working reconstruction may nevertheless combine multiple documented observations and necessary inferences when no source provides a complete architectural description; label that status explicitly in accordance with the Historical reconstruction method above.

## New evidence

When new evidence conflicts with the architecture WIP:

- do not silently alter the architecture;
- identify the conflicting statements;
- identify the evidence and provenance;
- determine whether the conflict can be explained;
- if unresolved, create or propose a research issue.

## Patentable ideas and public disclosure

This is a public repository. Before committing, opening an issue, pull request, discussion, or otherwise adding material that describes a new technical mechanism, implementation, architecture, instruction, capability mechanism, security mechanism, hardware design, or other potentially novel invention:

- consider whether the material could contain patentable subject matter;
- distinguish historical PP250 reconstruction and documented prior art from genuinely new design work;
- if the material appears potentially novel and technically useful, STOP before making it public;
- flag the idea to the repository owner and explain briefly why it may warrant patent review;
- do not disclose implementation details publicly until the repository owner explicitly confirms that publication is acceptable;
- when uncertain, treat the material as potentially patentable and ask before publishing.

In particular, apply this check to modern extensions of PP250 concepts, including capability-based processor design, capability-protected memory-mapped I/O, compartment/domain switching, interrupt handling, DMA protection, boot/root-capability construction, compact capability representation, FPGA/ASIC implementations, and security mechanisms.

This is a publication safeguard, not a determination that an idea is legally patentable. Patentability requires separate prior-art and legal review.

## Repository access

Do not process image files unless explicitly instructed by the user or permitted by a more specific AGENTS.md.

Do not perform broad repository scans when a narrower operation will answer the question.

Prefer changed files and Git diffs when reviewing recent work.

## Architecture

`architecture/` represents the current working reconstruction of the PP250 architecture.

It is not itself primary evidence.

Statements in architecture documents should be traceable to evidence or explicitly identified as inference.

## Transcriptions

Files under `transcriptions/` are transcriptions of source material.

Do not silently correct technical content in a transcription. Suspected transcription errors should be identified separately.

## Changes

Make small, logically coherent commits.

Commit messages should describe the architectural, documentary, or implementation change.

Do not modify unrelated files.
