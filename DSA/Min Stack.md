
# Minimum Stack

## Description of Problem 📥🧱📌

Design a stack class that supports the following operations:

- `MinStack()` initializes the stack object.
- `void push(int val)` pushes the element `val` onto the stack.
- `void pop()` removes the element on the top of the stack.
- `int top()` gets the top element of the stack.
- `int getMin()` retrieves the minimum element in the stack.

Each function should run in **O(1)** time.

LeetCode Link: [https://leetcode.com/problems/min-stack/description/](https://leetcode.com/problems/min-stack/description/)

---

## Constraints 📏🚧🧠

- -2^31 <= val <= 2^31 - 1
- `pop`, `top`, and `getMin` will always be called on non-empty stacks

---

## Algorithm/Data Structure to Use 🛠️📚

- **Stack**

---

## Solution 💡🚀✨

### Brute Force Approach ⏳🔨

1. The brute force way is to maintain a simple stack and, during the `getMin()` call, iterate through the stack to find the minimum value each time. This leads to **O(n)** time for `getMin()`.

### Optimized Approach (Using Two Stacks) ⚡📦🧠

1. All the operations except `getMin()` are naturally **O(1)** using Java's inbuilt `Stack` data structure.
2. To achieve **O(1)** for `getMin()`, we also need to store the current minimum value when each element is pushed.
3. If we only use a variable to store the minimum value, it cannot be reliably updated during `pop()` operations.
4. So, we maintain another stack called `minStack`, where each element corresponds to the minimum at that state of the main stack.
5. When pushing a value to the main stack:
   - If `minStack` is empty, push the value into it as the minimum.
   - If not, compare the new value being pushed with the current minimum value, which is at the top of the `minStack`. Then, push the smaller of the two into the `minStack`. Doing this ensures that every position in the `minStack` has the minimum value at that point in the main stack. So even when we pop from the main stack, the corresponding minimum at that state can be popped from `minStack`, keeping both stacks in sync and allowing `getMin()` to work in constant time.
   - This ensures consistency between both stacks.
6. During `pop()`, pop from both `mainStack` and `minStack`.
7. Now, `getMin()` simply returns `minStack.peek()`, an **O(1)** operation.

---

## Complexity Analysis 📊🕒🧮

- **Time Complexity**:
  - `push()`, `pop()`, `top()`, `getMin()` = **O(1)** each
- **Space Complexity**:
  - **O(n)** for the additional `minStack`

---

## Example 🧪📌

**Input:**

```
["MinStack", "push", 1, "push", 2, "push", 0, "getMin", "pop", "top", "getMin"]
```

**Output:**

```
[null, null, null, null, 0, null, 2, 1]
```

**Explanation:**

```
MinStack minStack = new MinStack();
minStack.push(1);
minStack.push(2);
minStack.push(0);
minStack.getMin(); // returns 0
minStack.pop();
minStack.top();    // returns 2
minStack.getMin(); // returns 1
```

---

## Summary 📌✅🧾

- **Brute Force**: Use a loop in `getMin()` — Time: O(n)
- **Optimized**: Maintain a second stack (`minStack`) — Time: O(1) for all operations

---

## Note 📎💭🧠

- Synchronizing both stacks ensures the minimum value is always tracked accurately.
- Always remember to update both stacks simultaneously during `push()` and `pop()`.
