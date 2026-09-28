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

Later Wheatley/Andrews patent material must be treated separately rather than projected backwards onto the original PP250. In that later representation, the 24-bit pointer has nine high-order form/access positions (bits 23–15) and fifteen identity positions (bits 14–0). The two FORM discrimination bits are separated: bits 23 and 15. Between them, bits 22–16 form the seven-bit Primary Access Field.

For a **system-store capability**, the complete high-order nine-bit pattern is:

```text
bit:    23  22  21  20  19  18  17  16  15
        -----------------------------------
        1   PP  RC  WC  EC  RD  WD  ED  0
        ^                               ^
      FORM                            FORM
```

where `PP` is PROPAGATION PERMIT. The patent defines bits 19–21 as the Capability Access Bits READ, WRITE and ENTER CAPABILITY, and bits 16–18 as the Data Access Bits READ, WRITE and EXECUTE DATA. Thus, when written from bit 23 down to bit 15, the order is exactly `1 PP RC WC EC RD WD ED 0`.

This pattern is specifically the **system-store form** (`bit 23 = 1`, `bit 15 = 0`). It must not be treated as the access interpretation for every later capability form: system-resource and passive forms use the same FORM positions but interpret portions of the intervening field differently.

### FORM combinations

In the later Andrews/Wheatley representation, bits 23 and 15 are the two separated FORM discriminators. They select the interpretation of the intervening field rather than merely labelling an otherwise uniform capability.

| Bit 23 | Bit 15 | FORM | Meaning |
|---:|---:|---:|---|
| `1` | `0` | `10` | System Store capability |
| `0` | `0` | `00` | System Resource capability |
| `1` | `1` | `11` | Passive capability |
| `0` | `1` | `01` | **Not defined in the material examined so far** |

FORM `01` therefore remains an open architectural possibility: its meaning, if any was assigned, has not yet been established from the surviving material.

For a **System Store capability (`10`)**, the seven intervening bits are the Primary Access Field:

```text
PP RC WC EC RD WD ED
```

For a **System Resource capability (`00`)**, bit 22 remains `PP`, while bits 21–16 are a six-bit **Resource Type field** (resource type zero is not permitted) rather than the six store access rights. ENTER access is implied for this form:

```text
23  22  21 20 19 18 17 16  15
 0  PP  <--- RESOURCE TYPE --->  0
```

For a **Passive capability (`11`)**, the remaining fields are interpreted according to the passive capability form; Local Store is one documented passive type.

**Architectural consequence:** FORM participates in determining how M interprets the remaining bits of the 24-bit pointer. In particular, the same six physical positions used for `RC WC EC RD WD ED` in System Store form are interpreted as the six-bit Resource Type field in System Resource form.

Accordingly, the later form discrimination is not simply the same contiguous two-bit table shown above. The later material includes distinctions among active/system-store, resource, passive/backing-store and ordinary-data representations and adds propagation control.

`PROPAGATION PERMIT` is therefore established for the later architecture but must not be assumed to have been the meaning of one of the original PP250's three non-rights positions without independent early evidence.

### Propagation Permit and access reduction evolved together

The later Wheatley/Andrews architecture introduces two complementary facilities together:

- `PROPAGATION PERMIT` is the seventh bit of the Primary Access Field, alongside the six existing rights `EC WC RC ED WD RD`. It controls whether the authority represented by a capability may be propagated.
- The masked capability-load operation, `LCM`, provides access reduction: it reduces access rights when deriving/loading reduced authority.

Neither mechanism should be projected backwards onto early PP250: we have no evidence that it had either Propagation Permit or LCM-style general attenuation.

**WORKING RECONSTRUCTION:** This is a coherent later architectural evolution: once general capability attenuation/propagation was introduced, the architecture acquired both a permission controlling propagation and a mechanism for reducing propagated authority.

This does **not** explain the earlier COS/POS one-bit displacement of the six access rights. The May 1976 Pocket Reference already records that displacement, whereas the Wheatley/Andrews enhancement is later. The reason for the earlier shift therefore remains **UNKNOWN**.

## Reconstruction rule

For reconstruction and emulator work:

1. Preserve the six named rights as documented semantic operations.
2. Preserve source/version-specific nine-bit encodings exactly as transcribed.
3. Do not collapse early class/type encoding, COS/POS layouts and later Wheatley/Andrews FORM/PROPAGATION encoding into one universal bit map.
4. Treat the meanings of the three positions outside `EC WC RC ED WD RD` in each early PP250/COS/POS representation as source-sensitive until reconciled.
5. In particular, do not assume that a non-zero access field is universally equivalent to a 'defined capability': early evidence explicitly admits an all-zero null capability.
