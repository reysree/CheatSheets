
# Container With Most Water

## Description of Problem 🌊📏💧

You are given an integer array `height` of length `n`. There are `n` vertical lines drawn such that the two endpoints of the `i-th` line are `(i, 0)` and `(i, height[i])`.

Find two lines that together with the x-axis form a container, such that the container contains the most water. 🌊📏💧

Return the maximum amount of water a container can store.

Notice that you may not slant the container.

LeetCode Link: [https://leetcode.com/problems/container-with-most-water/](https://leetcode.com/problems/container-with-most-water/)

---

## Constraints 📌🚧📝

- n == height.length
- 2 <= n <= 10^5
- 0 <= height[i] <= 10^4

---

## Algorithm/Data Structure to Use 🧠⚙️🔍

- **Two Pointer**

---

## Solution 💡🚀✨

### Brute Force Approach 🧱⏳🔨

1. The brute force way is to check every possible pair of lines, calculate the area, and keep track of the maximum.
2. This solution takes **O(n²)** time complexity.

### Optimized Approach (Using Two Pointer) 🚀⚡🏹

1. To find the maximum water that can be stored, we essentially calculate the area between two lines.
2. Given that height values represent lengths and the distance between indices represents breadth, the area is calculated as: 
   `min(height[i], height[j]) * (j - i)`.
3. The minimum height is chosen because water cannot exceed the height of the shorter wall.
4. Efficiently solve using **Two Pointers**:
   - One pointer (`start`) at the beginning and another pointer (`end`) at the end of the array.
5. Calculate the area between these two pointers and keep track of the maximum area found so far.
6. Move the pointers:
   - If `height[start]` is less than `height[end]`, increment `start`.
   - Otherwise, decrement `end`.
7. Continue this process until pointers overlap.

---

## Complexity Analysis 📈🕒📊

- **Time Complexity**: O(n), where `n` is the length of the array. It requires a single pass through the array.
- **Space Complexity**: O(1), constant extra space.

---

## Examples 🔎🎯🧩

### Example 1:

**Input:**height = [1,8,6,2,5,4,8,3,7]

**Output:**49

**Explanation:**The max area of water the container can contain is 49.

### Example 2:

**Input:**height = [1,1]

**Output:**1

**Explanation:**The maximum water that can be stored is 1.

---

## Summary 🗒️✅📚

- **Brute Force**: Check all pairs. Time: O(n²).
- **Optimized**: Two Pointers approach. Time: O(n), Space: O(1).

---

## Note 📝⚠️💡

- Always consider the shorter line for calculating area.
- Move the pointer pointing to the shorter line to find taller lines and maximize area.
