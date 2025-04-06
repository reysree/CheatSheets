
# Generate Parentheses

## Description of Problem 🧮📝🔗

You are given an integer `n`. Return all well-formed parentheses strings that can be generated using `n` pairs of parentheses.

LeetCode Link: [https://leetcode.com/problems/generate-parentheses/description/](https://leetcode.com/problems/generate-parentheses/description/)

---

## Constraints 📏📌

- 1 <= n <= 7

---

## Algorithm/Data Structure to Use 🛠️📐🧱

- **Backtracking**
- **Stack**
- **StringBuilder**

---

## Solution 💡🚀✨

### Brute Force Approach ⏳🔨

1. The brute force way is to generate all possible combinations of `2n` characters consisting of `'('` and `')'`, then filter the ones which are valid using a validation function.
2. This is inefficient and has exponential complexity as many invalid combinations are created and discarded.

### Optimized Way (Backtracking) ⚡🔁📦

1. From the problem statement, we understand that we have `n` pairs of parentheses, meaning `n` opening `'('` and `n` closing `')'` brackets.
2. For a string of parentheses to be valid (well-formed), every closing bracket must have a corresponding opening bracket before it.
3. This implies:
   - At any point, the count of `'('` should be **greater than or equal to** the count of `')'`.
4. We maintain two counters:
   - `openCount` → number of opening brackets `(` used
   - `closeCount` → number of closing brackets `)` used
5. We build the string using a backtracking approach:
   - If `openCount` < `n`, we add `'('` and recurse.
   - If `closeCount` < `openCount`, we add `')'` and recurse.
6. When both `openCount` and `closeCount` reach `n`, we have a valid combination which we add to our result list.
7. Backtracking ensures all valid combinations are explored and added, avoiding the need for post-generation validation.
8. This means that, unlike brute force where you generate all combinations first and then check for validity, here you are building valid combinations only as you go. So there's no need to filter out invalid ones later, saving both time and space.

---

## Example 🧩💬📘

### Example 1:
**Input:**
```
n = 1
```
**Output:**
```
["()"]
```

### Example 2:
**Input:**
```
n = 3
```
**Output:**
```
["((()))","(()())","(())()","()(())","()()()"]
```

---

## Complexity Analysis 📊⏱️🧠

- **Time Complexity**: O(4^n / √n) — Catalan number generation
- **Space Complexity**: O(4^n / √n) for storing all combinations + recursion stack

---

## Summary 📌✅🗂️

- **Brute Force**: Generate all permutations → validate each → slow and redundant
- **Optimized**: Backtracking with pruning → efficient and elegant

---

## Note 🧾🧠💡

- Ensure you never place a `)` unless there’s a matching `(` already placed.
- Backtracking allows clean rollback to try all possibilities while obeying constraints at every step.
