# Overlapping Choices — 1D State-Machine DP Mental Model

## When to Use

Three signals:
1. Entities on a **1D line** (robots, houses, stock prices)
2. Each entity makes a **mutually exclusive choice** (left vs right, rob vs skip, buy vs sell)
3. Adjacent entities **interfere** — one entity's choice limits what counts for the next

---

## The Trap

**Greedy fails here.** Asking "what's best for entity i independently?" double-counts shared resources (rooms, targets, gaps) between adjacent entities.

```
Robot 1 shoots Right → claims the room between Robot 1 and Robot 2
Robot 2 also shoots Left → claims the same room
Greedy counts that room's score twice. Wrong.
```

DP tracks what each entity *already committed to*, so the next entity can only claim what's left.

---

## The Two Mental Models

### 1. "The Segmented Chain" (Rooms and Doors)

> Because bullets / choices can't pass through other entities, the 1D line is split into isolated **Rooms** between consecutive entities.
> Only the two entities bordering a room can affect it.

```
Entity:  [0]    [1]    [2]    [3]
Room:        (A)    (B)    (C)

Room A → only Entity 0 and Entity 1 can score from it
Room B → only Entity 1 and Entity 2
Room C → only Entity 2 and Entity 3
```

This means: each transition in the DP only needs to consider one room — the room between entity `i-1` and entity `i`.

### 2. "The Sticky Notes" (State Tracking)

> Instead of committing entity `i` to a final decision, carry forward **two sticky notes**:
> - `left[i]` = max score if entity `i` chose LEFT
> - `right[i]` = max score if entity `i` chose RIGHT
>
> When computing entity `i+1`, look at entity `i`'s sticky notes, calculate overlap in the room between them, write new sticky notes for `i+1`.

```
left[i]  = best score with entity i pointing LEFT
right[i] = best score with entity i pointing RIGHT

Transitions at entity i+1:
    left[i+1]  depends on left[i] and right[i] — which of i's states avoid conflict?
    right[i+1] depends on left[i] and right[i] — same logic
```

---

## The Template (Java)

```java
// entities[] = the 1D array of entities
// score(room, direction) = score for a room based on entity's choice

int n = entities.length;
long[] left  = new long[n];   // max score if entity i shoots left
long[] right = new long[n];   // max score if entity i shoots right

// Base case: first entity
left[0]  = score(leftChoice,  entities[0]);
right[0] = score(rightChoice, entities[0]);

for (int i = 1; i < n; i++) {
    int room = /* value of room between entity i-1 and entity i */;

    // Entity i shoots LEFT: scores from the room to its left
    // Entity i-1 must have shot RIGHT (away from this room) OR also Left (no conflict in this room)
    left[i]  = Math.max(left[i-1], right[i-1]) + scoreLeft(room, entities[i]);

    // Entity i shoots RIGHT: scores from room to its right (computed when we process i+1)
    // Entity i-1 shooting RIGHT means it claimed the room between them — conflict check
    right[i] = Math.max(
        left[i-1]  + scoreRight(entities[i]),  // i-1 shot left, room unclaimed by i-1
        right[i-1] + scoreRight(entities[i])   // i-1 shot right but right means away — depends on problem
    );
}

return Math.max(left[n-1], right[n-1]);
```

---

## Problem-Specific Sticky Notes

### Robots in a Room (shooting left/right, claim gap score)

```java
// robots[] sorted positions, gaps[i] = score between robot i-1 and robot i
// Each robot shoots left (claims gap[i]) or right (claims gap[i+1])

long[] L = new long[n]; // L[i] = max if robot i shoots left
long[] R = new long[n]; // R[i] = max if robot i shoots right

L[0] = 0;              // no gap to the left of first robot
R[0] = gaps[1];        // first robot shoots right → claims gap between 0 and 1

for (int i = 1; i < n; i++) {
    long gap_left  = gaps[i];     // gap between robot i-1 and i
    long gap_right = gaps[i+1];   // gap between robot i and i+1 (0 if last)

    // Shoots left: claims gap_left
    // If prev also shot right into gap_left → conflict → score that gap once, not twice
    L[i] = Math.max(L[i-1], R[i-1] - gap_left) + gap_left;
    //                        ^^ prev claimed gap_left, undo it, then i claims it

    // Shoots right: claims gap_right (will be settled at i+1)
    R[i] = Math.max(L[i-1], R[i-1]) + gap_right;
}

return Math.max(L[n-1], R[n-1]);
```

### House Robber (rob vs skip)

```java
// Classic 1D state machine — two states: robbed, skipped
int[] rob  = new int[n]; // max if house i is robbed
int[] skip = new int[n]; // max if house i is skipped

rob[0]  = nums[0];
skip[0] = 0;

for (int i = 1; i < n; i++) {
    rob[i]  = skip[i-1] + nums[i]; // must skip previous to rob this one
    skip[i] = Math.max(rob[i-1], skip[i-1]); // free choice on previous
}

return Math.max(rob[n-1], skip[n-1]);
```

---

## Recognizing It Fast

```
1D line + mutual exclusive choices + adjacent interference
                    ↓
        Two arrays: left[i] and right[i]
        Transitions read previous sticky notes
        Final answer: max(left[n-1], right[n-1])
```

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Greedy per entity | DP — entity i's choice constrains entity i+1 |
| Single `dp[i]` instead of two states | Need `left[i]` AND `right[i]` — both states must survive until settled |
| Forgetting to subtract already-claimed gap | When switching from neighbor's Right to own Left, undo neighbor's claim first |
| Wrong room indexing | Room between entity `i-1` and `i` is `gaps[i]` — off-by-one common |
| Not handling boundary robots | First robot has no left gap; last robot has no right gap — seed base case carefully |

---

## Problems Using This Pattern

| LeetCode | Problem | Choices |
|---|---|---|
| 198 | House Robber | Rob vs Skip |
| 213 | House Robber II | Rob vs Skip (circular) |
| 2786 | Visit Array Positions to Maximize Score | Even vs Odd |
| 2731 | Movement of Robots | Left vs Right (segment scoring) |
| 122 | Best Time to Buy and Sell Stock II | Hold vs Sell |
| 309 | Best Time to Buy and Sell Stock with Cooldown | Buy / Sell / Cooldown states |
