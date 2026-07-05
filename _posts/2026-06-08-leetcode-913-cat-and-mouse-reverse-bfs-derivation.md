---
title: "LeetCode 913 Cat and Mouse - Complete Derivation via Reverse BFS"
date: 2026-06-08
categories: [Algorithm, game-theory, LeetCode]
tags: [reverse-bfs, game-tree, dynamic-programming, leetcode-hard]
---

## Problem Statement
LeetCode 913 "Cat and Mouse" is a two-player game on an undirected graph. **Rules (reminder):** Nodes are labeled `0 .. n`. Node `0` is the mouse's home (safe hole). Mouse starts at node `1`. Cat starts at node `2`. Mouse moves first. Players alternate turns. On each turn, the player must move to an adjacent node (cannot stay in place). If mouse reaches node `0` → mouse wins immediately. If cat and mouse are on the **same node** (and that node is not `0`) → cat wins immediately. Cat **cannot** enter node `0`. Both players play optimally. Return: `1` if mouse wins, `2` if cat wins, `0` if the game is a tie (infinite loop).

## State Definition
We define a **game state** as `(cat, mouse, turn)` where: `cat` ∈ `[1..n]` (cat cannot be at `0`), `mouse` ∈ `[0..n]` (mouse can be at `0`, which is terminal), `turn` ∈ `{0, 1}` with `0` = mouse's turn to move, `1` = cat's turn to move. Total number of possible states: at most `50 × 51 × 2 ≈ 5100`. This is small enough to build an explicit state graph.

## Terminal States (Known Outcomes)
The following states have **no outgoing edges** in the forward game graph (because the game ends immediately): (1) **Mouse wins**: `mouse == 0` (cat can be anywhere); (2) **Cat wins**: `cat == mouse` and `mouse != 0`. These are the **sources** for reverse propagation.

## Why Reverse BFS Instead of Forward Minmax?
A forward minmax with memoization would need to handle cycles carefully. Cycles in the state graph correspond to potential infinite play (ties). A naive recursive approach can get stuck or require complex three-color marking. **Reverse BFS** avoids this problem: start from **terminal states** (whose outcomes are known), propagate backwards along reverse edges, and any state that never gets reached from a terminal outcome is a tie. This is essentially a topological-like propagation on a graph with cycles, but we don't need to explicitly "detect" cycles — the propagation algorithm naturally leaves them unmarked.

## Building the State Graph
We first construct the **forward state graph**: nodes are all `(cat, mouse, turn)`. Directed edges go from a state to its possible **next states** according to game rules. If `turn == 0` (mouse moves): for each neighbor `m2` of `mouse` (including `0`), if moving to `m2` does not immediately end the game (we handle termination separately), then edge to `(cat, m2, 1)`. If `turn == 1` (cat moves): for each neighbor `c2` of `cat` where `c2 != 0`, edge to `(c2, mouse, 0)`. **Terminal states** have no outgoing edges in this forward graph because the game ends before another move.

## Reverse Propagation Rules
We propagate **from known outcomes backward** to their predecessors. Let `result(state)` be: `MOUSE_WIN`, `CAT_WIN`, or `UNKNOWN` (could be tie or not yet determined). We maintain a queue of states whose outcome has been determined but whose predecessors have not yet been processed.

**Initialization**: All terminal states (mouse at `0` → `MOUSE_WIN`, cat meets mouse → `CAT_WIN`) are pushed into the queue.

**Propagation Rules (Critical)**: When we pop a state `S`, we look at **all predecessors** `P` in the **reverse graph** (i.e., states that can move to `S` in one step). For each predecessor `P = (cat, mouse, turn)`, we decide if `P`'s outcome can now be determined.

If `turn == 0` (it was mouse's turn in `P`): mouse chooses the next state. If **any** child (successor) of `P` is `MOUSE_WIN` → then `P` is `MOUSE_WIN` (mouse will choose that move). If **all** children of `P` are `CAT_WIN` → then `P` is `CAT_WIN` (mouse has no winning move, and all moves lead to cat win). Otherwise (some children unknown, or mix of unknown and not-all-cat-win) → `P` remains `UNKNOWN` for now.

If `turn == 1` (it was cat's turn in `P`): cat chooses the next state. If **any** child of `P` is `CAT_WIN` → then `P` is `CAT_WIN`. If **all** children of `P` are `MOUSE_WIN` → then `P` is `MOUSE_WIN`. Otherwise → `UNKNOWN`.

**How to Implement "All Children Known"**: We need to track, for each state `P`: `total_children` = number of legal moves from `P`, and `resolved_children` = number of children whose outcome is known (`MOUSE_WIN` or `CAT_WIN`, not `UNKNOWN`). When `resolved_children == total_children`, we have all outcomes available. Then we apply the above rules. If rule yields `MOUSE_WIN` or `CAT_WIN` → we determine `P`, push `P` to queue. If after full propagation `P` is still `UNKNOWN`, it is a tie.

## Termination and Tie
The reverse BFS runs until the queue is empty. After that, any state with `result == MOUSE_WIN` or `CAT_WIN` is resolved, and any state still `UNKNOWN` is a **tie** (return `0`). The initial state is `(cat=2, mouse=1, turn=0)` (mouse starts). We look up its final result.

## Summary of the Algorithm
1. Enumerate all states `(cat, mouse, turn)`. 2. Identify terminal states (mouse at `0` → mouse win; cat meets mouse → cat win). Push them into a queue. 3. Build forward adjacency (children) and reverse adjacency (parents). 4. Maintain counts: for each state, track number of children and how many children are resolved. 5. Process queue: pop a resolved state `S`, for each parent `P` of `S`, increment `P`'s resolved counter. If `P`'s resolved counter == total children, apply turn-based rule to determine `P`'s outcome. If determined (mouse win or cat win), push `P` to queue. 6. After queue empty: any state not determined → tie. 7. Answer: result of initial state.

## Complexity
States: `O(n²)`. Transitions: each state has degree bounded by max degree of original graph. Total edges: `O(n² × Δ)`. With `n ≤ 50`, this is trivial.

## Connection to Minmax and Game Theory
The forward minmax definition is: For state with `turn == 0` (mouse): `value = MOUSE_WIN` if any child == `MOUSE_WIN`, `value = CAT_WIN` if all children == `CAT_WIN`, else unknown or tie. For state with `turn == 1` (cat): `value = CAT_WIN` if any child == `CAT_WIN`, `value = MOUSE_WIN` if all children == `MOUSE_WIN`, else unknown or tie. The reverse BFS is exactly an **efficient dynamic programming** that computes this fixpoint on a graph with cycles, without explicit recursion or cycle detection. Cycles that never reach a terminal become ties automatically.

## Discussion: What We Learned from Derivation
This problem is **not** World Final level — more like ICPC Regional or Codeforces ~2300. But it requires precise thinking about: state space modeling (including who moves), terminal conditions, direction of propagation, turn-dependent propagation rules, and handling "all children known" as a trigger. The reverse BFS approach is clean and robust, avoiding explicit cycle handling.

## References
- [LeetCode 913 - Cat and Mouse](https://leetcode.com/problems/cat-and-mouse/)
- Standard solution: reverse BFS + degree counting
- This derivation came from a Socratic "battle" with an AI, step by step, clarifying each ambiguity.

## Full Code Sketch (Python-like)
```python
def catMouseGame(graph):
    n = len(graph)
    MOUSE_WIN, CAT_WIN, TIE = 1, 2, 0
    result = [[[0]*2 for _ in range(n)] for __ in range(n)]
    degree = [[[0]*2 for _ in range(n)] for __ in range(n)]
    for cat in range(n):
        for mouse in range(n):
            degree[cat][mouse][0] = len(graph[mouse])
            degree[cat][mouse][1] = len([c for c in graph[cat] if c != 0])
    from collections import deque
    q = deque()
    for cat in range(1, n):
        for turn in [0,1]:
            result[cat][0][turn] = MOUSE_WIN
            q.append((cat,0,turn))
            for m in range(1, n):
                if cat == m:
                    result[cat][m][turn] = CAT_WIN
                    q.append((cat,m,turn))
    while q:
        cat, mouse, turn = q.popleft()
        cur_res = result[cat][mouse][turn]
        if turn == 0:
            for pre_cat in graph[cat]:
                if pre_cat == 0: continue
                # update degree and check for predecessor (pre_cat, mouse, 1)
        else:
            for pre_mouse in graph[mouse]:
                # update degree and check for predecessor (cat, pre_mouse, 0)
        # full implementation requires careful degree counting and rule checks
    return result[2][1][0]


## Final Thought
The hardest part of this problem is not coding — it's convincing yourself that the reverse propagation rules are correct for both players and that unmarked states indeed correspond to ties. Once that's clear, the implementation is straightforward. This blog post records the exact derivation process, including all the back-and-forth corrections. Hopefully it helps others who struggle with the same conceptual gaps.
