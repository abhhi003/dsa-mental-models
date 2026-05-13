# Space Optimization — 2D/3D DP to 1D Mental Model

## When to Use

Two signals:
1. Working Bottom-Up DP with `O(M×N)` or `O(M×N×K)` space
2. Code only ever reads `dp[i]` and `dp[i+1]` — never `dp[i-5]` or arbitrary past rows

**Brute force shape that signals this:**
`O(M×N×K)` space → optimize to `O(N×K)` or `O(N)`

---

## The Trap

Trying to jump directly from 3D array to 1D in your head.
Index confusion causes subtle bugs. Always take the two-step path:
`3D → two 2D arrays → (optionally) one 1D array`

---

## The Mental Model — "The Two-Row Bridge"

> You never need the whole grid. You only need **two rows at a time**: the row you're computing (`currRow`) and the row you already computed (`nextRow`).

**Rule:** If your loop body only reads `dp[i+1][...]`, you can replace the entire `dp` table with two arrays.

---

## The Translation (Step by Step)

### Step 1: Identify the rolling dimension

Look at your i loop. Does the body only read `dp[i+1][j][k]`?
If yes → `i` is the rolling dimension. Replace it.

```java
// Before — full 3D table
int[][][] dp = new int[m][n][K+1];
for (int i = m-1; i >= 0; i--) {
    for (int j = n-1; j >= 0; j--) {
        for (int k = 0; k <= K; k++) {
            dp[i][j][k] = f(dp[i+1][j][k], dp[i][j+1][k]);
        }
    }
}
return dp[0][0][K];
```

### Step 2: Replace `dp[i]` with `currRow`, `dp[i+1]` with `nextRow`

```java
int[][] currRow = new int[n][K+1];
int[][] nextRow = new int[n][K+1];

// seed nextRow with the base case (was dp[m-1])
for (int k = 0; k <= K; k++) nextRow[n-1][k] = baseCaseValue(k);

for (int i = m-2; i >= 0; i--) {
    for (int j = n-1; j >= 0; j--) {
        for (int k = 0; k <= K; k++) {
            currRow[j][k] = f(nextRow[j][k], currRow[j+1][k]);
            //               ↑ was dp[i+1]    ↑ was dp[i][j+1] — still currRow, same i
        }
    }
    // slide the window: what was current becomes next
    int[][] temp = nextRow;
    nextRow = currRow;
    currRow = temp;
    Arrays.fill(currRow, ... ); // reset if needed — depends on problem
}
return nextRow[0][K]; // last completed row is in nextRow after swap
```

### Step 3 (optional): If j also only reads `j+1`, collapse to 1D

```java
int[] curr = new int[K+1];
// process j right-to-left so j+1 is already computed
for (int j = n-1; j >= 0; j--) {
    for (int k = 0; k <= K; k++) {
        curr[k] = f(next[k], curr[k]); // reuse same array if safe
    }
}
```

---

## The Full Template (Java)

```java
// 3D → Two 2D rows
int[][] curr = new int[n + 1][K + 1];
int[][] next = new int[n + 1][K + 1];

// fill base row (i = m-1)
for (int j = n - 1; j >= 0; j--)
    for (int k = 0; k <= K; k++)
        next[j][k] = /* base case for row m-1 */;

for (int i = m - 2; i >= 0; i--) {
    // reset curr for this row
    for (int[] row : curr) Arrays.fill(row, 0);

    for (int j = n - 1; j >= 0; j--) {
        for (int k = 0; k <= K; k++) {
            int best = next[j][k];               // came from dp[i+1][j][k]
            if (j + 1 < n) best = Math.max(best, curr[j + 1][k]); // dp[i][j+1][k]
            if (k > 0)     best = Math.max(best, next[j][k - 1]); // used resource
            curr[j][k] = best + grid[i][j];
        }
    }

    // roll the window
    int[][] tmp = next;
    next = curr;
    curr = tmp;
}
return next[0][K];
```

---

## Recognizing It Fast

```
Bottom-Up dp[i][j][k] only reads dp[i+1][...] in the i-loop
                        ↓
        Replace with currRow / nextRow
        Swap at end of each i iteration
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Forgetting to reset `currRow` each iteration | `Arrays.fill(curr, ...)` at start of i loop |
| Reading from `currRow` before it's computed (same row, wrong direction) | Iterate j right-to-left so `j+1` is already done |
| Off-by-one — returning `currRow` instead of `nextRow` after swap | After the swap, completed row is in `nextRow` |
| Collapsing to 1D when j reads both `j-1` and `j+1` | Can't collapse to 1D — need 2D at minimum |
| Allocating `new int[n]` inside the i loop (GC pressure) | Allocate both arrays once outside; swap references |

---

## Problems Using This Pattern

| LeetCode | Problem | Before → After |
|---|---|---|
| 64 | Minimum Path Sum | O(M×N) → O(N) |
| 62 | Unique Paths | O(M×N) → O(N) |
| 63 | Unique Paths II | O(M×N) → O(N) |
| 174 | Dungeon Game | O(M×N) → O(N) |
| 741 | Cherry Pickup | O(N³) → O(N²) |
| 1463 | Cherry Pickup II | O(M×N²) → O(N²) |
