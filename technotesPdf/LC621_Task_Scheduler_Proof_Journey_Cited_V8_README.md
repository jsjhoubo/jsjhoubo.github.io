# LC 621 Task Scheduler — Proof Journey and Debug Notes (ZH)

**Document:** `LC621_Task_Scheduler_Proof_Journey_Cited_V8.pdf` (14 pages; PDF currently distributed as a ChatGPT attachment).

## Overview

Chinese technical research note documenting the process of deriving the solution to LeetCode 621: Task Scheduler. It deliberately preserves the back-and-forth reasoning, incorrect hypotheses, counterexamples, failures to establish a loop invariant, and the eventual row-based construction idea rather than presenting only the final formula.

## Core topics

1. Incorrect min-heap selection by earliest availability.
2. Highest-frequency skeleton `U_1` and the difficulty of packing other tasks into fixed gaps.
3. Residual capacity `x`, failed subproblem equivalence, and why stitching locally optimal schedules requires proof.
4. The **alignment thought attractor**: repeated attempts to repair fixed positions instead of changing the abstraction.
5. The row-based construction using `n + 1` positions, frequency-based selection, and alignment of repeated tasks across neighboring rows.
6. Tie-breaking to preserve original top-`k` frequency leaders in the relevant cases; treating the earlier replacement/permutation hypothesis as abandoned.
7. Examples, corrections to misread scheduling strings, and explicit separation of example validation from general optimality proof.
8. The standard lower bounds and the final task-time expression.

## References

- [1] [LeetCode 621 — Task Scheduler (original problem)](https://leetcode.com/problems/task-scheduler/)
- [2] [NeetCode — Task Scheduler (Greedy and Math)](https://neetcode.io/solutions/task-scheduler)
- [3] [Doocs LeetCode Wiki — Task Scheduler](https://leetcode.doocs.org/en/lc/621/)

**Provenance.** The original proof exploration, variable names (`U_1`, `x`, `k`), alignment issue, construction proposal, review errors, and revised invariants are primary discussion material from Bo Hou and ChatGPT. The three public links above support the external problem statement and published standard solution, not the claim that every intermediate hypothesis in the discussion is a theorem.

**PDF availability:** The cited PDF is generated as a conversation attachment; the PDF binary has **not** been uploaded to this GitHub repository. This page is the corresponding public citation and document index.
