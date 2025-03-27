
# Binary Search

## Description of Problem 🔍🔢📈

Given an array of integers `nums` which is sorted in ascending order, and an integer `target`, write a function to search for the target in `nums`. If the target exists, return its index. Otherwise, return `-1`.

You must write an algorithm with `O(log n)` runtime complexity.

LeetCode Link: [https://leetcode.com/problems/binary-search/](https://leetcode.com/problems/binary-search/)

---

## Constraints 📌🚧📏

- 1 <= nums.length <= 10^4
- -10^4 < nums[i], target < 10^4
- All the integers in `nums` are unique.
- `nums` is sorted in ascending order.

---

## Algorithm/Data Structure to Use 🛠️🧩📐

- **Divide and Conquer**

---

## Solution 💡🚀📚

### Brute Force Approach ⏳🔨🧱

1. The brute force approach would be to iterate through each element and check if it matches the target.
2. This approach has a time complexity of **O(n)**.

### Optimized Approach (Using Divide and Conquer) ⚡📈🏹

1. The goal is to solve this problem in **O(log n)** time, which indicates we should repeatedly divide the search space in half.
2. This suggests using a Divide and Conquer approach, specifically, binary search.
3. The binary search algorithm can be implemented recursively or iteratively. Here, we explore the recursive approach:
   - Initialize `start` at `0` and `end` at `nums.length - 1`.
4. Find the midpoint (`mid`) for each recursive call:
   - `mid = start + (end - start) // 2`
5. Base condition for recursion:
   - Continue searching while `start <= end`.
6. Check the middle element:
   - If `nums[mid]` is equal to the `target`, return `mid`.
   - If `nums[mid]` is greater than `target`, search in the left half (`end = mid - 1`).
   - If `nums[mid]` is less than `target`, search in the right half (`start = mid + 1`).
7. Continue recursively until the base condition fails. If the target isn't found, return `-1`.

---

## Complexity Analysis 📊⏱️🔍

- **Time Complexity**: O(log n), since we repeatedly divide the search interval by half.
- **Space Complexity**: O(log n), due to the recursive stack.

---

## Examples 🧩🎯📝

### Example 1:

**Input:**nums = [-1,0,3,5,9,12], target = 9

**Output:**4

**Explanation:**9 exists in nums, and its index is 4.

### Example 2:

**Input:**nums = [-1,0,3,5,9,12], target = 2

**Output:**-1

**Explanation:**2 does not exist in nums, so return -1.

---

## Summary 📘📌✅

- **Brute Force**: Check each element. Time: O(n).
- **Optimized**: Binary search (Divide and Conquer). Time: O(log n), Space: O(log n) for recursion.

---

## Note 📚⚠️💡

- Always choose the correct half to search based on comparison with the midpoint.
- Remember to handle edge cases properly, such as single-element arrays or when the target is at the edges.
