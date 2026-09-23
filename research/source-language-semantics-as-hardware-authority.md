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
