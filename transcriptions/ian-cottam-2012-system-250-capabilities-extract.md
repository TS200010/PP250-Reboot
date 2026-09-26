# Ian Cottam — System 250 capabilities extract

## Source

Ian Cottam, guest post, **“Experience and Good Taste in Software/Systems Design”**, *Walking Randomly*, 10 May 2012.

Original: https://walkingrandomly.com/?p=4284

## Provenance and status

This is a short verbatim extract from a later first-person retrospective by Ian Cottam. Cottam states that he worked for Plessey on System 250 in the mid-1970s. It is therefore useful first-hand retrospective evidence, but it is not contemporary 1970s documentation and should be treated accordingly when reconstructing precise machine mechanisms.

## Extract — “Capabilities”

> Later, but still in the mid 1970s, I worked for Plessey on their System 250 high-reliability, multiprocessor system. It was the first commercial example of a capability architecture. With such there is no supervisor state or privileged code rings or similar. If you held the capability to do something (e.g. read, write or enter another code context) you could do it. If you didn’t hold the appropriate capability, you could not.
>
> The only tiny section of code in the System 250 that was special was where capabilities were generated. No one else could make them.

## Reconstruction significance

The second paragraph is particularly important evidence for the unresolved capability-creation question. Cottam remembers capability generation as confined to a small section of code that was somehow “special”, while the preceding paragraph explicitly rejects supervisor state and privileged-code rings as the System 250 authority model.

The extract does **not** establish what mechanism made that code special. It does not by itself distinguish among a microcode/hardware-controlled mechanism, an undocumented instruction, or a documented instruction operating through a capability-authorised special path. It also does not establish that `LDP`/LOAD CAPABILITY was a capability-creation operation; loading an authorised capability and creating a new capability must remain distinct questions unless documentary evidence connects them.

Only the System 250-specific passage has been reproduced here. The remainder of Cottam’s article discusses later software-design experience and is available at the source URL above.
