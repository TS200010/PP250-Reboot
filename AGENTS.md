# PP250-Reboot Agent Instructions

## Purpose

This repository reconstructs and documents the architecture of the Plessey System 250 and provides experimental implementations of that architecture.

Historical accuracy takes precedence over filling gaps with plausible assumptions.

## Evidence policy

Distinguish clearly between:

- PRIMARY EVIDENCE
- SECONDARY EVIDENCE
- INFERENCE
- HYPOTHESIS
- UNKNOWN

Never promote an inference or hypothesis to established architecture without supporting evidence.

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

`docs/architecture/` represents the current working reconstruction of the PP250 architecture.

It is not itself primary evidence.

Statements in architecture documents should be traceable to evidence or explicitly identified as inference.

## Transcriptions

Files under `transcriptions/` are transcriptions of source material.

Do not silently correct technical content in a transcription. Suspected transcription errors should be identified separately.

## Changes

Make small, logically coherent commits.

Commit messages should describe the architectural, documentary, or implementation change.

Do not modify unrelated files.
