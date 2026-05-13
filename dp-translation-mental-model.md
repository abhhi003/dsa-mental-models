# DP Translation — Top-Down to Bottom-Up Mental Model

## When to Use

Three signals:
1. Working Top-Down (memoization) solution already exists
2. Need to eliminate recursion (stack overflow risk, O(n) call stack space)
3. Want to optimize space or iterate in a specific order

---

## The Traps

| Trap | Why It Fails |
|---|---|
| Writing `return` inside Bottom-Up loop | Bottom-Up has no caller to return to. Hits first valid cell, returns early, skips rest of table. |
| Forgetting a dimension | Every parameter that changes in Top-Down must become a loop in Bottom-Up. Miss one → wrong indices. |
| `Integer[]` / `Long[]` wrapper classes | NPE if cell never filled. Use primitive `int[]` / `long[]` + explicit sentinel initialization. |

---

## The Three Mental Models

### 1. "The Argument-to-Loop Translation"

> Every argument in `helper(x, y, magic)` that changes must become a `for` loop.

```
Top-Down signature: helper(int x, int y, int magic)
                              ↓         ↓        ↓
Bottom-Up loops:   for x ... for y ... for magic ...
```

Arguments that never change (like grid size) stay as constants — no loop for them.

### 2. "The Architect vs. The Explorer"

> Top-Down = Explorer. Stands at current cell, dives into future (recursion), collects result.
> Bottom-Up = Architect. Builds table brick by brick, never returns — just lays down values.

```java
// TOP-DOWN (Explorer) — calls future and returns
int helper(int i, int j) {
    return Math.max(helper(i+1, j), helper(i, j+1)) + grid[i][j];
}

// BOTTOM-UP (Architect) — builds; no return inside loop
for (int i = m-1; i >= 0; i--) {
    for (int j = n-1; j >= 0; j--) {
        dp[i][j] = Math.max(dp[i+1][j], dp[i][j+1]) + grid[i][j];
        // no return — keep laying bricks
    }
}
```

### 3. "The Time Machine"

> Top-Down stands at `i`, looks to the **future** (`i+1`).
> Bottom-Up must iterate so that when you compute `dp[i]`, `dp[i+1]` is already built.

```
If Top-Down looks at i+1:
    → Bottom-Up iterates i from high to low (build future first)

If Top-Down looks at i-1:
    → Bottom-Up iterates i from low to high (build past first)
```

---

## The Translation Checklist

```
Step 1: Identify all changing parameters in Top-Down signature
         → one for loop per parameter

Step 2: Identify base cases
         → fill dp table edges before the main loops (or handle with if-guards)

Step 3: Replace recursive calls with table lookups
         helper(i+1, j)   →   dp[i+1][j]
         helper(i, j+1)   →   dp[i][j+1]

Step 4: Determine loop direction
         Top-Down uses i+1  →  iterate i from (m-1) down to 0
         Top-Down uses i-1  →  iterate i from 0 up to (m-1)

Step 5: Replace memo[i][j] assignment with dp[i][j] assignment

Step 6: Replace final return helper(0, 0, K) with dp[0][0][K]
```

---

## The Template (Java)

```java
// TOP-DOWN (before translation)
int[][][] memo = new int[m][n][K+1];
int helper(int x, int y, int magic) {
    if (x == m-1 && y == n-1) return base(magic);
    if (memo[x][y][magic] != -1) return memo[x][y][magic];
    int best = Math.max(helper(x+1, y, magic), helper(x, y+1, magic));
    if (magic > 0) best = Math.max(best, helper(x+1, y, magic-1));
    return memo[x][y][magic] = best + grid[x][y];
}

// BOTTOM-UP (after translation)
int[][][] dp = new int[m+1][n+1][K+1];
// fill base case
for (int k = 0; k <= K; k++) dp[m-1][n-1][k] = base(k);

for (int x = m-1; x >= 0; x--) {        // ← iterates high→low because Top-Down uses x+1
    for (int y = n-1; y >= 0; y--) {
        for (int k = 0; k <= K; k++) {
            if (x == m-1 && y == n-1) continue; // base case already filled
            int best = Math.max(dp[x+1][y][k], dp[x][y+1][k]);
            if (k > 0) best = Math.max(best, dp[x+1][y][k-1]);
            dp[x][y][k] = best + grid[x][y];
        }
    }
}
return dp[0][0][K];
```

---

## Recognizing It Fast

```
Working Top-Down + need to remove recursion
                ↓
    Argument → Loop, return → assignment, future → past
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `return` inside Bottom-Up loop | Replace with `dp[i][j] = ...` and `continue` |
| Using `Integer[]` | Use `int[]` + fill with sentinel (`Integer.MIN_VALUE` or `-1`) |
| Wrong iteration direction | If Top-Down reads `i+1`, loop from `m-1` down to `0` |
| Off-by-one on dp array size | Add +1 padding for out-of-bounds reads: `dp[m+1][n+1]` |
| Base case not seeded | Fill base cases explicitly before main loop or guard with if-check |
