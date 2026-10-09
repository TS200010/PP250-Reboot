# System 250 Patents

This directory contains original patent documents used as primary sources for the reconstruction of the Plessey System 250.

## Repository convention

Keep the original patent PDF in this directory.

The current collection also contains RTF text extractions beside the US patent PDFs. Earlier transcriptions and figure captures remain in `transcriptions/`. These are RTF files, not plain TXT files. The intended home for new machine-readable transcriptions remains `transcriptions/`; no existing files have been moved.

Architectural conclusions derived from patents belong in `architecture/`.

The intended evidence flow is:

`patents/ → transcriptions/ → architecture/`

Do not silently correct, rewrite, or reinterpret the original source material.

## Current holdings — 9 October 2026

There are **15 patent PDFs and 14 RTF text extractions** in this directory: 14 US publications with PDF/RTF pairs, plus one GB family publication. See [PATENT-AUDIT.md](PATENT-AUDIT.md) for provenance, family relationships and architectural classification; its older holdings tables are dated historical records.

| US patent | Subject / scope | PDF | RTF |
|---|---|---|---|
| 3657736 | Subroutine assembly; architectural ancestry | [PDF](US-3657736-A.pdf) | [RTF](US-3657736-A.rtf) |
| 3680053 | Data transmission; designer lead, not established PP250 architecture | [PDF](US-3680053-A.pdf) | [RTF](US-3680053-A.rtf) |
| 3757307 | Program interrupt facilities | [PDF](US-3757307-A.pdf) | [RTF](US-3757307-A.rtf) |
| 3771146 | Capability-based interrupt arrangements | [PDF](US-3771146-A.pdf) | [RTF](US-3771146-A.rtf) |
| 3787813 | Capability registers | [PDF](US-3787813-A.pdf) | [RTF](US-3787813-A.rtf) |
| 3787818 | Multiprocessor system | [PDF](US-3787818-A.pdf) | [RTF](US-3787818-A.rtf) |
| 3814919 | Fault detection and isolation | [PDF](US-3814919-A.pdf) | [RTF](US-3814919-A.rtf) |
| 3879712 | Fault diagnostic arrangements | [PDF](US-3879712-A.pdf) | [RTF](US-3879712-A.rtf) |
| 4041460 | Peripheral equipment access units | [PDF](US-4041460-A.pdf) | [RTF](US-4041460-A.rtf) |
| 4050059 | Read-and-hold facility | [PDF](US-4050059-A.pdf) | [RTF](US-4050059-A.rtf) |
| 4121286 | Memory allocation and deallocation | [PDF](US-4121286-A.pdf) | [RTF](US-4121286-A.rtf) |
| 4383297 | Internal register addressing; later development | [PDF](US-4383297-A.pdf) | [RTF](US-4383297-A.rtf) |
| 4408274 | Memory protection; later development | [PDF](US-4408274-A.pdf) | [RTF](US-4408274-A.rtf) |
| 4486831 | Process suspension; later development | [PDF](US-4486831-A.pdf) | [RTF](US-4486831-A.rtf) |

[GB_1410631_A.pdf](GB_1410631_A.pdf) is a family counterpart of US3771146; there is no separate GB text transcription.

### Current text status

- `US-3787813-A.rtf` now contains the correct capability-register patent.
- `US-3771146-A.rtf` and `US-3787818-A.rtf` include the missing technical-description passages restored from their older transcriptions, with editorial provenance notes.
- `US-4121286-A.rtf` contains the allocation/deallocation text. The incorrectly labeled older `transcriptions/US4121286A.rtf` has been removed.
- `US-4486831-A.pdf` has been restored.
- **Outstanding technical-description repair:** `US-3657736-A.rtf` still stops during Step 7/10, before completing the transfer sequence.
- Figure transcription and verification are deferred to a separate pass. Claims-extraction errors are outside the current repair scope.

The text extractions retain OCR errors and, in some cases, webpage material. They have not been certified by a complete comparison against the scans; consult the PDFs for ambiguous wording, diagrams and bit fields.

US3999052 (IBM) and US4133029 (Siemens) have been removed from the active collection because no PP250 connection has been established. Their correct attribution and recovery reference are recorded in [the audit, Section 4](PATENT-AUDIT.md#4-correction-ibm-and-siemens-families-excluded--9-october-2026).

## Selected later-patent details

### US 4,408,274 — Memory protection system using capability registers

**Inventors:** Martyn P. Andrews; Nigel J. Wheatley  
**Assignee:** Plessey Overseas Limited

**PP250 relevance:** Capability representation and protection, capability classes/SCT mechanisms, propagation control, and hardware mechanisms associated with access-right reduction.

**Repository PDF:** [US-4408274-A.pdf](US-4408274-A.pdf)

**Status:** PDF and corresponding RTF text extraction are present.

---

### US 4,486,831 — Multi-programming data processing system process suspension

**Inventors:** Martyn P. Andrews; Nigel J. Wheatley  
**Assignee:** Plessey Overseas Limited

**PP250 relevance:** Process suspension and restoration, process dump-stack state, and saved execution/procedure state.

**Repository PDF:** [US-4486831-A.pdf](US-4486831-A.pdf)

**Status:** PDF and corresponding RTF text extraction are present.

---

### US 4,383,297 — Data processing system including internal register addressing arrangements

**Inventors:** Martyn P. Andrews; Nigel J. Wheatley  
**Assignee:** Plessey Overseas Limited

**PP250 relevance:** Internal register addressing and processor/register mechanisms relevant to the System 250 architecture.

**Repository PDF:** [US-4383297-A.pdf](US-4383297-A.pdf)

**Status:** PDF and corresponding RTF text extraction are present.

## Adding patent material

When a patent PDF is added:

1. Preserve the downloaded patent as an original primary-source artifact.
2. Record its provenance and source URL in this index.
3. Put any OCR or transcription in `transcriptions/`, not in this directory.
4. Check the transcription against the source where layout, diagrams, equations, bit fields, or ambiguous characters matter.
5. Update `architecture/` only with conclusions justified by the evidence.
6. Raise an issue rather than silently reconciling evidence that conflicts with the current architectural reconstruction.
