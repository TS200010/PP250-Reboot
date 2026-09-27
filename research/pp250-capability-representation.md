# PP250 capability representation

Status: research reconstruction, 27 September 2026. Material relocated from [PP250 execution and process model](pp250-execution-and-process-model.md). Evidence labels, source qualifications and unresolved questions are retained; this note does not amend the architecture specification.

## Stored and expanded capabilities: source comparison

**Documented, [EP-E1], paragraphs 16 and 19:** a stored capability combines access rights with a reference to an SCT entry. Loading it obtains base/limit information from that entry and combines it with the stored access rights. The result addresses a bounded segment with specified permitted operations.

```text
24-bit stored capability pointer
+------------------+------------------------------------+
| access/form code | SCT reference / index              |
+------------------+------------------+-----------------+
                                      |
                                      v
                              SCT entry: base / limit
                                      |
                                      v
48-bit capability-register representation
+-------------------------------------------------------+
| base address                                          |
+-------------------------------------------------------+
| access information and limit                          |
+-------------------------------------------------------+
       conceptual fields; not a universal bit allocation
```

The compact stored pointer is not a raw physical address. Nor is saving C0 at one Dump Stack word evidence for storing all 48 register bits there. Capability identity and rights can be preserved compactly and the expanded descriptor reconstructed.

**Exact layouts available, with provenance:** [EP-L1], Figure 4-1, labels its stored format as an 8-bit rights field and 16-bit SCT index (**SECONDARY EVIDENCE**). [EP-R1], p. 4, instead supplies the following nine-position access/prefix diagrams; it does not establish a complete universal 24-bit layout:

```text
COS:  1 1 EC WC RC ED WD RD 0
POS:  0 1  1 EC WC RC ED WD RD
```

The diagrams are reproduced as transcribed. They are not interchangeable. [EP-P4], “Capability Formats,” explicitly describes a 24-bit pointer with form/access information in the high nine bits and an identity in the low fifteen; its later form discrimination and propagation-permit rules introduce further distinctions. These source differences must be reconciled by machine/version and capability form before selecting emulator bit fields. The complete expanded-register bit map is likewise not reconstructed from schematic text alone.

The named rights are enter capability (EC), write/read capability (WC/RC), execute data (ED), and write/read data (WD/RD); [EP-E1], paragraph 16 and Figure 2, explains the corresponding operations. EC permits entry through a capability block; ED permits instruction execution. They are distinct rights.

### Sources for the relocated material

Source identifiers prefixed `EP-` retain the provenance and verification limits of the execution/process note; this reorganisation does not constitute a new source verification.

- **[EP-R1] PRIMARY EVIDENCE via repository transcription:** user's Plessey *System 250 Pocket Reference Book / Instruction Codes*, Issue 1, May 1976. [Title/contents transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg0-pg2%20transcription.txt); [pp. 3–4 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg3-pg4.pdf); [pp. 5–7 transcription](../transcriptions/System%20250%20Pocket%20Reference%20pg5-pg7%20transcription.txt) and [scan](../documentation/System%20250%20Pocket%20Reference%20pg5-pg7.pdf). Locators: p. 3 instruction codes, p. 4 access-code diagrams, p. 5 ROS/PDOS structures/state word, p. 6 Dump Stack, p. 7 Special Purpose CPU Registers/Internal Mode. Transcriptions checked; scans not independently rechecked here.
- **[EP-E1] PRIMARY EVIDENCE:** D. M. England, *Architectural Features of System 250* (1972), [repository paper](../sources/1972/1972-England-Architectural-Features-of-System-250.pdf). Paragraphs 4–9: topology; 16–20: rights, integrity and SCT; 21–26: domains/CALL/Dump Stack; 30: virtual store and disk capabilities; 31: process management and CPU independence. PDF text inspected; diagram bit boundaries not treated as verified from extraction.
- **[EP-P4] PRIMARY EVIDENCE:** US 4,408,274, *Memory protection system using capability registers*, [repository PDF](../patents/US4408274-memory-protection-capability-registers.pdf), [patent text](https://patents.google.com/patent/US4408274A/en). Locators: Description of Prior Art, Capability Formats, LC/load-on-use and SC descriptions, Figures 5–9 references. Text read; enhanced pointer format and propagation controls are version-qualified.
- **[EP-L1] SECONDARY EVIDENCE:** Henry M. Levy, *Capability-Based Computer Systems*, Digital Press, 1984, [chapter 4, “The Plessey System 250”](https://homes.cs.washington.edu/~levy/capabook/Chapter4.pdf), especially sections 4.2–4.5 and Figure 4-1. Read as corroboration and for terminology; it does not override the pocket reference or resolve version conflicts.
