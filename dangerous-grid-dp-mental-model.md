# Dangerous Grid — Top-Down DP Mental Model

## When to Use

Three signals must all appear:
1. Walking through a **2D grid** (usually Top-Left → Bottom-Right)
2. Grid contains **negative numbers** (penalties, enemies, damage)
3. You have a **limited resource** (magic, armor, skips, k-moves)

---

## The Traps

| Trap | Why It Fails |
|---|---|
| `if (magic == 0)` in same base case as `if (out_of_bounds)` | Running out of magic ≠ out of bounds. Magic=0 just disables the magic branch, game continues. |
| Returning `0` for invalid paths | Grid has negatives — `Math.max` will prefer 0 over a valid negative path. Silent wrong answer. |
| Forgetting magic on the final square | Base case `if (x == m-1 && y == n-1)` must still check if power-up is usable there. |

---

## The Three Mental Models

### 1. "Resources Are Not Walls"

> Running out of magic doesn't end the game — it just disables the magic if-branch.

```java
// WRONG — treats magic=0 like going out of bounds
if (r < 0 || c < 0 || r >= m || c >= n || magic < 0) return INVALID;

// RIGHT — two separate checks
if (r < 0 || c < 0 || r >= m || c >= n) return INVALID;   // hard stop
// magic < 0 just means: don't enter the magic branch below
```

### 2. "Out of Bounds is Lava" + "The Safe Infinity"

> Invalid path must return a score so bad that `Math.max` never picks it.

Because the grid has negatives, returning `0` for invalid paths is wrong — `Math.max(validNegative, 0)` will wrongly pick 0.

**Use Safe Infinity:**
```java
private static final int INVALID = (int) -1e9;  // safe: can add negatives without underflow

if (r < 0 || c < 0 || r >= m || c >= n) return INVALID;
```

Why `-1e9` not `Integer.MIN_VALUE`? Because you'll do `INVALID + grid[r][c]`, and `MIN_VALUE + negative` underflows to positive — wrong result.

### 3. "The Finish Line Is Still the Track"

> The base case at `(m-1, n-1)` is still a real square. Power-ups apply there too.

```java
if (r == m - 1 && c == n - 1) {
    // WRONG — forgot magic
    return grid[r][c];

    // RIGHT — magic still usable on exit square
    int normal = grid[r][c];
    int withMagic = (magic > 0) ? 0 : normal;  // skip damage if magic available
    return Math.max(normal, withMagic);
}
```

---

## The Template (Java)

```java
private int[][] grid;
private int[][][] memo;
private int m, n;
private static final int INVALID = (int) -1e9;

public int solve(int[][] grid) {
    this.grid = grid;
    m = grid.length; n = grid[0].length;
    int K = /* resource limit */;
    memo = new int[m][n][K + 1];
    for (int[][] layer : memo)
        for (int[] row : layer) Arrays.fill(row, Integer.MIN_VALUE);
    return dp(0, 0, K);
}

private int dp(int r, int c, int magic) {
    if (r < 0 || c < 0 || r >= m || c >= n) return INVALID;

    if (memo[r][c][magic] != Integer.MIN_VALUE) return memo[r][c][magic];

    int cell = grid[r][c];

    if (r == m - 1 && c == n - 1) {
        int normal = cell;
        int withMagic = (magic > 0) ? 0 : cell;
        return memo[r][c][magic] = Math.max(normal, withMagic);
    }

    // Move down and right
    int best = Math.max(
        dp(r + 1, c, magic),
        dp(r, c + 1, magic)
    );

    // Use magic if available
    if (magic > 0) {
        best = Math.max(best,
            Math.max(
                dp(r + 1, c, magic - 1),
                dp(r, c + 1, magic - 1)
            )
        );
        // magic on current cell (skip its damage)
        best = Math.max(best, dp(r + 1, c, magic - 1));
    }

    if (best == INVALID) return memo[r][c][magic] = INVALID;
    return memo[r][c][magic] = best + cell;
}
```

---

## Recognizing It Fast

```
2D grid + negative values + limited resource
                ↓
     Top-Down DP with 3D memo [r][c][resource]
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `return 0` for invalid paths | `return INVALID` = `-1e9` |
| `Integer.MIN_VALUE` for INVALID | Use `-1e9`; MIN_VALUE underflows when added to negatives |
| Magic check in bounds check | Separate the two conditions |
| Missing magic on final square | Final base case must also try using magic |
| `dp[r][c][magic]` instead of `dp[r][c][magic-1]` when using magic | Using magic costs 1 unit — decrement |

---

## Problems Using This Pattern

| LeetCode | Problem | Resource |
|---|---|---|
| 1219 | Path with Maximum Gold | No resource (pure grid max) |
| 1463 | Cherry Pickup II | Two pointers on same grid |
| 741 | Cherry Pickup | Round-trip path |
| 64 | Minimum Path Sum | No resource (baseline) |
| 2304 | Minimum Path Cost in a Grid | Edge weights |
