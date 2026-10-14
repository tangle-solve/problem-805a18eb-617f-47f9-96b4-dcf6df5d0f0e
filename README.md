<!-- solve-problem:805a18eb-617f-47f9-96b4-dcf6df5d0f0e -->
# Tenfold faster exact maximum flow on a frozen diverse corpus

A [Tangle Solve problem](https://solve.tangle.tools/problems/805a18eb-617f-47f9-96b4-dcf6df5d0f0e).

## What is the problem?

This is an **engineering moonshot in experimental algorithms**, not an asymptotic open problem. The task: build a maximum-flow solver (exact, i.e., provably optimal, no approximation) that achieves a **geometric-mean end-to-end speedup of at least 10×** over a declared baseline on a **preregistered, diverse corpus of instances**, where:

- **End-to-end time** includes everything: input parsing, graph construction, the flow computation itself, and verification of the optimality certificate (e.g., checking the accompanying min-cut: flow value equals cut capacity, conservation and capacity constraints hold). Timing only the kernel is not allowed.
- **Baseline** must be fixed in advance — most naturally, a strong production-quality exact solver such as a well-tuned push–relabel (Goldberg–Tarjan HIPR), Boost Graph Library `boykov_kolmogorov`/`push_relabel`, or LEMON's solvers, run with comparable exactness guarantees.
- **Corpus** must be preregistered (committed before development tuning) and diverse: random graphs, grid/vision instances, DIMACS-style hard families, real-world road/network/logistics graphs, degenerate and highly parallel graphs, tiny and huge instances.
- **Constraints:** exactness preserved (verified certificate, not just matching baseline objective values), and a stated memory budget not exceeded (e.g., comparable to baseline's, or a fixed multiple declared up front).

Formally: let T_new(I) and T_base(I) be end-to-end wall-clock times per instance I in corpus C. Success ⇔ (∏_I T_new/T_base)^(1/|C|) ≤ 0.1, with all certificates verified and memory ≤ budget.

Ambiguity to resolve in your write-up: how much parallelism (multi-core/GPU) counts, and whether the baseline may also be parallelized. State your convention explicitly.

## Why does it matter?

Maximum flow is a workhorse subroutine in vision (segmentation), logistics, scheduling, network design, and as an inner loop in min-cost flow and LP-based systems. The theory frontier (almost-linear time) has essentially no deployed implementations, so practitioners see none of the progress. A verified 10× on real, diverse workloads would (a) prove the theory-to-practice gap is closable, (b) produce a reusable benchmarking protocol (preregistered corpora, certificate-checked end-to-end timing) for the whole experimental-algorithms community, and (c) directly cut compute costs in every downstream application.

## What's already known?

- **Asymptotics are essentially solved in theory**: Chen et al. (2022) give almost-linear time (m^(1+o(1))) for max-flow/min-cost flow via interior-point methods with low-diameter graph representations (recursive routing structures). No public implementation exists; the constants and data-structure overheads are believed enormous.
- **Practical state of the art**: Goldberg–Tarjan push–relabel with heuristics (gap relabeling, global relabeling — the "HIPR" code) dominates in practice on CPU. BK-max-flow excels on vision instances. LEMON and Boost ship tuned versions.
- **Known engineering wins**: parallel push–relabel, GPU flow solvers report large speedups but usually on narrow instance families and often without certified exactness or end-to-end accounting.
- **Why prior attempts fall short**: speedups measured on cherry-picked corpora, kernel-only timing, no certificate verification, and baselines that are untuned. This is precisely what the preregistration + end-to-end + exactness protocol is designed to prevent.

## What would count as solving it?

1. **Primary deliverable**: a released solver + a preregistration document (corpus list, baseline binary/version/flags, memory budget, hardware, timing methodology committed and timestamped before tuning), and a measured geometric-mean end-to-end speedup ≥ 10× over the baseline on that corpus, with every instance's optimality certificate independently verified.
2. **Negative-result variant**: a rigorous demonstration that no combination of known techniques (parallel push–relabel, GPU, better heuristics) exceeds, say, 2–3× geometric mean across a diverse corpus — a well-instrumented "why it fails" study would also be a contribution.
3. **Protocol artifact**: the benchmark corpus + verification harness itself, usable by others.
A referee would require: reproducible runs, third-party verification scripts, and no post-hoc corpus changes.

## How to start

1. **Read Chen et al. (2022) Sections 1–3** only to understand what you're *not* implementing; then read Goldberg's HIPR documentation and the DIMACS challenge results to pick and freeze your baseline.
2. **Build the harness first**: corpus selection (aim for ≥ 5 families, spanning sparse/dense, tiny/huge, structured/random), certificate checker (independent of the solver), timing wrapper, preregistration write-up. This is 30–40% of the project and makes results credible.
3. **Profile the baseline**: on diverse corpora, much of the end-to-end time is often parsing/setup and output, not the flow kernel — meaning naive "faster algorithm" work can't yield 10×. Fast parsing, integer arithmetic choices, and certificate generation cost are first-order.
4. **Candidate techniques**: parallel/blocking push–relabel, SIMD/GPU kernels, better initial distance labels, warm starts, and aggressive memory-layout work. Main obstacle: geometric mean punishes any instance family where you're slow; vision-style instances and degenerate graphs behave very differently.

## Key references

- Chen, Kyng, Liu, Peng, Gutenberg, Sachdeva, "Maximum Flow and Minimum-Cost Flow in Almost-Linear Time," arXiv:2203.00671 (2022). Proves m^(1+o(1))-time exact min-cost flow via an interior-point method with dynamic low-diameter graph decompositions; no implementation exists. https://arxiv.org/abs/2203.00671
- Goldberg, A.V. and Tarjan, R.E., "A New Approach to the Maximum-Flow Problem," JACM 35(4), 1988. Introduces push–relabel, still the practical gold standard; basis for the HIPR benchmark code.
- Goldberg's HIPR implementation and DIMACS Implementation Challenge results (Network Flows area). Empirical data on which algorithms win on which instance families; use for baseline choice. (verify current hosting: dimacs.rutgers.edu / igsys.com)
- Boykov & Kolmogorov, "An Experimental Comparison of Min-Cut/Max-Flow Algorithms for Energy Minimization in Vision," PAMI 26(9), 2004. Shows instance-family-dependent winners — direct evidence a diverse corpus is essential. (verify)

## Warnings

- **The 10× must be geometric mean**: a 100× on one family and 1× on another gives ~10× on two instances but fails on diverse corpora. Any result measured on a narrow family is not a solution.
- **End-to-end is the trap**: parsing can be 20–50% of runtime on small/medium instances; certificate verification is often ignored entirely in prior GPU-flow papers.
- **Don't reimplement Chen et al. expecting practical wins** — the o(1) terms hide enormous constants; that path is likely a multi-year detour for this goal.
- **Fix the parallelism convention before starting**, or your result is unfalsifiable ("was it the algorithm or the 64 cores?").
- **Preregistration discipline**: if you tune against the corpus, you've invalidated the claim. Commit the corpus hash before optimization work.

---
*`ALGO-000215` · flagged for review*

## Acceptance guidance

1. **Primary deliverable**: a released solver + a preregistration document (corpus list, baseline binary/version/flags, memory budget, hardware, timing methodology committed and timestamped before tuning), and a measured geometric-mean end-to-end speedup ≥ 10× over the baseline on that corpus, with every instance's optimality certificate independently verified.
2. **Negative-result variant**: a rigorous demonstration that no combination of known techniques (parallel push–relabel, GPU, better heuristics) exceeds, say, 2–3× geometric mean across a diverse corpus — a well-instrumented "why it fails" study would also be a contribution.
3. **Protocol artifact**: the benchmark corpus + verification harness itself, usable by others.
A referee would require: reproducible runs, third-party verification scripts, and no post-hoc corpus changes.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/805a18eb-617f-47f9-96b4-dcf6df5d0f0e?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: ALGO-000215.

- [Public source catalog](https://drewstone.github.io/problems/#/p/ALGO-000215)
