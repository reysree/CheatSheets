
# Largest Rectangle in Histogram

## Description of Problem 📊📏🔍

Given an array of integers `heights` representing the histogram's bar heights, where the width of each bar is `1`, return the area of the largest rectangle in the histogram.

LeetCode Link: [Largest Rectangle in Histogram](https://leetcode.com/problems/largest-rectangle-in-histogram/description/)

---

## Constraints 📌🚧📏

- 1 <= heights.length <= 10^5
- 0 <= heights[i] <= 10^4

---

## Algorithm/Data Structure to Use 🛠️📈📚

- **Monotonic Stack**

---

## Solution 💡🚀✨

### Brute Force Approach ⏳🧱🔨

1. The brute force way is to consider each possible rectangle by iterating over all pairs of indices, resulting in **O(n²)** complexity.

### Optimized Approach (Using Monotonic Stack) ⚡📈📦

1. The problem is to find the largest rectangle in a histogram, essentially the largest rectangular area.
2. Given each bar's width is `1`, the rectangle's area will depend on the continuous sequence of heights.
3. To efficiently find the largest rectangle, we need to consider the current element relative to its neighbors (previous and next bars). This scenario indicates the use of a **Monotonic Stack**.
4. A monotonic stack stores elements either in strictly increasing or decreasing order. Here, we use increasing order.
5. Approach:
   - Continuously push indices of bars into the stack as long as the current height is greater than or equal to the height at the stack's peek.
   - When encountering a bar height smaller than the peek of the stack, start calculating areas.
6. Why do we calculate at this point? Because the current bar height is less, indicating the maximum area using heights from the stack may have been reached.
7. For each popped bar index, calculate the area as follows:
   - **Height:** The bar at the popped index.
   - **Width:** Calculate the width by finding the range from the previous bar remaining in the stack (`stack.peek()`) to the current bar.
     - If the stack is empty, it means the current bar is the smallest height so far. Hence, the width is the entire length up to the current index.
     - If the stack isn't empty, width is calculated by the difference between the current index and the index of the bar remaining on the stack minus one.

     **Additional Explanation:**
     - The width calculation is derived from the fact that the current bar (just popped from the stack) can extend backward only until it encounters a bar shorter than itself. Thus, the start of the range (`stack.peek() + 1`) points to the immediate right of the last shorter bar.
     - The end of the range (`current_index - 1`) represents the bar immediately to the left of the current position since the current bar itself has caused the condition to break (due to being shorter).
     - Hence, the width formula becomes:
     ```
     width = (current_index - 1) - (stack.peek() + 1) + 1
     width = current_index - stack.peek() - 1
     ```
8. The formula for calculating area:
   ```
   area = height[popped_index] * (current_index - stack.peek() - 1)
   ```
   or if the stack is empty:
   ```
   area = height[popped_index] * current_index
   ```
9. Compare the calculated area with the previously recorded max area and update accordingly.

---

## Complexity Analysis 📊🕒⚙️

- **Time Complexity**: O(n), each bar is pushed and popped once.
- **Space Complexity**: O(n), due to the stack.

---

## Examples 🧩🎯📚

### Example 1:

**Input:**
```
heights = [2,1,5,6,2,3]
```

**Output:**
```
10
```

**Explanation:**
The largest rectangle area is 10.

### Example 2:

**Input:**
```
heights = [2,4]
```

**Output:**
```
4
```

**Explanation:**
The largest rectangle area is 4.

---

## Summary 📌✅📘

- **Brute Force**: Consider all rectangles. Time: O(n²).
- **Optimized**: Monotonic Stack. Time: O(n), Space: O(n).

---

## Note 📚⚠️💡

- Always use monotonic increasing order to track indices of bar heights.
- Pay close attention to the width calculation, especially when the stack becomes empty.
- Ensure all elements are processed after the main loop by popping remaining indices from the stack and calculating their areas.
