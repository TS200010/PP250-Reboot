# PP250-Reboot Agent Instructions

## Editing safety and scope

- Review, inspect and analyse requests are read-only unless editing is explicitly authorised. Proposed changes or moves are not permission to implement them.
- Preserve source content when reorganising. Never summarise, condense, paraphrase, delete, opportunistically clean up or alter unrelated material unless explicitly requested. For narrow edits, change only the intended text; do not regenerate unaffected portions for convenience.
- Read the complete current file before replacing it. GitHub updates replace whole files: truncated, clipped or partial responses are not valid replacement inputs. Retrieve missing content through explicit line ranges or another lossless method, and verify the original end of file is present before writing.
- After writing, inspect the diff or compare complete before/after versions. Unexpected deletions, lost tail content or unrelated rewrites mean the update failed: stop further edits. API success proves acceptance, not preservation.
- Make small, logically coherent commits whose messages describe the change. Modify no unrelated files.
- Prefer narrow reads, changed files and Git diffs over broad scans. Do not process images unless explicitly instructed or permitted by a more specific AGENTS.md.

## Purpose and dependency order

Reconstruct and document the Plessey System 250 architecture, with experimental implementations. Historical accuracy takes precedence over plausible gap-filling. The root `README.md` is the authoritative concise scope statement; read it before reconstruction, M⟨H,T⟩, emulator or hardware work.

**Historical evidence -> architectural reconstruction -> develop M⟨H,T⟩ -> test it against the reconstruction -> emulator workbench -> FPGA**

Never reverse this dependency or project later theory or implementation backwards as historical evidence.

## Reconstruction and evidence

Sources are fragmentary, partial observations of one machine; no single document is necessarily complete or definitive. Reconstruct the simplest architecture explaining all credible observations, contradicting none of the reliable evidence, with the fewest unsupported mechanisms:

**Observation -> Constraint -> Reconstruction -> Prediction -> Falsification**

Record what sources say, show or necessarily demonstrate; derive constraints needed to reconcile them; construct a coherent model; predict consequences; actively seek contradictions. Independent observations not used to build the model strengthen it.

Keep source/evidence categories **PRIMARY EVIDENCE, SECONDARY EVIDENCE, INFERENCE, HYPOTHESIS, UNKNOWN** distinct, and label architectural conclusions as:

- **DOCUMENTED OBSERVATION:** directly source-supported.
- **NECESSARY INFERENCE:** required to reconcile documented observations.
- **WORKING RECONSTRUCTION:** current simplest evidence-explaining model.
- **SPECULATION:** possible but insufficiently constrained.
- **UNKNOWN:** observations do not adequately constrain an answer.

Keep observations traceable to sources and reconstructions distinguishable from observations. Never promote inference or hypothesis to established architecture without supporting evidence; a labelled working reconstruction may combine observations and necessary inferences without an explicit source statement of the whole.

Do not repeatedly demand documentary wording for synthesized conclusions. Search when it can add observations, discriminate between models, test predictions, falsify a model or materially change evidential status; first consider whether explicit confirmation should reasonably exist.

Before discarding apparently conflicting sources, consider machine version, operating system, abstraction level, terminology, viewpoint and implementation versus architecture. When new evidence conflicts with the reconstruction, identify the statements, evidence and provenance, investigate explanations, and create or propose a research issue if unresolved, within the authorised scope. Do not silently alter the architecture.

### Current-open-question gate

`research/pp250-open-questions.md` is the canonical current-status index for unresolved historical architecture. Before describing, proposing or pursuing an architectural issue as open, check that file and reconcile any older `UNKNOWN`, `HYPOTHESIS`, “unresolved” or “open question” wording against the current architecture and later research. Historical research-state labels do **not** automatically remain current.

Do not create an open question merely because representations differ between generations, operating systems or capability forms. Cross-generation bit mapping, chronology or exact encoding is not an architectural requirement unless the missing detail prevents reconstruction of observed behaviour. If a genuinely new blocker is found, state the missing behaviour/mechanism precisely and update the canonical open-question index as part of the authorised research update.

## Initial reconstruction configuration

The immediate reconstruction target is a minimal System 250 configuration: **one processor, one store, and peripherals**. Do not make reconstruction of the complete redundant multi-processor/multi-store system a prerequisite for initial architectural understanding or the emulator workbench.

Prioritise the processor, store, peripheral interfaces and mechanisms required to operate this single-processor, single-store configuration, including startup, capabilities, process execution, interrupts and I/O. Defer redundant processors and stores, failover, and multi-unit reconfiguration to later stages. Continue to record historical evidence about those features where it illuminates the basic mechanisms; deferral is a research priority, not a claim that such mechanisms did not exist or a permanent exclusion from scope.

## Bootstrap boundary and authority

The immediate objective is an evidence-backed chain from inert/power-on state to the first legitimate execution of ordinary PP250 software, not recreation of ROS/POS or the whole historical OS.

- **Below the boundary:** hardware, microcode-visible mechanisms and required initial state.
- **At the boundary:** complete relevant first-process state and provenance of every required protected object/capability.
- **Above the boundary:** only enough to prove ordinary software can build a self-sustaining capability system.

For every bootstrap proposal ask: Where did authority originate? Which documented data structure holds it? How was that structure created from the preceding state? Is unexplained capability fabrication required? Is undocumented supervisor/privileged machinery introduced? Can normal PP250 mechanisms take over?

Prefer **data structures -> necessary algorithms** over invented OS behaviour. Label missing links as inference, hypothesis or unknown. Completion requires hardware/microcode to reach a valid first process with sufficient legitimate authority to construct subsequent software-managed resources without undocumented privilege or arbitrary capability fabrication.

Read the relevant detailed documents:

- `research/reconstruction-sufficiency-and-workbench-boundaries.md`: boundary, proof of sufficiency, completion criterion and workbench boundaries.
- `research/pp250-boot-and-processor-startup.md`: cold start, faults, CHP, Dump Stacks, initial C6/C7, processor initialisation/admission and first-process transition.
- `research/capability-genesis-and-resource-lifecycle.md`: capability genesis, SCT/resource lifecycle, primordial authority and dynamic resource admission/removal.

## M⟨H,T⟩ status

**H** (Church machine) and **T** (Turing machine) are concepts identified in surviving System 250 material; use H because C denotes capability registers. **M** is developing reconstruction theory, not an established historical term or third peer machine.

The motivation is the apparent combination within the Church machine of capability machinery and machinery acting on combined H/T state: CHP through the Dump Stack can replace a complete process state containing both Turing state and Church capability state.

Keep open whether M is a meta-machine manipulating H/T, underlying machinery implementing them, or a more precise formulation. No useful mathematical formulation is thereby settled. Develop and test M⟨H,T⟩ against the reconstruction; on failure revise the theory or re-examine reconstruction/evidence, rather than force the machine to fit.

Preserve substantial reasoning, definitions, invariants, predictions and falsification tests in research documents, not an expanded README.

## Workbench and FPGA

The emulator/simulator is an executable research workbench downstream of reconstruction and theory. Its faithful baseline implements the evidence-backed reconstruction, exposes uncertain assumptions and tests theory against behaviour. Keep later patents and new experimental mechanisms explicitly separate from that baseline.

FPGA work is downstream again: a new machine informed by reconstruction, theory and workbench results, not a historical System 250 hardware reproduction or evidence for it.

Inter-computer capability authority remains unsolved; transport, authentication or cryptography alone do not solve the authority-to-reconstruct problem.

## Academic and PhD potential

Proactively flag credible original contributions to the repository owner: unifying reconstructions; independently supported non-obvious predictions; principles beyond the usual capability/protection description; M⟨H,T⟩ abstractions, formal models, invariants, security properties or decompositions; distinct semantic/authority/privilege/trust boundaries versus later architectures; experiments testing general claims; and consequential negative or contradictory results.

For candidate conference/journal, historical, formal-security or experimental-architecture papers and PhD contributions, state: the contribution; potential significance/novelty; required evidence or experiment; needed prior-art/literature search; and suitability for a standalone paper, PhD or both. Treat novelty as a research lead pending literature review; lack of explicit historical wording alone does not disqualify reconstruction.

Preserve doctoral-scale work in `research/phd-research-programme.md` where appropriate and paper-sized arguments in research notes, retaining the reasoning chain. Academic flagging is separate from, and does not override, the patent safeguard.

## Patentable ideas and public disclosure

This repository is public. Before any commit, issue, PR, discussion or other disclosure of a new technical mechanism/design, distinguish historical reconstruction and documented prior art from new invention and assess potential patentable subject matter.

For potentially novel, technically useful material—or uncertainty—STOP before public disclosure, flag it privately to the owner with the reason for patent review, and obtain explicit permission before publishing implementation details. This applies particularly to modern capability processors, protected memory-mapped I/O, domain switching, interrupts, DMA protection, boot/root-capability construction, compact capabilities, FPGA/ASIC designs and security mechanisms.

This is a publication safeguard, not a legal patentability determination; separate prior-art and legal review is required.

## Research document types

`research/` contains research records, not the canonical architecture. Duplication between research documents is acceptable and can be evidentially useful. Do not consolidate, rewrite or delete research merely because the same conclusion appears elsewhere.

There are two distinct research-document lifecycles:

- **Topic research** is a living investigation of a question, such as capability genesis or bootstrap. It may be revisited and updated as new evidence, reasoning, competing hypotheses or revised reconstructions emerge. Preserve the reasoning chain rather than reducing it to the latest conclusion.
- **Source review** records the examination of a particular paper, patent, manual or other new piece of evidence: what it says, what it newly reveals, its implications, conflicts and questions. Once the review is complete, treat it as a fixed research record. Do not later rewrite it to match the current architecture or eliminate duplication. Amend it only to correct an actual error in the review itself, such as a mistaken citation, transcription error or factual misidentification.

Before modifying an existing research document, determine which type it is. Do not turn a source review into a living synthesis document, and do not freeze topic research merely because an earlier version recorded a conclusion.

## Architecture and document integrity

Research may be overlapping; architecture must be canonical. Each architectural element or mechanism should have one authoritative home under `architecture/`. Other architecture documents should reference that canonical treatment rather than independently defining or maintaining a second version of the same mechanism.

An architecture document may cite multiple research notes, source reviews, transcriptions, patents, papers or other evidence that independently led to or support the same reconstruction. Do not remove duplicated research in order to manufacture a single provenance chain.

Use **proportionate provenance** in architecture documents. The architecture is a usable specification/reconstruction, not a paper requiring citations on every statement. Basic, well-established architectural facts do not require inline references. Cite evidence or research for non-obvious, obscure, reconstructed, disputed, version-dependent or otherwise significant claims. Use multiple references only when their independent agreement or differing perspectives materially support the reconstruction; do not accumulate citations merely because an elementary fact appears in many sources. Clearly label inference, uncertainty and unresolved disagreement.

- `architecture/architecture.md` is the navigation index. Put substantive reconstruction in the relevant subject document under `architecture/`, identified through that index.
- Architecture documents are working reconstruction, not primary evidence. Apply proportionate provenance rather than requiring every statement to carry a source reference.
- When reviewing architecture, distinguish between material the document canonically owns and architectural material that belongs in another subject document. Report proposed moves before making them unless editing has explicitly been authorised.
- `transcriptions/` contains source transcriptions. Never silently correct technical content; identify suspected transcription errors separately.
