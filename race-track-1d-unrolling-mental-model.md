# Race Track — 1D Unrolling Mental Model

## When to Use

Two signals:
1. Object moves in a **continuous loop** around a perimeter (rectangle, circle, polygon)
2. Steps can be **very large** (num can be 10^9+) — simulating step-by-step is too slow

---

## The Trap

Simulating movement wall-by-wall using nested `if/else` bounds checking.

```java
// WRONG — simulates every step or every wall segment
while (steps > 0) {
    if (dir == RIGHT && x < W - 1) { x++; steps--; }
    else if (dir == RIGHT) { dir = DOWN; }
    // ... combinatorial explosion of edge cases
}
```

Breaks on massive inputs, complex to maintain, easy to get off-by-one on corners.

---

## The Mental Model — "The Race Track"

> Take the perimeter of the grid. Cut it at (0,0) with scissors.
> Unroll it into a single straight 1D line.
>
> Instead of tracking (x, y, dir), track one integer: `pos` — distance from the start line.

**Perimeter length:**
```
perimeter = 2 * (W - 1) + 2 * (H - 1)
```

**Taking a massive step:**
```java
pos = (pos + steps) % perimeter;
```

One modulo. No simulation. O(1) per move.

---

## The Two Sub-Models

### 1. "The Unrolled Loop"

Map positions to a straight line:

```
Corner positions on the 1D track (clockwise from top-left):

(0,0) ──► (W-1,0) ──► (W-1,H-1) ──► (0,H-1) ──► back to (0,0)
  0         W-1        W-1+H-1       2*(W-1)+H-1    perimeter
```

| Segment | pos range | Side |
|---|---|---|
| Top edge | `[0, W-2]` | Moving Right |
| Right edge | `[W-1, W+H-3]` | Moving Down |
| Bottom edge | `[W+H-2, 2W+H-4]` | Moving Left |
| Left edge | `[2W+H-3, 2W+2H-5]` | Moving Up |

### 2. "The Anchor Points"

To translate `pos` back to `(x, y)`:

1. **Which edge?** — use `if` conditions on `pos`
2. **How far past the corner?** — subtract that corner's distance from `pos`

```java
int getX(int pos, int W, int H) {
    int right  = W - 1;
    int bottom = right + (H - 1);
    int left   = bottom + (W - 1);

    if (pos <= right)  return pos;                    // top edge
    if (pos <= bottom) return W - 1;                  // right edge (fixed x)
    if (pos <= left)   return left - pos;             // bottom edge (moving left)
    return 0;                                          // left edge (fixed x=0)
}

int getY(int pos, int W, int H) {
    int right  = W - 1;
    int bottom = right + (H - 1);
    int left   = bottom + (W - 1);

    if (pos <= right)  return 0;                      // top edge (fixed y=0)
    if (pos <= bottom) return pos - right;            // right edge
    if (pos <= left)   return H - 1;                  // bottom edge (fixed y)
    return (W - 1 + H - 1 + W - 1 + H - 1) - pos;   // left edge
}
```

---

## The Template (Java)

```java
class Solution {
    int W, H, perimeter;
    int pos = 0; // current 1D position on the track

    void init(int width, int height) {
        W = width; H = height;
        perimeter = 2 * (W - 1) + 2 * (H - 1);
    }

    void move(int steps) {
        pos = (pos + steps) % perimeter;
    }

    int[] getCoords() {
        return new int[]{ getX(), getY() };
    }

    private int getX() {
        int r = W - 1, b = r + H - 1, l = b + W - 1;
        if (pos <= r) return pos;
        if (pos <= b) return W - 1;
        if (pos <= l) return l - pos;
        return 0;
    }

    private int getY() {
        int r = W - 1, b = r + H - 1, l = b + W - 1;
        if (pos <= r) return 0;
        if (pos <= b) return pos - r;
        if (pos <= l) return H - 1;
        return perimeter - pos;
    }
}
```

---

## Recognizing It Fast

```
Object moves in loop/perimeter + can take huge steps
                    ↓
        pos = (pos + steps) % perimeter
        Only convert to (x, y) when asked
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Simulating step-by-step | Use `% perimeter` — one operation |
| Off-by-one on perimeter | Corners count once: `2*(W-1) + 2*(H-1)`, not `2*W + 2*H` |
| Converting to (x,y) too early | Keep 1D as long as possible; only translate when reading coordinates |
| Wrong anchor subtraction | Draw the segment on paper: `bottom_edge_x = left_corner - pos`, not `pos - left_corner` |
| Forgetting `% perimeter` can return 0 | pos=0 is valid — it's back at start. Don't treat 0 as invalid. |

---

## Problems Using This Pattern

| LeetCode | Problem | Cycle |
|---|---|---|
| 2257 | Count Unguarded Cells in Grid | Partial — directional |
| 1996 | The Number of Weak Characters | 1D line sorting variant |
| 874 | Walking Robot Simulation | Grid movement (partial) |
| Custom | Rectangle Perimeter Walker | Full perimeter race track |
