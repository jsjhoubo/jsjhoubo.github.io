# LeetCode 3464 — Maximize the Distance Between Points on a Square

## A Derivation-Oriented Technical Report

This note is not written as a standard editorial. It records the reasoning path that leads from a seemingly hard pairwise-distance selection problem to a one-dimensional cyclic greedy feasibility check.

The important part of this problem is not simply “binary search + greedy.” The real difficulty is explaining why a problem that initially looks like a general k-clique-style pairwise constraint becomes tractable because all points lie on the boundary of a square, and because the condition k >= 4 forces the optimal answer to be at most side.

---

## 1. Problem Restatement

We are given a square with side length side. All given points lie on the boundary of this square.

We need to choose exactly k points such that the minimum Manhattan distance among all chosen pairs is as large as possible.

For two points:

\[
p_i=(x_i,y_i),\quad p_j=(x_j,y_j)
\]

their Manhattan distance is:

\[
dist(p_i,p_j)=|x_i-x_j|+|y_i-y_j|
\]

The objective is:

\[
\max_{|S|=k}\ \min_{p_i,p_j\in S,\ i\ne j} dist(p_i,p_j)
\]

---

## 2. Why the Problem Initially Looks Hard

A natural first step is to binary search the answer.

Given a candidate distance d, the feasibility question becomes:

Can we choose k points such that every pair has Manhattan distance at least d?

That is:

\[
\forall p_i,p_j\in S,\quad dist(p_i,p_j)\ge d
\]

If these were arbitrary points in the plane, this would look like a graph problem.

Build a graph where:

- each point is a vertex;
- two vertices are connected if their Manhattan distance is at least d.

Then the feasibility question becomes:

Does this graph contain a clique of size k?

This is the first warning sign. In a general graph, finding a k-clique is hard. Brute force would require checking:

\[
\binom{n}{k}
\]

subsets, which is impossible for large n.

So the problem cannot be solved by treating the points as arbitrary vertices with arbitrary pairwise constraints.

The geometry must matter.

---

## 3. Why Local Nearest-Neighbor Thinking Fails

One tempting idea is:

The final minimum distance must be determined by some pair of selected points. Therefore, maybe for each point we only need to inspect its nearest k neighbors.

This does not work.

The closest point to a selected point inside the chosen subset is not necessarily among its nearest k neighbors in the entire point set.

Many globally nearby points may simply not be selected.

So the following implication is false:

\[
\text{nearest in chosen set} \Rightarrow \text{nearest in global set}
\]

This means local k-nearest-neighbor information is not enough.

The selected set is a global object.

---

## 4. Why Single-Center Geometry Also Fails

Another attempt is to use Manhattan balls.

For a fixed point p, all points within distance d form a diamond:

\[
|x-x_p|+|y-y_p|<d
\]

One may try to count how many points lie outside such a diamond.

But this only checks distance from one center point.

The original constraint is all-to-all:

\[
\forall p_i,p_j\in S,\quad dist(p_i,p_j)\ge d
\]

A single-center condition cannot guarantee that the other selected points are mutually far apart.

So this idea is too weak.

The mistake is replacing a pairwise constraint with a one-center constraint.

---

## 5. The k-Clique View Is Useful, But Too General

At this point, the feasibility check still looks like:

Choose k points such that all pairwise distances are at least d.

In a general point set, this is essentially an independent-set or clique-style selection problem.

That viewpoint is useful because it tells us what not to do:

We should not try to enumerate subsets.
We should not try to locally repair nearest-neighbor choices.
We should not try to use one point as the only anchor.

The reason the problem is solvable is the special structure:

All points lie on the boundary of a square.

---

## 6. Mapping Boundary Points to a Cycle

Since all points are on the square boundary, we can sort them in clockwise order along the perimeter.

For example, we can map each boundary point to a one-dimensional coordinate pos on the perimeter.

One possible mapping is:

- bottom edge y = 0: pos = x
- right edge x = side: pos = side + y
- top edge y = side: pos = 3 * side - x
- left edge x = 0: pos = 4 * side - y

Then all boundary points become points on a cycle of length:

\[
P = 4\cdot side
\]

After sorting these positions, the original 2D boundary order becomes a 1D circular order.

This is the main structural reduction:

\[
\text{square boundary} \rightarrow \text{one-dimensional cycle}
\]

---

## 7. The Subtle Issue: Perimeter Distance Is Not Always Manhattan Distance

At first, it is tempting to say:

If two selected points are far enough along the perimeter, then their Manhattan distance is also far enough.

But this needs care.

For points on opposite sides, perimeter distance and Manhattan distance can differ significantly.

Example:

\[
p=(0,y),\quad q=(side,y)
\]

Then:

\[
dist(p,q)=side
\]

but their shortest perimeter distance can be around:

\[
2\cdot side
\]

So perimeter distance is not always equal to Manhattan distance.

This was a key source of confusion.

If we only use cyclic perimeter gaps, we may accidentally ignore pairs on opposite sides.

So why does the cycle model still work?

The missing condition is:

\[
k\ge 4
\]

---

## 8. Why k >= 4 Is the Key Condition

Since we must choose k points on a perimeter of length:

\[
4\cdot side
\]

among the k cyclic gaps between consecutive selected points, at least one gap is at most:

\[
\frac{4\cdot side}{k}
\]

Because the problem has:

\[
k\ge 4
\]

we get:

\[
\frac{4\cdot side}{k}\le side
\]

Therefore, the optimal minimum distance cannot exceed side:

\[
answer \le side
\]

This is the turning point.

It means that when checking a candidate distance d, we only need to consider:

\[
0\le d\le side
\]

For this range, the relationship between Manhattan distance and circular perimeter distance becomes safe.

For boundary points on a square:

\[
dist(p,q)\ge \min(\text{circular perimeter distance}(p,q),\ side)
\]

So if:

\[
d\le side
\]

and the circular perimeter distance between two boundary points is at least d, then their Manhattan distance is also at least d.

Conversely, Manhattan distance is never greater than the shortest path along the boundary, so if Manhattan distance is at least d, the circular boundary distance is also at least d.

Thus, for d <= side, the Manhattan constraint is equivalent to requiring enough spacing on the boundary cycle.

This is where k >= 4 becomes essential.

Without k >= 4, the answer might be larger than side, and the simple cycle-spacing model would no longer be automatically valid.

---

## 9. From Pairwise Manhattan Constraint to Cyclic Spacing

After mapping boundary points to perimeter positions and knowing that the target distance satisfies:

\[
d\le side
\]

the feasibility condition becomes:

Can we select k positions on a cycle of length P such that every adjacent selected pair in cyclic order has circular gap at least d?

That is, if the selected positions in clockwise order are:

\[
p_1,p_2,\dots,p_k
\]

then we need:

\[
p_{i+1}-p_i\ge d
\]

for all consecutive pairs, and also the wrap-around gap:

\[
P-(p_k-p_1)\ge d
\]

This is much simpler than checking all pairwise Manhattan distances.

The reason this works is that on a cycle, the closest pair in the selected set must appear as adjacent selected points in cyclic order.

So instead of checking all pairs, it is enough to check adjacent cyclic gaps.

This is the collapse from a k-clique-like pairwise problem to a one-dimensional cyclic spacing problem.

---

## 10. Binary Search on the Answer

Feasibility is monotonic.

If distance d is feasible, then any smaller distance is also feasible.

Formally:

\[
check(d)=true \Rightarrow check(d')=true\quad \forall d'\le d
\]

Therefore, we binary search d.

The search range is:

\[
0\le d\le side
\]

not:

\[
0\le d\le 2\cdot side
\]

because k >= 4 guarantees:

\[
answer\le side
\]

This upper bound is not just a minor optimization. It is part of why the one-dimensional cycle model is valid.

---

## 11. Greedy Feasibility on the Cycle

Now consider a fixed candidate distance d.

We have sorted perimeter positions:

\[
pos[0] < pos[1] < \dots < pos[n-1]
\]

To handle the cycle, duplicate the positions:

\[
pos[i+n] = pos[i] + P
\]

where:

\[
P=4\cdot side
\]

Now the cycle becomes a doubled line.

For each possible starting point s, we greedily choose the earliest next point whose perimeter position is at least d away from the last chosen point.

The greedy choice is natural:

Given a current selected point, choosing the earliest possible next point leaves the most remaining perimeter space for future selections.

This is the same exchange principle used in many one-dimensional spacing problems.

If a later point q can work as the next selected point, then the earliest valid point u before q is no worse, because it leaves at least as much room for the rest of the cycle.

The earlier confusion was whether this greedy choice is safe under Manhattan distance.

It becomes safe after the k >= 4 argument, because we have reduced the problem to perimeter spacing for d <= side.

So the greedy check is:

Start from each possible point.
Repeatedly jump to the earliest point at least d away.
Try to select k points.
After selecting k points, check whether the wrap-around gap back to the start is also at least d.

If yes, d is feasible.

---

## 12. Why This Is Not Reverse-Engineered

The derivation path is important:

1. The problem first appears to be a pairwise selection problem.
2. Pairwise selection suggests a k-clique-style graph formulation.
3. That is too hard in general.
4. Nearest-neighbor and single-center geometry attempts fail because they do not enforce all pairwise constraints.
5. The boundary condition suggests sorting points around the perimeter.
6. Opposite-side examples show that perimeter distance is not always Manhattan distance.
7. The condition k >= 4 proves answer <= side.
8. For d <= side, boundary cyclic spacing is enough.
9. The problem becomes selecting k points on a cycle with adjacent gaps at least d.
10. This admits a greedy feasibility check.
11. Binary search gives the maximum feasible d.

The key conceptual jump is not “use binary search.” The key jump is recognizing that the apparently general k-clique constraint collapses into a cyclic spacing constraint because all points lie on the square boundary and k >= 4.

---

## 13. Complexity

Let n be the number of boundary points.

Preprocessing:

- Map each point to a perimeter coordinate.
- Sort all coordinates.

This costs:

\[
O(n\log n)
\]

For each candidate d, a simple greedy check can be done by trying starts and jumping to the next coordinate at least d away.

With binary search over d:

\[
O(\log side)
\]

rounds are needed.

Depending on implementation, the feasibility check can be done in:

\[
O(nk\log n)
\]

using binary search for each jump, or improved with precomputed next pointers / two pointers.

Since k is at most small in this problem, the \(O(nk\log n)\) form is already structurally reasonable.

The important algorithmic complexity reduction is from:

\[
\binom{n}{k}
\]

or a general clique-style feasibility problem, down to a polynomial cyclic greedy check.

---

## 14. Main Pitfalls

The main pitfalls are:

1. Treating the problem as a general k-clique problem and getting stuck.
2. Trying to use nearest neighbors in the original point set.
3. Trying to use a single Manhattan diamond around one center point.
4. Assuming perimeter distance always equals Manhattan distance.
5. Forgetting that opposite-side points can have large perimeter distance but smaller Manhattan distance.
6. Missing the role of k >= 4.
7. Binary searching up to 2 * side instead of realizing answer <= side.
8. Checking only local geometric regions instead of using cyclic order.
9. Forgetting the final wrap-around gap on the cycle.

---

## 15. Final Summary

The final idea is:

Map every boundary point to its clockwise perimeter coordinate.

The square boundary becomes a cycle of length:

\[
4\cdot side
\]

Because:

\[
k\ge 4
\]

the optimal answer is at most:

\[
side
\]

For candidate distances:

\[
d\le side
\]

the Manhattan distance constraint on boundary points can be checked through cyclic perimeter spacing.

So the feasibility check becomes:

Can we select k points on the cycle such that adjacent selected points, including the wrap-around pair, are at least d apart?

This can be checked greedily.

Then binary search d.

The true reasoning path is:

\[
\text{pairwise Manhattan constraint}
\rightarrow
\text{k-clique-like warning}
\rightarrow
\text{boundary cycle}
\rightarrow
k\ge 4 \Rightarrow answer\le side
\rightarrow
\text{cyclic spacing}
\rightarrow
\text{greedy check}
\rightarrow
\text{binary search}
\]

This is why the problem is not a generic geometry problem and not a generic graph problem. It is a boundary-order problem whose key condition is k >= 4.
