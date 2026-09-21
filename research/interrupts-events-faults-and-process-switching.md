# Interrupts, events, faults and process switching

> Status: research/design note. It records both the historical interpretation developed so far and the modern design questions that followed from it. The exact PP250 dispatch sequence after SIW/NIB detection still requires documentary confirmation.

## Why this topic arose

A modern PP250-derived machine cannot simply be assessed as a historical recreation. Modern peripherals expect low-latency event handling, while one of the distinctive System 250 choices appears to have been the deliberate avoidance of conventional external device interrupts.

The discussion therefore separated two questions:

1. Why did the original System 250 avoid ordinary peripheral interrupts?
2. Can a modern implementation obtain low event latency without allowing an external device to seize or redirect processor execution state?

## Historical direction established so far

The literature examined so far describes System 250 I/O as memory mapped and identifies a polling/event mechanism rather than conventional peripheral-to-CPU interrupt vectors.

The emerging interpretation is that this was not merely a performance-era implementation accident. It fits the machine's fault-containment and capability philosophy.

A conventional interrupt can allow an external hardware source to cause an asynchronous processor control transfer. In a capability machine, that creates an important architectural question: what authority and execution state does the interrupt handler inherit, and how is the external source prevented from bypassing the capability model?

System 250 appears to have avoided that route for ordinary device activity.

The broad flow reconstructed in discussion is:

    device / I/O activity
          |
          v
    shared protected state / SIW
          |
          v
    processor periodically observes pending work
          |
          v
    protected software machinery
          |
          v
    appropriate driver/process activity

The System Interrupt Word (SIW), Normal Interrupt Block (NIB), interval-timer behaviour and exact process activation sequence need to be tied to primary sources in a later pass.

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

The exact historical relationship among normal interrupt handling, CALL, fault handling and CHP remains to be established from the documentation.

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

## Open historical questions

The following should be answered from original sources before the modern mechanism is specified:

1. What exactly happens after the interval mechanism detects relevant SIW state?
2. How is the appropriate process or handler identified?
3. What role does the NIB play?
4. Does normal interrupt processing invoke CHP, construct process/dump-stack state, perform a CALL-like protected transition, or use another mechanism?
5. Which state is saved, and by what hardware/microcode sequence?
6. What does the external I/O module actually have permission to modify?
7. How are simultaneous/multiple device events represented?
8. How is this coordinated in a multiprocessor configuration?
9. What are the measured/documented costs of CHP and the cheaper protected transitions?

## Modern design question to preserve

Once the historical mechanism is understood, investigate whether a modern PP250-derived processor can replace periodic polling latency with a hardware pending-event observation mechanism **without reintroducing external authority over CPU control flow**.

The goal is not "add interrupts to PP250." It is:

> retain the PP250 fault-containment boundary while allowing modern event latency.

That distinction should remain explicit in subsequent design work.
