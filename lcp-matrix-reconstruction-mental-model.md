# LCP Matrix Reconstruction — 5-Model Mental Model

## Problem Class

Given a matrix describing relationships between elements (e.g., `lcp[i][j]` = longest common prefix of suffixes at i and j), reconstruct the **lexicographically smallest valid string** — or return `""` if the matrix is contradictory.

**Canonical problem:** LC 2573 — Find the String with LCP

---

## The 5-Model System

Use these in order. Each model handles one phase of the solution.

---

## Model 1: "Equivalence Class" (Matrix as a Social Network)

**When:** Matrix describes pairwise relationships (friends, same-group, shared property).

**The Trap:** Reading the matrix row-by-row and applying overlapping rules naively.

**Mental Model:**
> Treat the matrix as a social network.
> `matrix[i][j] > 0` means i and j are "friends."
> Friends of friends are in the same group (Equivalence Class).

**Apply it:** Use Union-Find. If `lcp[i][j] > 0`, call `union(i, j)`. Each connected component is one equivalence class — all indices that must share the same character.

```java
int[] parent = new int[n];
Arrays.fill(parent, -1); // -1 = own group initially

// Union-Find helpers omitted for brevity
for (int i = 0; i < n; i++)
    for (int j = i + 1; j < n; j++)
        if (lcp[i][j] > 0) union(parent, i, j);
```

---

## Model 2: "Greedy Lock-In" (Left-to-Right, Best-First)

**When:** Problem asks for lexicographically smallest / largest / first valid sequence.

**The Trap:** Generating all combinations, sorting, picking the best → TLE.

**Mental Model:**
> Index 0 is the most important position. Lock in the best valid choice ('a'), apply it to its entire Equivalence Class, never revisit.
> Move to index 1. Pick the next best valid choice. Repeat.

**Apply it:** Iterate `i` from 0 to n-1. If `s[i]` is unassigned, assign the smallest available character. Then assign that same character to every member of `find(i)`'s equivalence class.

```java
char[] s = new char[n];
char next = 'a';

for (int i = 0; i < n; i++) {
    if (s[i] == 0) {                   // unassigned
        // assign 'next' to whole group
        for (int j = i; j < n; j++)
            if (find(parent, j) == find(parent, i))
                s[j] = next;
        next++;
        if (next > 'z') return "";     // ran out of characters
    }
}
```

---

## Model 3: "Trust, but Verify" (Forward Build + Backward Lie-Detector)

**When:** Problem says "return empty string if no valid answer" — the input might be contradictory.

**The Trap:** Writing dozens of upfront validation checks to catch every contradiction before building.
You will almost always miss an edge case.

**Mental Model:**
> Phase 1 (Trust): Blindly build a candidate answer using greedy, assuming input is honest.
> Phase 2 (Verify): Run the original process on your candidate. Does it reproduce the input exactly?
> If yes → return candidate. If no → return `""`.

```java
String candidate = build(lcp); // trust phase — greedy + equivalence class

int[][] computed = computeLCP(candidate);
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++)
        if (computed[i][j] != lcp[i][j]) return ""; // lie detected

return candidate; // verified
```

---

## Model 4: "Time-Travel DP" (Let the Formula Dictate Loop Direction)

**When:** DP transition depends on a future index: `dp[i][j] = 1 + dp[i+1][j+1]`.

**The Trap:** Looping 0→n and hitting `IndexOutOfBoundsException`, or reading uncomputed future values.

**Mental Model:**
> Look at the transition. `dp[i][j]` needs `dp[i+1][j+1]`.
> Data flows from end → beginning.
> Loop **backwards** — start where base cases naturally exist (end of string), build toward index 0.

```java
int[][] dp = new int[n + 1][n + 1]; // +1 for safe i+1, j+1 access

// Base cases: dp[i][i] = 0 (empty suffix) and dp[n][...] = 0 — already zeroed

for (int i = n - 1; i >= 0; i--) {        // ← backwards
    for (int j = n - 1; j >= 0; j--) {
        if (s[i] == s[j])
            dp[i][j] = 1 + dp[i + 1][j + 1]; // safe because we built i+1 already
        else
            dp[i][j] = 0;
    }
}
```

---

## Model 5: "Shadowing" (Kill the Second Matrix)

**When:** You built a full `dp[n][n]` just to compare against the input `lcp[n][n]`.

**The Trap:** Allocating O(n²) extra space when you don't need to store intermediate results.

**Mental Model:**
> You're iterating backwards. By the time you check `dp[i][j]`, you already verified `dp[i+1][j+1]`.
> So instead of storing `dp[i][j]`, just **compute the expected value on the fly and compare immediately.**

```java
// Before (O(n²) space)
dp[i][j] = (s[i] == s[j]) ? 1 + dp[i+1][j+1] : 0;
if (dp[i][j] != lcp[i][j]) return "";

// After (O(1) extra space — shadows the input matrix itself)
int expected = (s[i] == s[j]) ? 1 + lcp[i+1][j+1] : 0;
if (expected != lcp[i][j]) return "";
// No dp array needed — use lcp[i+1][j+1] directly since it was already validated
```

**Space: O(n²) → O(1)** (beyond the input matrix itself).

---

## The 30-Second Decision Checklist

```
1. "Grid of pairwise relationships?"       → Equivalence Classes (Union-Find)
2. "Asks for lex smallest/largest?"        → Greedy Lock-In (left-to-right, best-first)
3. "Input could be invalid / contradictory?"→ Trust, but Verify (build then lie-detect)
4. "Transition depends on future indices?" → Backwards DP (Time-Travel)
5. "Building matrix just to compare it?"  → Shadowing (eliminate the second matrix)
```

---

## Full Solution Skeleton (Java)

```java
public String findTheString(int[][] lcp) {
    int n = lcp.length;
    int[] parent = new int[n];
    Arrays.fill(parent, -1);

    // Model 1: Build equivalence classes
    for (int i = 0; i < n; i++)
        for (int j = i + 1; j < n; j++)
            if (lcp[i][j] > 0) union(parent, i, j);

    // Model 2: Greedy assign characters to each class
    char[] s = new char[n];
    char next = 'a';
    for (int i = 0; i < n; i++) {
        if (s[i] != 0) continue;
        if (next > 'z') return "";
        for (int j = i; j < n; j++)
            if (find(parent, j) == find(parent, i)) s[j] = next;
        next++;
    }

    // Model 3 + 4 + 5: Backwards DP verify (shadow the lcp matrix)
    for (int i = n - 1; i >= 0; i--) {
        for (int j = n - 1; j >= 0; j--) {
            int expected;
            if (s[i] == s[j]) {
                expected = (i + 1 < n && j + 1 < n) ? 1 + lcp[i+1][j+1] : 1;
            } else {
                expected = 0;
            }
            if (expected != lcp[i][j]) return "";
        }
    }

    return new String(s);
}
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Upfront matrix validation instead of verify phase | Build first, verify after — catch all contradictions in one pass |
| Looping forward for LCP DP | Loop backwards — `dp[i][j]` needs `dp[i+1][j+1]` |
| Allocating `int[][] dp` for verification | Use shadow technique — compute expected from `lcp[i+1][j+1]` directly |
| Not union-ing on `lcp[i][j] > 0` | Any nonzero LCP means same first character — they're in the same class |
| Running out of letters silently | Check `next > 'z'` before assigning; return `""` immediately |

---

## Problems Using This Pattern

| LeetCode | Problem | Models Used |
|---|---|---|
| 2573 | Find the String with LCP | All 5 |
| 1202 | Smallest String With Swaps | 1 + 2 (Union-Find + Greedy) |
| 128 | Longest Consecutive Sequence | 1 (Union-Find equivalence) |
| 1971 | Find if Path Exists in Graph | 1 (Union-Find) |
| 300 | Longest Increasing Subsequence | 4 (DP direction) |
