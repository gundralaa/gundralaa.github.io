---
layout: post
title: "SMT Algorithms for Temporal Constraint Checking in Safety-Critical Systems"
---

Temporal constraints show up everywhere in safety-critical engineering: a brake command must be acknowledged within a deadline, a watchdog must be serviced often enough, a flight-control mode transition must not happen too quickly, and so on. The tricky part is that these requirements live at the intersection of **logic** (“this must never happen”), **time** (“within 50ms”), and **implementation detail** (“with fixed-width integers, saturating arithmetic, and stateful software”).

Satisfiability Modulo Theories (SMT) is one of the most practical bridges across that intersection. It lets you express system behavior as constraints over domains engineers actually use—bit-vectors, integers, reals, arrays, uninterpreted functions—while still getting push-button automation from modern solvers.

### What “SMT” really buys you

SAT solves propositional formulas. SMT generalizes SAT to richer mathematical domains (“theories”). For safety work, this matters because you usually don’t want to abstract away the things that cause bugs:

* Fixed-width arithmetic (overflow, wraparound) via **bit-vectors**
* Quantities like time and rates via **integers/reals** (LIA/LRA)
* Memory and IO buffers via **arrays**
* Components that are black-boxed but still structured via **uninterpreted functions**

In temporal checking, SMT is often the engine behind:

* **Bounded model checking (BMC)** of temporal properties
* **\(k\)-induction** proofs of invariants (safety properties)
* **Counterexample generation** as concrete traces (great for debugging)
* **Automatic test input generation** from constraints

### The solver side: algorithms you’ll see in practice

The “classic” architecture powering many industrial solvers is often described as **DPLL(T)** (or its modern variant **CDCL(T)**):

* **CDCL SAT core**: searches for a satisfying assignment of Boolean structure, learns clauses from conflicts, backjumps, and restarts.
* **Theory solvers**: check consistency of the SAT core’s tentative assignment with respect to a domain:
  * **Bit-vectors**: bit-blasting to SAT, word-level reasoning, and specialized rewriting
  * **Linear arithmetic** (LIA/LRA): simplex-style methods and cutting planes (solver-dependent)
  * **EUF** (uninterpreted functions): congruence closure
  * **Arrays**: reasoning with select/store axioms and solver-specific heuristics

Two “engineering” features matter a lot for temporal problems:

* **Incremental solving** (`push`/`pop`): BMC adds constraints step-by-step as you extend the time horizon.
* **Unsat cores / assumptions**: useful for traceability (“which requirements/assumptions caused UNSAT?”) and for debugging inconsistent specs.

### Temporal constraint checking: from “over time” to “constraints”

At a high level, checking a temporal requirement usually means building a model of the system’s evolution:

* A **state** vector \(s_t\) at time step \(t\)
* A **transition relation** \(T(s_t, s_{t+1}, i_t)\) describing how the state updates given inputs \(i_t\)
* A **property** \(P(s_t, i_t)\) capturing what must hold

Then you ask: “Is there a trace of length \(k\) that violates the property?”

#### Bounded model checking (the workhorse)

For a bound \(k\), you “unroll” the transition relation:

\[
I(s_0) \wedge \bigwedge_{t=0}^{k-1} T(s_t, s_{t+1}, i_t) \wedge \neg \text{PropertyUpTo}(k)
\]

If the SMT solver says **SAT**, it provides a counterexample trace \(s_0, \dots, s_k\). If it says **UNSAT**, there is no counterexample *within the bound*.

This is often enough in practice because many safety requirements have meaningful finite horizons (deadlines, timeouts, bounded response times), and because counterexamples tend to be short.

#### Temporal requirements you can encode naturally

* **Deadline/response**: “If event A happens, event B happens within N ticks.”
* **Minimum separation**: “Events of type X are at least N ticks apart.”
* **No glitches**: “Once in mode M, stay in M for at least N ticks.”
* **Rate bounds**: “Signal changes by at most Δ per tick.”
* **Scheduling feasibility**: constraints over release times, execution times, and precedence.

### A tiny SMT-LIB sketch: “ack within 3 cycles”

Here’s a small, bounded example: if `cmd` is asserted at time \(t\), then `ack` must be true at least once in \([t, t+3]\). This is the kind of property that appears in bus protocols, actuator handshakes, and watchdog-style supervision.

```lisp
; Bounded horizon k=10, discrete-time steps 0..10
(set-logic QF_BV)

(define-fun within3 ((ack0 Bool) (ack1 Bool) (ack2 Bool) (ack3 Bool)) Bool
  (or ack0 ack1 ack2 ack3))

; cmd_t and ack_t are Boolean signals over time
(declare-fun cmd ( (_ BitVec 4) ) Bool)
(declare-fun ack ( (_ BitVec 4) ) Bool)

; We look for a counterexample at some time t <= 7 (so t+3 is in-bounds)
(declare-const t (_ BitVec 4))
(assert (bvule t #x7))

; Violation: cmd(t) happens, but ack is false for the next 4 steps
(assert (cmd t))
(assert (not (within3 (ack t)
                      (ack (bvadd t #x1))
                      (ack (bvadd t #x2))
                      (ack (bvadd t #x3)))))

(check-sat)
(get-model)
```

Real models add the system dynamics \(T(s_t, s_{t+1}, i_t)\), bounds, type constraints, and usually use incremental solving to grow the horizon without rebuilding everything from scratch.

### Languages and tools industry professionals can actually use

There are two layers here: the **solver interface language** (how you talk to the SMT engine) and the **system modeling language** (how engineers describe the system before it’s encoded).

#### Solver interface languages (practical in industry)

* **SMT-LIB 2**: the standard, tool-agnostic text format supported by major solvers; great for reproducibility and long-term archival.
* **Solver APIs** (common in production tooling):
  * **Python** (e.g., Z3Py, cvc5 Python bindings): prototyping, analysis pipelines, test generation.
  * **C/C++**: embedding in verification tools with tight performance requirements.
  * **Java**: integration with JVM-based toolchains; common in enterprise settings.
  * **.NET/C#**: integration into Windows-heavy engineering environments.

Typical solvers used in practice include **Z3**, **cvc5**, **Yices**, and **Boolector** (especially strong for bit-vectors/arrays). In regulated environments, teams often pin solver versions and keep solver invocation deterministic (fixed seeds, bounded resources, archived queries).

#### System/specification languages (where requirements start)

Depending on the organization and domain, you’ll see:

* **Synchronous languages** (strong fit for control + discrete-time reasoning): Lustre / SCADE-style models.
* **Model-based design**: Simulink / Stateflow models (often compiled into verification backends).
* **Architecture + contracts**: AADL with contract extensions (commonly used to tie architecture to timing/assurance arguments).
* **Formal spec languages**: TLA+, Alloy, or similar (often at the requirements/architecture level).
* **Model checking formalisms**: SMV-family languages, Promela-style process models, and timed-automata tools (useful when timing is the main complexity).

### A modeling paradigm that scales: contract-based, synchronous, and compositional

For temporal safety constraints, a robust modeling paradigm is:

* **Synchronous, discrete-time state machines / dataflow**: the system advances in logical ticks, which makes “within N cycles” properties direct and avoids ambiguities about sampling.
* **Contract-based design** (assume/guarantee):
  * Each component has an **assumption** about its environment and a **guarantee** about its outputs/state evolution.
  * Contracts compose, enabling scalable analysis (and clearer traceability for assurance cases).
* **SMT-powered checking**:
  * Encode contracts and component transitions into SMT.
  * Use BMC for deadline-style properties and \(k\)-induction/invariant checking for “always” safety properties.
  * Use incremental solving and unsat cores to iterate as requirements evolve.

This aligns well with how safety-critical systems are built in the real world: independently-developed subsystems, clear interface expectations, and strong pressure for traceability from requirements → analysis artifacts → evidence.

### Closing thoughts

SMT isn’t “one algorithm”—it’s a set of mature solver techniques plus modeling discipline. The payoff is practical: you can ask precise temporal questions about realistic models (with real numeric types and implementation constraints) and get either a proof (within a method’s limits) or a concrete counterexample trace that engineers can reproduce and fix.
