# Difference Array — Mental Model

## When to Use

Three conditions must hold:
1. There is a shared **axis** (time, index, target value, rotation angle, etc.)
2. Each contributor affects a **contiguous range** of that axis
3. You need min / max / sum across the axis

**Brute force shape that signals this:**
`O(n × range)` → optimize to `O(n + range)`

---

## The One Rule

```
Add v to range [l, r]:
    diff[l]   += v
    diff[r+1] -= v

Read actual value at position t:
    prefix_sum(diff[0..t])
```

- Always `r+1` to close the range
- Signs are always **opposite** at the two ends

---

## Identifying r Correctly

> **Ask: is the right boundary included or excluded?**

| Boundary type       | r        | Close diff at |
|---------------------|----------|---------------|
| Inclusive  `[l, end]`  | `end`    | `end + 1`     |
| Exclusive  `[l, end)`  | `end - 1`| `end`         |

**Examples:**
- Car Pooling: drop off AT `to` → not riding there → range `[from, to-1]` → `diff[to] -= v`
- Complementary Array: cost-1 range `[lo+1, hi+limit]` fully inclusive → `diff[hi+limit+1] += v`
- Population Year: dead AT `death` → alive during `[birth, death-1]` → `diff[death-1950] -= 1`

---

## Multi-Layer Updates

When cost has tiers, stack multiple updates on the same diff array.
Prefix sum collapses all layers into the final value at each position simultaneously.

**Example — Minimum Moves to Make Array Complementary:**

For each pair `(a, b)`, `lo = min(a,b)`, `hi = max(a,b)`, target range `[2, 2*limit]`:

```
T range                   Cost   Diff update
------------------------  -----  ----------------------------
[2, 2*limit]              +2     diff[2]+=2,    diff[2*limit+1]-=2
[lo+1, hi+limit]          -1     diff[lo+1]-=1, diff[hi+limit+1]+=1
a+b (exact point)         -1     diff[a+b]-=1,  diff[a+b+1]+=1
```

Net cost at T=a+b → 0, at one-change zone → 1, elsewhere → 2.

---

## Array Sizing

```
size = max_index_you_write_to + 1
```

- Valid range `[2, 2*limit]` → you write at index `2*limit+1` to close → size = `2*limit+2`
- Year range `[1950, 2050]` → offset by 1950 → indices `[0, 100]` → size = `101`

---

## Sweep to Answer

```java
int ans = Integer.MAX_VALUE, curr = 0;
for (int t = start; t <= end; t++) {
    curr += diff[t];           // prefix sum = actual value at t
    ans = Math.min(ans, curr); // or Math.max, or threshold check
}
```

---

## Cost Table for Complementary Array (per pair)

| T range                   | Moves needed | Reason                        |
|---------------------------|--------------|-------------------------------|
| `[2, lo]`                 | 2            | Neither element can stay      |
| `[lo+1, hi+limit]`        | 1            | Keep one, change other        |
| `a+b` (exact)             | 0            | Already correct               |
| `[hi+limit+1, 2*limit]`   | 2            | Neither element can stay      |

**Keep lo:** need `T - lo` in `[1, limit]` → `T in [lo+1, lo+limit]`  
**Keep hi:** need `T - hi` in `[1, limit]` → `T in [hi+1, hi+limit]`  
Union = `[lo+1, hi+limit]`

---

## Problems Using This Pattern

| LeetCode | Problem                              | Axis          | Goal             |
|----------|--------------------------------------|---------------|------------------|
| 1094     | Car Pooling                          | Stop index    | Max occupancy    |
| 1109     | Corporate Flight Bookings            | Seat index    | Sum of bookings  |
| 1854     | Maximum Population Year              | Year          | Max population   |
| 1589     | Maximum Sum Obtained of Any Permutation | Index      | Max range sum    |
| 798      | Smallest Rotation with Highest Score | Rotation angle| Max score        |
| 2536     | Increment Submatrices by One         | 2D grid       | Range update     |
| 1893     | Check if All the Integers in a Range Are Covered | Number line | Coverage check |
| 2381     | Shifting Letters II                  | String index  | Net shift        |
| 2381 (hard) | Minimum Moves to Make Array Complementary | Target sum | Min moves    |

---

## Java Template

```java
// 1. Build diff array
int[] diff = new int[maxVal + 2]; // +2 for safe r+1 writes

for (each contributor with range [l, r] and value v) {
    diff[l]   += v;
    diff[r+1] -= v;
}

// 2. Prefix sum sweep
int curr = 0, ans = Integer.MAX_VALUE; // or MIN_VALUE for max
for (int i = start; i <= end; i++) {
    curr += diff[i];
    ans = Math.min(ans, curr); // adjust based on goal
}
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| `diff[r+1]` for exclusive boundary | Use `diff[r]` — boundary is already one past the range |
| Wrong array size | Size = max index you write + 1 |
| `+= -1` instead of `-= 1` | Equivalent but easy to misread; prefer `-= 1` |
| Sweeping wrong range | Start and end must cover the full valid axis |
