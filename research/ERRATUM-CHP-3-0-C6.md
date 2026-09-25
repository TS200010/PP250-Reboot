# Erratum: `CHP 3 0 C6` is not evidence

Status: correction, 25 September 2026.

The sequence `CHP 3 0 C6` appeared in `research/pp250-execution-and-process-model.md` as a user-supplied example and was subsequently used there to motivate a working interpretation of CHP.

That recollection is now explicitly withdrawn as unreliable. It must **not** be treated as evidence for the PP250 instruction syntax, CHP operand semantics, the role of C6, offset 3, or the relationship between a Process Base and a Dump Stack.

Accordingly, the following propositions in the existing note are withdrawn where they depend on that recollection:

- that `C6 + 3` is the normal CHP route to an incoming Dump Stack;
- that C6 should be used as the capability field for CHP on the strength of this example;
- any inference from the literal values `3` or `0` in that recalled instruction.

The surviving evidence is independent of that recollection: the Pocket Reference documents both store-mode and direct-mode CHP and marks CHP's register field as unused; the processor self-test paper describes CHP as dumping the current register context and reloading it from an indicated block in store; and the separate patent-derived material concerning CHANGE PROCESS must be evaluated on its own provenance.

Until the original note is textually revised, this erratum takes precedence over its `CHP 3 0 C6` passage and any conclusions derived specifically from that example.
