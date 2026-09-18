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
