---
layout: post
title: "LeetCode 1665: Deriving the Greedy Strategy from First Principles"
date: 2026-05-14
categories: [LeetCode, Greedy, Algorithms]
tags: [leetcode, greedy, sorting, proof, derivation]
---

## Problem Abstraction

Each task is represented as:

\[
[a_i, m_i]
\]

where:

- \(a_i\) is the actual energy cost after completing the task.
- \(m_i\) is the minimum energy required before starting the task.

If the current energy is \(E\), then task \(i\) can be started only if:

\[
E \ge m_i
\]

After completing the task:

\[
E \leftarrow E - a_i
\]

The goal is to find the minimum initial energy needed to complete all tasks in some order.

---

## 1. Brute Force Is Exponential

The most direct idea is to try every possible task order.

However, with \(n\) tasks, there are:

\[
n!
\]

possible permutations.

So brute force is not acceptable.

This means the key challenge is not simulation.  
The key challenge is discovering a structural ordering rule.

---

## 2. A Feasibility Perspective

One natural direction is binary search on the answer.

Given an initial energy \(X\), we can ask:

> Is there an order of tasks that allows us to finish all tasks?

The answer is monotonic:

- If \(X\) is feasible,
- then any larger initial energy is also feasible.

So binary search seems possible.

However, the check function still needs to decide a good task order.

A tempting local strategy is:

> If multiple tasks are currently available, do the task with smaller actual cost first.

The intuition is that smaller actual cost leaves more remaining energy.

This local idea is reasonable but not globally reliable.

The issue is that each task has not only a cost \(a_i\), but also a minimum requirement \(m_i\).  
If a high-minimum task is delayed too much, it may become impossible to start later.

So sorting only by actual cost is not enough.

---

## 3. The Key Quantity: Minimum Surplus

For each task:

\[
[a_i, m_i]
\]

the important quantity is:

\[
m_i - a_i
\]

This value can be interpreted as follows:

> If we start task \(i\) with exactly \(m_i\) energy, then after finishing it, we have \(m_i - a_i\) energy left.

So \(m_i - a_i\) measures how much energy remains after satisfying the task's minimum requirement and paying its actual cost.

This quantity connects the two competing forces:

- the entry requirement \(m_i\),
- and the energy consumption \(a_i\).

Therefore, the greedy ordering should be based on \(m_i - a_i\), not on \(m_i\) or \(a_i\) alone.

---

## 4. Greedy Ordering

The correct ordering is:

\[
m_i - a_i \quad \text{in descending order}
\]

That is, tasks with larger \(m_i - a_i\) should be done earlier.

Intuitively, these are tasks that have relatively high minimum requirements but do not consume as much energy compared with that requirement.

Doing them earlier avoids the risk of losing too much energy before reaching them.

After sorting, the remaining problem becomes a linear scan.

---

## 5. Rewriting the Answer

Regardless of the order, the total actual energy consumed is fixed:

\[
\sum_i a_i
\]

So the minimum initial energy can be written as:

\[
\sum_i a_i + extra
\]

where:

- \(\sum_i a_i\) is the unavoidable energy needed to pay all actual costs;
- \(extra\) is the additional energy required to satisfy the minimum requirements along the chosen order.

This is the main accounting insight.

Instead of trying to guess the answer directly, we start from the total actual cost and add only the extra energy needed when a minimum requirement is not met.

---

## 6. Linear Scan After Sorting

After sorting tasks by \(m_i - a_i\) in descending order, define:

\[
base = \sum_i a_i
\]

Initialize:

\[
cur = base
\]

and:

\[
extra = 0
\]

Here, \(cur\) represents the current available energy during the scan.

For each task \([a_i, m_i]\):

If:

\[
cur < m_i
\]

then the current energy is not enough to start the task.  
We must add exactly:

\[
m_i - cur
\]

extra energy.

So:

\[
extra \leftarrow extra + (m_i - cur)
\]

and after adding this energy:

\[
cur = m_i
\]

Then we complete the task:

\[
cur \leftarrow cur - a_i
\]

If \(cur \ge m_i\), no extra energy is needed, and we simply do:

\[
cur \leftarrow cur - a_i
\]

At the end, the answer is:

\[
base + extra
\]

---

## 7. Scan Invariant

During the scan, we maintain the following invariant:

> After processing the current prefix of the sorted task list, `extra` is the minimum additional energy needed to make this prefix feasible.

For each task:

- If \(cur \ge m_i\), then the task is already feasible.
- If \(cur < m_i\), then at least \(m_i - cur\) additional energy is necessary.
- Adding more than that is unnecessary.
- Adding less than that is insufficient.

Therefore, every increment to `extra` is forced and minimal.

---

## 8. Example

Consider:

\[
[[1,7],[2,8],[3,9],[4,10],[5,11],[6,12]]
\]

For every task:

\[
m_i - a_i = 6
\]

So the order remains the same.

The total actual cost is:

\[
1 + 2 + 3 + 4 + 5 + 6 = 21
\]

Thus:

\[
base = 21
\]

Initialize:

\[
cur = 21,\quad extra = 0
\]

Now scan the tasks.

### Task \([1,7]\)

\[
cur = 21 \ge 7
\]

After finishing:

\[
cur = 21 - 1 = 20
\]

### Task \([2,8]\)

\[
cur = 20 \ge 8
\]

After finishing:

\[
cur = 20 - 2 = 18
\]

### Task \([3,9]\)

\[
cur = 18 \ge 9
\]

After finishing:

\[
cur = 18 - 3 = 15
\]

### Task \([4,10]\)

\[
cur = 15 \ge 10
\]

After finishing:

\[
cur = 15 - 4 = 11
\]

### Task \([5,11]\)

\[
cur = 11 \ge 11
\]

After finishing:

\[
cur = 11 - 5 = 6
\]

### Task \([6,12]\)

Now:

\[
cur = 6 < 12
\]

We need to add:

\[
12 - 6 = 6
\]

So:

\[
extra = 6
\]

After adding energy:

\[
cur = 12
\]

After finishing the task:

\[
cur = 12 - 6 = 6
\]

Therefore, the final answer is:

\[
base + extra = 21 + 6 = 27
\]

---

## 9. Why This Derivation Matters

A standard solution might simply say:

> Sort by \(m_i - a_i\) descending.

But that alone is not very informative.

The important reasoning path is:

1. Brute force over all orders is exponential.
2. Binary search on the answer is possible, but the feasibility check still needs a task order.
3. Sorting by actual cost alone is not sufficient.
4. The important quantity is the interaction between minimum requirement and actual cost.
5. That interaction is captured by \(m_i - a_i\).
6. After sorting, the answer can be computed as:
   \[
   \sum_i a_i + extra
   \]
7. The scan only adds energy when the current energy is below the task's minimum requirement.

This gives a more robust understanding than memorizing the final greedy rule.

---

## 10. Summary

The core idea of LeetCode 1665 is not implementation.  
It is the greedy modeling.

The final structure is:

1. Sort tasks by:

\[
m_i - a_i
\]

in descending order.

2. Let:

\[
base = \sum_i a_i
\]

3. Scan the sorted tasks while maintaining current energy \(cur\).

4. If:

\[
cur < m_i
\]

then add:

\[
m_i - cur
\]

to `extra`.

5. Return:

\[
base + extra
\]

The key insight is:

> The minimum initial energy consists of the unavoidable total actual cost plus the minimum extra energy needed to satisfy all minimum requirements under the greedy order.
