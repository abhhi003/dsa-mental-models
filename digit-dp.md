# Digit DP — Mental Model

## When to Use
- Range [L, R] with R > 10^5
- Property defined **per digit** (count, sum, uniqueness, pattern)
- "Can I decide contribution digit by digit, left to right?" → YES = Digit DP

## Core Formula
```
answer = f(R) - f(L-1)
f(N) = compute property for all numbers in [0, N]
```

## Universal Template (Java)

### State
| Variable | Meaning | Size |
|---|---|---|
| `pos` | current digit position | len(N) |
| `tight` | bounded by N? | 2 |
| `started` | first non-zero placed? | 2 |
| `...extra` | problem-specific | varies |

### Skeleton
```java
private T helper(int pos, boolean tight, boolean started, ...extra) {
    if (pos == n) return BASE_CASE;  // count→1, sum→0

    if (memoized) return memo[...];

    int limit = tight ? s.charAt(pos) - '0' : 9;

    for (int digit = 0; digit <= limit; digit++) {
        boolean newTight   = tight && (digit == limit);
        boolean newStarted = started || (digit != 0);

        if (!started && digit == 0) {
            // leading zero — don't update mask/state
            recurse(pos+1, false, false, ...unchanged_extra);
        } else {
            // real digit — update state, check constraints
            recurse(pos+1, newTight, newStarted, ...updated_extra);
        }
    }

    return memo[...] = result;
}
```

## Return Type by Problem

| Problem Type | Return | Base Case | Contribution |
|---|---|---|---|
| Count numbers | `int` | `1` | `+1` per valid number |
| Sum of property | `int[]{count, sum}` | `{1, 0}` | `sum += digit_val * child[0]` |
| Count digit ones | `int[]{count, ones}` | `{1, 0}` | `if digit==1: ones += child[0]` |

## Extra State by Problem

| Constraint | Extra State | Size |
|---|---|---|
| Digit uniqueness | `mask` (bitmask) | 1024 (2^10) |
| Adjacent relation | `lastDigit` | 10 |
| Two-digit relation | `last, secondLast` | 10 × 10 |
| Digit sum % k | `sum % k` | k |
| Running count | `totalOnes` | len(N) |

## Key Rules

### Tight
```
tight=true  → limit = N[pos]  (bounded by N at this position)
tight=false → limit = 9       (free, any digit valid)
newTight    = tight && (digit == limit)
```
Once free, always free. Never goes back to tight.

### Started (Leading Zeros)
```
newStarted = started || (digit != 0)
```
- Leading zero → DON'T update mask/extra state
- Once started, every digit (including 0) is a real digit of the number

### Memoization
```java
// Init
Arrays.fill(memo, -1);  // or nested loop for multi-dim

// Lookup
if (memo[pos][tight][started] != -1) return memo[pos][tight][started];

// Store
return memo[pos][tight][started] = result;
```

### {count, sum} Pattern
When you need sum of a property (not just count):
- Return `long[]{count, propertySum}`
- Contribution of current digit = `digit_value * child[0]` (count of completions)
- Deep contributions = `child[1]` (sum from recursive calls)
- Update BOTH accumulators in EVERY branch (leading zero and real digit)

## Complement Trick
"At least one X" is hard → compute "all with zero X" and subtract:
```java
// LC 1012: numbers with repeated digit = total - numbers with all unique digits
numDupDigits(N) = N - countSpecialNumbers(N);
```

## Bitmask for Digit Uniqueness
```java
// 10 digits → 10 bits → 1024 states
int mask = 0;

// Check if digit d already used
if ((mask >> d & 1) == 1) continue;

// Mark digit d as used
int newMask = mask | (1 << d);
```
Array dim: `[len][1024][2][2]`

## Problem → Pattern Map

| Problem | Key Extra State | Return Type |
|---|---|---|
| LC 233 Count Digit One | none | `{count, ones}` |
| LC 2376 Special Integers | `mask` (bitmask) | `int` |
| LC 1012 Repeated Digits | reuse LC 2376 | `N - special(N)` |
| LC 3753 Waviness Sum | `last, secondLast` | `{count, waveSum}` |
| LC 357 Unique Digits | no tight needed | pure math |

## When NOT Digit DP
- Property on the number as whole (prime, perfect square) → different technique
- Arithmetic sequences → math
- XOR over range → bit manipulation
- Quick check: "Can I evaluate digit by digit?" → No = not digit DP

## Mental Checklist Before Coding
```
□ f(R) - f(L-1) setup done
□ tight: true at start, newTight = tight && digit == limit
□ started: false at start, newStarted = started || digit != 0
□ leading zero branch skips state update (mask, last, etc.)
□ base case matches return type (count→1, sum→0)
□ memo initialized to -1
□ {count, sum} — both arrays updated in every branch
□ array dimensions cover all state ranges
```
