# System 250 Architecture — Capability Representation

## Capability access rights

Page 4 identifies six named access rights: `EC`, `WC`, `RC`, `ED`, `WD`, `RD`.

The diagrams are transcribed as:

- COS capability pointer: `1 1 EC WC RC ED WD RD 0`
- POS access codes: `0 1 1 EC WC RC ED WD RD`

These diagrams are evidence for OS/version-specific encodings around the same six named access semantics. They must not be treated as proof that all nine positions have one fixed meaning across every System 250 generation.

## Early capability class/type encoding

Early Plessey patent material describes a two-bit classification associated with a capability/access code:

| Two-bit value | Meaning |
|---|---|
| `11` | Active store-segment capability |
| `10` | Passive/backing-store segment capability |
| `01` | Resource capability |
| `00` | Null capability — no usable authority |

The early patent material also describes the null/zero case as an all-zero access code. Thus zero must not automatically be interpreted as merely an uninitialised capability register: it is also an architecturally meaningful null capability representation in this generation.

The six access rights (`EC WC RC ED WD RD`) are conceptually distinct from this class/type information.

## Later capability form encoding

Later Wheatley/Andrews patent material must be treated separately rather than projected backwards onto the original PP250. In that later representation, the 24-bit pointer has nine high-order form/access positions and fifteen identity positions. Two separated form-discrimination bits participate in classifying the pointer, while a seven-bit primary access field contains the six familiar rights plus a propagation-permit bit.

Accordingly, the later form discrimination is not simply the same contiguous two-bit table shown above. The later material includes distinctions among active/system-store, resource, passive/backing-store and ordinary-data representations and adds propagation control.

`PROPAGATION PERMIT` is therefore established for the later architecture but must not be assumed to have been the meaning of one of the original PP250's three non-rights positions without independent early evidence.

## Reconstruction rule

For reconstruction and emulator work:

1. Preserve the six named rights as documented semantic operations.
2. Preserve source/version-specific nine-bit encodings exactly as transcribed.
3. Do not collapse early class/type encoding, COS/POS layouts and later Wheatley/Andrews FORM/PROPAGATION encoding into one universal bit map.
4. Treat the meanings of the three positions outside `EC WC RC ED WD RD` in each early PP250/COS/POS representation as source-sensitive until reconciled.
5. In particular, do not assume that a non-zero access field is universally equivalent to a 'defined capability': early evidence explicitly admits an all-zero null capability.
