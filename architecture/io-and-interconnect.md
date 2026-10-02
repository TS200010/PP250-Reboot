# System 250 Architecture — I/O and Interconnect

## Protected channel transfer

**DOCUMENTED OBSERVATION:** US3787818A, Description 48–50 and 63–90, describes autonomous channel modules with source/destination capability registers and a transfer Dump Stack containing pointer pairs. The channel reads descriptors through its SCT capability, validates their sumcheck/base/limit information and checks source/destination bounds during transfer. Special channel capabilities include the transfer Dump Stack, System Interrupt Word and SCT. Processor backdoor writes to those special registers are allowed while the channel is offline (48).

This is channel-side protection, distinct from processor checks on ordinary instructions. The description does not establish every access-right check, the complete authority governing channel setup, or coordination of active channels during relocation. The peripheral/control register accesses are memory-mapped; a separate I/O instruction set is not required by the described scheme (4).

## Access-unit read-and-hold

**DOCUMENTED OBSERVATION:** US4050059A, Description 13, describes READ-AND-HOLD locking the whole access unit to the requesting bus/port. A WRITE or RESET from that bus releases the hold; the described automatic timeout is 10 microseconds. Description 16–21 uses parity inversion on the write following a hold to verify hold integrity, including multiplexed arrangements, with failures entering the fault-recovery mechanism described in US3814919A.

These observations establish an atomic-access facility and its integrity checks. They do not establish an arbitrary-duration software mutex or a lock spanning a complete relocation transaction.

## Bus diagnosis and version boundaries

**DOCUMENTED OBSERVATION:** US4041460A, Description 17–19, exposes the address received by the access unit and its inverse through an address-monitor register. This permits checks and localisation of address/bus faults during operation without adding capability authority.

Reset-decoder wording differs between source descriptions: US4041460A describes reset for codes other than 1, 2 and 4, while other descriptions specify a reset code. This remains a source/version discrepancy; no universal combined decoder is inferred.

## Sources

- [US3787818A transcription](../transcriptions/US3787818A.rtf).
- [US4050059A transcription](../transcriptions/US4050059A.rtf).
- [US4041460A transcription](../transcriptions/US4041460A.rtf).
- [Completed batch review and limitations](../research/pp250-patent-transcriptions-review-2026-10-02.md).
