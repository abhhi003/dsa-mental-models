# Stack — Chain Reaction Pattern Mental Model

## When to Use

Three signals must all appear:
1. Array or string of items on a **1D line**
2. Items **move toward each other**, combine, or destroy each other
3. Problem asks for the **final state** of the array

---

## The Trap

Using a single `if/else` to compare incoming item against `stack.peek()`.

```java
// WRONG — only one interaction
if (!stack.isEmpty() && collides(stack.peek(), incoming)) {
    stack.pop();
} else {
    stack.push(incoming);
}
```

This lets the incoming item interact **once then stops**. Chain reactions require repeated interactions.

---

## The Mental Model — "The Bowling Ball"

> An incoming item moving against the flow is a **bowling ball**.
> It doesn't hit one pin and stop — it keeps rolling until the lane is empty or it hits something heavier.

**The incoming item must fight the entire stack, not just the top.**

---

## The Template (Java)

```java
Deque<Integer> stack = new ArrayDeque<>();

for (int item : items) {
    boolean survived = true;

    while (!stack.isEmpty() && collides(stack.peek(), item)) {
        if (stack.peek() beats item) {
            survived = false;
            break;
        } else if (stack.peek() equals item) {
            survived = false;
            stack.pop();
            break;
        } else {
            // item wins this round — keep rolling
            stack.pop();
        }
    }

    if (survived) stack.push(item);
}
```

Three outcomes per collision:
| Result | Action |
|---|---|
| Incoming **loses** | `survived = false; break` |
| **Tie** | `survived = false; stack.pop(); break` |
| Incoming **wins** | `stack.pop()` — continue while loop |

---

## Collision Condition (`collides`)

Only trigger the while loop when a real collision is possible.

**Example — Asteroid Collision (LC 735):**
- Positive = moving right, Negative = moving left
- Collision only when `stack.peek() > 0 && item < 0` (right meets left)
- No collision: two rights, two lefts, or left then right (moving apart)

```java
// collision condition
!stack.isEmpty() && stack.peek() > 0 && item < 0
```

---

## Signal Keywords

| Problem says... | Think... |
|---|---|
| "moving left/right, destroy on collision" | Chain reaction stack |
| "remove adjacent duplicates" | Chain reaction stack |
| "final state after all interactions" | Chain reaction stack |
| "items cancel/combine when they meet" | Chain reaction stack |
| "score/size changes on collision" | Chain reaction stack |

---

## Recognizing It Fast

```
1D array + items interact with neighbors + ask final state
             ↓
         Stack + while loop
```

If you catch yourself writing nested `if/else` to handle "but what if it also destroys the next one" — stop. That's the chain reaction signal. Switch to `while`.

---

## Problems Using This Pattern

| LeetCode | Problem | Collision Rule |
|---|---|---|
| 735 | Asteroid Collision | right(+) hits left(-), larger survives |
| 2216 | Minimum Deletions to Make Array Beautiful | pair destruction |
| 1047 | Remove All Adjacent Duplicates in String | equal neighbors cancel |
| 1209 | Remove All Adjacent Duplicates II | k equal neighbors cancel |
| 2390 | Removing Stars from String | star destroys left neighbor |
| 844 | Backspace String Compare | `#` destroys left character |

---

## Common Mistakes

| Mistake | Fix |
|---|---|
| Single `if/else` instead of `while` | Always `while(!stack.isEmpty() && collides(...))` |
| Forgetting `survived` flag | Without it, item gets pushed even after it dies |
| Wrong collision condition | Only collide when items actually face each other |
| Tie case forgotten | Equal-size collision: both die → `pop()` + `survived=false` |
| Using `Stack<>` class | Use `Deque<> stack = new ArrayDeque<>()` — faster in Java |

---

## Why Stack (Not Array/List)

The chain reaction always involves the **most recent survivor** — that's the stack top. Each item only ever fights the item directly in front of it. Stack gives O(1) peek + pop for exactly this access pattern.

Total complexity: **O(n)** — each item pushed and popped at most once.
