# System 250 Overall Architecture

Status: research reconstruction, 27 September 2026. System-level context relocated from the execution/process research note; not a wiring diagram or an emulator specification.

## Scope

This note describes the physical System 250 setting: processors, shared Store Modules, peripheral subsystems and their interconnection. Documented observations and architectural inferences remain distinguished. Process state and transitions are developed in [PP250 execution and process model](pp250-execution-and-process-model.md).

## Physical system and interconnection

**Documented, [EP-E1], paragraphs 4–9:** System 250 comprises multiple processors, shared Store Modules and peripheral subsystems. A processor can reach each Store Module. Devices expose controller registers through the interconnection and can be addressed using ordinary data instructions.

The following is a **conceptual common system/CPU-bus view**, showing peer subsystems, not a wiring diagram:

```text
   +-------------+     +-------------+     +-------------+
   | Processor 0 |     | Processor 1 | ... | Processor n |
   +------+------+     +------+------+     +------+------+
          |                   |                   |
==========+===================+===================+===========
          Common system / CPU-bus interconnection (conceptual)
==========+===================+===================+===========
          |                   |                   |
   +------+------+     +------+------+     +------+----------+
   | Store       |     | Store       | ... | Disk / secondary|
   | Module A    |     | Module B    |     | storage subsystem|
   | shared      |     | shared      |     | via controller /|
   +-------------+     +-------------+     | bus interface   |
                                          +-----------------+
```

Disk storage is off the bus as a peer bus-connected subsystem. It does **not** hang from a Store Module. This does not imply that disk sectors are directly accessible like primary-memory words: controller-register access and backing-store transfer are distinct operations.

**Physical-topology qualification:** England describes each CPU's own parallel CPU bus, with a port at each Store Module's access unit; bus multiplexors connect CPU buses to peripheral buses. [EP-P2], description of Figure 1, likewise has separate CB1/CB2 paths terminating at access-unit ports. Thus “common bus” above means a common reachable system interconnection, not one electrically shared CPU wire bundle. Multiplexors, duplicated peripheral paths and access-unit arbitration are collapsed in the diagram.

**Strong inference from [EP-E1], paragraphs 16–19, and [EP-P2]:** the bus transports addresses, information and control/status signals; it is capability/data agnostic in the sense that a transmitted bit pattern does not acquire authority merely by travelling on it. The processor's capability checks and microcode enforce permitted operations and bounds. This does not deny transport parity, control codes or interface checks, and should not be read as saying the bus carries only unqualified data bits.

## Related architectural subjects

- [Processor control](pp250-processor-registers-and-internal-mode.md): processor registers and Internal Mode.
- [Capability representation](pp250-capability-representation.md) and [System Capability Table](pp250-sct-segments-and-virtual-memory.md): protected references, segments and their physical realisation.
- [Execution and process model](pp250-execution-and-process-model.md): running and suspended processes, CALL and CHP, and processor mobility.
- [Boot and processor startup](pp250-boot-and-processor-startup.md): establishment of executable processor state.
- [Normal interrupt and system dispatch](pp250-normal-interrupt-and-system-dispatch.md): operational entry and dispatch.


### Sources for the relocated material

Source identifiers prefixed `EP-` retain the provenance and verification limits of the execution/process note; this reorganisation does not constitute a new source verification.

- **[EP-E1] PRIMARY EVIDENCE:** D. M. England, *Architectural Features of System 250* (1972), [repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf). Paragraphs 4–9: topology; 16–20: rights, integrity and SCT; 21–26: domains/CALL/Dump Stack; 30: virtual store and disk capabilities; 31: process management and CPU independence. PDF text inspected; diagram bit boundaries not treated as verified from extraction.
- **[EP-P2] PRIMARY EVIDENCE:** US 4,383,297, *Data processing system including internal register addressing arrangements*, [repository PDF](../patents/US4383297-internal-register-addressing.pdf), [patent text](https://patents.google.com/patent/US4383297A/en). Locators: illustrative embodiment/Figure 1 description; special data and capability registers; Internal Mode Operation General and restrictions. Text read; later register map kept distinct from [EP-R1].
- **[EP-R1] PRIMARY EVIDENCE via repository transcription:** user's Plessey *System 250 Pocket Reference Book / Instruction Codes*, Issue 1, May 1976. [Title/contents transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg0-pg2%20transcription.txt); [pp. 3–4 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg3-pg4.pdf); [pp. 5–7 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg5-pg7.pdf). Locators: p. 3 instruction codes, p. 4 access-code diagrams, p. 5 ROS/PDOS structures/state word, p. 6 Dump Stack, p. 7 Special Purpose CPU Registers/Internal Mode. Transcriptions checked; scans not independently rechecked here.
