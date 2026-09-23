# PP250 Application Binary Interface

## 1. Scope

This document defines the Application Binary Interface (ABI) for programs executing on the PP250 architecture.

The ABI specifies the binary-level conventions required for independently compiled and assembled program components to interoperate. It defines how language-level constructs are represented at the PP250 binary interface and how those constructs are mapped onto the PP250 execution and protection model.

The ABI specifies, in particular:

- the machine-level data representation and alignment requirements;
- the use and preservation of PP250 registers;
- the procedure calling convention;
- the representation and passing of arguments and return values;
- the representation and invocation of capabilities;
- the representation of executable entries and enter capabilities;
- the representation of objects, interfaces and method dispatch;
- the layout and invocation of separately compiled modules;
- the conventions required for linking and relocation;
- the mapping of supported language and compiler constructs onto PP250 facilities; and
- the security properties that shall be preserved by conforming implementations.

The ABI does not define the PP250 instruction set, instruction encodings, processor implementation, or microarchitectural behaviour. Those are defined by the PP250 Architecture Specification.

The ABI does not prescribe the implementation of a particular programming language. A language implementation may expose the facilities defined by this ABI through its own source-language abstractions, provided that generated code conforms to the requirements of this specification.

In particular, the capability architecture defined by this ABI is an execution and protection mechanism and need not be exposed directly to the source programmer. A compiler may use capabilities to implement ordinary language constructs such as function calls, objects, methods, interfaces and modules.

An ABI-conforming compiler shall generate code whose externally observable binary behaviour conforms to this specification. An ABI-conforming assembler and linker shall preserve the representations and authority relationships required by this specification.

Where this specification refers to a capability, the term denotes a PP250 capability as defined by the PP250 Architecture Specification. A capability shall not be treated as an ordinary integer or conventional machine address unless explicitly permitted by this specification.

This specification is intended to permit separately compiled components to interoperate while preserving the authority and protection semantics of the PP250 architecture.
