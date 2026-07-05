---
layout: post
title: "LeetCode 1000: Interval DP with Modulo Feasibility"
date: 2026-05-20
categories: [leetcode, dynamic-programming, algorithms]
tags: [leetcode, interval-dp, dynamic-programming, modulo, proof, derivation]
---
# Technique Notes: Interval DP with Modulo Feasibility — LeetCode 1000

**Problem:** LeetCode 1000 — *Minimum Cost to Merge Stones*  
**Topic:** Interval DP, modulo feasibility, state compression  
**Date:** 2026-05-20

---

## 1. Problem Restatement

We are given an array `stones`, where each element is an initial pile.

In one operation, we may merge exactly `K` consecutive piles into one pile.  
The cost of this operation is the total number of stones in those `K` piles.

The goal is to merge the entire array into one pile with minimum total cost.  
If this is impossible, return `-1`.

---

## 2. First Structural Observation: `K piles -> 1 pile`

One legal merge operation transforms:

```text
K piles -> 1 pile
```

So the number of piles decreases by:

```text
K - 1
```

each time.

Starting from `n` piles and ending with one pile, the total decrease in the number of piles is:

```text
n - 1
```

Therefore, we need:

```text
(n - 1) % (K - 1) == 0
```

The global feasibility condition is:

```cpp
if ((n - 1) % (K - 1) != 0) return -1;
```

Equivalently:

```cpp
if (n % (K - 1) != 1) return -1;
```

This is not just an implementation trick. It is a direct invariant from the operation rule.

---

## 3. Why This Is Interval DP

Every operation merges consecutive piles.

Therefore, every pile formed during the process must correspond to a continuous interval of the original array.

So any intermediate pile has the form:

```text
[l, r]
```

This naturally suggests interval dynamic programming.

However, a normal two-dimensional interval DP is not immediately enough, because an interval may need to be reduced to multiple piles before it can be merged further.

---

## 4. Three-Dimensional State

Define:

```text
dp[l][r][t]
```

where:

```text
dp[l][r][t] = minimum cost to merge stones[l...r] into exactly t piles
```

Here:

- `K` is the number of piles required by one merge operation.
- `t` is the target number of piles remaining after reducing interval `[l, r]`.

These two meanings must not be confused.

For example:

```text
dp[l][r][1]
```

means the minimum cost to reduce interval `[l, r]` into one pile.

But to compute this state, we may first need:

```text
dp[l][r][K]
```

because the final merge into one pile can only happen after the interval has already been reduced to exactly `K` piles.

---

## 5. Local State Feasibility

For interval `[l, r]`, let:

```text
len = r - l + 1
```

Initially, this interval has `len` piles.

If it is reduced to `t` piles, then the total number of removed piles is:

```text
len - t
```

Since each operation removes exactly `K - 1` piles, the state is reachable only if:

```text
(len - t) % (K - 1) == 0
```

and also:

```text
len >= t
```

So in DFS or DP, invalid states can be pruned by:

```cpp
if (len < piles) return;
if ((len - piles) % (K - 1) != 0) return;
```

This is the local version of the global feasibility condition.

---

## 6. Two Semantics: Composition vs Actual Merge

The main subtlety is that the DP contains two different semantic actions.

### 6.1 Composition

Suppose we want to reduce `[l, r]` into `t` piles.

We can split the interval at `mid`:

```text
[l, mid] and [mid + 1, r]
```

If the left interval becomes `x` piles and the right interval becomes `t - x` piles, then together they form `t` piles.

The general transition is:

```text
dp[l][r][t]
=
min over mid, x {
    dp[l][mid][x] + dp[mid + 1][r][t - x]
}
```

This transition only composes two already-reduced subintervals.

No new merge operation happens here, so no interval sum is added.

### 6.2 Actual Merge

Cost is paid only when exactly `K` piles are merged into one pile.

So for the same interval `[l, r]`, once we know how to reduce it to exactly `K` piles, we can perform one final merge:

```text
dp[l][r][1] = dp[l][r][K] + sum(l, r)
```

This equation is the key bridge between interval composition and actual merging.

---

## 7. Implementation-Friendly Recurrence

A convenient recurrence is to form `piles` piles by separating the last pile.

If the last pile comes from suffix interval `[mid + 1, r]`, then:

```text
[l, mid]      -> piles - 1 piles
[mid + 1, r] -> 1 pile
```

So:

```text
dp[l][r][piles]
=
min over mid {
    dp[l][mid][piles - 1] + dp[mid + 1][r][1]
}
```

This is valid because every pile in an intermediate state corresponds to a continuous original interval.

A common mistake is to write the suffix cost as:

```text
sum(mid + 1, r)
```

This is wrong.

The suffix may need internal merges before becoming one pile, so the correct term is:

```text
dp[mid + 1][r][1]
```

The interval sum is paid only when `K` already-formed piles are merged into one pile.

---

## 8. Base Case

A single stone is already one pile and requires no merge operation.

Therefore:

```cpp
dp[i][i][1] = 0;
```

not:

```cpp
dp[i][i][1] = stones[i];
```

The stone value contributes to the cost only when it is included in an actual merge operation.

---

## 9. Top-Down DP Skeleton

A recursive function can be defined as:

```cpp
dfs(l, r, piles)
```

meaning:

```text
compute dp[l][r][piles]
```

The pruning logic:

```cpp
if (l > r) return;
if (piles < 1 || piles > K) return;

int len = r - l + 1;

if (len < piles) return;
if ((len - piles) % (K - 1) != 0) return;
```

If `piles == 1`, use the actual merge transition:

```cpp
dfs(l, r, K);

if (dp[l][r][K] != INF) {
    dp[l][r][1] = dp[l][r][K] + rangeSum(l, r);
}
```

If `piles > 1`, use the composition transition:

```cpp
for (int mid = l; mid < r; mid++) {
    dfs(l, mid, piles - 1);
    dfs(mid + 1, r, 1);

    if (dp[l][mid][piles - 1] == INF) continue;
    if (dp[mid + 1][r][1] == INF) continue;

    dp[l][r][piles] =
        min(dp[l][r][piles],
            dp[l][mid][piles - 1] + dp[mid + 1][r][1]);
}
```

The `INF` checks are necessary to avoid invalid transitions and overflow.

---

## 10. Implementation Details

### DP Dimension

Since `piles` can be equal to `K`, the third dimension must be `K + 1`:

```cpp
vector<vector<vector<int>>> dp(
    n, vector<vector<int>>(n, vector<int>(K + 1, INF))
);
```

### Prefix Sum

Use prefix sums to compute interval costs.

With exclusive prefix:

```cpp
rangeSum(l, r) = prefix[r + 1] - prefix[l];
```

With inclusive prefix:

```cpp
rangeSum(l, r) = sum[r] - (l > 0 ? sum[l - 1] : 0);
```

### Avoid Overflow

Before adding two DP values:

```cpp
dpA + dpB
```

check:

```cpp
if (dpA == INF) continue;
if (dpB == INF) continue;
```

Before doing:

```cpp
dp[l][r][K] + rangeSum(l, r)
```

check:

```cpp
if (dp[l][r][K] != INF)
```

---

## 11. Complexity

There are roughly:

```text
O(n^2 * K)
```

states.

Each state may enumerate:

```text
O(n)
```

split points.

So the total complexity is:

```text
O(n^3 * K)
```

If `K` is treated as up to `n`, this can be viewed as:

```text
O(n^4)
```

For the original constraints of LeetCode 1000, this version is usually acceptable with pruning.

---

## 12. Two-Dimensional Optimization

There is a known optimization that compresses away the third dimension.

For an interval of length `len`, the minimum possible remaining pile count is determined by:

```text
((len - 1) % (K - 1)) + 1
```

So one can define:

```text
dp[l][r] = minimum cost to reduce interval [l, r] as much as possible
```

Then use:

```cpp
dp[l][r] = min(dp[l][r], dp[l][mid] + dp[mid + 1][r]);
```

When the interval can be reduced to exactly one pile:

```cpp
if ((len - 1) % (K - 1) == 0) {
    dp[l][r] += rangeSum(l, r);
}
```

This gives roughly:

```text
O(n^3)
```

especially when split points are stepped by `K - 1`.

---

## 13. Common Mistakes

### Mistake 1: Thinking `t < K` is invalid

Wrong.

The final answer itself is `1` pile, and `1 < K`.

`t` is the target number of piles after reducing an interval.  
It is not the number of piles merged in one operation.

### Mistake 2: Setting `dp[i][i][1] = stones[i]`

Wrong.

A single pile needs no merge operation:

```cpp
dp[i][i][1] = 0;
```

### Mistake 3: Treating the last pile as only an interval sum

Wrong.

For suffix `[mid + 1, r]`, the cost to make it one pile is:

```text
dp[mid + 1][r][1]
```

not:

```text
sum(mid + 1, r)
```

### Mistake 4: Forgetting modulo feasibility

The state:

```text
dp[l][r][piles]
```

is valid only if:

```text
(len - piles) % (K - 1) == 0
```

---

## 14. Core Mental Model

The whole problem is about separating two meanings.

Composition means:

```text
left interval forms some piles
right interval forms some piles
together they form more piles
```

Actual merge means:

```text
K piles become 1 pile
pay interval sum
```

The bridge is:

```text
dp[l][r][1] = dp[l][r][K] + sum(l, r)
```

Once this bridge is understood, the DP becomes systematic.

---

## 15. Final Reasoning Chain

```text
consecutive merge
=> interval DP

K piles -> 1 pile
=> pile count decreases by K - 1

interval [l, r] into t piles
=> dp[l][r][t]

valid state
=> (len - t) % (K - 1) == 0

actual cost only when K piles become 1
=> dp[l][r][1] = dp[l][r][K] + sum(l, r)
```

This is the essential technique behind LeetCode 1000.
