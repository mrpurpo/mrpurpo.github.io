---
title: "Solving the Rook Exchange Problem"
date: 2026-11-21
draft: false
---

*Write-up of my work for the ENIM 2026 Demuestratón ([full paper (PDF)](/papers/rook-optimality.pdf), [verification code](https://github.com/mrpurpo/16rooks)).*

## The Problem

Consider a standard $8 \times 8$ chessboard with 8 white rooks placed along the first row and 8 black rooks along the last row, moving according to standard FIDE rules (without captures). What is the minimum number of moves required for the two sets of rooks to completely exchange positions, so that all white rooks end up on the eighth row and all black rooks end up on the first row?

From the statement alone, we can identify the natural parameters of the problem, which also give us a way to construct simpler variants or generalizations:

- **Board dimensions:** $m \times n$ (rows by columns, with $m \ge 3$ and $n \ge 2$; for $m = 2$ there are no empty squares and no piece can move).
- **Piece type:** king, knight, bishop, rook, queen, or pawn.

To formalize the rules, we assume two important things:
1. **Unlabeled pieces:** Rooks of the same color are indistinguishable. It does not matter whether the white rook starting on `a1` ends up on `a8`, `b8`, or `h8`, as long as the final configuration has all white rooks on row $m$ and all black rooks on row 1.
2. **No alternating turns:** Unlike a standard game of chess, we do not need to alternate between white and black moves; we can move any piece of either color at any step, and every traversed or destination square must be empty.

Our goal is to find a sequence of legal moves $M = (m_1, m_2, \dots, m_T)$ that reaches the target configuration while minimizing $T = |M|$.

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

Extending this pattern across the entire $8 \times 8$ board gives a solution in **33 moves** (grouping the final one-step descent onto row 1 at the end, as in the paper):

```text
1. Preparation (1 move):
   B a1-a3
2. Sweep columns b..h (21 moves):
   N b8-b2, N b2-a2, B b1-b8,
   N c8-c2, N c2-b2, B c1-c8,
   N d8-d2, N d2-c2, B d1-d8,
   N e8-e2, N e2-d2, B e1-e8,
   N f8-f2, N f2-e2, B f1-f8,
   N g8-g2, N g2-f2, B g1-g8,
   N h8-h2, N h2-g2, B h1-h8
3. Close column a (4 moves):
   N a8-a4, N a4-h4, B a3-a8, N h4-h1
4. Final descent to row 1 (7 moves):
   N g2-g1, N f2-f1, N e2-e1, N d2-d1, N c2-c1, N b2-b1, N a2-a1
```

More generally, each additional column after the first adds 4 moves (3 moves in the sweep plus 1 in the final descent), yielding a construction of **$4n + 1$ moves** ($1 + 3(n - 1) + 4 + (n - 1) = 4n + 1$) whenever the board has **$m \ge 5$ rows** and $n \ge 2$ columns.

## Dead Ends, Structure, and the $m \in \{3, 4\}$ Regime Shift

Working through the problem surfaced several traps that shaped the final proof:

- **The independence trap:** Assuming at first that decomposing the $8 \times 8$ board into smaller independent $8 \times 2$ sub-boards (36 moves) was optimal, before realizing that a single global cycle across all $n$ columns pays the initial column-clearing overhead only once instead of once per pair.
- **Swapping in-place vs. cyclic column shift:** It is tempting to think each black rook swaps places with the white rook directly below it. In the optimal 33-move construction, the white rooks in columns `b` through `h` move straight up their own columns (`b1-b8`, ..., `h1-h8`), while the black rooks shift cyclicly one column to the left (`b8` ends up on `a1`, `c8` on `b1`, ..., and `a8` wraps all the way across row 4 to `h1`).
- **The hidden row threshold ($m \ge 5$ vs. $m \in \{3, 4\}$):** Notice that the $4n + 1$ construction above uses **five distinct rows**: rows 1 and $m$, plus three intermediate "parking" rows in column `a` (the black rook arriving from column `b` parks on row 2, the white rook from `a1` waits on row 3, and the black rook from `a8` crosses along row 4). At first it looks like you might be able to compress those intermediate rows on a $3 \times n$ or $4 \times n$ board—but you cannot! On boards with only $m \in \{3, 4\}$ rows, there are at most two intermediate rows, forcing one extra move and raising the exact minimum to **$4n + 2$**.

This gives the complete piecewise formula for rooks ($m \ge 3, n \ge 2$):

$$T_{\min}(m, n) = \begin{cases} 4n + 1 & \text{if } m \ge 5, \\ 4n + 2 & \text{if } m \in \{3, 4\}. \end{cases}$$

## Proving the Lower Bound

Why is it impossible to beat $4n + 1$ moves (or 33 moves on $8 \times 8$)? The proof decomposes the total number of moves $T = H + V$ into horizontal moves $H$ and vertical moves $V$, and tracks which rooks make a **long vertical move** (a single vertical move all the way from row 1 to row $m$ or vice versa):

1. **Vertical moves ($V \ge 4n - L$):** Every rook must cover a vertical distance of $m - 1$ rows. A rook that makes at least one long move uses at least 1 vertical move; any other vertical move advances by at most $m - 2$ rows, so a rook without long moves needs at least 2 vertical moves. If $L$ is the number of rooks that make a long move, then
   $$V \ge L \cdot 1 + (2n - L) \cdot 2 = 4n - L.$$
2. **Horizontal exits per column ($\text{out}_j \ge \ell_j$):** Let $\text{out}_j$ be the number of horizontal moves leaving column $c_j$ (so $H = \sum_{j=1}^n \text{out}_j$) and $\ell_j$ be the number of rooks that make a long move in $c_j$ (so $L \le \sum_{j=1}^n \ell_j$). Because rooks cannot jump over each other, a long move in $c_j$ requires the entire column $c_j$ to contain no other rook at that turn. In particular, at least one original rook must exit $c_j$ before its first long move, and between any two successive long moves in $c_j$ the rook that just landed must exit $c_j$ before the next one can cross. These exits occur in disjoint time intervals, giving $\text{out}_j \ge \ell_j$ (and always $\text{out}_j \ge 1$).
3. **The identity for $T - 4n$ and the extremal principle:** Combining $T = H + V$ with the bounds above yields
   $$T - 4n = \bigl(V - (4n - L)\bigr) + \sum_{j=1}^n (\text{out}_j - \ell_j) + \left(\sum_{j=1}^n \ell_j - L\right),$$
   where all three terms on the right are non-negative integers. This immediately proves $T \ge 4n$. Could $T = 4n$? If so, every term would vanish, meaning $\text{out}_j = \ell_j \ge 1$ in *every* column $c_j$. A column with no slack ($\text{out}_j = \ell_j$) has a rigid structure: before its first long move at turn $\tau_j$, exactly one original rook exits $c_j$ and *no rook can enter $c_j$*. Choosing the column $c_a$ whose first long move $\tau_a = \min_j \tau_j$ happens earliest in the entire game gives a contradiction: the rook that exits $c_a$ before turn $\tau_a$ must enter some other column $c_b$ at a turn prior to $\tau_a \le \tau_b$, violating the no-entry condition on $c_b$. Therefore $T \ge 4n + 1$.

To prove $T \ge 4n + 2$ when $m \in \{3, 4\}$, we analyze the equality case $T = 4n + 1$. When $T - 4n = 1$, all the slack concentrates in a single exceptional column $c_*$ with $\ell_* = 0$ and $\text{out}_* = 1$, while every other column has $\text{out}_j = \ell_j \ge 1$. Using time-reversal and color-reflection symmetries, one can show that three distinct rooks pass through $c_*$ without making long moves—one entering rook $E$, one resident rook $Q$ that never leaves $c_*$, and one exiting rook $S$—and their vertical ordering forces them to park on three distinct intermediate rows $2 \le p_E < p_Q < p_S \le m - 1$ inside $c_*$. That requires at least three intermediate rows, which is only possible when $m \ge 5$.

## Variations with Other Chess Pieces on $8 \times 8$

The second part of the paper solves the exchange problem on the $8 \times 8$ board for all the remaining standard chess pieces, revealing a sharp structural divide between **leapers** (fixed-step pieces, which reduce to *multi-color pebble motion* on a fixed adjacency graph) and **sliders** (whose reach changes dynamically as intermediate squares are blocked):

| Piece | Minimum Moves ($8 \times 8$) | General / Structural Key |
| :--- | :---: | :--- |
| **Pawns** | $\infty$ (Impossible) | Without captures, pawns cannot change columns or pass the opposing pawn in the same column. |
| **Kings** | **112** | $T_{\min}(m, n) = 2n(m - 1)$ for $m \ge 3, n \ge 2$. Each of the $2n$ kings advances $m - 1$ rows at 1 row per move, bypassing the opposing king via two diagonal steps into an adjacent column with zero vertical waste. |
| **Queens** | **23** | Classical result by Tokarev (2004/2005). Splitting the 16 queens into 6 central column pairs (`b`–`g`, costing $\ge 3$ moves per pair since central diagonals cannot reach the opposite row) plus the 4 corner queens ($\ge 5$ moves) gives $6 \cdot 3 + 5 = 23$. |
| **Bishops** | **30** | Decomposes into dark-square and light-square bishops ($15 + 15$). On dark squares, only the main diagonal `a1–h8` connects rows 1 and 8 directly; 1 corner bishop crosses in 1 move once cleared, while the other 7 bishops take 2 moves each ($1 + 7 \cdot 2 = 15$). |
| **Knights** | **64** | Each jump changes the row by at most 2, so each of the 16 knights needs at least $\lceil 7/2 \rceil = 4$ moves ($16 \cdot 4 = 64$). Achieved via a centrally symmetric 64-move construction where every knight makes four forward jumps ($+2, +2, +2, +1$). |

## Computational Verification

While the proofs are purely analytical, I wrote a C++ verification tool, [`swaprows`](https://github.com/mrpurpo/16rooks), to independently test the theorems on smaller boards via exhaustive breadth-first search (BFS), count all optimal sequences, and validate all explicit constructions on $8 \times 8$:

- **Exact minima across 25 board sizes:** Exhaustive BFS on $m \times n$ boards up to $5 \times 5$ ($\approx 2.5 \times 10^8$ states; by comparison, $8 \times 8$ has $\binom{64}{8}\binom{56}{8} \approx 6.3 \times 10^{18}$ states) confirmed $T_{\min}(m, n) = 4n + 2$ for $m \in \{3, 4\}$ and $4n + 1$ for $m \ge 5$, including the exact regime jump between $m = 4$ and $m = 5$.
- **Rigidity of all optimal sequences:** Enumerating all **1,217,568** optimal $4n + 1$ sequences across six boards ($5 \times 2$, $6 \times 2$, $7 \times 2$, $8 \times 2$, $5 \times 3$, and $6 \times 3$) confirmed that *every single one* satisfies the three structural invariants from the equality-case proof (a unique exceptional column $c_*$ with $\ell_* = 0, \text{out}_* = 1$; exact vertical bound $V = 4n - L$; and three rooks parking on distinct intermediate rows in $c_*$).
- **Other pieces:** Exhaustive BFS on the $8 \times 8$ bishop subproblem ($\approx 1.2 \times 10^8$ states per square color, solved in 76 seconds) independently confirmed the 30-move minimum, while assignment lower bounds and move-sequence checks certified the optimal constructions for kings (112), knights (64), queens (23), and rooks (33).

You can read the complete proofs, diagrams, and explicit move tables in the [full paper (PDF)](/papers/rook-optimality.pdf) or explore the C++ solver in the [GitHub repository](https://github.com/mrpurpo/16rooks).

