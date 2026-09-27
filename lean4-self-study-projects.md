# Lean 4 Learning Projects

## Context for the tutor

The learner is an experienced software engineer with some Haskell and OCaml, a background in knowledge representation (Cyc, truth maintenance systems), and experience teaching SQL and data modeling. The goal is to learn Lean 4 through projects that use its prover and its general-purpose programming infrastructure together, with emphasis on what Lean does that Haskell and OCaml don't.

Adjustments to watch for coming from Haskell/OCaml:

- Lean is strict, not lazy.
- Typeclass resolution (`outParam`, default instances, instance priorities) behaves differently from Haskell.
- Termination is checked, not assumed. `partial` is an escape hatch that costs the ability to reason about the function.
- `do` notation supports `mut`, `for`, early `return`, and nested actions, so imperative-looking code is idiomatic.
- Metaprogramming (custom syntax, macros, elaborators) is a first-class tool, not an exotic extension.

References, with the abbreviations used in the reading lists below:

- **FPIL**: *Functional Programming in Lean* (Christiansen), the programming side. Chapters 1–9 plus two interludes.
- **TPIL**: *Theorem Proving in Lean 4* (Avigad, de Moura, Kong, Ullrich), the logic and tactics. Chapters 1–12.
- **MPIL**: *Metaprogramming in Lean 4* (Paulino et al.), needed from Project 2 onward. Its chapter dependencies are: Expressions → MetaM → Elaboration, Syntax → Macros, Syntax + MetaM → Elaboration → Embedding DSLs.

Each project has a **Required reading** list, meant to be read before or alongside the milestones it supports. Readings aren't repeated in later projects, so the lists are cumulative.

Suggested order: Project 1 as a warm-up (about a week), then Project 2 as the main project. Projects 3 and 4 are optional follow-ons.

---

## Project 1: Verified computus

**Idea.** Implement the Easter date calculation two ways and prove the two agree:

1. A direct transcription of the tabular definition: golden number, epact, paschal full moon, next Sunday.
2. A compact arithmetic algorithm: Gauss, or the anonymous Gregorian algorithm popularised by Meeus.

**Why it's a good first project.** The correctness question is real but small. The Julian computus has a 532-year period. Once periodicity is proven, the remaining cases can be closed with `decide`. The Gregorian cycle is 5,700,000 years, which is too long for brute force, so that case needs real reasoning with `omega` and modular arithmetic lemmas.

**Milestones.**

1. A `lake` project with a CLI: take a year and a calendar (Julian or Gregorian) as arguments and print Easter Sunday.
2. Implement both algorithms for the Julian calendar and prove periodicity.
3. Close the Julian equivalence with `decide`. Compare with `native_decide` and discuss what extra trust `native_decide` requires (it trusts the compiler).
4. Gregorian equivalence by proof rather than enumeration.

**Required reading.**

- FPIL ch. 1 *Getting to Know Lean* (skim; much will be familiar from Haskell and OCaml)
- FPIL ch. 2 *Hello, World!*: `IO`, `lake`, and the `cat` worked example. This is the model for milestone 1.
- FPIL *Interlude: Propositions, Proofs, and Indexing*: the first contact with propositions as types
- FPIL ch. 6 *Monad Transformers*: read for the extended `do` features (mutable variables, loops, early return) rather than for transformers as such
- TPIL ch. 2–5: *Dependent Type Theory*, *Propositions and Proofs*, *Quantifiers and Equality*, *Tactics*. This is the core proof vocabulary for milestones 2–4.
- TPIL ch. 12 *Axioms and Computation*: background for the `decide` vs `native_decide` discussion in milestone 3, in particular what the kernel will and won't evaluate

**Lean features exercised.** `IO` and `do` notation, `lake`, `Decidable` and `decide`, `omega`, structuring proofs with lemmas about `%` and `/`.

---

## Project 2: Typed relational algebra engine (recommended main project)

**Idea.** A small relational engine in which relations are indexed by their schema, so ill-formed queries are type errors.

**Milestones.**

1. Represent schemas as type-level data (for example, lists of attribute name and type pairs) and relations as collections of schema-conforming tuples.
2. Implement selection, projection, rename, natural join, union, and difference, typed so that projecting onto a missing attribute or unioning incompatible schemas doesn't typecheck.
3. Give the algebra a denotational semantics.
4. Write optimizer rewrites (selection pushdown, projection elimination, join commutativity and associativity) and prove each one preserves the semantics.
5. Use syntax extension to embed a SQL-like surface syntax that elaborates into the typed algebra.
6. Load CSV files, run queries, and print results.

**Why.** This uses all three of Lean's distinctive strengths at once: dependent types that do real work, proofs about a transformation that matters in practice, and metaprogramming. The idea that "a query rewrite is a theorem" is also directly reusable in teaching SQL.

**Lean features exercised.** Dependent types and type-level computation, `Decidable` equality on schemas, inductive semantics, `simp` and rewriting proofs, `syntax`, `macro_rules`, and elaborators, file IO.

**Required reading.**

- FPIL ch. 3 *Overloading and Type Classes*: needed for equality on schema-dependent values
- FPIL ch. 7 *Programming with Dependent Types*, all of it. Section 7.3 *Worked Example: Typed Queries* encodes a subset of relational algebra with indexed families and is the direct starting point for milestones 1–2. The *Pitfalls* section is worth reading before choosing a representation.
- FPIL *Interlude: Tactics, Induction, and Proofs*
- TPIL ch. 7 *Inductive Types* and ch. 8 *Induction and Recursion*: needed for the semantics and the rewrite proofs in milestones 3–4
- TPIL ch. 11 *The Conversion Tactic Mode*: rewriting inside subterms, which optimizer proofs need constantly
- MPIL *Syntax* and *Macros*: the surface syntax in milestone 5
- MPIL *Expressions*, *MetaM* (skim), *Elaboration*, and *Embedding DSLs By Elaboration*: needed if the SQL syntax has to be elaborated with schema information and not merely macro-expanded

**Tutor note.** FPIL 7.3 already solves a simplified version of milestones 1–2. It uses linked lists, a three-type universe, and operators that only loosely match SQL. Have the learner work through it first, then treat the project as a critique and extension of that encoding rather than a from-scratch design. The schema representation chosen in step 1 drives how hard steps 2 to 4 are. Consider spending a session on alternatives before committing, for example lists versus finite maps, and name-indexed versus position-indexed attributes.

---

## Project 3: Datalog evaluator, then a JTMS

**Idea.**

1. Write a naive Datalog evaluator, then a semi-naive one.
2. Prove that semi-naive evaluation computes the same least model as naive evaluation.
3. Extend it into a justification-based truth maintenance system, with well-founded support given as an inductive definition.
4. Prove that the labelling algorithm only labels a node IN if it has a non-circular justification.

**Why.** Termination is the teaching point. Evaluation either gets a well-founded measure (a finite Herbrand base with monotone growth) or falls back to `partial` and becomes unprovable. The JTMS extension turns an invariant usually left as a comment into a theorem.

**Required reading.**

- FPIL ch. 8 *Programming, Proving, and Performance*, sections 8.1–8.4: *Tail Recursion*, *Proving Equivalence*, *Arrays and Termination*, *More Inequalities*. Section 8.2 is the template for proving semi-naive evaluation equivalent to naive evaluation. Sections 8.3–8.4 cover `termination_by` and decreasing measures. The chapter summary gives a practical workflow: start with `partial`, then replace it with a termination proof.
- TPIL ch. 8 *Induction and Recursion*, reread with attention to well-founded recursion
- TPIL ch. 7 *Inductive Types*, reread with attention to inductive families. Well-founded support in the JTMS is an inductive predicate.

**Lean features exercised.** Well-founded recursion and `termination_by`, least fixpoints, inductive predicates, `Finset` and `HashSet`.

---

## Project 4: Untrusted solver, verified checker

**Idea.**

1. Write a CDCL SAT solver as ordinary, performance-oriented, unverified Lean code, using arrays, in-place updates, and heuristics.
2. Have it emit LRAT (or DRAT) certificates.
3. Write a certificate checker and verify it.

**Why.** Search cleverly, check with proofs. This is the architecture most large-scale verification uses, including Lean's own `bv_decide`. The project shows where proof effort pays off and where it doesn't.

**Required reading.**

- FPIL ch. 8, sections 8.5–8.7: *Bounded Numbers*, *Insertion Sort and Array Mutation*, *Special Types*. Section 8.6 explains reference counting, in-place mutation, and `dbgTraceIfShared` for checking that an array really is updated in place. It also stresses benchmarking a compiled `lake` executable rather than code run in the editor. This is the core reading for the solver half.
- FPIL ch. 4–6 on monads and monad transformers, for structuring solver state
- The three books don't cover LRAT or DRAT certificates. The tutor should supply a short description of the format before milestone 2.

**Lean features exercised.** Reference-counted in-place updates ("functional but in place"), `Array` performance idioms, profiling compiled Lean code, soundness proofs over a clause database.
