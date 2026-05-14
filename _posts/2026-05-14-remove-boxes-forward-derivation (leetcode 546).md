---
layout: post
title: "A Forward Derivation of Remove Boxes DP"
date: 2026-05-14
categories: algorithms
tags: leetcode dynamic-programming interval-dp state-compression
---

# A Forward Derivation of Remove Boxes DP

> LeetCode 546 — Remove Boxes
>
> A technical note on deriving `dp[l][r][k]` from first principles,
> instead of reverse-engineering the final state definition.

Most explanations of Remove Boxes begin with the famous state:

```cpp
dp[l][r][k]
```

followed by a brief statement such as:

```text
k = the number of boxes equal to boxes[r]
attached to the right.
```

For many people, this feels almost impossible to invent naturally.

Questions immediately appear:

```text
Why right?
Why not left?
Why is only the count needed?
Why does this not lose information?
Why does this compress an exponential subset space?
```

This note tries to answer those questions from the ground up.

The goal is NOT to present the final DP immediately.

Instead, we derive it step by step:

```text
root-driven search
→ leaf-driven reasoning
→ retained subsets
→ sequence recursion
→ gap decomposition
→ intervalization
→ equivalence relation
→ state compression
→ dp[l][r][k]
```

The important part is not memorizing the final state.

The important part is understanding:

```text
what exponential object is being compressed,
and why the compression is lossless.
```

---

# 1. Starting From the Original Search Tree

Consider the smallest interesting example:

```text
1 2 1
```

The naive search tree looks like:

```text
remove left 1
remove middle 2
remove right 1
```

Very quickly we notice:

```text
removing 2 first
allows the two 1's to merge later.
```

This gives the first crucial observation.

---

## Observation 1

```text
Removing intermediate gaps may create future merges
between same-colored boxes.
```

This immediately implies:

```text
subintervals are NOT independent.
```

A standard interval DP is unlikely to work directly.

---

# 2. Why Root-Driven Search Becomes Messy

The natural perspective is:

```text
At the current state,
what should we remove next?
```

This is a root-driven search.

However, it quickly becomes chaotic.

Why?

Because local actions fundamentally change future topology.

For example:

```text
1 2 1
```

Removing:

```text
2
```

changes the future connectivity structure.

So reasoning entirely from the root is difficult.

The branching factor is enormous.

More importantly:

```text
future merge structures depend heavily on earlier decisions.
```

---

# 3. The Critical Perspective Shift: Leaf → Root

Instead of asking:

```text
What should be removed first?
```

we ask:

```text
What survives until the very end?
```

This is the turning point.

We stop viewing the search tree from:

```text
root → leaf
```

and instead reason from:

```text
leaf → root
```

This changes the entire structure of the problem.

---

# 4. The Emergence of Sequence Recursion

Consider:

```text
1 3 2 2 2 3 4 3 1
```

Suppose the final removed group is:

```text
(3,3,3)
```

Then all non-3 regions must already have disappeared.

This induces the decomposition:

```text
(1) | (222) | (4) | (1)
```

At this moment, the problem no longer looks like:

```text
"Which move should I make now?"
```

Instead it becomes:

```text
"Which boxes survive until the final layer?"
```

This naturally suggests a recursion over retained sequences.

---

# 5. A First Sequence-Based DP Attempt

At this stage, a natural idea appears.

Suppose:

```text
F(sequence)
```

means:

```text
maximum obtainable score from this remaining sequence.
```

For example:

```text
F(132223431)
```

may branch into:

```text
remove final 1-group
remove final 3-group
remove final 2-group
remove final 4-group
```

leading to decompositions like:

```text
9 + F(122231)
9 + F(133431)
1 + F(13222343)
```

This viewpoint is actually very deep.

It correctly identifies:

```text
The true combinatorial object is:
which boxes survive until the final merge.
```

However, this representation is still exponential.

Why?

Because different retained subsets generate different residual sequences.

So although this sequence recursion captures the correct structure,
it is still computationally infeasible.

---

# 6. From Sequence Recursion to Interval Structure

Now comes another conceptual transition.

Suppose the final retained group is:

```text
{1,5,7}
```

corresponding to the three 3's in:

```text
1 3 2 2 2 3 4 3 1
```

Then everything between surviving 3's must disappear first.

So the sequence decomposes into independent gaps:

```text
[0,0]
[2,4]
[6,6]
[8,8]
```

This is the moment where:

```text
sequence recursion
```

starts transforming into:

```text
interval recursion.
```

The retained subset determines:

```text
how the sequence is partitioned into gap intervals.
```

This is the true bridge between:

```text
sequence DP
```

and:

```text
interval DP.
```

---

# 7. The First Natural Interval DP — And Why It Fails

At this point, a very natural attempt is:

```text
F(l,r,c)
```

meaning:

```text
Inside interval [l,r],
color c is removed last.
```

Then:

```text
all non-c gaps are solved first.
```

This idea is extremely natural.

In fact:

```text
it passes many test cases.
```

But it is wrong.

---

# 8. The Real Difficulty: Partial Retention

Consider the classic counterexample:

```text
1 2 2 1 2 2 1
```

Optimal strategy:

```text
remove the middle 1 first → 1 point

remaining:
1 2 2 2 2 1

remove four 2's → 16 points

remove two 1's → 4 points

Total = 21
```

This reveals the core difficulty.

---

## Observation 2

```text
Among boxes of the same color,
some may be removed early,
while others survive until the final merge.
```

Therefore:

```text
"color 1 is removed last"
≠
"all 1's are removed last"
```

So:

```text
F(l,r,c)
```

does not contain enough information.

---

# 9. The True Exponential Object

Now consider:

```text
1 2 1 2 1 2 1
```

Positions of color `1`:

```text
0 2 4 6
```

If we only ask which `1` boxes are retained until some final merge, then in principle every subset of these four positions is possible:

```text
{}
{0}
{2}
{4}
{6}
{0,2}
{0,4}
{0,6}
{2,4}
{2,6}
{4,6}
{0,2,4}
{0,2,6}
{0,4,6}
{2,4,6}
{0,2,4,6}
```

That is:

```text
2^4 = 16
```

If we ignore the empty subset, there are:

```text
2^4 - 1 = 15
```

non-empty retained subsets.

This is the true source of exponential complexity.

---

## Observation 3

```text
The exponential object is:
which same-colored boxes survive until the final merge.
```

The problem is fundamentally about retained subsets.

The next question is:

```text
Can we group these 2^m subsets so that many of them become equivalent?
```

---

# 10. First Compression: Partition by Rightmost Element

Suppose:

```text
S = {p1 < p2 < ... < pt}
```

Every non-empty subset has a unique:

```text
max(S) = pt
```

So instead of enumerating all subsets directly,
we first partition them by:

```text
their rightmost surviving element.
```

For example, fixing:

```text
rightmost = 6
```

gives exactly the subsets that contain `6` and contain no retained `1` to the right:

```text
{6}
{0,6}
{2,6}
{4,6}
{0,2,6}
{0,4,6}
{2,4,6}
{0,2,4,6}
```

This is the first compression.

Originally, for four positions:

```text
2^4 = 16 subsets
```

After fixing the rightmost element as `6`, the remaining three positions `0,2,4` are optional, so this partition contains:

```text
2^3 = 8 subsets
```

Still exponential,
but now every subset shares the same rightmost anchor.

---

# 11. The Most Important Step: Recursive Partitioning

Now comes the key insight.

Suppose:

```text
rightmost anchor = 6
```

Instead of enumerating the entire subset,
we ask:

```text
Which surviving same-colored box
is the nearest retained partner to the left of 6?
```

Suppose:

```text
m = 4
```

Then:

```text
(5,5)
```

must be completely removed first.

After that:

```text
4 and 6 become adjacent.
```

Now the remaining problem becomes:

```text
solve [0,4],
while knowing that
there is already one same-colored box to the right of 4
that will eventually merge with it.
```

This is the exact moment where:

```text
k
```

emerges naturally.

Not from magic.

Not from reverse engineering.

But directly from recursive decomposition.

---

# 12. The Emergence of dp(l,r,k)

We now arrive naturally at:

```text
dp(l,r,k)
```

meaning:

```text
Inside interval [l,r],
boxes[r] belongs to the final retained group,
and there are already k same-colored boxes
collected on its right.
```

Important clarification:

```text
k does NOT refer to boxes inside [l,r].
```

Instead:

```text
k is inherited from parent recursion.
```

It records:

```text
how many same-colored boxes
have already been merged into the future final group.
```

This is not an arbitrary state definition.

It is induced naturally by:

```text
recursive subset decomposition.
```

---

# 13. The Equivalence Relation

This is the real heart of the compression.

Consider these subsets:

```text
{4,6}
{2,4,6}
{0,4,6}
{0,2,4,6}
```

Once recursion reaches:

```text
dp(0,4,1)
```

all of them become equivalent.

Why?

Because future decisions no longer depend on:

```text
which exact subset history produced this state.
```

Future behavior depends only on:

```text
1. current anchor position
2. merged same-color count
```

Formally:

```text
Different retained subsets become equivalent
once future recursion cannot distinguish them.
```

And future recursion only depends on:

```text
(anchor, merged_count)
```

NOT the full subset history.

This is the actual state compression.

Most explanations skip this step entirely.

But this is the real mathematical reason the DP works.

---

# 14. Transition Derivation

Now the transitions become straightforward.

## Case 1 — Stop Merging Further Left

Suppose:

```text
boxes[r]
will NOT merge with any same-colored box inside [l,r-1].
```

Then:

```text
first solve [l,r-1]
```

and finally remove:

```text
boxes[r]
+
its k collected same-colored boxes.
```

Thus:

```text
dp(l,r,k)
=
dp(l,r-1,0)
+
(k+1)^2
```

Important clarification:

This does NOT mean:

```text
there are no same-colored boxes in [l,r-1].
```

It only means:

```text
those boxes are not part of the current retained group.
```

---

## Case 2 — Merge With a Left Partner

Suppose:

```text
boxes[m] == boxes[r]
```

and we want:

```text
m and r to belong to the same final retained group.
```

Then:

```text
(m+1,r-1)
```

must be fully removed first.

So:

```text
r becomes part of m's future retained group.
```

Therefore:

```text
dp(l,r,k)
=
max(
    dp(l,m,k+1)
    +
    dp(m+1,r-1,0)
)
```

where:

```text
boxes[m] == boxes[r]
```

Why `k+1`?

Because from the perspective of `m`:

```text
its retained group now contains:
- the previous k boxes
- plus r itself
```

Again:

```text
k is recursively induced.
```

---

# 15. Why This Covers All Possibilities

Suppose the final retained subset is:

```text
S = {s1 < s2 < ... < st}
```

Fix:

```text
st
```

as the rightmost anchor.

Then:

```text
s_{t-1}
```

must be the nearest retained partner to its left.

Therefore:

```text
(s_{t-1}+1, st-1)
```

must disappear first.

Then recursion continues on:

```text
{s1,...,s_{t-1}}
```

Thus every retained subset corresponds to:

```text
a unique recursive decomposition path.
```

So the DP is complete.

---

# 16. What Is Actually Being Compressed?

A common misunderstanding is:

```text
The number of subsets somehow decreases.
```

It does not.

The actual phenomenon is:

```text
many different subsets become equivalent
under future recursion.
```

The DP does NOT eliminate subsets directly.

Instead:

```text
it quotients the subset space
by a future-behavior equivalence relation.
```

The equivalence class is represented by:

```text
(l,r,k)
```

This is the true structure behind the famous DP.

---

# 17. Final Implementation

```cpp
class Solution {
public:
    vector<int> boxes;
    int memo[100][100][100];

    int dp(int l, int r, int k) {
        if (l > r) return 0;

        int &ans = memo[l][r][k];
        if (ans != -1) return ans;

        ans = dp(l, r - 1, 0) + (k + 1) * (k + 1);

        for (int m = l; m < r; m++) {
            if (boxes[m] == boxes[r]) {
                ans = max(
                    ans,
                    dp(l, m, k + 1)
                    + dp(m + 1, r - 1, 0)
                );
            }
        }

        return ans;
    }

    int removeBoxes(vector<int>& input) {
        boxes = input;
        memset(memo, -1, sizeof(memo));
        return dp(0, boxes.size() - 1, 0);
    }
};
```

---

# 18. Final Remarks

The hardest part of Remove Boxes is not implementation.

It is not memoization.

It is not interval DP syntax.

The real difficulty is understanding:

```text
what exponential object is being compressed,
and why the compression is lossless.
```

Once that becomes clear:

```text
dp[l][r][k]
```

no longer feels magical.

It becomes a natural consequence of:

```text
recursive subset decomposition.
```

```
```
