# Reconstruction Sufficiency and Workbench Boundaries

This note preserves project principles that were previously stated at length in the root README. They are retained here so that the README can remain a concise statement of project direction without losing the criteria against which reconstruction and later implementation work are judged.

## Reconstruction boundary

The primary historical reconstruction runs from an inert PP250/System 250 machine to the first legitimate execution of ordinary PP250 software.

The reconstruction must account for the hardware and protected state required to cross that boundary. It must not hide bootstrap problems behind an assumed operating system. If the first program requires capabilities, SCT entries, C6/C7, a Dump Stack, executable code or other protected state, their provenance must be explained by documented behaviour or by explicitly identified reconstruction hypotheses.

Detailed investigation of cold start, faults, CHP, Dump Stacks, initial C6/C7 and processor admission is maintained in `research/pp250-boot-and-processor-startup.md`. Capability genesis, SCT/resource lifecycle and primordial authority are maintained in `research/capability-genesis-and-resource-lifecycle.md`.

## Proof of sufficiency

Reaching the first ordinary instruction is not by itself enough. The reconstructed initial state must contain sufficient legitimate ancestral authority for ordinary PP250 mechanisms to construct a continuing capability-managed system.

A successful reconstruction should therefore make it possible for software, without arbitrary capability fabrication or an undocumented privileged escape, to establish and manage protected resources, derive restricted authority, construct process/Dump-Stack state, create another process and transfer execution to it using normal machine mechanisms.

This is the **proof-of-sufficiency criterion**:

> The reconstructed hardware and initial state must provide enough legitimate authority for the first ordinary software to construct all subsequent software-managed resources using normal PP250 capability mechanisms.

A proposed bootstrap is incomplete if the first program must manufacture authority from ordinary data. A reconstruction that requires a persistent undocumented supervisor mode is likewise suspect. A reconstruction that provides sufficient ancestral authority and then permits that authority to be legitimately derived, restricted, transferred and consumed is architecturally viable and can be tested against the surviving evidence.

The purpose of this criterion is not to reconstruct ROS, POS or the complete historical operating system. A small demonstrator can establish sufficiency if it proves that the reconstructed machine can become self-sustaining above the hardware/software boundary.

## Emulator and simulation workbench contract

The emulator/simulator is downstream of the historical reconstruction and the developing M⟨H,T⟩ theory. Its first responsibility is to provide an executable **faithful baseline** for the architecture supported by the evidence.

The workbench must therefore:

- implement documented behaviour where it is known;
- identify reconstructed or uncertain behaviour rather than silently inventing it;
- make assumptions traceable to their evidence status;
- provide tests against documented and reconstructed behaviour;
- permit architectural hypotheses, including M⟨H,T⟩, to be exercised against concrete machine behaviour; and
- keep later patents, alternative historical developments and new experimental mechanisms explicitly separate from the faithful baseline.

Experimental convenience must never silently change the historical model.

## FPGA boundary

A later FPGA machine is downstream of the reconstruction, M⟨H,T⟩ investigation and emulator workbench. It is not evidence about the historical System 250 and need not reproduce the PP250's 1970s implementation choices.

Its purpose is to investigate what a real machine informed by the architectural principles that survive the reconstruction and theoretical work should look like using contemporary technology. It may therefore use modern FPGA technology, RAM, interconnects, peripherals, DMA, storage, networking and development/debugging interfaces.

The historical reconstruction and the new hardware architecture must remain distinguishable. The FPGA design may simplify, alter or extend mechanisms where justified, but those choices must not be back-projected into the reconstructed PP250.

## Inter-computer authority boundary

Inter-computer capability authority remains a separate research problem rather than an established feature of either the reconstructed PP250 or the FPGA design.

The central problem is not simply serialising a capability. Once transmitted through an ordinary channel, a representation is data and can be copied or manufactured. The architectural question is what legitimate authority permits a receiving machine to transform a received representation into local capability authority.

Authentication, encryption or secure transport do not by themselves answer that authority question. Until a satisfactory authority model is established, inter-computer capability work remains a research hypothesis and should not be presented as settled architecture.