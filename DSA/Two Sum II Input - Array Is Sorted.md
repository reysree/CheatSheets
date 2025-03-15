
# Two Sum II - Input Array Is Sorted

## Description of Problem

Given a 1-indexed array of integers `numbers` that is already sorted in non-decreasing order, find two numbers such that they add up to a specific target number. Let these two numbers be `numbers[index1]` and `numbers[index2]` where `1 <= index1 < index2 <= numbers.length`.

Return the indices of the two numbers, `index1` and `index2`, added by one as an integer array `[index1, index2]` of length 2.

The tests are generated such that there is exactly one solution. You may not use the same element twice.

Your solution must use only constant extra space.

**LeetCode Link**: [Two Sum II - Input Array Is Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/description/)

---

## Algorithm/Data Structure to Use

- **Two Pointer**

---

## Solution

### Brute Force Approach

1. The brute force way is to find the sum of each pair of elements in the array to match the target value.
2. This solution takes **O(n^2)** time complexity, where `n` is the length of the array.

### Optimized Approach (Using Two Pointer)

1. As the input array is sorted, we don't need to store the values like the regular Two Sum problem.
2. Since the array is sorted and the solution involves adding two elements `x + y = target`, the Two Pointer algorithm is a perfect fit.
3. We initialize two pointers: one at the beginning of the array (`beg`) and one at the end of the array (`end`).
4. In every iteration, we check if `numbers[beg] + numbers[end] == target`. If so, we found our solution.
5. If not found, we check if the sum is greater than the target:
   - If **sum > target**, it means the value at the `end` pointer is too large, so we decrement `end`.
6. If the sum is less than the target:
   - If **sum < target**, it means the value at the `beg` pointer is too small, so we increment `beg`.
7. We repeat this process until the `beg` and `end` indices overlap.

---

## Complexity Analysis

- **Time Complexity**: O(n), where `n` is the length of the array. We traverse the array at most once.
- **Space Complexity**: O(1), constant extra space.

---

## Example

**Input:**\
numbers = [2, 7, 11, 15], target = 9

**Output:**\
[1, 2]

**Explanation:**\
The sum of numbers[1] and numbers[2] is 2 + 7 = 9. Therefore, we return [1, 2].

---

## Summary

- **Brute Force**: Check all pairs. Time: O(n^2).
- **Optimized**: Two Pointer approach for sorted array. Time: O(n), Space: O(1).

---

## Note

- Make sure to return indices as 1-indexed.
- You can return the indices in any order as long as `index1 < index2`.
