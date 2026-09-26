# Swift Closures as PP250 Authority

## Purpose

This note captures a specific modern-language experiment arising from the broader source-language authority investigation: how Swift first-class functions and capturing closures might map onto a PP250-style capability architecture.

It is a **research hypothesis and compiler experiment**, not a claim about the historical PP250 compiler or historical use of the architecture.

## First-class functions and closures

A Swift closure can be viewed conceptually as two related things:

```text
closure
  |
  +-- invocation / code
  |
  +-- captured environment
```

For example:

```swift
func makeAdder(_ x: Int) -> (Int) -> Int {
    return { y in x + y }
}
```

The returned function must preserve both the ability to invoke the closure body and access to the captured value `x` after `makeAdder` has returned.

On a conventional architecture this relationship is commonly lowered toward a code pointer plus some form of environment/context reference. On a PP250-like capability architecture the more interesting interpretation is:

```text
invocation authority     -> capability / Enter authority
captured environment     -> bounded data authority
ordinary arguments       -> ordinary data where appropriate
```

The significant point is not that PP250 hardware should understand a "Swift closure". It should not. The language/runtime/compiler understands the closure; the machine enforces the authority relationships used to realise it.

A closure therefore suggests the general interpretation:

> **A first-class function value carries invocation authority together with the authority required by its captured environment.**

Two closures can consequently execute the same code while possessing different captured environments and therefore different authority.

## Captured state

For immutable captured state, the environment may require only the authority necessary to read that state. For mutable captured state, such as:

```swift
func counter() -> () -> Int {
    var n = 0
    return {
        n += 1
        return n
    }
}
```

the escaping closure requires continuing authority to the storage holding `n`, including whatever write authority is required by the implementation.

This gives escaping and non-escaping closures a useful experimental distinction. A non-escaping closure may be able to use authority tied to the current activation. An escaping closure requires an environment whose lifetime and legitimate authority survive that activation.

The exact PP250 representation, register allocation, stored-capability form and calling sequence are deliberately left open until the historical architecture and ABI requirements justify them.

## The LLVM information boundary

The important correction is that arbitrary LLVM IR cannot in general be expected to reconstruct all of the original Swift semantics after those semantics have been lowered away.

If a Swift closure has become something structurally equivalent to a code reference plus an environment pointer, ordinary LLVM IR may preserve enough structure to execute it correctly without preserving the fact that the source construct was specifically a Swift closure. Conversely, optimisation or lowering may erase distinctions that would have allowed a more precise authority mapping.

The compiler problem is therefore not:

> Can arbitrary LLVM IR recreate Swift semantics?

It is:

> **Which authority relationships needed for a PP250 implementation survive Swift -> SIL -> LLVM lowering, and what is the minimum information that must be preserved when they do not?**

This is why Swift SIL is particularly interesting. It provides a point above LLVM IR at which closure capture, ownership, lifetime, access and invocation relationships may still be explicit enough to identify the intended authority before conventional lowering obscures it.

## Capability provenance as a compiler invariant

A PP250 backend must not weaken the architecture merely to accommodate conventional pointer operations appearing in LLVM IR.

A useful working invariant is:

> **Every authority-bearing pointer or reference lowered to PP250 machine code must have a legitimate capability provenance.**

This does not mean that every LLVM pointer operation requires a newly manufactured capability. Ordinary displacement within an already authorised object may remain computation: a PP250 capability supplies authority over the object while ordinary data arithmetic supplies the displacement.

Likewise, an LLVM operation that superficially resembles address arithmetic may be safely lowerable if the compiler can prove that it remains an operation under existing authority.

The critical distinction is between **computing a position within existing authority** and **manufacturing new authority from a number**.

Thus an `inttoptr`, or a `ptrtoint` / integer-arithmetic / `inttoptr` sequence, must not automatically become an arbitrary PP250 capability. If the compiler can establish a legitimate capability provenance and interpret the operation without increasing authority, it may be lowerable. If no such provenance can be established, the operation is not representable by ordinary PP250 capability semantics and should be rejected or require an explicitly defined ABI/runtime mechanism.

The prohibition is therefore not simply "PP250 does not support `inttoptr`". It is:

> **ordinary computation must not create authority that the computation did not already possess.**

## Minimal experiment

The initial objective should not be "compile all Swift for PP250".

A smaller experiment can test the architectural proposition directly. Use a sequence of tiny Swift programs covering:

1. an ordinary function;
2. a non-capturing first-class function value;
3. a closure capturing immutable state;
4. a closure capturing mutable state;
5. a non-escaping closure;
6. an escaping closure.

For each example trace:

```text
Swift semantics
      -> SIL representation
      -> LLVM IR representation
      -> authority information retained or lost
      -> proposed PP250 authority representation
      -> required ABI/backend operation
```

The experiment should determine whether invocation and captured-environment relationships can reach PP250 code generation without being reduced to forgeable numerical addresses, and identify the earliest compiler stage at which additional authority information becomes necessary.

A successful demonstration would therefore be more than "Swift code runs on a PP250 emulator". It would test whether a modern language's first-class-function semantics can be realised as hardware-enforced authority on a reconstructed early capability architecture.

## Relationship to the broader compiler investigation

This note is a focused extension of `research/source-language-semantics-as-hardware-authority.md`.

That broader work asks whether source-language relationships can be preserved or recovered as hardware authority. Swift closures provide a particularly clean test because the source construct visibly combines **invocation** with **captured state and lifetime**. The experiment should therefore be used to test the general authority-preservation hypothesis, not to bake Swift-specific concepts into PP250 hardware.