# System 250 Architecture — Capability Representation

## Capability access rights

Page 4 identifies six named access rights: `EC`, `WC`, `RC`, `ED`, `WD`, `RD`.

The diagrams are transcribed as:

- COS capability pointer: `1 1 EC WC RC ED WD RD 0`
- POS access codes: `0 1 1 EC WC RC ED WD RD`

The exact relationship between these OS-specific diagrams and the more general capability-form/type encoding below must be kept version- and source-sensitive.

## Stored capability type/form field

Contemporary Plessey patent material describes the high-order two-bit classification of a stored capability/reference. These two bits determine what kind of protected reference the remaining word represents and therefore what processing is appropriate.

| Two-bit value | Meaning |
|---|---|
| `11` | Active store-segment capability — refers through the System Capability Table (SCT) to a segment represented by an SCT identity |
| `10` | Passive/backing-store segment capability — represents a segment in backing store rather than an immediately usable active SCT reference |
| `01` | Resource capability — represents a non-store logical/system resource |
| `00` | Null capability — no usable capability/authority |

These values are architecturally important: the two-bit field is not merely another pair of access permissions. It classifies the form of capability and determines the interpretation of the rest of the stored representation.

The six access rights (`EC WC RC ED WD RD`) are conceptually distinct from this form/type classification. A capability therefore carries both a statement of **what kind of protected reference it is** and, where applicable, **what operations are permitted through it**.
