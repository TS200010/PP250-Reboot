# System 250 Architecture — I/O and Interconnect

## Protected channel transfer

**DOCUMENTED OBSERVATION:** US3787818A, Description 48–50 and 63–90, describes autonomous channel modules with source/destination capability registers and a transfer Dump Stack containing pointer pairs. The channel reads descriptors through its SCT capability, validates their sumcheck/base/limit information and checks source/destination bounds during transfer. Special channel capabilities include the transfer Dump Stack, System Interrupt Word and SCT. Processor backdoor writes to those special registers are allowed while the channel is offline (48).

This is channel-side protection, distinct from processor checks on ordinary instructions. The description does not establish every access-right check, the complete authority governing channel setup, or coordination of active channels during relocation. The peripheral/control register accesses are memory-mapped; a separate I/O instruction set is not required by the described scheme (4).

## May 1976 Pocket Reference — peripheral register interfaces

**DOCUMENTED OBSERVATION (pp. 17–18):** The peripheral controllers expose registers at fixed offsets in their respective module address spaces. Typical interfaces comprise an address-monitor register (`MAR`), a second address register (`TAR` or `IAR` as named in the tables), `STATUS`, `COMMAND`, and `DATA IN`/`DATA OUT`. Register counts and offsets vary between disc, magnetic tape, line printer, parallel/serial teletype, serial punch/reader, and the serial-parallel adaptor. The tables should be used per-device; they do not define a universal register layout. The documented `MAR` and inverse-address facilities correspond to the address-monitor diagnostics in US4041460A, Description 17–19.

**DOCUMENTED OBSERVATION (pp. 19–21):** The Backing and Initialising Store Control Unit, Console Control Unit and Exchangeable Disc System each have a larger address structure. These include status/command addresses, forecast and buffer-transfer functions, and addressable data-buffer regions. The Exchangeable Disc System additionally exposes `WORK ITEM`, `WORK STATUS`, and last-sector-address regions. Some offsets are explicitly assigned different meanings for reads and writes (for example, an all-ones read versus a command write at octal `0003`). These are device-specific register actions, not ordinary passive memory cells.

**DOCUMENTED OBSERVATION (pp. 22–34):** The individual command/status bit tables include full/empty, stop, offline, overrun, fault and transfer-condition indications. The disc material also describes a `TRANSFER CAPABILITY` with disc-address/transfer fields and a check code. Its precise diagram requires checking against the scan before assigning individual bit positions; it must not simply be equated with an ordinary System 250 memory capability.

**DOCUMENTED OBSERVATION (pp. 3–4):** `MOVE` is a direct-mode instruction (octal `74`) using source `C2/D2`, destination `C3/D3`, sumcheck `D5`, and count `D0`. The reference says overflow is reset before `MOVE`, is set on exit by the STATUS wire, and that a nonzero remaining count indicates device BUSY. The exact partial-transfer update/termination sequence is not supplied by this summary.

**PATENT CORROBORATION:** US4041460A, Description 2 and 5–16, documents module/register selection and the access-unit bus commands, handshaking and response indications. US3787818A, Description 54–62, documents peripheral status/data registers and polling; Description 32–39 and 63 onward describes an *earlier patented* independent capability-protected channel transfer facility. This channel design is evidence for that patent's architecture, not proof that the identical channel implementation remained in service in May 1976.

**INTERPRETATION:** The 1976 peripheral tables support substantial continuity of the memory-mapped access-unit interface; they do not call for postulating a wholesale redesign of disc, tape and other peripheral interfaces. Peripheral status examination provides a software-visible readiness/completion mechanism. The separate [architectural-generations SIW timeline](../research/pp250-architectural-generations.md#evidence-update--11-october-2026-system-interrupt-word-siw) records the working reconstruction that SIW-based completion notification had apparently been removed by May 1976; that conclusion is not established merely by the presence of status registers.

## Access-unit read-and-hold

**DOCUMENTED OBSERVATION:** US4050059A, Description 13, describes READ-AND-HOLD locking the whole access unit to the requesting bus/port. A WRITE or RESET from that bus releases the hold; the described automatic timeout is 10 microseconds. Description 16–21 uses parity inversion on the write following a hold to verify hold integrity, including multiplexed arrangements, with failures entering the fault-recovery mechanism described in US3814919A.

These observations establish an atomic-access facility and its integrity checks. They do not establish an arbitrary-duration software mutex or a lock spanning a complete relocation transaction.

## Bus diagnosis and version boundaries

**DOCUMENTED OBSERVATION:** US4041460A, Description 17–19, exposes the address received by the access unit and its inverse through an address-monitor register. This permits checks and localisation of address/bus faults during operation without adding capability authority.

Reset-decoder wording differs between source descriptions: US4041460A describes reset for codes other than 1, 2 and 4, while other descriptions specify a reset code. This remains a source/version discrepancy; no universal combined decoder is inferred.

## Physical resource identity and capability authority

**DOCUMENTED OBSERVATION:** Peripheral/control interfaces described here are memory-mapped, without requiring a distinct peripheral instruction set. Physical addressability, physical presence, software interpretation and capability authority are separate concepts.

**HYPOTHESIS / UNRESOLVED:** An exclusive primordial allocator might remove an entire address range from an unallocated pool and manufacture its first capability with only the requested rights. Neither memory-mapped I/O nor the ordinary Store Allocator's length-and-access interface proves this. Removing hardware does not itself invalidate surviving capabilities; replacement at the same address requires safe handling of stale references and capabilities already expanded in processor registers. SCT indirection may help, but physical admission, revocation and identity reuse remain unresolved. See [resource-lifecycle reconstruction, section 22](../research/capability-genesis-and-resource-lifecycle.md#22-8-october-2026--exclusive-primordial-allocation-and-dynamic-physical-reconfiguration).

## Sources

- [Pocket Reference pp. 3–4](../transcriptions/System%20250%20Pocket%20Reference%20pg3-pg4%20transcription.txt).
- [Pocket Reference pp. 17–21](../transcriptions/System%20250%20Pocket%20Reference%20pg17-pg21.txt).
- [Pocket Reference pp. 22–27](../transcriptions/System%20250%20Pocket%20Reference%20pg22-pg27.txt).
- [Pocket Reference pp. 28–36](../transcriptions/System%20250%20Pocket%20Reference%20pg28-pg36.txt).
- [US3787818A transcription](../transcriptions/US3787818A.rtf).
- [US4050059A transcription](../transcriptions/US4050059A.rtf).
- [US4041460A transcription](../transcriptions/US4041460A.rtf).
- [Completed batch review and limitations](../research/pp250-patent-transcriptions-review-2026-10-02.md).
