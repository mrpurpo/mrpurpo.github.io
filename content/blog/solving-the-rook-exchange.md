---
title: "Solving the Rook Exchange Problem"
date: 2026-10-31
draft: true
---

*Write-up of my work for the ENIM 2026 Demuestratón.*

## The Problem

Consider a standard $8 \times 8$ chessboard with 8 white rooks placed along the first row and 8 black rooks along the last row, moving according to standard FIDE rules (without captures). What is the minimum number of moves required for the two sets of rooks to completely exchange positions, so that all white rooks end up on the eighth row and all black rooks end up on the first row?

From the statement alone, we can identify the natural parameters of the problem, which also give us a way to construct simpler variants or generalizations:

- **Board dimensions:** $m \times n$ (rows by columns).
- **Piece type:** king, knight, bishop, rook, or queen (pawns are excluded since they cannot change columns without capturing).

To formalize the rules, we assume two important things:
1. **Unlabeled pieces:** Rooks of the same color are indistinguishable. It does not matter whether the white rook starting on `a1` ends up on `a8`, `b8`, or `h8`, as long as the final configuration has all white rooks on row 8 and all black rooks on row 1.
2. **No alternating turns:** Unlike a standard game of chess, we do not need to alternate between white and black moves; we can move any piece of either color at any step.

Our goal is to find a sequence of legal moves $M = (m_1, m_2, \dots, m_k)$ that reaches the target configuration while minimizing $|M| = k$.

## First Contact: Starting with an $8 \times 2$ Toy Example

With nothing more than the problem statement, my first instinct was to sit down with pen and paper. Because keeping track of all 16 rooks on an $8 \times 8$ board at once was too messy by hand, I decided to reduce the problem to an $8 \times 2$ board (8 rows and 2 columns, `a` and `b`) with 2 white rooks on row 1 and 2 black rooks on row 8. Any valid algorithm for the $8 \times 2$ board immediately gives an upper bound for the $8 \times 8$ board simply by repeating it 4 times across pairs of columns.

If rooks could pass through one another without obstacles, each of the 4 rooks would only need a single straight vertical move to reach the other side, taking 4 moves in total.

Once we account for collisions, a rook cannot cross its column until the opposing rook in the same column gets out of the way. My first idea on paper was a symmetric pattern: to let a black rook reach the bottom row in a single move, the white rook in its column first steps forward (`a1-a2`) and then sideways into the adjacent column (`a2-b2`), clearing column `a`. Applying this idea symmetrically to each rook takes 3 moves per rook, or **12 moves** in total:

```text
(a1-a2, a2-b2, b8-b7, b7-a7, b2-b8, a7-a1, b1-b2, b2-a2, a8-a7, a7-b7, a2-a8, b7-b1)
```

However, playing around with variations of this pattern quickly showed that symmetry was wasting moves. As soon as we clear a column, the opposing rook on that column does not need to step out of the way first; it can immediately travel all the way to its final square in a single turn! In doing so, it both reaches its destination and vacates its starting square for the next piece. Using this observation drops the count to **10 moves**:

```text
(a1-a2, a2-b2, a8-a1, b2-a2, a2-a8, b1-b2, b2-a2, b8-b1, a2-b2, b2-b8)
```

Here, the 2 black rooks only make 1 move each (`a8-a1` and `b8-b1`), while the 2 white rooks do all the maneuvering (4 moves each: 2 to clear the column and 2 to return and cross).

Finally, we can shave off one more move by starting with `a1-a3` instead of `a1-a2`. The key insight is that by pushing the first white rook one square further up, the black rook on `b8` can drop down to `b2` and tuck into `a2` *behind* the white rook on `a3`. In a single maneuver, it clears column `b` (allowing `b1-b8` in one move), has a clear path down to `a1`, and never blocks the white rook on `a3`. This gives a solution in just **9 moves**:

```text
(a1-a3, b8-b2, b2-a2, a2-a1, b1-b8, a8-a7, a7-b7, b7-b1, a3-a8)
```

## Scaling Up to the $8 \times 8$ Board: From 36 to 33 Moves

With the 9-move solution for the $8 \times 2$ board in hand, we can simply apply it to 4 independent pairs of columns to solve the full $8 \times 8$ board in $9 \times 4 = 36$ moves.

Can we do better than 36? Notice why the $8 \times 2$ board took 9 moves instead of 8: we paid 1 extra setup move at the beginning (`a1-a3`) to "park" a rook out of the way, and then resolved the rest of the columns in a chain. Instead of paying that setup cost 4 separate times on four $8 \times 2$ strips, we can pay it *once* on column `a` and chain the column-clearing maneuver across all 8 columns from `b` through `h` before closing the loop with `a8` and `a3-a8`.

Extending this pattern across the entire $8 \times 8$ board gives a solution in **33 moves**:

```text
(a1-a3,
 b8-b2, b2-a2, a2-a1, b1-b8,
 c8-c2, c2-b2, b2-b1, c1-c8,
 d8-d2, d2-c2, c2-c1, d1-d8,
 e8-e2, e2-d2, d2-d1, e1-e8,
 f8-f2, f2-e2, e2-e1, f1-f8,
 g8-g2, g2-f2, f2-f1, g1-g8,
 h8-h2, h2-g2, g2-g1, h1-h8,
 a8-a7, a7-h7, h7-h1, a3-a8)
```

More generally, each additional column after the first adds 4 moves (3 moves for the top rook to snake around into the previous column's bottom square, plus 1 direct move for the bottom rook to shoot straight up to the top row), yielding a general construction of **$4n + 1$ moves** for any $m \times n$ board with $m \ge 3$ and $n \ge 2$.

---

## Outline for the Remaining Sections (To Do)

### 1. Dead Ends and Mistakes Along the Way
- **The independence trap:** Assuming at first that decomposing the $8 \times 8$ board into smaller independent sub-boards ($8 \times 2$) would be optimal, before realizing that a single global cycle across all $n$ columns saves 1 move per boundary.
- **Labeled vs. unlabeled pieces:** Initially thinking of rooks as swapping with the rook directly opposite them in the same column, rather than shifting columns cyclically (notice how in the 33-move solution, the black rook from `b8` actually ends up on `a1`, `c8` ends up on `b1`, and `a8` wraps around to `h1`!).
- **Early lower-bound misconceptions:** Any initial guesses at the minimum number of moves that turned out to be too high or too loose.

### 2. Proving the Lower Bound ($4n + 1$ is Optimal)
- **The unobstructed baseline ($2n$ moves):** Every one of the $2n$ rooks starts on the wrong row and must make at least 1 vertical move, giving a trivial lower bound of $2n$.
- **Why every column requires horizontal detours (+$2n$ moves):** Since each column starts with both a white and a black rook facing each other, neither can reach the opposite row in 1 move while the other remains in the column. At least one rook per column must leave its column horizontally and later a rook must enter horizontally to fill the vacated target square.
- **The +1 tie-breaker move:** Why the very first column to be cleared requires an extra intermediate move (giving $4n + 1$ rather than $4n$), completing the proof that 33 moves cannot be beaten on $8 \times 8$.

### 3. Computational Verification & AI Assistance
- How a program / search algorithm (BFS or SAT/MIP solver on small $m \times n$ boards) and AI assistance were used to test small cases ($3 \times 2$, $4 \times 3$, etc.) and confirm that no hidden shortcut beats $4n + 1$.

### 4. Extensions to Other Pieces
- **Kings and Queens:** How diagonal moves change the bottleneck.
- **Bishops and Knights:** Color-complex constraints for bishops and graph shortest-path formulation for knights.
