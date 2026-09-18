# System 250 Patents

This directory contains original patent documents used as primary sources for the reconstruction of the Plessey System 250.

## Repository convention

Keep the original patent PDF in this directory.

Machine-readable transcriptions or text extractions belong in `transcriptions/`.

Architectural conclusions derived from patents belong in `architecture/`.

The intended evidence flow is:

`patents/ → transcriptions/ → architecture/`

Do not silently correct, rewrite, or reinterpret the original source material.

## Patent register

### US 4,408,274 — Memory protection system using capability registers

**Inventors:** Martyn P. Andrews; Nigel J. Wheatley  
**Assignee:** Plessey Overseas Limited

**PP250 relevance:** Capability representation and protection, capability classes/SCT mechanisms, propagation control, and hardware mechanisms associated with access-right reduction.

**Preferred repository filename:**  
`US4408274-memory-protection-capability-registers.pdf`

**Status:** PDF not yet added to this directory.

---

### US 4,486,831 — Multi-programming data processing system process suspension

**Inventors:** Martyn P. Andrews; Nigel J. Wheatley  
**Assignee:** Plessey Overseas Limited

**PP250 relevance:** Process suspension and restoration, process dump-stack state, and saved execution/procedure state.

**Preferred repository filename:**  
`US4486831-process-suspension.pdf`

**Status:** PDF not yet added to this directory.

---

### US 4,383,297 — Data processing system including internal register addressing arrangements

**Inventors:** Martyn P. Andrews; Nigel J. Wheatley  
**Assignee:** Plessey Overseas Limited

**PP250 relevance:** Internal register addressing and processor/register mechanisms relevant to the System 250 architecture.

**Preferred repository filename:**  
`US4383297-internal-register-addressing.pdf`

**Status:** PDF not yet added to this directory.

## Adding patent material

When a patent PDF is added:

1. Preserve the downloaded patent as an original primary-source artifact.
2. Record its provenance and source URL in this index.
3. Put any OCR or transcription in `transcriptions/`, not in this directory.
4. Check the transcription against the source where layout, diagrams, equations, bit fields, or ambiguous characters matter.
5. Update `architecture/` only with conclusions justified by the evidence.
6. Raise an issue rather than silently reconciling evidence that conflicts with the current architectural reconstruction.
