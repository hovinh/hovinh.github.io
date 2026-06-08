---
#layout: post
title: "Turing Machine: The Board Game That Makes You Think Like a Computer"
description: >
  A deep dive into how the board game Turing Machine works — from its punch card mechanism and Verification card design to puzzle generation and the genuine computer science concepts playing out on your table.
author: author1
comments: true
---

The name alone caught my attention. *Turing Machine* — a board game named after one of the most important theoretical constructs in computer science. I came across reviews describing how it used physical punch cards as a coding device, and I was curious enough to buy a copy and find out whether the connection was more than just a marketing nod.

After playing it and eventually passing it on, I found myself still wanting to fully decode how it worked. This post is that decoding. We go from the rules of the game, through the engineering of its physical components, through the mathematics of puzzle generation, and finally to the conceptual core: what it actually means that this game is named after a Turing Machine.

- [The Game: Rules and Key Concepts](#the-game-rules-and-key-concepts)
- [The Punch Card Mechanism](#the-punch-card-mechanism)
- [How the Verification Card Works](#how-the-verification-card-works)
- [How Puzzles Are Generated](#how-puzzles-are-generated)
- [The Real Turing Connection](#the-real-turing-connection)
- [Summary](#summary)

---

## The Game: Rules and Key Concepts

*Turing Machine* is a deduction game for 1–4 players designed by Yoann Levet and Fabien Gridel (Scorpion Masqué, 2022). A game takes about 20 minutes. The premise is simple: find a secret 3-digit code before your opponents do.

If you have played classic code-breaking games — where you guess a combination and coloured pegs tell you how many digits are in the right position or exist somewhere in the code — think of this as a more sophisticated cousin. In those games, the criteria are fixed and transparent from the start. In *Turing Machine*, the criteria themselves are hidden, and figuring out what each criterion even *is* forms half the puzzle.

### The Code, the Verifiers, and the Punch Cards

The secret **Code** is a 3-digit sequence — a triangle digit, a square digit, and a circle digit — each between 1 and 5. For example: triangle=2, square=4, circle=1.

The code is not revealed directly. Instead, the puzzle is set up with **Verifiers** — fictional AI judges, each checking one hidden **criterion** about the code. A criterion might be *"the triangle number is even"* or *"square is the smallest of the three digits"*. Players do not know which criterion each Verifier is checking. That is what they must deduce.

Each Verifier has a **Criteria card** placed face-up. It tells you the *category* of check — for example, "this Verifier compares one digit to a fixed number" — and lists 2–3 possible criteria. Exactly one is the true hidden criterion for this puzzle. The actual hidden answer is encoded on a face-down **Verification card**, which players interact with physically each round.

The **punch cards** are the 45 cards players use to compose proposals — three sets of 15 cards numbered 1–5, one set per digit shape (triangle, square, circle). Selecting one card from each set and overlaying them forms a 3-digit proposal to test against the Verifiers.

![Fig01](/assets/blog/2026-06-07/game_concept_overview.png){:data-width="1440" data-height="640"}
Fig. 1. How a puzzle is structured, using Problem 01 from the rulebook. The secret code (triangle=2, square=4, circle=1) satisfies all four hidden criteria simultaneously. The example query shows how a failing answer still narrows down the possibilities.
{:.figure}

### Round Structure

Each round, every player acts simultaneously and independently:

![Fig02](/assets/blog/2026-06-07/round_flow.png){:data-width="1440" data-height="560"}
Fig. 2. The four steps of each round in Turing Machine.
{:.figure}

**1. Compose** — Select one triangle card, one square card, and one circle card (each numbered 1–5) and overlay them to form a 3-digit proposal. Write it on your note sheet.

**2. Question** — Query up to 3 Verifiers using your proposal. For each query, place the Verifier's Verification card under your stacked punch cards and look through the single hole that appears. A pass means your proposal satisfies that Verifier's criterion. A fail means it does not. Record every result — this is mandatory and determines tiebreakers.

**3. Deduce** — Analyse your answers. Cross off eliminated possibilities in your note sheet's digit grids. Write down criteria you have confirmed.

**4. End of Round** — Everyone simultaneously signals thumb-up (confident they know the code) or thumb-down (still working). If one or more players signal up, they write their code secretly and check the solution. The player who found the correct code using the fewest total questions wins.

### The key design guarantee

Two constraints make the game work: first, **only one code satisfies all criteria simultaneously**. Second, **every Verifier is necessary** — no Verifier repeats information that the others already cover. These are enforced mathematically at puzzle generation time, as we will see later.

---

## The Punch Card Mechanism

Before we can understand the Verification card, we need to understand what punch cards do as a general mechanism — and crucially, what problem they are solving.

### The core idea: mechanical lookup

A punch card system is a way to **retrieve a pre-stored answer for a given combination of inputs, purely through physical means — no arithmetic, no logic gates, no electricity**.

The idea is to do the "computation" in advance and encode the result spatially into the physical structure of the card itself. A hole at a certain position means *yes*; the absence of a hole means *no*. To retrieve the answer, you simply find the right position on the card and observe whether a hole is there or not.

This matters because it separates two concerns that we often conflate: *answering* a question (retrieving a stored result) versus *reasoning* about a question (deriving a result on the fly). Punch cards handle the first. They are lookup devices, not reasoning engines — and that distinction becomes important when we get to the Turing connection.

The key principle: **position = meaning, hole = yes, no hole = no**.

### From a simple list to a two-dimensional table

The best way to build intuition is to start small and scale up.

**One parameter — a simple list:** Suppose you have 5 students and want to record whether each passed or failed. Make one card per student: punch a hole if they passed, leave it solid if they failed. To answer "did student 3 pass?" — pick card 3, look at the hole. Done. No arithmetic needed.

![Fig03](/assets/blog/2026-06-07/punch_card_1param.png){:data-width="1440" data-height="500"}
Fig. 3. One-parameter lookup. Each card encodes a single pre-stored answer via the presence or absence of a hole.
{:.figure}

**Two parameters — a table:** Now suppose you want to look up any entry in a 5×5 multiplication table without doing arithmetic. The naive approach would be one card per combination — 25 cards total. That is not terrible for a 5×5 table, but for a 10×10 table it becomes 100 cards; for larger tables, the count grows as N². More importantly, this approach treats every (row, column) pair as an independent fact with no structure, when in reality there is clear structure we can exploit.

The punch card insight is to design two separate sets of cards — one set for rows, one for columns — and let them work together:

A **row card** for number N opens its entire row — all 5 column positions in that row are punched through, all other rows are solid. A **column card** for number N opens its entire column — all 5 row positions are punched through, all other columns are solid.

Now place a row card on top of a column card on top of the pre-printed answer grid. The only position where both cards are open simultaneously — where light passes through both layers — is the intersection of that row and column. Exactly one cell survives. That cell has the answer pre-printed.

Why *exactly* one cell? This is the key geometric property. Row cards and column cards divide the grid into two orthogonal partitions — like a grid of streets where rows run east-west and columns run north-south. Any east-west strip and any north-south strip cross at exactly one intersection. No cell can be in two different rows at once, and no cell can be in two different columns at once. So there is no way to get zero intersections (the target cell is always in both) or more than one (no other cell shares both the same row *and* the same column).

![Fig04](/assets/blog/2026-06-07/punch_card_2param.png){:data-width="1440" data-height="580"}
Fig. 4. Two-parameter lookup: stacking a row card and a column card leaves exactly one cell exposed on the pre-printed answer grid.
{:.figure}

**Three parameters — the Turing Machine case:** The game has 3 digits (triangle, square, circle) each ranging 1–5, giving 5 × 5 × 5 = 125 possible proposals. The physical card is 2D, so the three parameters must be folded onto two axes:

- **Rows (5 positions):** the triangle digit
- **Columns (25 positions):** square and circle combined — 5 bands of 5 sub-columns each

This gives a **5 × 25 = 125 cell grid**, one cell per unique proposal. The three punch cards each eliminate one independent dimension:

| Card | What it opens | Holes open |
|---|---|---|
| Triangle card (value N) | Entire row N — all 25 columns | 25 |
| Square card (value N) | Entire band N — all 5 rows × 5 sub-cols | 25 |
| Circle card (value N) | Sub-col N within every band — all 5 rows | 5 |

Stacking all three leaves exactly one surviving cell — the one at (row triangle, band square, sub-col circle). That cell is pre-printed with pass or fail for the criterion encoded on the Verification card.

![Fig05](/assets/blog/2026-06-07/punch_card_stacking.png){:data-width="1440" data-height="560"}
Fig. 5. Three-card addressing in Turing Machine. Each card eliminates one independent dimension. Their intersection is always exactly 1 of the 125 cells.
{:.figure}

---

## How the Verification Card Works

A **Verification card** is the pre-printed answer grid for one specific criterion. It encodes a complete truth table: for every one of the 125 possible proposals, it stores whether that proposal passes or fails the criterion.

### Building a Verification card from scratch

Take a simple criterion: *"triangle is even"*.

For every possible proposal (triangle, square, circle), evaluate whether the triangle digit is in {2, 4}. This produces 125 binary answers — 50 passes and 75 failures. Map these onto the 5×25 grid. Since the criterion depends only on triangle (the row), the result is clean: rows 2 and 4 are entirely pass, all other rows are entirely fail.

Now take a complex criterion: *"square is the smallest digit"*, meaning square < triangle AND square < circle. This depends on all three digits simultaneously. The pass cells scatter irregularly across the grid — clustering toward the top of each band, vanishing entirely in the square=5 band (since no digit can be smaller than 5). Only 20 of 125 cells pass.

In both cases the physical card is identical in format — a 5×25 grid. The criterion complexity only affects the *pattern* inside it.

![Fig06](/assets/blog/2026-06-07/verification_card_patterns.png){:data-width="1440" data-height="540"}
Fig. 6. Two Verification cards side by side. Left: "triangle is even" — clean horizontal stripes. Right: "square is smallest" — irregular scatter. Same 5×25 format, very different truth table.
{:.figure}

---

## How Puzzles Are Generated

Understanding the physical mechanism raises the next question: how do the designers ensure that any given combination of Verifiers produces a puzzle with exactly one solution, with no Verifier being redundant? The answer is a clean combinatorial algorithm.

### The two validity conditions

A puzzle is valid if and only if:

1. **Unique solution:** Exactly one code satisfies all criteria simultaneously.
2. **No redundancy:** Remove any single Verifier and the solution is no longer unique — at least two codes survive.

These are enforced computationally, not manually.

### Criteria as bit vectors

Each criterion is represented as a **125-bit vector** — one bit per possible code, set to 1 if that code passes the criterion and 0 if it fails. The full library has 95 such vectors.

Given a candidate set of criteria, the surviving solution space is their **bitwise AND**:

```
solution = criterion_A AND criterion_B AND criterion_C AND criterion_D
```

Count the 1-bits. A valid puzzle has exactly one. This is computationally cheap — a single pass over 125 bits — making exhaustive search over millions of combinations feasible.

### The generation algorithm

```
1. Pick target code (triangle, square, circle)
        ↓
2. Find all 95 criteria compatible with the target
   (those whose bit vector has a 1 at the target's position)
        ↓
3. Search combinations of 4–6 compatible criteria
   whose bitwise AND = exactly 1 surviving code
        ↓
4. For each candidate set, test non-redundancy:
   remove each criterion in turn — does the AND expand?
        ↓
5. Compute difficulty metrics:
   min questions for optimal solver + variance by query order
        ↓
6. Output: valid, rated, non-redundant puzzle
```

The search space for 4-Verifier puzzles is C(95, 4) ≈ 3.26 million combinations. Early termination — pruning candidate sets the moment their intersection exceeds one non-target code — makes this tractable. For 6-Verifier puzzles the space reaches ~735 million, but the same pruning applies.

### What makes a puzzle hard?

Not all criteria are equally informative. The key metric is **passing set size P** — how many of the 125 codes satisfy the criterion. A criterion with P near 63 (≈ 50%) is most informative: it cuts the solution space roughly in half per query regardless of whether the answer is pass or fail. Very low P is nearly a giveaway; very high P barely filters anything.

![Fig07](/assets/blog/2026-06-07/passing_set_size.png){:data-width="1440" data-height="560"}
Fig. 7. Criterion informativeness by passing set size P. Criteria near P=63 halve the solution space per query. Extremes at either end make for weaker puzzles.
{:.figure}

But P alone is not enough. Two criteria with identical P can be heavily **correlated** — they eliminate mostly the same codes — or largely **orthogonal** — they eliminate different regions. The best puzzles pair orthogonal criteria that independently address different dimensions: one checking a row property (triangle alone), one a band property (square alone), one a sub-column property (circle alone), and one a cross-digit relationship.

The **Difficulty Factor** on each Problem card is the minimum number of questions an optimal solver needs. The **Luck Factor** is the variance across query orderings — a high luck factor means one particular sequence of queries is much faster than others, introducing a chance element even for skilled players.

---

## The Real Turing Connection

Here is the central observation, which you may have already sensed: the punch cards are not what makes this game *Turing Machine*. They are a historical metaphor — a nod to the era of early computing. The actual Turing connection runs much deeper.

### What a Turing Machine is

In 1936, Alan Turing proposed an abstract model of computation so simple it seems almost trivial — and yet powerful enough to describe anything a modern computer can do. The model has four components:

1. A **tape** — an infinite sequence of cells, each holding a symbol. This is the memory: the complete state of the world.
2. A **read head** — a pointer to one cell at a time. It reads the symbol there, then moves one step left or right.
3. An **internal state** — a finite register encoding what the machine "knows so far." It changes at every step.
4. A **transition function** — a lookup table that says: given (current state + symbol just read), write a new symbol, move the head, and transition to a new state. Or halt.

The machine runs this loop indefinitely until it hits a halt condition. Nothing more.

![Fig08](/assets/blog/2026-06-07/turing_machine_model.png){:data-width="1440" data-height="580"}
Fig. 8. The abstract Turing Machine model, with a concrete example: finding the first 1 on a tape of 0s and 1s. The machine reads one symbol at a time, updates its state, and halts when the condition is met.
{:.figure}

Turing's insight was that *any* computation — sorting, arithmetic, language parsing, chess-playing — can be reduced to this loop. Complexity emerges not from the mechanism itself but from the transition rules and the length of the tape. The model is minimal by design: strip away everything non-essential and this is what remains.

### The player is the machine

Now map the game onto this model:

![Fig09](/assets/blog/2026-06-07/game_as_turing_machine.png){:data-width="1440" data-height="560"}
Fig. 9. Every element of the abstract Turing Machine has a direct counterpart in the game. The player is running the machine.
{:.figure}

Each round, the player reads one bit from the world — a pass or fail answer from a Verifier — then updates their state by crossing off codes that are now impossible. The machine halts when **entropy reaches zero**: that is, when the number of remaining candidate codes has been reduced from 125 to exactly 1. At that point there is nothing left to be uncertain about. The tape holds one symbol, the state is FOUND, and the player signals thumb-up.

The note sheet is not just a score tracker. It is the tape of the Turing Machine made visible — a physical record of every state transition the player has made since the game began.

### The information-theoretic lower bound

The solution space has 125 codes, requiring at minimum log₂(125) ≈ 6.97 bits to uniquely identify. Since each query returns exactly 1 bit (pass or fail), at least 7 perfectly efficient questions are needed. The game's 3-queries-per-round budget and typical 3–4 round solve time (9–12 questions total) sits close to this theoretical minimum — accounting for the fact that real queries are rarely perfectly informative.

The Difficulty Factor is precisely this minimum: the fewest binary queries needed to guarantee halting, which is the computational complexity of that specific constraint satisfaction instance.

### The game is harder than it looks: constraint learning

At its core the game is a **constraint satisfaction problem (CSP)**:

- **Variables:** triangle, square, circle each with domain {1, 2, 3, 4, 5}
- **Constraints:** the hidden criterion each Verifier checks
- **Goal:** find the unique assignment satisfying all constraints

But standard CSP assumes the constraints are *known*. Here they are *hidden*. The player must simultaneously **learn the constraints** and **satisfy them**:

```
Standard CSP:      known constraints → find solution
Turing Machine:    unknown constraints → learn constraints → find solution
```

This double loop — inferring what the rules are while using inferred rules to eliminate solutions — is what separates skilled players from beginners. A fail answer is often more valuable than a pass, because failing a criterion with a large passing set eliminates more candidates in one step.

### The Bletchley connection

There is a second Turing reference worth drawing out. Turing is famous not only for the abstract machine but for his wartime codebreaking at Bletchley Park, where he led the effort to crack the German Enigma cipher.

The challenge Bletchley faced was a combinatorial explosion: the Enigma machine had roughly 10^23 possible settings at any given moment. Brute force was completely infeasible — even checking one setting per second, it would take longer than the age of the universe. The bombe machine Turing designed did not try every possibility. Instead, it exploited the logical *structure* of the problem: known fragments of plaintext (called cribs) placed constraints on what the rotor settings could be, and the bombe propagated those constraints mechanically, using contradictions to eliminate whole regions of the search space in one move. What remained after propagation was a drastically smaller set of candidates to check manually.

That is exactly what a skilled *Turing Machine* player does each round. You are not enumerating 125 codes one by one. You are exploiting the structure of each answer — pass or fail against a known criterion category — to eliminate whole regions of the solution space at once. Every crossed-off digit value on your note sheet is a constraint propagated. The note sheet is the bombe in miniature.

### Why the punch card is set dressing

The punch card answers a pre-computed question mechanically. It contributes no reasoning, no constraint propagation, no state update. It is the equivalent of a dictionary lookup — useful infrastructure, but not the intelligence.

The intelligence is entirely in the player's reasoning process. The punch card honours the historical setting — Turing's era *did* use punch cards as the primary input/output medium for early computers. It is thematically accurate and physically satisfying. But the soul of the game is the logic engine the player runs internally, round by round, until they halt.

---

## Summary

*Turing Machine* layers three things on top of each other, and they are worth keeping distinct.

**The physical mechanism** — punch cards implement mechanical lookup via orthogonal set intersection. Three stacked cards address one of 125 cells in a pre-printed truth table. It is elegant engineering, but it is a lookup device, not a computer.

**The puzzle design** — each puzzle is a carefully generated constraint satisfaction instance. Validity (unique solution) and non-redundancy are enforced algorithmically using bitwise AND over 125-bit criterion vectors. Difficulty is measured by information-theoretic efficiency and query-order variance.

**The conceptual core** — the player is a Turing Machine. Each query reads one bit. The note sheet is the tape. Eliminating codes is the state transition. Finding the solution is the halt. The game is simultaneously a Turing Machine computation and a codebreaking exercise in the tradition of Bletchley Park.

What makes the game elegant is that it achieves all of this without ever requiring the player to know any of it. You can sit down, learn the rules in five minutes, and have a genuinely satisfying logical experience — while unknowingly simulating one of the most fundamental models in theoretical computer science.

That, to me, is what makes it worth decoding.

---