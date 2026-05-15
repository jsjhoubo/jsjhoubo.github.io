# LeetCode 3225 — Maximum Score From Grid Operations

## A Derivation-Oriented Technical Report

This note is not a standard editorial. It records the reasoning process that leads to the final DP formulation: why the obvious states fail, how the correct state naturally emerges, and how the transition is optimized from \(O(n^4)\) to \(O(n^3)\).

The main value of this problem is not simply knowing the final recurrence. The hard part is discovering the right state representation and verifying that the state is actually closed.

---

## 1. Problem Compression

The operation is:

> Pick a cell \((i,j)\), then turn all cells from row \(0\) to row \(i\) in column \(j\) black.

After any number of operations, each column must have a black prefix followed by a white suffix:

    black
    black
    ...
    white
    white

Therefore, the final state of each column can be represented by a single height:

\[
h[j] = \text{number of black cells in column } j
\]

So:

\[
h[j]=0
\]

means column \(j\) is fully white, and:

\[
h[j]=n
\]

means column \(j\) is fully black.

For a cell \((r,j)\):

\[
(r,j)\text{ is black} \iff r < h[j]
\]

\[
(r,j)\text{ is white} \iff r \ge h[j]
\]

This compresses the original grid operation problem into choosing a height sequence:

\[
h[0], h[1], \dots, h[n-1]
\]

where:

\[
0 \le h[j] \le n
\]

A brute force search over all final height assignments would have:

\[
(n+1)^n
\]

possibilities, which is impossible.

This is the first important modeling step: the operation sequence itself is irrelevant. Only the final black height of each column matters.

---

## 2. Why \(dp[j][h[j]]\) Is Not Closed

A natural first attempt is to define:

\[
dp[j][h[j]]
\]

as the maximum score we can get up to column \(j\), assuming column \(j\) has black height \(h[j]\).

This looks reasonable, but it is not closed.

A white cell \((r,j)\) contributes if at least one of its horizontal neighbors is black. In terms of heights, \((r,j)\) contributes iff:

\[
r \ge h[j]
\]

and:

\[
r < h[j-1] \quad \text{or} \quad r < h[j+1]
\]

So the contribution of column \(j\) depends on three heights:

\[
h[j-1],\ h[j],\ h[j+1]
\]

Knowing only \(h[j]\) is insufficient.

This is the first failed state attempt.

The problem is not that the recurrence is hard to write. The problem is that the state has lost information required to evaluate the score.

A one-column state cannot determine a column's score, because a column's white cells are scored based on both neighboring columns.

---

## 3. Why Simple Delayed Scoring Still Fails

A second attempt is to delay the scoring of the current column.

For example, define:

\[
f(j,h[j])
\]

as a state where column \(j\)'s height is fixed, but its white cells are not fully scored yet.

The idea is reasonable: if the right neighbor is not known yet, maybe we can postpone scoring.

But this still fails if the state only stores \(h[j]\).

If column \(j\) is not scored yet, then later scoring it requires both:

\[
h[j-1]
\]

and:

\[
h[j+1]
\]

However, if the state only stores \(h[j]\), then \(h[j-1]\) has already been compressed away.

So delaying the score does not solve the problem unless the state also preserves the boundary information needed to compute that delayed score later.

The lesson is:

\[
\text{Delayed scoring is useful only if the delayed score remains computable from the state.}
\]

This is the key closure issue.

---

## 4. The Natural Three-Column Dependency

Since column \(j\)'s contribution depends on:

\[
h[j-1],\ h[j],\ h[j+1]
\]

we should wait until all three are known before scoring column \(j\).

Let:

\[
a = h[j-1]
\]

\[
b = h[j]
\]

\[
c = h[j+1]
\]

Once \(a,b,c\) are known, column \(j\)'s contribution is completely determined.

This naturally suggests a sliding-window state:

\[
dp_{\text{prev}}[a][b]
\]

where \(a=h[j-1]\) and \(b=h[j]\) are the two known adjacent heights before scoring column \(j\).

Then we choose:

\[
c = h[j+1]
\]

and transition to:

\[
dp_{\text{now}}[b][c]
\]

This gives:

\[
dp_{\text{now}}[b][c]
=
\max
\left(
dp_{\text{now}}[b][c],
dp_{\text{prev}}[a][b] + score(j,a,b,c)
\right)
\]

The window moves as:

\[
(a,b) \rightarrow (b,c)
\]

This is the closure point.

To score column \(j\), we need exactly three heights:

\[
h[j-1],\ h[j],\ h[j+1]
\]

The previous state provides the first two, and the transition chooses the third.

Therefore the state is closed.

---

## 5. Scoring a Column

Now derive \(score(j,a,b,c)\).

Recall:

\[
a=h[j-1],\quad b=h[j],\quad c=h[j+1]
\]

A cell \((r,j)\) is white iff:

\[
r \ge b
\]

It has a black left neighbor iff:

\[
r < a
\]

It has a black right neighbor iff:

\[
r < c
\]

So \((r,j)\) contributes iff:

\[
r \ge b
\]

and:

\[
r < a \quad \text{or} \quad r < c
\]

Equivalently:

\[
b \le r < \max(a,c)
\]

Therefore:

\[
score(j,a,b,c)
=
\sum_{r=b}^{\max(a,c)-1} grid[r][j]
\]

If:

\[
\max(a,c) \le b
\]

then there is no contributing row, so:

\[
score(j,a,b,c)=0
\]

Using column prefix sums, where:

\[
prefix[r][j] = \sum_{t=0}^{r} grid[t][j]
\]

and:

\[
prefix[-1][j]=0
\]

we get:

\[
score(j,a,b,c)
=
prefix[\max(a,c)-1][j] - prefix[b-1][j]
\]

when \(\max(a,c)>b\), otherwise \(0\).

This \(O(1)\) scoring formula is necessary for the later optimization.

---

## 6. Boundary Handling

There is no column \(-1\) and no column \(n\).

We introduce two virtual boundary columns:

\[
h[-1]=0
\]

\[
h[n]=0
\]

Height \(0\) means fully white, so these virtual columns never provide black neighbors.

The initial states are:

\[
dp_{\text{prev}}[0][h_0]=0
\]

for all:

\[
0 \le h_0 \le n
\]

This represents:

\[
h[-1]=0,\quad h[0]=h_0
\]

For the last real column \(j=n-1\), the right neighbor is fixed:

\[
h[n]=0
\]

So when scoring the last column, \(c=0\).

---

## 7. Naive DP Complexity: \(O(n^4)\)

The direct transition is:

\[
dp_{\text{now}}[b][c]
=
\max_a
\left(
dp_{\text{prev}}[a][b] + score(j,a,b,c)
\right)
\]

Naively:

- There are \(O(n)\) columns.
- There are \(O(n^2)\) states \((a,b)\) or \((b,c)\).
- For each state, we may enumerate \(O(n)\) choices of \(a\).

So the total complexity is:

\[
O(n^4)
\]

This is already much better than \((n+1)^n\), but it is still not the final optimized form.

---

## 8. Key Observation for Optimization

For fixed \(j\) and fixed \(b\), the transition is:

\[
dp_{\text{now}}[b][c]
=
\max_a
\left(
dp_{\text{prev}}[a][b] + score(j,a,b,c)
\right)
\]

The important observation is:

\[
score(j,a,b,c)
\]

depends on \(a\) and \(c\) only through:

\[
\max(a,c)
\]

This lets us split the maximization over \(a\) into two cases.

---

## 9. Case 1: \(a \le c\)

If:

\[
a \le c
\]

then:

\[
\max(a,c)=c
\]

So:

\[
score(j,a,b,c)
=
prefix[c-1][j] - prefix[b-1][j]
\]

provided \(c>b\); otherwise the score is \(0\).

In this case, the score no longer depends on \(a\). Therefore the best contribution from this case requires:

\[
\max_{a\le c} dp_{\text{prev}}[a][b]
\]

This can be maintained by a prefix maximum over \(a\).

Define:

\[
leftMax[c]
=
\max_{a\le c} dp_{\text{prev}}[a][b]
\]

Then the candidate from \(a\le c\) is:

\[
leftMax[c] + prefix[c-1][j] - prefix[b-1][j]
\]

when \(c>b\). If \(c\le b\), the score part is \(0\), and the zero-score case must be handled correctly.

---

## 10. Case 2: \(a > c\)

If:

\[
a > c
\]

then:

\[
\max(a,c)=a
\]

So:

\[
score(j,a,b,c)
=
prefix[a-1][j] - prefix[b-1][j]
\]

provided \(a>b\); otherwise the score is \(0\).

The expression becomes:

\[
dp_{\text{prev}}[a][b] + prefix[a-1][j] - prefix[b-1][j]
\]

For fixed \(b\) and \(c\), we need:

\[
\max_{a>c}
\left(
dp_{\text{prev}}[a][b] + prefix[a-1][j]
\right)
\]

This can be maintained by a suffix maximum over \(a\).

Define:

\[
rightMax[c]
=
\max_{a>c}
\left(
dp_{\text{prev}}[a][b] + prefix[a-1][j]
\right)
\]

Then the candidate from \(a>c\) is:

\[
rightMax[c] - prefix[b-1][j]
\]

again with invalid states ignored.

---

## 11. Optimized Transition

For each column \(j\), and for each fixed current height \(b\), we compute:

\[
leftMax[c] = \max_{a\le c} dp_{\text{prev}}[a][b]
\]

and:

\[
rightMax[c] =
\max_{a>c}
\left(
dp_{\text{prev}}[a][b] + prefix[a-1][j]
\right)
\]

Then we can compute every:

\[
dp_{\text{now}}[b][c]
\]

in \(O(1)\).

Therefore, for each fixed \(b\), all \(c\) values can be processed in \(O(n)\).

Since there are \(O(n)\) choices of \(b\), each column layer costs:

\[
O(n^2)
\]

Across \(n\) columns, the total time is:

\[
O(n^3)
\]

Using rolling arrays, the space is:

\[
O(n^2)
\]

This is the final optimized complexity.

---

## 12. Why the Optimization Is Natural

The \(O(n^3)\) optimization is not a random trick.

It comes directly from the formula:

\[
score(j,a,b,c)
=
\sum_{r=b}^{\max(a,c)-1} grid[r][j]
\]

The only nonlinear interaction between \(a\) and \(c\) is:

\[
\max(a,c)
\]

So the natural split is:

\[
a \le c
\]

and:

\[
a > c
\]

In the first case, \(c\) controls the score.

In the second case, \(a\) controls the score.

This is exactly why prefix maximum and suffix maximum appear.

The optimization is therefore a direct consequence of understanding the score structure, not a memorized DP trick.

---

## 13. Implementation Pitfalls

The implementation is subtle. The most common mistakes are:

1. Confusing the meaning of \(h\). Here \(h\) is the number of black cells, not the last black row.
2. Black rows are \(0,1,\dots,h-1\).
3. White rows are \(h,h+1,\dots,n-1\).
4. The score range is rows \([b,\max(a,c)-1]\).
5. Prefix indices must be \(\max(a,c)-1\) and \(b-1\).
6. If \(\max(a,c)\le b\), the score is \(0\).
7. The virtual boundary heights are \(0\), not \(-1\).
8. Invalid DP states must be initialized to negative infinity.
9. \(a\) and \(c\) cannot be merged into one variable.
10. The optimization must split \(a\le c\) and \(a>c\).
11. The zero-score case must not be lost during optimization.

A particularly easy bug is to write:

\[
prefix[\max(a,c)][j] - prefix[b][j]
\]

instead of:

\[
prefix[\max(a,c)-1][j] - prefix[b-1][j]
\]

Another easy bug is to forget that the score is effectively:

\[
\max(0,\ prefix[\max(a,c)-1][j] - prefix[b-1][j])
\]

That zero case must be preserved when optimizing the transition.

---

## 14. Final Algorithm Summary

The algorithm can be summarized as follows:

1. Compress each column into a black height \(h[j]\).
2. Observe that column \(j\)'s score depends on \(h[j-1]\), \(h[j]\), and \(h[j+1]\).
3. Use delayed scoring: only score column \(j\) when all three heights are known.
4. Use the sliding-window DP state:

\[
dp_{\text{prev}}[a][b] \rightarrow dp_{\text{now}}[b][c]
\]

where:

\[
a=h[j-1],\quad b=h[j],\quad c=h[j+1]
\]

5. Compute \(score(j,a,b,c)\) using column prefix sums.
6. The naive transition is \(O(n^4)\).
7. Since \(score\) only depends on \(\max(a,c)\), split into \(a\le c\) and \(a>c\).
8. Use prefix maximum and suffix maximum to optimize the transition to \(O(n^3)\).

Final complexity:

\[
Time = O(n^3)
\]

\[
Space = O(n^2)
\]

---

## 15. Reflection

The main difficulty of this problem is state discovery.

A one-column DP is not closed. Delaying the score is not enough unless the state preserves the necessary boundary information. The correct state emerges only after recognizing the three-column dependency.

The real core of the problem is:

- column-height compression,
- three-column dependency,
- delayed scoring,
- closed sliding-window DP,
- \(O(n^4)\) transition,
- \(O(n^3)\) prefix/suffix maximum optimization.

This makes the problem much closer to a competitive programming DP modeling problem than a routine interview DP.

The final solution feels natural only after understanding why the simpler states fail. That reasoning process is the most valuable part of the problem.
