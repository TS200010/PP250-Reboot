# CORAL 250 — capability-based procedure profiling

## Status and provenance

**Source:** first-hand recollection of Anthony John Stanners, 26 September 2026.

This note records a recollection from the CORAL 250 compiler project. It is not yet corroborated by surviving contemporary documentation. The implementation details should therefore be preserved as first-hand evidence rather than silently promoted to documented architectural fact.

## Ian David Cottam's compiler metrication work

Anthony Stanners recalls that Ian David Cottam worked on a **metrication/profiling system for the PP250 CORAL compiler**, intended to measure where the compiler spent its execution time so that its performance could be improved.

Stanners recalls that the CORAL 250 compiler maintained a capability-based list of its procedures, using **Enter Data (ED) capabilities**. Cottam duplicated this structure and modified the duplicate so that procedure invocation was redirected through a short instrumentation path before reaching the original procedure. This allowed the time spent in individual compiler procedures to be measured, providing profiling information for compiler performance improvement.

Stanners recalls the mechanism clearly but does not now remember which capability register held or referenced the procedure list. It should therefore **not be identified as C5, C6, or any other specific register without corroborating documentation**.

## Architectural significance

This is a concrete example of the practical expressive power of the System 250 capability architecture. The capability structure was not merely protecting memory: it also represented part of the callable structure of a substantial program. By duplicating and interposing on that capability-based procedure structure, instrumentation could be introduced systematically without rewriting every compiler procedure or every call site.

In modern terminology, this resembles an **instrumented dispatch table or interposition layer**, but with the dispatch structure represented through System 250 capabilities rather than ordinary forgeable procedure pointers.

The example is particularly relevant to the PP250 Reboot project's top-down investigation of capabilities as program and object structure. It provides a remembered historical instance in which changing capability-level procedure binding changed the behaviour of the whole compiler in a controlled way while preserving the underlying procedures.

## Points to corroborate

If contemporary CORAL 250 compiler material is recovered, look specifically for:

- references to **metrication**, measurement, timing, profiling or performance analysis;
- the compiler's procedure capability list and its exact architectural representation;
- use of **Enter Data (ED)** capabilities for the procedure entries;
- duplicated or alternative procedure capability structures;
- a redirection, wrapper or instrumentation routine inserted before procedure entry;
- timing counters or the mechanism used to accumulate time per procedure; and
- any reports showing compiler performance measurements or optimisations derived from them.

The precise capability register designation and call sequence should remain unresolved until supported by documentation; this note deliberately records the remembered mechanism without inventing those details.
