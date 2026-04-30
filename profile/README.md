# hylic-recursion

`hylic` is a Rust library for catamorphic computation over trees and DAGs. The recursion does four things at each node: `init` builds a per-node heap, `visit` yields each child, each child's result is recursively computed and `accumulate`d into the heap, then `finalize` closes the heap into the node's result. The library packages that pattern as three independent values (a `Fold<N, H, R>`, a `Treeish<N>`, and an executor that drives the recursion) and ships sequential and parallel executors that consume them through a uniform `executor.run(&fold, &graph, &root)` interface.

The parallel executor is `Funnel`, a work-stealing engine bundled in the core crate. Each policy preset is a triple of static types (queue topology × accumulation strategy × wake policy), all generic parameters on `Funnel<P>`, so picking a preset compiles a different specialisation of the entire walk; there is no runtime dispatch on strategy. Continuations are defunctionalised and live in arenas released at end of pool lifetime. On the published 14-workload [Matrix benchmark](https://hylic-recursion.github.io/hylic-docs/cookbook/benchmarks.html#matrix), a `Funnel` variant wins ten rows outright against a handrolled Rayon recursion and a scoped pool, and lands within a few percent of the winner on the rest.

`hylic-pipeline` is a typestate pipeline builder over the same primitives. The same `(grow?, graph, fold, lifts...)` is constructed by chaining methods (`.filter_seeds(...).wrap_init(...).zipmap(...).run(...)`), with the chain's stage and shape enforced at the type level. Both surfaces share the same fold/treeish/executor decomposition; the difference is in how the chain is built.

The long-form material lives at <https://hylic-recursion.github.io/hylic-docs>: a [quick start](https://hylic-recursion.github.io/hylic-docs/quickstart.html), [the recursive pattern](https://hylic-recursion.github.io/hylic-docs/concepts/separation.html), the [Funnel deep-dive](https://hylic-recursion.github.io/hylic-docs/funnel/overview.html), the [interactive benchmark viewer](https://hylic-recursion.github.io/hylic-docs/cookbook/benchmarks.html), and a [cookbook](https://hylic-recursion.github.io/hylic-docs/cookbook/fibonacci.html) of worked examples (filesystem summary, expression evaluation, module resolution, configuration inheritance, cycle detection, parallel execution).

The repositories that ship the code:

- [`hylic`](https://github.com/hylic-recursion/hylic) — core library; published on [crates.io](https://crates.io/crates/hylic).
- [`hylic-pipeline`](https://github.com/hylic-recursion/hylic-pipeline) — typestate pipeline builder; published on [crates.io](https://crates.io/crates/hylic-pipeline).
- [`hylic-benchmark`](https://github.com/hylic-recursion/hylic-benchmark) — Criterion harness behind the published numbers.
- [`hylic-docs`](https://github.com/hylic-recursion/hylic-docs) — mdBook source for the documentation site.
