# Interrupts, events, faults and process switching

> Status: research/design note. The historical PP250 interrupt and dispatch mechanism is reconstructed in [pp250-normal-interrupt-and-system-dispatch.md](pp250-normal-interrupt-and-system-dispatch.md). This note preserves the distinct modern architecture/design questions that follow from that reconstruction.

## Why this topic arose

A modern PP250-derived machine cannot simply be assessed as a historical recreation. Modern peripherals expect low-latency event handling, while one of the distinctive System 250 choices appears to have been the deliberate avoidance of conventional external device interrupts.

The discussion therefore separated two questions:

1. Why did the original System 250 avoid ordinary peripheral interrupts?
2. Can a modern implementation obtain low event latency without allowing an external device to seize or redirect processor execution state?

## Historical basis

The historical PP250 mechanism is maintained in [pp250-normal-interrupt-and-system-dispatch.md](pp250-normal-interrupt-and-system-dispatch.md). That reconstruction is the source of truth for the original SIW, NIB, `C(N)`, D15, Program Trap and automatic-`CHP` mechanisms.

This note does not duplicate that reconstruction. Its concern is the architectural question that follows from it: whether a modern PP250-derived machine can retain the original authority boundary while providing event latency appropriate to modern peripherals.

## Three meanings of "interrupt" must not be conflated

The literature uses related terminology for mechanisms that are architecturally different.

### 1. External peripheral/processor interrupt

The conventional model in which a device or another processor asynchronously causes the current CPU to vector into a handler.

This is the mechanism System 250 appears deliberately to have avoided for ordinary I/O.

### 2. Internally generated processor event

Faults, timer expiration and related internally recognised conditions are different. The processor itself detects the condition and enters a defined architectural sequence.

### 3. Protected/program interrupt machinery

System 250 documentation also describes protected interrupt arrangements involving capability/access machinery and the NIB. This must not automatically be equated with a conventional external IRQ line.

Keeping these three concepts separate is essential when reconstructing the machine.

## The modern question: can an event behave more like a fault?

A possible modern direction arose from the historical distinction.

An external device could be permitted to alter only a capability-authorised piece of event state:

    DEVICE
       |
       v
    protected event bit / queue
       |
       v
    CPU notices pending event
       |
       v
    internally generated architectural event
       |
       v
    protected dispatch machinery

The critical idea is that the peripheral would **not** supply an execution address, C6/C7, arbitrary capability state, or a processor context.

A useful design invariant expressed during the discussion was:

> External hardware may change capability-authorised system state, but only the processor may convert that state into a change of execution.

This could be regarded as accelerated/event-driven observation of shared state rather than restoring a conventional device-controlled interrupt vector.

This is a modern design hypothesis, not a claim about the original PP250.

## Event arrival must not imply CHP

This led directly to the observation that CHP was remembered as an expensive instruction.

That matters because low-latency event notification should not automatically require a complete process switch.

The architectural concepts should be separated:

    event becomes pending
        !=
    process is changed

A device may generate many events while the currently running process continues. The event mechanism can record pending work. Scheduling policy can then decide whether another process should run.

Only if a different process is actually selected should a heavyweight process-state transition such as CHP become relevant.

This suggests three qualitatively different costs:

    ordinary execution

    protected event recognition / fault-like entry

    full CHP process change

For the historical PP250, normal interrupt entry is now reconstructed as an automatic `CHP` through the `C(N)`/NIB mechanism; the detailed evidence and epistemic status belong in the historical reconstruction rather than here. The modern design question is separate: event arrival need not itself force an immediate process change.

## Why this matters for fault containment

A faulty device may be able to set its permitted event state repeatedly. That is a denial-of-service/rate-control problem.

It should not thereby acquire the ability to:

* manufacture capabilities;
* alter arbitrary C registers;
* choose C6 or C7;
* provide an unchecked handler address;
* access arbitrary physical memory;
* inject authority into another process.

That is the important architectural distinction between "device can request attention" and "device can redirect the CPU."

## Comparison with CHERIoT

CHERIoT retains conventional hardware interrupt mechanisms but its software architecture recognises that directly invoking arbitrary application ISRs would undermine compartment isolation. Interrupt handling is therefore mediated by trusted RTOS machinery, with events delivered to threads rather than simply allowing an arbitrary handler to inherit interrupted state.

System 250 appears to have attacked the problem from the other direction: do not give ordinary external peripherals the asynchronous processor-control mechanism in the first place.

This makes interrupt/event handling one of the most valuable PP250-versus-CHERIoT comparison points.

## Modern design question to preserve

Once the historical mechanism is understood, investigate whether a modern PP250-derived processor can replace periodic polling latency with a hardware pending-event observation mechanism **without reintroducing external authority over CPU control flow**.

The goal is not "add interrupts to PP250." It is:

> retain the PP250 fault-containment boundary while allowing modern event latency.

That distinction should remain explicit in subsequent design work.
