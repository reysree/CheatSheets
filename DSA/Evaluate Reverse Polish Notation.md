
# Evaluate Reverse Polish Notation

## Description of Problem 🧮🔢🧠

You are given an array of strings `tokens` that represents a valid arithmetic expression in **Reverse Polish Notation**.

Return the integer that represents the evaluation of the expression.

- The operands may be integers or results of other operations.
- The operators include `+`, `-`, `*`, and `/`.
- Division between integers should always truncate toward zero.

LeetCode Link: [https://leetcode.com/problems/evaluate-reverse-polish-notation/description/](https://leetcode.com/problems/evaluate-reverse-polish-notation/description/)

---

## Constraints 📏📚✅

- 1 <= tokens.length <= 1000
- `tokens[i]` is one of `"+"`, `"-"`, `"*"`, `"/"`, or a string representing an integer in the range `[-100, 100]`

---

## Algorithm/Data Structure to Use 🧰📐🧱

- **Stack**

---

## Solution 💡🚀✨

### Brute Force Approach ⏳🔨

1. The brute force way is to convert the postfix notation to infix and then evaluate the expression by applying the BODMAS rule.
2. But this becomes unnecessarily complex and inefficient.

### Optimized Way (Using Stack) ⚡📦🧠

1. From the problem, we understand that every two numbers in the array are followed by an operator (postfix form), and we must evaluate them following mathematical rules.
2. This is a classic use case of using a **Stack**.
3. Algorithm:
   - Traverse the array `tokens` from left to right.
   - For each element:
     - If it's a number, push it onto the stack.
     - If it's an operator (`+`, `-`, `*`, `/`):
       - Pop the top two elements from the stack.
       - Perform the operation: `secondOperand operator firstOperand`.
         (Note: the second popped value is actually the first operand in the operation due to stack order.)
       - Push the result back onto the stack.
   - After the loop ends, the stack will contain one final result, which is our answer.

---

## Example 🧩🔢💬

### Example 1:

**Input:**
```
tokens = ["1","2","+","3","*","4","-"]
```

**Output:**
```
5
```

**Explanation:**
```
((1 + 2) * 3) - 4 = 5
```

---

## Complexity Analysis 📊🧠🧮

- **Time Complexity**: O(n), where n is the number of tokens.
- **Space Complexity**: O(n), for the stack.

---

## Summary 📘✅🗂️

- **Brute Force**: Convert to infix, apply BODMAS — complex and inefficient.
- **Optimized**: Use a stack to evaluate directly — Time: O(n), Space: O(n).

---

## Note 🧾⚠️🧠

- Always remember that in stack operations, the **last popped** is the **first operand**, and the **second popped** is the **second operand** in arithmetic operations.
- Ensure that integer division truncates towards zero (language-specific behavior may vary).
