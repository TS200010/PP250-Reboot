# From Source-Language Relationships to Hardware Authority

## Why this line of enquiry started

This note deliberately preserves the reasoning path rather than reducing it to a conclusion.

The immediate discussion began with LLVM. The first temptation was to ask how a PP250 LLVM backend could represent capabilities: perhaps a special LLVM address space, an unusually wide pointer type, capability-specific intrinsics, or capability-aware instruction selection.

That framing turned out to be backwards for the experiment we actually want to perform.

The important observation was that a PP250 capability need never appear explicitly in ordinary LLVM IR. LLVM can continue to describe the program in its normal terms: objects, pointers, GEPs, loads, stores, calls, function arguments, indirect calls and so forth. The PP250 code generator can then decide that certain relationships expressed by that IR are best realised using hardware capabilities.

That led to a more fundamental question:

> Instead of asking which LLVM constructs should become capabilities, suppose we were writing a C++ compiler from scratch for a capability machine. What would we naturally want to implement as capabilities?

This question moves the analysis above LLVM. It lets us first decide what authority exists in the source-language semantics, and only afterwards ask whether LLVM preserves enough evidence for a backend to recover it.

## The distinction that matters

A conventional compiler tends eventually to turn many source-language relationships into addresses.

A pointer becomes an address. A reference becomes an address. `this` becomes an address. A function pointer becomes a code address. A virtual call eventually obtains a code address from a table. Object state is accessed by calculating addresses.

On a capability machine there is no requirement to collapse all of those relationships into numbers.

A useful working distinction is:

```text
VALUES                          RELATIONSHIPS / AUTHORITY

int                             pointer
float                           reference
bool                            this
enum                            object access
ordinary arithmetic             function pointer
                                invocation
                                delegation
                                protected state
```

On a PP250-like architecture this suggests another distinction:

```text
values / computation      -> D world
relationships / authority -> C world
```

This is not yet a claim that every item in the right-hand column must become a distinct capability. It is the hypothesis we want to test.

## Objects and `this`

Consider:

```cpp
class Account {
    int balance;
    int history[100];

public:
    int getBalance();
    void deposit(int);
};
```

An instance of `Account` is naturally a bounded storage object. A capability to that object can mean authority to access that particular instance's storage.

```text
C1 ------> Account object
           +----------------+
           | balance        |
           | history[100]   |
           +----------------+
```

C1 is not merely the numerical location of the Account. It is the authority under which accesses to the Account can occur.

For `this->balance`, a conventional implementation treats `this` as a pointer and calculates the address of `balance`. A capability-native implementation can instead treat `this` as the authority for the current object:

```text
C(this) + offset(balance)
```

The offset is computation. The C register supplies the authority.

## Arrays

Arrays are perhaps the simplest case:

```cpp
int a[100];
a[i]
```

The natural implementation is:

```text
C(a) = authority over the array object
D(i) = position/displacement within that object
```

The memory operation uses the capability plus displacement. Hardware checks whether the requested operation lies within the segment and whether the capability permits the operation.

There is no need to create a new capability representing `&a[i]` merely to perform the access.

This is important: PP250 gives us the possibility of retaining one capability representing authority over the object while ordinary D-register computation identifies a position within that authority.

## Pointers

For `Account *p;`, the natural capability-machine question is not initially "what integer representation should p have?" It is: what relationship does possession of p represent?

At the language level, p permits the program to identify and access an Account object subject to the language rules. A capability-native implementation can therefore regard a pointer as a transferable reference carrying authority to the referenced object.

Assignment `p = q` then has an authority interpretation: authority represented by q is delegated/copied into p. Dereference `p->balance` means accessing the object under that authority.

A practical PP250 compiler cannot reserve a physical C register forever for every source-language pointer. Dormant references therefore need an architecturally protected representation in memory and a legitimate mechanism for loading them into capability registers. This is precisely why the PP250 distinction between capability registers and protected in-memory capability representation matters.

## References

C++ references expose the relationship even more directly:

```cpp
void update(Account &a);
```

The source-language meaning is that `update` is being given access to an existing Account object. On a capability machine this looks naturally like delegation:

```text
caller C3 ------> callee C1
```

The argument need not first become an ordinary numerical address and then somehow be protected after the fact. The call can transfer the authority itself.

This suggests that function calling conventions may be one of the most important places for a capability-native compiler.

## `const` and attenuation

Consider:

```cpp
void print(const Account &a);
```

At the C++ language level, the callee is restricted in how it may modify the object through a. That resembles capability attenuation:

```text
caller:
C3 = RD | WD

        |
        | attenuation
        v

callee:
C1 = RD
```

If the language semantics and architecture allow this correspondence safely, a property normally enforced only by the compiler could become hardware-enforced authority reduction.

However, ordinary C++ `const` must not automatically be equated with a security boundary. C++ has `mutable`, `const_cast`, aliasing and other semantics that complicate such an interpretation.

The important observation is therefore not "C++ const is a read-only capability". It is that source-language restrictions may contain enough information to derive reduced machine authority, and a language designed explicitly for capabilities could make this correspondence much stronger.

## Function pointers

For:

```cpp
void (*f)(int);
```

a conventional machine eventually represents f as a code address. But semantically the useful relationship is that the holder is able to invoke this function.

That is much closer to invocation authority than to data. On a PP250-like machine a function pointer is therefore a natural candidate for an Enter capability.

Then `f(42)` does not have to mean "jump indirectly to the numerical address stored in f". It can mean: exercise the authority represented by f to enter the permitted execution domain/entry point.

## Virtual functions and vtables

For:

```cpp
Shape *s = ...;
s->draw();
```

a conventional C++ implementation generally follows an object/vptr/vtable/function-pointer chain and eventually performs an indirect call to a code address.

A capability-native implementation could instead make the dispatch table contain invocation authority:

```text
Shape object
     |
     v
dispatch table
     |
     +----> capability: draw
     +----> capability: move
     +----> capability: destroy
```

The entry is no longer interesting because it contains the address of draw. It is interesting because possession of the entry confers the authority to invoke draw.

If PP250 Enter establishes an execution environment through C6/C7, the operation can establish both an authorised entry point and the appropriate domain/environment.

Virtual dispatch therefore becomes a particularly interesting candidate for direct architectural expression rather than merely protected indirect branching.

## Private state and encapsulation

In conventional C++, `private` is principally a language/compiler restriction. Once machine code exists, the processor does not know that a member was declared private.

A capability-native implementation opens a stronger possibility. The private representation can belong to a protection domain whose capability is not possessed by arbitrary outside code. Public operations can be exposed through Enter capabilities.

```text
outside code
     |
     | possesses
     v
public Enter capability
     |
     v
Account implementation domain
     |
     | possesses
     v
capability to private state
```

Then outside code does not merely refrain from accessing private state because the C++ compiler rejected the source expression. It lacks the machine authority required to access the state.

This illustrates why the research question is broader than memory-safe pointers. Capability architecture may directly implement object/module boundaries and invocation relationships.

## `new`, allocation and capability genesis

For:

```cpp
Account *p = new Account;
```

the natural sequence is:

```text
request storage
      |
      v
create/allocate bounded storage object
      |
      v
obtain legitimate capability to it
      |
      v
construct Account within it
      |
      v
return/delegate authority
```

The important unresolved architectural question is capability genesis: exactly which authority permits creation/allocation of the new bounded storage capability?

The compiler must not introduce an operation equivalent to:

```text
raw address + length -> arbitrary new capability
```

available to ordinary code, because that would defeat the authority model. Allocation must ultimately descend from existing legitimate authority.

`delete` introduces a different problem: lifetime and revocation. Capabilities alone do not automatically solve use-after-free. That must remain a separate research question rather than being hidden by the compiler mapping.

## Function arguments and return values

Once pointers/references are interpreted as authority, ordinary function boundaries acquire another meaning.

A pointer/reference argument is potentially delegation of authority from caller to callee. A pointer/reference return value is potentially delegation in the opposite direction.

This suggests that an ABI for a capability-native language should not begin by asking how conventional pointer-sized words are passed. It should classify arguments according to whether they are values or authority-bearing relationships and choose D or C machinery accordingly.

This could make authority transfer visible at exactly the place where software abstractions cross boundaries.

## Resource handles

The same reasoning extends beyond C++ memory objects.

A file descriptor, socket handle, device handle, service reference or similar entity is semantically not valuable because of the integer used to represent it. Its value is the authority to perform operations on the represented resource.

On a capability machine these are therefore obvious candidates for actual capabilities rather than indices into privileged software tables.

This may be one of the places where the top-down derivation begins to converge with the original System 250 idea that capabilities protect resources and namespaces rather than merely memory addresses.

## Provisional mapping

| Source-language concept | Natural capability interpretation |
| --- | --- |
| object | bounded storage authority |
| array | bounded storage authority |
| pointer | transferable authority/reference |
| reference | delegated authority |
| `this` | authority to current object |
| `const` reference | possible attenuated authority |
| function pointer | invocation authority |
| virtual function | invocation / Enter capability |
| vtable entry | stored invocation authority |
| function argument | possible authority delegation |
| pointer/reference return | authority delegation |
| `new` | creation/allocation under existing authority |
| class private state | possible protected domain/state authority |
| module/library/service | possible authority domain |
| file/socket/device handle | authority to an external resource |

This table is not intended as a final design. Each row needs to be tested against actual C++ semantics, the PP250 architecture, and the practical compiler implementation.

## Why this changes the LLVM question

Initially we were asking:

> Which LLVM operations should become capabilities?

That question starts too low.

The better sequence is:

```text
C++ semantic relationship
        |
        v
authority interpretation
        |
        v
LLVM representation
        |
        v
can the relationship still be recovered?
        |
        v
PP250 capability implementation
```

For each source-language relationship we now need to establish three things:

1. What authority, if any, does the language relationship naturally represent?
2. What does the C++ frontend turn that relationship into in LLVM IR?
3. After normal LLVM optimisation, is enough information still present for the PP250 backend to recover the authority relationship correctly?

That final question determines where the implementation belongs.

If the relationship remains recoverable from ordinary LLVM IR, capability knowledge can remain below LLVM IR in the PP250 lowering/backend.

If optimisation destroys information required to distinguish authority relationships, we then have evidence that something must be preserved earlier in compilation.

We should not introduce PP250-specific IR merely because it is convenient. We should discover the minimum point at which authority information must become explicit.

## Relationship to CHERI

This also clarifies why the experiment is not simply a CHERI-style compiler port.

CHERI's pointer model makes machine capabilities represent pointers. LLVM therefore needs enough awareness of the unusual pointer representation and external state to avoid transformations that destroy capability semantics.

The PP250 experiment being considered here asks a different question:

> Can ordinary source-language relationships be implemented as authority without making every intermediate LLVM pointer itself the machine capability?

For an array access, for example, we may be able to retain:

```text
C = authority over the whole array
D = computed displacement within it
```

rather than deriving a new capability corresponding to every intermediate pointer value.

Similarly, a virtual call may be recognised not merely as a protected code pointer but as an invocation-authority transition.

Whether this works in practice is precisely what the compiler research must establish.

## Emerging hypothesis

The line of reasoning suggests a broader hypothesis for the top-down PP250 programme:

> A conventional compiler frequently lowers source-language relationships into addresses and relies on software conventions and later protection mechanisms to constrain their use. A capability-native compiler can instead preserve selected relationships as authority and lower that authority directly into hardware capability mechanisms.

The hypothesis is broader than "make pointers safe".

It potentially includes bounded storage authority, authority delegation through calls and returns, attenuation of authority, invocation authority, object and module protection domains, protected private state, and authority to external resources.

The particularly interesting possibility is that a language already contains much of the semantic structure needed to identify these relationships. The compiler's task may therefore be less about adding security metadata and more about **not throwing away authority relationships by prematurely reducing them to numerical addresses**.

## Next experiment

The next step should not be a large compiler implementation.

Take a set of very small C++ programs, one for each candidate relationship: local array access; object member access through `this`; pointer parameter; reference parameter; `const` reference; pointer return; function pointer; virtual function call; `new` object; access to private state.

For each, inspect the Clang-generated LLVM IR before and after representative optimisation.

Then trace:

```text
source semantic relationship
        -> LLVM evidence
        -> surviving LLVM evidence
        -> proposed PP250 C/D representation
        -> required PP250 operation
```

The purpose is not merely to demonstrate that PP250 can execute compiled C++.

The experiment asks whether the source-language authority structure can survive the compiler pipeline sufficiently for a PP250 backend to realise it directly in hardware.


## A layered route to capability-aware compilation

The discussion subsequently exposed an important correction to the earlier framing. It is not necessary to solve the richest source-language-to-authority mapping before capability compilation becomes useful. There is a continuum, ranging from treating an entire program as opaque to preserving rich source semantics such as those available in Swift SIL.

This matters because each level can stand independently and all levels can target the same capability architecture.

### Level 1 — Opaque program or module containment

At the coarsest level, we need know nothing about the internal semantics of the program or module.

Conceptually:

```text
              ENTER capability
                    |
                    v
          +-------------------+
          |   opaque module   |
          |                   |
          |  arbitrary code   |
          |  arbitrary data   |
          |                   |
          +-------------------+
                    |
             only explicitly
             granted capabilities
```

The module is entered with an execution environment containing only the capabilities it has legitimately been granted. We do not care how badly behaved its internal computation may be: provided the architecture is sound, it cannot manufacture authority that is absent from that environment.

The guarantee is therefore:

> The component may do anything permitted by the capabilities supplied to it, but it cannot access authority outside that set.

This requires no source-language knowledge and potentially no understanding of the internal program at all. It demonstrates a fundamental distinction between capability architecture and compiler-derived memory safety: useful authority confinement exists even for opaque code.

### Level 2 — Structural raising from ordinary LLVM IR

The next level uses ordinary LLVM IR without requiring source-language authority metadata.

LLVM still exposes structural facts such as allocations, globals, GEP-derived locations, loads and stores. Those facts may be sufficient to recognise bounded storage objects and accesses derived from them.

```text
alloca / global / allocation
             |
             v
           object
             |
            GEP
             |
             v
       derived access
        |          |
      load       store
```

The compiler need not know that an object was a Swift class, C++ object, Rust value or C structure. It can make only the claim justified by the IR: there is an identifiable storage object and accesses are derived from it.

This allows a PP250 lowering to investigate representing the object by a capability and using ordinary D-register computation for displacement:

```text
C = bounded authority over object
D = computed displacement
```

This is less semantically rich than source-derived authority, but it can still turn ordinary LLVM storage structure into hardware-enforced bounded authority.

### Level 3 — Behavioural and flow analysis of ordinary LLVM IR

There is a possible intermediate level between simple object bounding and source-semantic authority.

LLVM exposes function boundaries, calls, arguments, returns, aliases, captures, read/write behaviour and data flow. Conservative analysis may therefore reveal authority-transfer opportunities even when the original source semantics are no longer known.

For example:

```text
Function A
    |
    | passes object X
    v
Function B
    |
    +-- reads X
    +-- does not retain X
```

This may permit the compiler to realise the call as a temporary delegation of authority to X.

Likewise, if analysis establishes that a function requires only capabilities A, B and C, the execution environment constructed for it may be narrower than that of its caller.

This level must remain conservative. It cannot recreate source semantics that have disappeared. It can only derive authority relationships justified by behaviour actually visible in LLVM IR.

Nevertheless, this moves beyond simple bounds protection toward inferred least-authority execution.

### Level 4 — Preserve semantic authority from a richer IR such as Swift SIL

At the richest end, the compiler acts before source-language semantics have been reduced to ordinary LLVM operations.

Swift is particularly interesting because its existing compiler pipeline contains SIL, a Swift-specific intermediate representation above LLVM IR. SIL retains concepts including ownership, borrowing, access, object relationships, dispatch and lifetime that are largely absent after conventional LLVM lowering.

The experimental path therefore becomes:

```text
Swift semantics
      |
      v
     SIL
      |
      +-- ownership
      +-- borrowing
      +-- object relationships
      +-- access
      +-- invocation / dispatch
      +-- lifetime
      |
      v
authority interpretation
      |
      v
LLVM-preserved authority representation
      |
      v
PP250 lowering
```

This does not imply writing a new compiler. It means investigating the existing SIL-to-LLVM boundary and determining which semantic relationships can usefully be retained as authority.

The same principle could later apply to other frontends if they possess sufficiently rich pre-LLVM representations.

### Passing authority through LLVM

If authority identified above LLVM affects correctness, it cannot simply be placed in disposable descriptive metadata and hoped to survive optimisation.

The working direction is to investigate LLVM's existing semantic extension mechanisms—particularly intrinsics and operand bundles—and determine whether they can preserve a small, language-independent authority vocabulary through optimisation without requiring a new fundamental LLVM type.

Conceptually that vocabulary might eventually express relationships such as:

```text
authority root
delegation
attenuation
access requirement
invocation authority
domain transition
```

These are deliberately authority concepts, not Swift or C++ concepts. Swift SIL might recognise one relationship through ownership or dispatch while another frontend reaches the same authority operation through completely different source semantics.

No final representation has yet been selected. The important design requirement is that authority affecting correctness must survive optimisation with defined semantics.

### The continuum

The resulting research programme is therefore not a binary choice between an unaware compiler and a fully capability-native language:

```text
             increasing semantic knowledge
                       -------------------->

 OPAQUE          STRUCTURAL          FLOW          SEMANTIC
 MODULE            LLVM              LLVM            SIL
   |                 |                 |               |
   v                 v                 v               v
 domain           bounded          inferred        language
 isolation        objects          delegation      authority
   |                 |                 |               |
   +-----------------+--------+--------+---------------+
                              |
                              v
                       SAME CAPABILITY
                         ARCHITECTURE
```

Each level is independently useful.

Failure to recover rich source semantics does not invalidate opaque-domain protection. Failure to infer useful delegation from ordinary LLVM does not invalidate bounded-object raising. Conversely, richer semantic information can progressively produce more precise authority without requiring a different underlying machine.

### Architectural consequence

This gives the modern capability-machine work a strong design criterion:

> **Hardware protects authority; software defines meaning.**

The processor need not know whether a capability arose because a module was sandboxed, LLVM recognised a bounded object, flow analysis inferred delegation, or Swift SIL identified a semantically meaningful object relationship.

An Enter capability protecting an opaque module and an Enter capability produced from rich language semantics are the same kind of architectural authority. What differs is how much knowledge the software/compiler possessed when constructing the authority graph.

This is important because it keeps source-language complexity out of the hardware while allowing progressively richer compiler understanding to exploit the same underlying capability mechanisms.

### Where to resume this investigation

When this work is resumed, the useful next questions are:

1. Define the minimum authority environment required to execute an opaque LLVM module under an Enter capability.
2. Take small ordinary LLVM IR examples and determine exactly which bounded-object relationships can be raised without additional metadata.
3. Determine how far conservative LLVM flow/capture analysis can reduce delegated authority at function boundaries.
4. Take a small Swift example and inspect its actual SIL for ownership, access, object and dispatch relationships that disappear during IRGen.
5. Test which existing LLVM semantic mechanisms can carry those relationships through optimisation without modifying LLVM's fundamental IR model.

The objective is not to commit prematurely to one compiler integration strategy. It is to establish experimentally how much authority can be extracted or preserved at each level and show that all of those levels converge on the same hardware authority model.

## Swift closures as PP250 authority

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

### Captured state

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

### The LLVM information boundary for closures

Arbitrary LLVM IR cannot in general be expected to reconstruct all of the original Swift semantics after those semantics have been lowered away.

If a Swift closure has become something structurally equivalent to a code reference plus an environment pointer, ordinary LLVM IR may preserve enough structure to execute it correctly without preserving the fact that the source construct was specifically a Swift closure. Conversely, optimisation or lowering may erase distinctions that would have allowed a more precise authority mapping.

The compiler problem is therefore not:

> Can arbitrary LLVM IR recreate Swift semantics?

It is:

> **Which authority relationships needed for a PP250 implementation survive Swift -> SIL -> LLVM lowering, and what is the minimum information that must be preserved when they do not?**

This is why Swift SIL is particularly interesting. It provides a point above LLVM IR at which closure capture, ownership, lifetime, access and invocation relationships may still be explicit enough to identify the intended authority before conventional lowering obscures it.

### Capability provenance as a compiler invariant

A PP250 backend must not weaken the architecture merely to accommodate conventional pointer operations appearing in LLVM IR.

A useful working invariant is:

> **Every authority-bearing pointer or reference lowered to PP250 machine code must have a legitimate capability provenance.**

This does not mean that every LLVM pointer operation requires a newly manufactured capability. Ordinary displacement within an already authorised object may remain computation: a PP250 capability supplies authority over the object while ordinary data arithmetic supplies the displacement.

Likewise, an LLVM operation that superficially resembles address arithmetic may be safely lowerable if the compiler can prove that it remains an operation under existing authority.

The critical distinction is between **computing a position within existing authority** and **manufacturing new authority from a number**.

Thus an `inttoptr`, or a `ptrtoint` / integer-arithmetic / `inttoptr` sequence, must not automatically become an arbitrary PP250 capability. If the compiler can establish a legitimate capability provenance and interpret the operation without increasing authority, it may be lowerable. If no such provenance can be established, the operation is not representable by ordinary PP250 capability semantics and should be rejected or require an explicitly defined ABI/runtime mechanism.

The prohibition is therefore not simply "PP250 does not support `inttoptr`". It is:

> **ordinary computation must not create authority that the computation did not already possess.**

### Minimal Swift experiment

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

Swift closures provide a particularly clean test of the broader authority-preservation hypothesis because the source construct visibly combines **invocation** with **captured state and lifetime**. The experiment should test that hypothesis without baking Swift-specific concepts into PP250 hardware.