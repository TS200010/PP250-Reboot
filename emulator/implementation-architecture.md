# PP250 Emulator Implementation Architecture

## Purpose

This document records the implementation direction for the executable PP250 reconstruction.

The implementation begins as a faithful bare PP250 machine emulator. It is deliberately structured so that it can later grow into a wider System 250 simulation environment capable of hosting newly written PP250 system software and, where surviving evidence permits, reconstructed historical operating systems such as COS, POS, PDOS and ROS.

The layers must remain distinct. Operating-system policy must not be built into the emulated hardware.

## 1. PP250 machine

The first implementation target is the bare machine.

The processor model should contain the complete PP250 processor state required by the reconstructed machine, including state that is accessible only internally by the microcode and is not directly exposed by machine opcodes. There is no need to classify that state separately before implementation: the machine instructions themselves determine which portions of the state they expose or manipulate.

The preferred implementation is now a **clocked hardware-state simulation**, not an instruction-by-instruction interpreter. The processor's current clocked state and incoming condition signals determine the next state through a reconstructed combinational control network. Instructions such as CHP emerge from that network's microprogram sequences rather than from special Swift handlers. Historical operating-system policy is not embedded in the machine.

The initial implementation should remain close to the reconstructed hardware rather than introducing unnecessary modern virtual-machine abstractions.

### Clocked microprogram simulation — current design direction (23 October 2026)

**Documented:** The processor self-test paper describes a synchronous PP250 with a 300 ns basic clock period, typical instruction time 0.9 μs, and a control-unit matrix of 83 microinstructions (slots) by 150 control signals. It explains that results are captured at the end of the clock pulse and that conditional inputs for a succeeding slot are established using the appropriate preceding state. The processor diagrams identify incoming conditional groups including SOHCS (store response), AUCS, CIS, ICS and FICS. The 150 columns are microprogram *control signals*, not a demonstrated count of external input lines or of individual physical gates.

**Implementation direction:** A tight, non-suspending Swift loop invokes `evaluateLogic()` once per simulated processor clock. Within that one invocation, successive layers of combinational logic settle against the current clocked state; the resulting register, indicator and microprogram-sequencer state is then committed together for the next invocation. No separate instruction dispatcher, per-instruction CHP routine, clock function or compulsory cycle counter is required in the outer loop:

```swift
while running {
    evaluateLogic()
}
```

Pack control-bit patterns in machine words: the full 150-bit microprogram row occupies three `UInt64` values (83 rows = 1,992 bytes before container overhead), while any verified subset of at most 64 independently controlled gate enables can fit in one. Gate-count estimates of about 30–40 remain **unverified working estimates** and must not be confused with the 150 control-signal count. Swift/LLVM can evaluate many Boolean conditions with word-wide AND/OR/XOR operations, but correct dependencies still require successive combinational stages and datapath/register work. Real-time execution at 3.33 million simulated cycles per second on a circa-2020 MacBook Pro is a **benchmark target, not an established result**.

**Store interaction:** The processor remains a clocked state machine. Its outgoing store request and incoming SOHCS status participate in the same control network; while a required response is absent, processor clock cycles are consumed without advancing past the dependent operation. A separate store access unit can arbitrate and handshake independently of the processor clock, but it does not asynchronously execute or resume processor instructions. The exact historical next-slot/hold circuitry is not yet recovered, so the detailed waiting-state implementation remains a reconstruction question. Store modules and the interconnect retain their own simulated hardware state, without assumed instantaneous read/write semantics.

**Boundary:** This is a hardware-oriented implementation hypothesis constrained by documented timing and interfaces, not a claim that all 83 slot contents, 150 microbit assignments or internal Boolean equations have been recovered. Preserve historical gaps rather than inserting generic CPU semantics.

## 2. Component model

The emulator should model the PP250 as a collection of hardware components rather than as a single processor object containing an abstract memory.

Initial components include:

- PP250 processor instances;
- PP250 memory-module instances; and
- the shared system bus.

The design must permit multiple processors and multiple memory modules to be attached to the same bus. A memory module should be represented as a physical machine component; the historically small module sizes are useful reminders that this is a compact machine by modern standards rather than a large virtual address-space abstraction.

Peripherals can be added later as further bus-connected components where the reconstructed architecture requires them.

## 3. Bus

Processors and memory communicate through the PP250 bus rather than by direct processor-to-memory calls.

The bus is capability/data agnostic. It transports the machine's bus transactions; it does not decide whether transferred information represents ordinary data or capability information and must not acquire separate invented `data` and `capability` transaction semantics.

The capability/data distinction and the rules governing legitimate manipulation of capabilities belong in the processor mechanisms reconstructed from the PP250 architecture, historically implemented beneath the ordinary process interface by microcode. A process running on the machine must not be able to evade those rules by choosing a different software interpretation of the transferred bits.

The bus model may eventually need to reproduce addressing, arbitration, timing or other documented System 250 behaviour, but these details should be introduced from evidence rather than assumed from a modern bus architecture.

## 4. Assembler

A native PP250 assembler is an early implementation component, separate from the processor.

Its purpose is to translate actual PP250 assembly language into the machine instruction representation executed by the emulator. It should be developed from the reconstructed instruction formats and encodings rather than from a generic assembler framework that imposes foreign assumptions on the machine.

The assembler also provides an important consistency test: an instruction-set reconstruction that cannot be represented accurately by the assembler is probably incomplete or misunderstood.

The current implementation direction is therefore:

1. model complete processor state and the evidenced microprogram control/condition interface;
2. implement clocked state transitions and reconstruct the microprogram slot-selection network;
3. connect processor, bus and store modules through their documented control and response signals;
4. develop the native assembler and test bare-machine instruction behaviour as a consequence of the hardware network; and
5. extend the executable baseline with software environments and toolchain facilities when justified.

The assembler may be developed alongside processor reconstruction. **Instruction-level emulation is not the preferred approach at present**; it remains a possible reference/testing technique, not the architecture of the faithful simulator.

## 5. Bare machine versus operating system

The emulator does not initially attempt to reproduce COS, POS, PDOS, ROS, or any other historical PP250/System 250 operating system.

This distinction is especially important for process mechanisms. The processor must implement the documented semantics of mechanisms such as CHP and Dump-Stack state. It does not implement a scheduler merely because an operating system may use those mechanisms for scheduling.

A Dump Stack is process state from which a process can be resumed. How software decides when to suspend or resume a process, how runnable processes are selected, and how historical operating systems allocate processors are software-policy questions unless evidence establishes a hardware mechanism.

Thus the initial boundary is:

```text
PP250 machine
    processor(s)
    memory module(s)
    bus
    documented machine mechanisms
```

not:

```text
PP250 machine + assumed historical scheduler or operating system
```

## 6. Growth into a PP250 system

The bare machine is only the first layer. The implementation should be capable of expanding into a PP250/System 250 environment by adding documented hardware components and supporting software.

A useful conceptual layering is:

```text
PP250 MACHINE
    processor(s)
    complete processor state
    instruction execution
    memory module(s)
    shared bus
          |
          v
PP250 SYSTEM
    configured machine
    peripherals
    I/O and other documented hardware
    assembler/loading/debug facilities
          |
          v
PP250 SOFTWARE ENVIRONMENTS
    diagnostics and test programs
    prototype operating system
    experimental system software
    reconstructed historical software where evidence permits
        COS
        POS
        PDOS
        ROS
```

Each lower layer should remain usable independently of the layers above it.

## 7. Future operating systems

Once the bare machine is sufficiently complete, operating systems can be written in native PP250 instructions and run on it.

This may begin with a deliberately small prototype operating system whose purpose is to exercise process creation, process changing, protection, resource management and other machine facilities. It may later extend to attempts to reconstruct COS, POS, PDOS or ROS if sufficient historical material survives.

Historical operating systems should run *on* the reconstructed machine. They should not be simulated by special cases inside it.

If a reconstructed historical operating system exposes a discrepancy in the machine model, the discrepancy should be investigated against the evidence. The emulator should not be altered merely to make historical software appear to work.

## 8. User interface

The hardware-component model also provides a natural basis for optional graphical inspection and control.

Individual components may acquire views appropriate to them: for example, a processor view could expose its state and provide Reset, Step and Run controls; memory modules could have inspection views; and later bus or peripheral views could expose useful diagnostic information.

The GUI is not part of the PP250 architecture. It observes and controls emulator components through clean interfaces and must not become the location where machine semantics are implemented.

## 9. Design principle

The implementation should preserve the elegance of the original separation of concerns.

The PP250 capability rules are machine-enforced rules, not conventions that cooperating software is expected to follow. The bus need not understand the semantic distinction between data and capability information, and operating-system code need not be trusted to enforce the fundamental capability/data rules on behalf of the processor.

The emulator should reproduce that property wherever the surviving evidence allows: ordinary PP250 software should be constrained by the same machine mechanisms regardless of whether the software is a tiny bare-metal test, a newly written operating system, or a reconstruction of historical System 250 software.
