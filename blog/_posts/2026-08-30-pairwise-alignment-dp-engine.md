---
layout: post
title: "Sequence Alignment: One DP Engine, Thirteen Rosalind Problems"
description: >
  Sequence alignment on Rosalind looks like thirteen unrelated puzzles, until you notice
  most of them are the same dynamic programming recurrence wearing different clothes.
  This post traces how one DP engine, parameterized by a scoring function, a gap model,
  and four free-start/free-end flags, grows from counting point mutations into global,
  local, overlap, semiglobal, affine-gap, and multiple alignment.
author: author1
comments: true
---

**Prerequisite**: dynamic programming, Python.

- [A Few Concepts First](#a-few-concepts-first)
- [Ground Zero: Comparing Sequences Without Alignment](#ground-zero-comparing-sequences-without-alignment)
- [Allowing Indels: A DP Table for Edit Distance](#allowing-indels-a-dp-table-for-edit-distance)
- [One Table, Three Questions: Score, Witness, All Witnesses](#one-table-three-questions-score-witness-all-witnesses)
- [Real Biology Enters: Scoring Matrices and Needleman-Wunsch](#real-biology-enters-scoring-matrices-and-needleman-wunsch)
- [When an Indel Is One Event, Not Many](#when-an-indel-is-one-event-not-many)
- [Finding an Island: Local Alignment](#finding-an-island-local-alignment)
- [Local vs. Global: A Decision Framework](#local-vs-global-a-decision-framework)
- [The Many Shapes of "Aligned": Free Ends](#the-many-shapes-of-aligned-free-ends)
- [Modeling Real Indels: Affine Gap Penalties](#modeling-real-indels-affine-gap-penalties)
- [From a Grid to a Hypercube: Multiple Alignment](#from-a-grid-to-a-hypercube-multiple-alignment)
- [A Different Question Entirely: OSYM](#a-different-question-entirely-osym)
- [Summary](#summary)

[Rosalind](https://rosalind.info/problems/list-view/) is a problem set for learning bioinformatics through code, in the spirit of Project Euler: a few hundred bite-sized problems, grouped by topic, each unlocked by the ideas the previous ones taught you. I've been working through it problem by problem, and the **Alignment** topic is the one that stood out as worth a deep dive. It starts as the simplest possible task: comparing two biological sequences character by character, lining them up and counting the differences. But it doesn't stay simple for long — problem by problem, it grows more variants of the same dynamic programming idea, generalized along a handful of independent axes: how a mismatch is scored, how a gap is priced, what "aligned" is even allowed to mean, and what question you ask of the finished table.

This post walks that recurrence's evolution problem by problem, in the order the ideas actually build on each other, with real, runnable code at every step rather than pseudocode. Along the way: why plain distance breaks the moment biology gets involved, why one variant refuses to fit the class hierarchy, and why the whole approach stops working past four sequences.

## A Few Concepts First

A **sequence** here just means a string over a small, fixed alphabet: DNA over `{A, C, G, T}`, RNA over `{A, C, G, U}`, or a protein over its 20 amino acid letters. Each single letter in that string is a **residue** — a generic word for "one unit of the sequence," used because the same idea applies whether that unit is a DNA base, an RNA base, or an amino acid. Comparing two sequences means asking how one could have turned into the other through evolution's two basic edits: a **substitution**, where one residue is swapped for another without changing the sequence's length, and an **indel** — insertion or deletion — where a residue is added or removed, shifting everything downstream of it.

Aligning two sequences means proposing a specific correspondence between their residues: which position in one sequence is meant to represent the same original spot as which position in the other. Wherever an indel breaks the simple "position *i* matches position *i*" correspondence, a **gap** — a placeholder (`-`) inserted into one sequence — restores it:

```
first:   G A T T A C A
second:  G A T - A C A
```

Reading column by column, every position matches except one: `second` is missing the `T` that `first` has there, so a gap stands in for it instead of forcing a wrong match one column over. That's what "aligning" two sequences means throughout this post — choosing exactly where to place gaps like this one so the two strings line up as well as possible — and the rest of the post is about how to score an alignment, and how to efficiently search over every way of placing those gaps for the best-scoring one.

## Ground Zero: Comparing Sequences Without Alignment

**[HAMM — Counting Point Mutations](https://rosalind.info/problems/hamm/)**

The most literal reading of "compare two sequences" is Hamming distance: walk both strings position by position and count where they disagree.

```python
def hamming_distance(first: str, second: str) -> int:
    return sum(a != b for a, b in zip(first, second))
```

This is exactly right for counting point mutations between two sequences you already know correspond position-for-position — two people's copies of the same gene, say, differing only by a handful of single-letter changes. But it silently assumes something big: that position *i* in one sequence really does correspond to position *i* in the other. Real sequences don't cooperate. Evolution doesn't just substitute residues, it inserts and deletes them too — a stretch of DNA copies itself into the middle of the sequence, or a chunk simply drops out. The moment either sequence gains or loses a residue, every position downstream of that event shifts, and comparing index-to-index stops meaning anything.

Once indels are on the table, "how different are these two sequences" stops being a fixed lookup and becomes a search problem: *which* correspondence between the two sequences' residues minimizes the total difference? That search is what alignment actually is, and it's why the rest of this post exists.

## Allowing Indels: A DP Table for Edit Distance

**[EDIT — Edit Distance](https://rosalind.info/problems/edit/) · [EDTA — Edit Distance Alignment](https://rosalind.info/problems/edta/)**

The classical answer is **edit distance** (Levenshtein, 1965): the minimum number of substitutions, insertions, and deletions needed to turn one string into the other. It's solved with a DP table where cell $$(i,j)$$ holds the edit distance between the first $$i$$ characters of one string and the first $$j$$ of the other:

$$D[i,j] = \min \begin{cases} D[i-1,j-1] + \text{cost}(a_i, b_j) & \text{substitution (0 if a match)} \\ D[i-1,j] + 1 & \text{deletion} \\ D[i,j-1] + 1 & \text{insertion} \end{cases}$$

```python
def build_distance_table(first: str, second: str) -> list[list[int]]:
    rows, cols = len(first), len(second)
    distances = [[0] * (cols + 1) for _ in range(rows + 1)]
    for i in range(rows + 1):
        distances[i][0] = i          # deleting the first i chars of `first`
    for j in range(cols + 1):
        distances[0][j] = j          # inserting the first j chars of `second`
    for i in range(1, rows + 1):
        for j in range(1, cols + 1):
            substitution_cost = 0 if first[i - 1] == second[j - 1] else 1
            distances[i][j] = min(
                distances[i - 1][j] + 1,                     # deletion
                distances[i][j - 1] + 1,                     # insertion
                distances[i - 1][j - 1] + substitution_cost,  # substitution / match
            )
    return distances
```

EDIT only asks for the number in the bottom-right corner (see Fig. 1 below). EDTA asks for more: not just *how different* are these sequences, but *which* residues correspond to which. That matters biologically — a raw distance of 4 tells you nothing about whether those four changes cluster near a known active site or scatter randomly. Getting the actual correspondence means tracing back through the table from $$(m,n)$$ to $$(0,0)$$, at each cell picking whichever neighbor's value plus its move cost actually explains the current cell:

```python
def edit_alignment(first: str, second: str) -> tuple[str, str]:
    distances = build_distance_table(first, second)
    aligned_first: list[str] = []
    aligned_second: list[str] = []
    i, j = len(first), len(second)

    while i > 0 or j > 0:
        substitution_cost = 1 if i == 0 or j == 0 or first[i - 1] != second[j - 1] else 0
        if i > 0 and j > 0 and distances[i][j] == distances[i - 1][j - 1] + substitution_cost:
            aligned_first.append(first[i - 1])   # diagonal: match or substitution
            aligned_second.append(second[j - 1])
            i, j = i - 1, j - 1
        elif i > 0 and distances[i][j] == distances[i - 1][j] + 1:
            aligned_first.append(first[i - 1])   # up: deletion from `first`
            aligned_second.append("-")
            i -= 1
        else:
            aligned_first.append("-")            # left: insertion into `first`
            aligned_second.append(second[j - 1])
            j -= 1

    aligned_first.reverse()
    aligned_second.reverse()
    return "".join(aligned_first), "".join(aligned_second)
```

![Fig01](/assets/blog/2026-08-30/edit-distance-fill.gif){:data-width="560" data-height="400"}
Fig. 1. Filling the edit-distance table for "ACGT" against "AGT" row by row, then tracing back from the bottom-right corner: the highlighted path reads, right to left, T-T, G-G, C-\-, A-A.
{:.figure}

## One Table, Three Questions: Score, Witness, All Witnesses

**[CTEA — Counting Optimal Alignments](https://rosalind.info/problems/ctea/)**

EDTA's traceback makes a choice every time two or more neighbors tie for best. Call one complete trace through the table — one fully specified sequence of substitutions and indels that actually achieves the minimum — a **witness** to the optimal distance. CTEA asks: how many distinct witnesses are there?

The answer matters more than it might sound. If a pair of sequences has exactly one optimal witness, EDTA's alignment is unambiguous — trust it as *the* story. If it has billions (common with repetitive sequences), then minimizing edit count alone can't tell you which of those billion equally-cheap histories actually happened; parsimony has run out of information, and you'd need something like a probabilistic model to pick among them. CTEA turns this from a philosophical worry into a number, by generalizing the traceback from "follow one tied direction" to "sum every tied direction":

```python
def count_optimal_alignments(first: str, second: str, modulus: int = 134_217_727) -> int:
    distances = build_distance_table(first, second)
    rows, cols = len(first), len(second)
    counts = [[0] * (cols + 1) for _ in range(rows + 1)]
    counts[0][0] = 1
    for i in range(1, rows + 1):
        counts[i][0] = 1
    for j in range(1, cols + 1):
        counts[0][j] = 1

    for i in range(1, rows + 1):
        for j in range(1, cols + 1):
            substitution_cost = 0 if first[i - 1] == second[j - 1] else 1
            total = 0
            if distances[i][j] == distances[i - 1][j - 1] + substitution_cost:
                total += counts[i - 1][j - 1]      # tied diagonal move
            if distances[i][j] == distances[i - 1][j] + 1:
                total += counts[i - 1][j]           # tied deletion
            if distances[i][j] == distances[i][j - 1] + 1:
                total += counts[i][j - 1]           # tied insertion
            counts[i][j] = total % modulus

    return counts[rows][cols]
```

Same table, three different questions asked of it: the score (EDIT), one proof (EDTA), every proof (CTEA).

## Real Biology Enters: Scoring Matrices and Needleman-Wunsch

**[GLOB — Global Alignment with Scoring Matrix](https://rosalind.info/problems/glob/)**

Everything so far treats every substitution as equally bad — swapping in Leucine for Isoleucine (two amino acids that are chemically almost interchangeable, usually harmless) costs the same as swapping in Leucine for Aspartate (a much bigger chemical change, often disruptive to how the protein folds or functions). Real protein comparison shouldn't do that. **BLOSUM62** is a substitution matrix built from observed substitution frequencies in real protein alignments — conservative swaps score positively, disruptive ones score negatively.

Plugging a matrix in instead of a flat mismatch cost, and flipping from minimizing cost to maximizing score, gives exactly **Needleman-Wunsch**:

$$S[i,j] = \max \begin{cases} S[i-1,j-1] + \text{score}(a_i, b_j) \\ S[i-1,j] - g \\ S[i,j-1] - g \end{cases}$$

It's worth being precise about what actually changed from EDIT's table, because it's less than it looks: same recurrence shape, same three-way choice per cell. Unit-cost edit distance is the special case `match=0, mismatch=1, gap=1`, negated. The two names come from two different fields arriving at the same DP independently — Levenshtein (1965) from string algorithms, Needleman & Wunsch (1970) from molecular biology, five years apart, for different reasons.

Rather than duplicate this loop per variant, it's worth writing it once as a class parameterized by a scoring function and a handful of boolean flags — flags that do nothing yet (every variant so far is "all false"), but will carry the rest of this post:

```python
from typing import Callable


class PairwiseAlignment:
    """Base DP engine for a linear-gap pairwise alignment variant.

    Subclasses declare their variant purely through class attributes:
    FREE_START_FIRST/FREE_END_FIRST/FREE_START_SECOND/FREE_END_SECOND mark
    which end of `first`/`second` may be skipped for free rather than
    mandatorily consumed, and ALLOW_RESTART floors the table at 0 (Smith-
    Waterman's "abandon a negative-scoring alignment and restart here").
    """

    FREE_START_FIRST = False
    FREE_END_FIRST = False
    FREE_START_SECOND = False
    FREE_END_SECOND = False
    ALLOW_RESTART = False

    def __init__(self, first: str, second: str, score_fn: Callable[[str, str], int], gap_penalty: int):
        self.first = first
        self.second = second
        self._score_fn = score_fn
        self.gap_penalty = gap_penalty

    @classmethod
    def from_substitution_matrix(
        cls, first: str, second: str, substitution_matrix: dict, gap_penalty: int
    ) -> "PairwiseAlignment":
        return cls(first, second, lambda a, b: substitution_matrix[a][b], gap_penalty)

    def align(self) -> tuple[int, str, str]:
        """Return (score, first_aligned, second_aligned) for this variant's best alignment."""
        scores = self._fill_table()
        rows, cols = len(self.first), len(self.second)
        score, (end_row, end_col) = self._extract_score(scores, rows, cols)
        i, j, core_first, core_second = self._traceback(scores, end_row, end_col)
        first_aligned, second_aligned = self._finalize(i, j, end_row, end_col, core_first, core_second)
        return score, first_aligned, second_aligned

    def _fill_table(self) -> list[list[int]]:
        rows, cols = len(self.first), len(self.second)
        scores = [[0] * (cols + 1) for _ in range(rows + 1)]
        free_col0 = self.FREE_START_FIRST or self.ALLOW_RESTART
        free_row0 = self.FREE_START_SECOND or self.ALLOW_RESTART

        for i in range(1, rows + 1):
            scores[i][0] = 0 if free_col0 else -self.gap_penalty * i
        for j in range(1, cols + 1):
            scores[0][j] = 0 if free_row0 else -self.gap_penalty * j

        self._best_score, self._best_cell = 0, (0, 0)
        for i in range(1, rows + 1):
            for j in range(1, cols + 1):
                substitution_score = self._score_fn(self.first[i - 1], self.second[j - 1])
                candidates = [
                    scores[i - 1][j - 1] + substitution_score,  # diagonal
                    scores[i - 1][j] - self.gap_penalty,        # gap in second
                    scores[i][j - 1] - self.gap_penalty,        # gap in first
                ]
                if self.ALLOW_RESTART:
                    candidates.append(0)  # Smith-Waterman: abandon and restart here
                scores[i][j] = max(candidates)
                if self.ALLOW_RESTART and scores[i][j] > self._best_score:
                    self._best_score, self._best_cell = scores[i][j], (i, j)

        return scores

    def _extract_score(self, scores: list[list[int]], rows: int, cols: int) -> tuple[int, tuple[int, int]]:
        """Return (score, (end_row, end_col)) for this variant's free-end rule."""
        if self.ALLOW_RESTART:
            return self._best_score, self._best_cell
        if not self.FREE_END_FIRST and not self.FREE_END_SECOND:
            return scores[rows][cols], (rows, cols)

        last_row_best = max(scores[rows])
        last_col_best = max(scores[i][cols] for i in range(rows + 1))
        if self.FREE_END_SECOND and not self.FREE_END_FIRST:
            return last_row_best, (rows, scores[rows].index(last_row_best))
        if self.FREE_END_FIRST and not self.FREE_END_SECOND:
            best_row = max(range(rows + 1), key=lambda i: scores[i][cols])
            return last_col_best, (best_row, cols)
        if last_row_best >= last_col_best:
            return last_row_best, (rows, scores[rows].index(last_row_best))
        best_row = max(range(rows + 1), key=lambda i: scores[i][cols])
        return last_col_best, (best_row, cols)

    def _traceback(self, scores: list[list[int]], end_row: int, end_col: int) -> tuple[int, int, str, str]:
        """Walk the score table backward from (end_row, end_col).

        Stops as soon as a free-start axis reaches 0 (its unconsumed prefix
        is simply left out, no gap padding); a mandatory axis keeps padding
        with explicit gap characters until it also reaches 0.
        """
        first_aligned: list[str] = []
        second_aligned: list[str] = []
        i, j = end_row, end_col

        while True:
            if self.ALLOW_RESTART and scores[i][j] == 0:
                break
            if i == 0 and j == 0:
                break
            if i == 0:
                if self.FREE_START_SECOND:
                    break
                first_aligned.append("-")
                second_aligned.append(self.second[j - 1])
                j -= 1
                continue
            if j == 0:
                if self.FREE_START_FIRST:
                    break
                first_aligned.append(self.first[i - 1])
                second_aligned.append("-")
                i -= 1
                continue

            substitution_score = self._score_fn(self.first[i - 1], self.second[j - 1])
            if scores[i][j] == scores[i - 1][j - 1] + substitution_score:
                first_aligned.append(self.first[i - 1])
                second_aligned.append(self.second[j - 1])
                i, j = i - 1, j - 1
            elif scores[i][j] == scores[i - 1][j] - self.gap_penalty:
                first_aligned.append(self.first[i - 1])
                second_aligned.append("-")
                i -= 1
            else:
                first_aligned.append("-")
                second_aligned.append(self.second[j - 1])
                j -= 1

        first_aligned.reverse()
        second_aligned.reverse()
        return i, j, "".join(first_aligned), "".join(second_aligned)

    def _finalize(self, i, j, end_row, end_col, core_first, core_second) -> tuple[str, str]:
        """Return the final (first_aligned, second_aligned). No-op unless overridden."""
        return core_first, core_second
```

That's the whole engine. The variant for this section is almost nothing on top of it:

```python
class GlobalAlignment(PairwiseAlignment):
    """Needleman-Wunsch: both strings fully, mandatorily aligned end to end."""
```

```python
score, _, _ = GlobalAlignment.from_substitution_matrix(
    first, second, BLOSUM62, gap_penalty=5
).align()
```

`GlobalAlignment` adds nothing — every flag stays at its default — which is the point: plain Needleman-Wunsch is what this engine does when none of its variation axes are switched on. Every alignment variant for the rest of this post is this same base class, distinguished only by which class attributes it flips.

## When an Indel Is One Event, Not Many

**[GCON — Global Alignment with Constant Gap Penalty](https://rosalind.info/problems/gcon/)**

GLOB's linear gap cost (`gap_penalty × gap_length`) treats a 10-residue gap as ten times as unlikely as a 1-residue gap. Biologically, that's often wrong: a single event — one multi-letter chunk copying itself in, or one copying mistake dropping several letters at once — can create a multi-residue gap in one shot, not one residue at a time. GCON's answer is the opposite extreme: **any** gap, long or short, costs one flat penalty, modeling "one event" rather than "one event per residue."

Neither extreme is fully realistic on its own — that tension is exactly what motivates affine gaps later in this post. But GCON earns its place in the arc for a more technical reason too: a flat per-gap cost genuinely **breaks** the "look at the one cell above/left" recurrence, because a gap can now open from arbitrarily far back in the same row or column at no extra cost for the distance. Filling the table still takes $$O(n \cdot m)$$ time, but only by tracking a running best-score-so-far per row and per column as you go, rather than a single predecessor cell:

```python
def global_alignment_score_constant_gap(
    first: str, second: str, substitution_matrix: dict, gap_penalty: int
) -> int:
    rows, cols = len(first), len(second)
    scores = [[0] * (cols + 1) for _ in range(rows + 1)]
    for i in range(1, rows + 1):
        scores[i][0] = -gap_penalty
    for j in range(1, cols + 1):
        scores[0][j] = -gap_penalty

    col_best = [scores[0][j] for j in range(cols + 1)]
    for i in range(1, rows + 1):
        row_best = scores[i][0]
        for j in range(1, cols + 1):
            substitution_score = substitution_matrix[first[i - 1]][second[j - 1]]
            scores[i][j] = max(
                scores[i - 1][j - 1] + substitution_score,  # diagonal, same as before
                col_best[j] - gap_penalty,                  # open a gap from anywhere in this column
                row_best - gap_penalty,                     # open a gap from anywhere in this row
            )
            row_best = max(row_best, scores[i][j])
            col_best[j] = max(col_best[j], scores[i][j])

    return scores[rows][cols]
```

That's why it stays a standalone function instead of another `PairwiseAlignment` subclass — its recurrence needs state the class's per-cell model doesn't carry. Not every variant in this family fits the clean hierarchy, and that's a more honest lesson than pretending it does.

![Fig02](/assets/blog/2026-08-30/gcon-fill.gif){:data-width="560" data-height="420"}
Fig. 2. Filling the constant-gap table for "MEANLY" against "MHNLY": blue cells win from the diagonal, exactly like GLOB; orange cells win by jumping to the best score anywhere earlier in their row or column, the long-range move a flat gap penalty allows.
{:.figure}

## Finding an Island: Local Alignment

**[LOCA — Local Alignment with Scoring Matrix](https://rosalind.info/problems/loca/)**

GLOB and GCON both force the *entire* length of both sequences into the alignment. That's the wrong model when two sequences are only expected to share one small, conserved stretch amid otherwise unrelated sequence — say, two proteins that do unrelated jobs but happen to share the one small **domain** (a self-contained, reusable functional chunk of a protein) responsible for grabbing onto DNA. **Smith-Waterman** local alignment finds the single best-scoring substring pair instead, by flooring the DP table at zero — a negative-scoring stretch is simply abandoned and the alignment restarts from 0 wherever it likes:

```python
class LocalAlignment(PairwiseAlignment):
    """Smith-Waterman: the table floors at 0, so the alignment may start and end anywhere."""
    ALLOW_RESTART = True
```

That one flag is all it takes: `_fill_table` already appends a free `0` candidate and tracks the running best cell whenever `ALLOW_RESTART` is set, and `_traceback` already stops the moment it walks back onto a 0.

LOCA pairs this with **PAM250** rather than BLOSUM62, and that pairing isn't incidental: PAM250 is built from more distantly related sequences, which is exactly the regime where you're hoping to spot a faint shared domain between otherwise-unrelated proteins rather than compare close relatives end to end.

![Fig03](/assets/blog/2026-08-30/loca-fill.gif){:data-width="560" data-height="460"}
Fig. 3. Filling the local-alignment table for "PAWHEAE" against "HEAGAWG" under PAM250: pink cells are floored to 0 — potential restart points — and the ringed cell is the best score anywhere in the table, not the bottom-right corner. The traceback grows outward from a 0 cell to that peak, ignoring everything else.
{:.figure}

## Local vs. Global: A Decision Framework

Worth stating explicitly, since it's the first real branch point in the arc:

- **Global** (Needleman-Wunsch): the two sequences are expected to correspond over their *entire* length — similar size, presumably related end to end. Two people's copies of the same gene; a mutated protein against its normal, unmutated version.
- **Local** (Smith-Waterman): the sequences may be very different lengths or origins, and are only expected to share a *subregion* — a domain, a motif, a functional site. Database search: does this short query share a domain with a much longer hit.
- **Overlap / semiglobal** (next section): the sequences are expected to correspond at their *ends*, not necessarily their whole length.

The question to ask before reaching for any of these: do I expect these two sequences to correspond over their whole length, just a piece of them, or specifically their ends?

## The Many Shapes of "Aligned": Free Ends

**[OAP — Overlap Alignment](https://rosalind.info/problems/oap/) · [SMGB — Semiglobal Alignment](https://rosalind.info/problems/smgb/)**

A natural question: LOCA already restarts anywhere, so what's left to add? The answer is that "restart anywhere" and "free ends" solve genuinely different problems.

Local alignment can *abandon* a weak stretch mid-alignment and never has to reach either sequence's true start or end — perfect for finding a clean island and discarding noisy flanks around it. But genome assembly needs the opposite guarantee: two sequencing reads that overlap at their ends must be glued *continuously*, mismatches and all, all the way to their literal endpoints — you can't discard a weak patch in the middle of the overlap and call it done. Overlap alignment enforces exactly that: once it starts, it runs uninterrupted (no restart) from some interior point of `first` through to `first`'s true end, and from `second`'s true start through to some interior point of `second`. It reports "how well do these ends actually overlap," not "is there a short match somewhere."

`PairwiseAlignment` expresses this as which end of each string is allowed to be skipped **for free** — a leading or trailing run of gaps that costs nothing, as opposed to a mandatory end where every gap is charged the usual penalty:

| Variant | free start of `first` | free end of `first` | free start of `second` | free end of `second` | typical use |
|---|:---:|:---:|:---:|:---:|---|
| `GlobalAlignment` | | | | | two full, comparable sequences |
| `OverlapAlignment` | ✓ | | | ✓ | genome assembly: suffix of a read meets prefix of another |
| `SemiglobalAlignment` | ✓ | ✓ | ✓ | ✓ | read-vs-reference mapping, containment either direction |

```python
class OverlapAlignment(PairwiseAlignment):
    """A global alignment between some suffix of first and some prefix of second."""
    FREE_START_FIRST = True
    FREE_END_SECOND = True


class SemiglobalAlignment(PairwiseAlignment):
    """A global alignment where a leading/trailing gap run on either string is free."""
    FREE_START_FIRST = FREE_END_FIRST = FREE_START_SECOND = FREE_END_SECOND = True
```

Nothing else has to change: `_extract_score` already reads `FREE_END_FIRST`/`FREE_END_SECOND` to pick the best cell in the last row or column instead of insisting on the corner, and `_traceback` already breaks out early the moment it hits a free-start boundary instead of padding it with gaps.

SMGB — free on all four ends — is the general "one sequence might overhang or sit inside the other, and I don't know which direction" case, which is exactly the read-mapping scenario (align a short read to a long reference without penalizing the reference's flanks on either side).

Being free on an end has a real implementation cost, too. `OverlapAlignment` gets away with the base class's do-nothing `_finalize`: whatever prefix/suffix its traceback skipped over is simply excluded from the result, which is correct for "report the overlap, however short" — a shorter pair of aligned strings is fine. `SemiglobalAlignment` can't get away with that: it needs both full sequences represented, since the whole point is showing where a short read sits inside a long reference. So it overrides `_finalize` to pad the untouched region back on with gap characters instead of dropping it, keeping both aligned strings the same length as their originals:

```python
class SemiglobalAlignment(PairwiseAlignment):
    """A global alignment where a leading/trailing gap run on either string is free."""
    FREE_START_FIRST = FREE_END_FIRST = FREE_START_SECOND = FREE_END_SECOND = True

    def _finalize(self, i, j, end_row, end_col, core_first, core_second) -> tuple[str, str]:
        rows, cols = len(self.first), len(self.second)

        if i > 0:
            leading_first, leading_second = self.first[:i], "-" * i
        elif j > 0:
            leading_first, leading_second = "-" * j, self.second[:j]
        else:
            leading_first = leading_second = ""

        if end_row < rows:
            trailing_first, trailing_second = self.first[end_row:], "-" * (rows - end_row)
        elif end_col < cols:
            trailing_first, trailing_second = "-" * (cols - end_col), self.second[end_col:]
        else:
            trailing_first = trailing_second = ""

        return (
            leading_first + core_first + trailing_first,
            leading_second + core_second + trailing_second,
        )
```

![Fig04](/assets/blog/2026-08-30/free-end-flags.svg){:data-width="900" data-height="630"}
Fig. 4. Global, overlap, semiglobal, and local, drawn as which portion of each string is mandatory (penalized), free (skippable), or simply excluded from the alignment altogether.
{:.figure}

## Modeling Real Indels: Affine Gap Penalties

**[GAFF — Global Alignment with Affine Gap Penalty](https://rosalind.info/problems/gaff/) · [LAFF — Local Alignment with Affine Gap Penalty](https://rosalind.info/problems/laff/)**

GCON's flat gap cost and GLOB's linear gap cost are both crude approximations of the same underlying biology, from opposite directions: linear overcharges long single-event indels; flat undercharges scattered independent small ones. The realistic middle ground is an **affine** gap penalty — a steep one-time cost to *open* a gap, then a much cheaper cost to *extend* it one more residue. This discourages many small scattered gaps (each pays the steep open cost) without punishing one genuinely large indel as if it were many independent events.

These aren't arbitrary numbers, either: GAFF and LAFF use `gap_open=11, gap_extend=1` with BLOSUM62 — the actual default gap costs used by BLAST, the search tool virtually every biologist uses to find sequences similar to a given one. This pair of problems is less "a teaching variant" and more "reproduce what real-world aligners actually run."

The catch is that a single score table genuinely can't express this — at any cell, the next move's price depends on whether the *previous* move was already a gap extension or something else, and a plain running-max table has no memory of that. **Gotoh's algorithm** tracks three tables per cell instead: $$M$$ (ends in a match/substitution), $$D$$ (ends in a gap eating `first`), $$I$$ (ends in a gap eating `second`) — a gap only ever extends from its own table, or opens fresh from $$M$$:

```python
_NEG_INF = float("-inf")


class AffineGapPairwiseAlignment:
    """Shared three-table (Gotoh) DP engine for affine-gap alignment variants."""

    ALLOW_RESTART = False

    def __init__(self, first: str, second: str, score_fn, gap_open_penalty: int, gap_extend_penalty: int):
        self.first = first
        self.second = second
        self._score_fn = score_fn
        self.gap_open_penalty = gap_open_penalty
        self.gap_extend_penalty = gap_extend_penalty

    def _build_tables(self):
        """Fill the M/D/I score tables, honoring ALLOW_RESTART's floor/border rules."""
        rows, cols = len(self.first), len(self.second)
        match_border = 0 if self.ALLOW_RESTART else _NEG_INF
        match_scores = [[match_border] * (cols + 1) for _ in range(rows + 1)]
        deletion_scores = [[_NEG_INF] * (cols + 1) for _ in range(rows + 1)]
        insertion_scores = [[_NEG_INF] * (cols + 1) for _ in range(rows + 1)]
        match_scores[0][0] = 0

        if not self.ALLOW_RESTART:
            for i in range(1, rows + 1):
                deletion_scores[i][0] = -(self.gap_open_penalty + self.gap_extend_penalty * (i - 1))
            for j in range(1, cols + 1):
                insertion_scores[0][j] = -(self.gap_open_penalty + self.gap_extend_penalty * (j - 1))

        best_score, best_cell = 0, (0, 0)
        for i in range(1, rows + 1):
            for j in range(1, cols + 1):
                substitution_score = self._score_fn(self.first[i - 1], self.second[j - 1])
                best_predecessor = max(
                    match_scores[i - 1][j - 1], deletion_scores[i - 1][j - 1], insertion_scores[i - 1][j - 1]
                )
                match_value = best_predecessor + substitution_score
                match_scores[i][j] = max(0, match_value) if self.ALLOW_RESTART else match_value
                deletion_scores[i][j] = max(
                    match_scores[i - 1][j] - self.gap_open_penalty,       # open a new gap
                    deletion_scores[i - 1][j] - self.gap_extend_penalty,  # extend the existing one
                )
                insertion_scores[i][j] = max(
                    match_scores[i][j - 1] - self.gap_open_penalty,
                    insertion_scores[i][j - 1] - self.gap_extend_penalty,
                )
                if self.ALLOW_RESTART and match_scores[i][j] > best_score:
                    best_score, best_cell = match_scores[i][j], (i, j)

        if not self.ALLOW_RESTART:
            best_score = max(match_scores[rows][cols], deletion_scores[rows][cols], insertion_scores[rows][cols])
            best_cell = (rows, cols)

        return match_scores, deletion_scores, insertion_scores, best_score, best_cell
```

`GAFF`/`LAFF` reuse the exact same "floor at zero" trick from LOCA, just scoped to the $$M$$ table alone — `D`/`I` need no floor of their own, since $$M$$'s floor already supplies the restart option whenever a gap-ending alignment would need to abandon and restart:

```python
class GlobalAffineGapAlignment(AffineGapPairwiseAlignment):
    """Global alignment (Gotoh) under an affine gap penalty."""


class LocalAffineGapAlignment(AffineGapPairwiseAlignment):
    """Local alignment (best-scoring substrings) under an affine gap penalty."""
    ALLOW_RESTART = True
```

![Fig05](/assets/blog/2026-08-30/gotoh-state-machine.svg){:data-width="520" data-height="400"}
Fig. 5. The three tables as a small state machine: a gap can only ever open fresh from M or extend from its own table — never jump directly between D and I.
{:.figure}

![Fig06](/assets/blog/2026-08-30/gaff-fill.gif){:data-width="1250" data-height="480"}
Fig. 6. The three Gotoh tables for "SANTLEY" against "MEANLY", filling in side by side — note how empty D and I start out (most of both tables is `-inf`, an alignment that can't possibly end in a gap there). One traceback path is drawn across all three panels at once, jumping from M to D and back exactly where a gap opens and closes.
{:.figure}

This engine is a *sibling* to `PairwiseAlignment`, not a subclass of it — none of the affine-gap problems here need the free-start/free-end machinery, so it only ever varies along `ALLOW_RESTART`. Two families, two class hierarchies, sharing an idea but not a base class.

## From a Grid to a Hypercube: Multiple Alignment

**[MULT — Multiple Alignment](https://rosalind.info/problems/mult/)**

Every variant so far compares two sequences. Real biology often wants more — a gene family across several species, to spot conserved motifs or to feed a phylogenetic tree builder. The direct generalization: instead of a 2-D grid of prefix-length pairs $$(i, j)$$, use a $$k$$-dimensional grid of prefix-length tuples $$(i_1, \dots, i_k)$$, one coordinate per sequence. GLOB's "3 predecessors per cell" (diagonal / gap-first / gap-second) becomes "every nonempty subset of sequences advances one character this column, everyone else contributes a gap" — $$2^k - 1$$ predecessors per cell:

```python
from itertools import combinations, product


def nonempty_subsets(count: int) -> list[tuple[int, ...]]:
    """Return every nonempty subset of range(count), as tuples of indices."""
    subsets = []
    for size in range(1, count + 1):
        subsets.extend(combinations(range(count), size))
    return subsets


def column_at(sequences, state, subset, sequence_count) -> tuple[str, ...]:
    """The column produced by advancing subset: those sequences contribute
    their next character, every other sequence contributes a gap."""
    return tuple(
        sequences[index][state[index] - 1] if index in subset else "-"
        for index in range(sequence_count)
    )


def multiple_alignment_score(sequences: list[str], match_score: int = 0, mismatch_score: int = -1) -> int:
    sequence_count = len(sequences)
    lengths = [len(sequence) for sequence in sequences]
    subsets = nonempty_subsets(sequence_count)
    pairs = list(combinations(range(sequence_count), 2))
    scores: dict = {}

    for state in product(*(range(length + 1) for length in lengths)):
        if all(index == 0 for index in state):
            scores[state] = 0
            continue
        best_score = None
        for subset in subsets:                      # every nonempty subset of sequence indices
            if not all(state[index] >= 1 for index in subset):
                continue
            predecessor = tuple(
                state[index] - 1 if index in subset else state[index]
                for index in range(sequence_count)
            )
            column = column_at(sequences, state, subset, sequence_count)
            column_score = sum(
                match_score if column[a] == column[b] else mismatch_score for a, b in pairs
            )
            candidate = scores[predecessor] + column_score
            if best_score is None or candidate > best_score:
                best_score = candidate
        scores[state] = best_score

    return scores[tuple(lengths)]
```

`itertools.product` over the state space happens to visit cells in a valid topological order for free, which is what makes the fill loop this simple. But the cost is real: the state space size is $$\prod_i (n_i + 1)$$ — multiplying, not adding, with every extra sequence — and each cell now has $$2^k - 1$$ transitions instead of 3. Total work is roughly $$O(2^k \cdot n^k)$$. That's exponential in $$k$$, which is why Rosalind's own MULT only ever asks for 4 short sequences: exact hypercube DP is already impractical past that. It's not just this implementation being slow — optimal sum-of-pairs multiple alignment is NP-hard in the number of sequences (Wang & Jiang, 1994). Real MSA tools (ClustalW, MUSCLE, MAFFT) don't run exact DP past the pairwise case at all; they build a guide tree and progressively merge pairwise/profile alignments, trading optimality for tractability. MULT is the exact answer, and simultaneously the clearest demonstration of why nobody computes it this way at scale.

![Fig07](/assets/blog/2026-08-30/mult-hypercube.svg){:data-width="820" data-height="420"}
Fig. 7. Adding a third sequence turns the pairwise grid into a cube: the state space multiplies rather than adds, and each cell's 3 predecessors become 7.
{:.figure}

The cube is easier to see as what it actually is on a computer: a stack of ordinary 2-D grids, one per prefix length of the third sequence, each filled in exactly like every 2-D table earlier in this post. Laid out side by side rather than stacked, all three fill in together — and the actual answer is just one cell in the last of them:

![Fig08](/assets/blog/2026-08-30/mult-cube-fill.gif){:data-width="1350" data-height="500"}
Fig. 8. The 3-sequence hypercube for "AT" x "AC" x "GT", flattened into its three 2-D slices — one per prefix length of the third sequence — laid out side by side and filled in together. The rightmost slice (third sequence fully consumed) ends with the ringed cell: the final MULT score.
{:.figure}

## A Different Question Entirely: OSYM

**[OSYM — Isolating Symbols in Alignments](https://rosalind.info/problems/osym/)**

Every problem so far has asked some version of "what's the best alignment." OSYM asks something structurally different: for *every* possible forced pairing of positions $$(j, k)$$ — insisting that residue $$j$$ of `first` aligns directly with residue $$k$$ of `second` — what's the best alignment score achievable under that constraint? Then it sums all of those over every $$(j,k)$$ pair.

Recomputing full DP per pair would be $$O(n^3 m)$$-ish and wasteful. Instead: build the ordinary forward table (best score of prefixes — exactly GLOB's table, just scored via a flat match/mismatch instead of a substitution matrix) and a mirror-image backward table (best score of *suffixes*, filled from the bottom-right corner backward), then glue them at each candidate pair:

$$M[j,k] = \text{forward}[j-1][k-1] + \text{score}(a_j, b_k) + \text{backward}[j][k]$$

```python
def backward_scores(first: str, second: str, match_score: int, mismatch_score: int, gap_penalty: int):
    """scores[i][j] is the optimal global alignment score of the suffixes first[i:] and second[j:]."""
    rows, cols = len(first), len(second)
    scores = [[0] * (cols + 1) for _ in range(rows + 1)]
    for i in range(rows - 1, -1, -1):
        scores[i][cols] = scores[i + 1][cols] - gap_penalty
    for j in range(cols - 1, -1, -1):
        scores[rows][j] = scores[rows][j + 1] - gap_penalty

    for i in range(rows - 1, -1, -1):
        for j in range(cols - 1, -1, -1):
            substitution_score = match_score if first[i] == second[j] else mismatch_score
            scores[i][j] = max(
                scores[i + 1][j + 1] + substitution_score,
                scores[i + 1][j] - gap_penalty,
                scores[i][j + 1] - gap_penalty,
            )
    return scores


def isolated_symbol_matrix_sum(forward, backward, first: str, second: str, match_score: int, mismatch_score: int) -> int:
    rows, cols = len(first), len(second)
    matrix_sum = 0
    for j in range(1, rows + 1):
        for k in range(1, cols + 1):
            substitution_score = match_score if first[j - 1] == second[k - 1] else mismatch_score
            matrix_sum += forward[j - 1][k - 1] + substitution_score + backward[j][k]
    return matrix_sum
```

One forward pass, one backward pass, and every forced-pair score falls out in $$O(1)$$ each.

Why would anyone want this? It's the deterministic (max-plus) sibling of the **forward-backward algorithm** used with HMMs — the same computational shape used to compute *posterior confidence* in an alignment: how sure are we that residue 37 of sequence A really corresponds to residue 42 of sequence B, versus that being one arbitrary choice among many nearly-tied alternatives? Consistency-based aligners (ProbCons and relatives) use exactly this pattern to flag which columns of a multiple alignment are robust versus ambiguous — information that matters downstream, since a phylogenetic tree built on a shaky alignment column is only as trustworthy as that column. It's also a natural bridge for a later post: swap the max/+ semiring here for sum/×, and the identical forward-backward gluing trick becomes genuinely probabilistic.

## Summary

Laid end to end, this family of Rosalind problems generalizes along four largely independent axes:

- **Cost model** — unit cost → arbitrary substitution matrix + linear gap → flat per-gap cost → affine open/extend gap.
- **Alignment extent** — mandatory whole-string → restart-anywhere (local) → free on one end of each string (overlap) → free on all four ends (semiglobal).
- **The question asked of the table** — best score → one witness alignment → count of all witnesses → per-position forced-pairing scores.
- **Dimensionality** — pairwise 2-D grid → $$k$$-way hypercube.

Two of those axes collapse cleanly into one class hierarchy (`PairwiseAlignment`'s boolean flags cover global/local/overlap/semiglobal in a single DP loop). Two of them genuinely don't — GCON's flat gap needs running-maximum state the per-cell model doesn't carry, and MULT's hypercube needs a different state space entirely, not just different transition costs. That split is, honestly, the more useful takeaway than "everything reduces to one abstraction": know which axis of variation your problem is generalizing along, and you'll know in advance whether a clean subclass will do, or whether you need a structurally different engine.
