# LeetCode 887: Super Egg Drop — A Minimax DP Derivation

## Problem

We are given:

- `k` eggs
- `n` floors

There exists an unknown critical floor `f`, where:

- If we drop an egg from a floor higher than `f`, the egg breaks.
- If we drop an egg from floor `f` or below, the egg does not break.

The goal is to determine the exact value of `f` in the minimum number of moves in the worst case.

---

## 1. Why the State Is Subtle

At first glance, it is tempting to define a state involving the exact value of `f`.

For example:

\[
dp(n, k, f)
\]

where \(f\) is the true critical floor.

But this is not the right perspective.

The algorithm does not know \(f\).  
The goal is not to optimize for a known \(f\), but to design a strategy that works for every possible \(f\).

So the problem is not:

> Given \(f\), how many moves do we need?

It is:

> Without knowing \(f\), what is the minimum number of moves needed to guarantee that we can determine \(f\) in the worst case?

This makes the problem closer to a minimax decision process.

---

## 2. Minimax Interpretation

At each step, we choose a floor to drop an egg from.

Suppose we choose floor \(i\).

There are two possible outcomes:

1. The egg breaks.
2. The egg does not break.

Since we need a worst-case guarantee, after choosing floor \(i\), the adversary can be viewed as choosing the worse of these two outcomes.

Therefore, the structure is:

\[
\text{choose } i \quad \rightarrow \quad \max(\text{break case}, \text{not break case})
\]

Since we control the choice of \(i\), we want the best such choice:

\[
\min_i \max(\cdots)
\]

This is the core minimax structure.

---

## 3. State Definition

Define:

\[
dp(n, k)
\]

as the minimum number of moves needed to determine the critical floor in the worst case, given:

- \(n\) floors remaining
- \(k\) eggs available

This state is closed because it only depends on:

- how many floors are still possible
- how many eggs remain

The exact identity of the floor interval does not matter; only its length matters.

---

## 4. Transition

Suppose we drop an egg from floor \(i\), where:

\[
1 \le i \le n
\]

There are two cases.

### Case 1: The egg breaks

If the egg breaks at floor \(i\), then the critical floor must be below \(i\).

So we need to search among:

\[
i - 1
\]

floors, and we have one fewer egg:

\[
k - 1
\]

The cost of this branch is:

\[
dp(i - 1, k - 1)
\]

### Case 2: The egg does not break

If the egg does not break at floor \(i\), then the critical floor is at least \(i\).

So we only need to search the floors above \(i\), which gives:

\[
n - i
\]

remaining floors.

The egg is not broken, so we still have:

\[
k
\]

eggs.

The cost of this branch is:

\[
dp(n - i, k)
\]

### Worst-case branch

Since we need a worst-case guarantee, after choosing \(i\), the number of remaining moves is:

\[
\max(dp(i - 1, k - 1), dp(n - i, k))
\]

We also spent one move dropping the egg from floor \(i\).

Therefore:

\[
dp(n, k)
=
1 + \min_{1 \le i \le n}
\max(dp(i - 1, k - 1), dp(n - i, k))
\]

This is the basic minimax DP recurrence.

---

## 5. Base Cases

Several base cases are natural.

If there are no floors:

\[
dp(0, k) = 0
\]

because there is nothing to determine.

If there is one floor:

\[
dp(1, k) = 1
\]

because one drop is enough.

If there is only one egg:

\[
dp(n, 1) = n
\]

because we must test floors linearly from bottom to top in the worst case.

---

## 6. Naive Complexity

The number of states is:

\[
O(nk)
\]

For each state \((n,k)\), the naive transition enumerates:

\[
i = 1,2,\dots,n
\]

So the time complexity is:

\[
O(kn^2)
\]

For LeetCode constraints such as:

\[
n \le 10000,\quad k \le 100
\]

this is too slow.

So we need to exploit more structure.

---

## 7. Monotonicity Observation

For fixed \(n\) and \(k\), define:

\[
A(i) = dp(i - 1, k - 1)
\]

and:

\[
B(i) = dp(n - i, k)
\]

As \(i\) increases:

- \(A(i)\) increases, because the break case has more lower floors to search.
- \(B(i)\) decreases, because the not-break case has fewer upper floors to search.

So:

\[
A(i)
\]

is monotone non-decreasing, and:

\[
B(i)
\]

is monotone non-increasing.

The objective for a fixed \(i\) is:

\[
\max(A(i), B(i))
\]

This function is minimized near the point where:

\[
A(i) \approx B(i)
\]

In other words, the best first drop is near the crossing point of the two monotonic branches.

---

## 8. Binary Search Optimization

Since:

\[
A(i) - B(i)
\]

is monotone non-decreasing, we can binary search for the crossing point.

The optimal \(i\) is near the first position where:

\[
A(i) \ge B(i)
\]

Because \(i\) is discrete, the best answer may be at the crossing point or immediately next to it.

The important idea is:

> We do not need to scan all floors \(i\).  
> We can use monotonicity to locate the balance point between the break and not-break branches.

This reduces the transition cost from:

\[
O(n)
\]

to approximately:

\[
O(\log n)
\]

per state.

So the DP can be optimized from:

\[
O(kn^2)
\]

to:

\[
O(kn \log n)
\]

using binary search inside the minimax recurrence.

---

## 9. Why This Is a Minimax Problem

This problem can be viewed as a decision tree.

At every node:

- We choose a floor to test.
- Nature reveals either "break" or "not break".
- We must be prepared for the worse branch.

Thus, the recurrence has the form:

\[
\min_i \max(\text{left branch}, \text{right branch})
\]

This is structurally similar to adversarial games, where:

- our strategy minimizes the cost,
- the adversary chooses the branch that maximizes the cost.

The unknown floor \(f\) is not directly part of the state, because it is precisely what the strategy is trying to discover.

---

## 10. Summary

The core derivation is:

1. Do not define the state using the true unknown \(f\).
2. Define:

\[
dp(n,k)
\]

as the worst-case minimum number of moves needed for \(n\) floors and \(k\) eggs.

3. If we drop from floor \(i\):

- break branch:

\[
dp(i-1,k-1)
\]

- not-break branch:

\[
dp(n-i,k)
\]

4. Since we need worst-case guarantee:

\[
\max(dp(i-1,k-1), dp(n-i,k))
\]

5. Since we choose the best \(i\):

\[
dp(n,k)
=
1+\min_i \max(dp(i-1,k-1), dp(n-i,k))
\]

6. The two branches are monotonic in opposite directions:

\[
dp(i-1,k-1) \text{ increases with } i
\]

\[
dp(n-i,k) \text{ decreases with } i
\]

7. Therefore, the optimal \(i\) lies near their crossing point, which enables binary search optimization.

---

## Reflection

The main difficulty of this problem is not implementation.

The hard part is changing perspective:

- from "what is the true floor?"
- to "what strategy guarantees success under the worst outcome?"

Once the problem is viewed as a minimax decision tree, the recurrence becomes natural.

The next key insight is that the two branches move in opposite directions as the first drop floor changes.  
This monotonic structure allows the naive enumeration over all floors to be optimized by binary search.
