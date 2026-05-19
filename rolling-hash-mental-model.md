# Rolling Hash — Mental Model

## Core Idea

| Approach | Cost per window | Total |
|---|---|---|
| Recompute hash from scratch | O(k) | O(n×k) |
| Rolling hash | O(1) | O(n) |

Rolling hash: drop leftmost char, add rightmost char — O(1) per slide.

---

## When to Use (Pattern Recognition)

### Signal Keywords
| Phrase in problem | Signal |
|---|---|
| "repeated substring of length k" | Fixed window + duplicate check |
| "duplicate subarray/substring" | Rolling hash + HashSet |
| "longest duplicate substring" | Binary search on length + rolling hash |
| "all substrings of length k" | Enumerate + deduplicate via hash |
| "find pattern in text" | Rabin-Karp |

### 3-Question Test
1. Fixed-length window sliding across string/array?
2. Need to compare window **identity** (equality), not just track a running value?
3. Naive O(n×k) too slow for constraints?

**All 3 yes → Rolling Hash.**

### Rolling Hash vs Similar Patterns
| Pattern | Use when |
|---|---|
| Sliding Window | Variable-size window, track running value (sum/count) |
| Two Pointers | Sorted input, shrink/grow from both ends |
| Rolling Hash | Fixed-size window, need identity check (equality) |
| KMP / Z-algo | Single known pattern — no hashing needed |

> Key distinction: **Sliding window tracks a value. Rolling hash tracks identity.**

---

## The Math

### Polynomial Hash (Horner's Method)

**What it is:** Algorithm by William George Horner (1819) to evaluate polynomials efficiently.
Reduces O(n²) multiplications to O(n) by factoring out powers.

**Direct formula (expensive):**
```
hash("AACT") = 1×4³ + 1×4² + 2×4¹ + 4×4⁰  =  92
```
Needs `Math.pow` or manual power computation.

**Horner's factored form (cheap):**
```
= ((1×4 + 1)×4 + 2)×4 + 4  =  92
```
Same result. No powers needed — just multiply and add, left to right.

Built iteratively:
```
hash = 0
hash = 0×4 + 1  = 1       (A)
hash = 1×4 + 1  = 5       (AA)
hash = 5×4 + 2  = 22      (AAC)
hash = 22×4 + 4 = 92      (AACT)
```

Pattern: `hash = hash × BASE + charValue`

### Rolling Forward
```
old window: [A A C T]   hash = val[0]×4³ + val[1]×4² + val[2]×4 + val[3]
new window: [A C T G]

Step 1 — drop left:   hash - val[0] × BASE^(LEN-1)
Step 2 — shift left:  × BASE
Step 3 — add right:   + val[new]
```

Formula:
```
newHash = (oldHash - leftVal × BASE^(LEN-1)) × BASE + rightVal
```

### Modular Arithmetic (critical)
```java
// Always mod to prevent overflow
hash = (hash * BASE + charVal) % MOD;

// Subtraction can go negative — add MOD before mod
hash = (hash - leftVal * highPow % MOD + MOD) % MOD;
hash = (hash * BASE + rightVal) % MOD;
```

---

## Java Template

```java
public List<String> rollingHashTemplate(String s, int LEN) {
    final long BASE = 31;           // or 4 for DNA (ACGT)
    final long MOD  = 1_000_000_007L;

    // Precompute BASE^(LEN-1)
    // Starts at 1 (= BASE^0), multiply LEN-1 times → BASE^(LEN-1)
    long highPow = 1;
    for (int i = 0; i < LEN - 1; i++) {
        highPow = (highPow * BASE) % MOD;
    }

    Set<Long>   seen   = new HashSet<>();
    Set<String> result = new HashSet<>();   // Set prevents duplicate results

    // Hash first window [0 .. LEN-1]
    long hash = 0;
    for (int i = 0; i < LEN; i++) {
        hash = (hash * BASE + val(s.charAt(i))) % MOD;
    }
    seen.add(hash);

    // Slide: i = start index of current window
    for (int i = 1; i <= s.length() - LEN; i++) {
        long leftVal  = val(s.charAt(i - 1));           // char leaving
        long rightVal = val(s.charAt(i + LEN - 1));     // char entering

        hash = (hash - leftVal * highPow % MOD + MOD) % MOD;
        hash = (hash * BASE + rightVal) % MOD;

        if (!seen.add(hash)) {
            // Collision possible — verify with actual string
            result.add(s.substring(i, i + LEN));
        }
    }

    return new ArrayList<>(result);
}

// For general strings
private long val(char c) { return c - 'a' + 1; }

// For DNA strings (ACGT only)
// A=1, C=2, G=3, T=4  →  use map
```

---

## Key Implementation Details

| Detail | Why |
|---|---|
| `highPow` loop: `i < LEN-1` | Starts at BASE^0=1, multiply LEN-1 times → BASE^(LEN-1). If `i < LEN` → BASE^LEN (wrong) |
| Sliding loop starts at `i=1` | `i` = window start index. First window (i=0) hashed before loop |
| `+ MOD` before `% MOD` | Subtraction can produce negative — MOD keeps it positive |
| `result` is `Set<String>` | Same substring appearing 3+ times → only one entry in result |
| Verify on hash match | Hash collision: two different strings, same hash. `.equals()` confirms |

---

## Variable Naming Clarity

```
s = "AAAAACCCCCAAAAACCCCC"
     ↑         ↑
     i-1       i+LEN-1   (at slide step i)

Window at step i = s[i .. i+LEN-1]
Left char leaving  = s[i-1]
Right char entering = s[i+LEN-1]
```

---

## Binary Search + Rolling Hash (Advanced)

For "longest duplicate substring" type problems:

```
binary search on window length k
    lo = 1, hi = n-1
    mid = (lo + hi) / 2

    check(mid):
        use rolling hash to find all substrings of length mid
        if any duplicate exists → return true

    if check(mid) → lo = mid + 1  (can go longer)
    else          → hi = mid - 1  (too long)
```

Time: O(n log n)

---

## Collision Handling

Two different strings can hash to same value (false positive).

```java
if (!seen.add(hash)) {
    result.add(s.substring(i, i + LEN));  // actual string, not hash
}
```

Using `Set<String> result` handles this — even if false positive adds wrong string,
duplicate problem is about real equality.

For higher safety: **double hashing** (two different BASE/MOD pairs).

---

## Practice Problems

| Problem | Key Pattern | Difficulty |
|---|---|---|
| LC 187 — Repeated DNA Sequences | Rolling hash + HashSet | Medium |
| LC 1044 — Longest Duplicate Substring | Binary search + rolling hash | Hard |
| LC 718 — Maximum Length of Repeated Subarray | Rolling hash or DP | Medium |
| LC 1316 — Distinct Echo Substrings | Rolling hash + counting | Hard |
| LC 28 — Find Index of First Occurrence | Rabin-Karp / rolling hash | Easy |

---

## Mental Checklist Before Coding

- [ ] What is BASE? (4 for DNA, 26/31 for lowercase alpha)
- [ ] What is LEN? Fixed by problem or binary searched?
- [ ] `highPow` = BASE^(LEN-1) — loop runs LEN-1 times
- [ ] Hash first window before sliding loop
- [ ] Sliding loop: `i` is window start, runs `i=1` to `i <= n-LEN`
- [ ] Left char = `s[i-1]`, right char = `s[i+LEN-1]`
- [ ] `+ MOD` before `% MOD` on subtraction
- [ ] Use `Set<String>` for result (deduplication)
- [ ] Verify on hash match (collision safety)
