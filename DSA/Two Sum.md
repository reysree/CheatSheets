
# Two Sum Problem

## Description of Problem

Given an array of integers `nums` and an integer `target`, return indices of the two numbers such that they add up to `target`.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

You can return the answer in any order.

**LeetCode Link**: [Two Sum Problem on LeetCode](https://leetcode.com/problems/two-sum/)

---

## Algorithm/Data Structure to Use

- **HashMap / HashTable**

---

## Solution

### Brute Force Approach

1. The brute force way is to find the sum of each pair of elements in the array to match the target value.
2. This solution takes **O(n^2)** time complexity, where `n` is the length of the array.

### Optimized Approach (Using HashMap)

1. If we analyze the problem, what we are trying to find is `x + y = target`, where `x` and `y` are elements in the array.
2. So in every iteration, if we somehow find `y`, given `target - x`, then we have found our solution.
3. To do this efficiently, we can use a **HashMap**, where we store the key as the element and the value as its index in the array.
4. Now we iterate through the array and check if `target - x` is present in the HashMap. If it is present, that means we found the pair `(x, y)`.
5. If found, return an array with the indices of `x` and `y`.
6. If not found, add `x` and its index to the map, which might be helpful for the remaining numbers in the array.

---

## Complexity Analysis

- **Time Complexity**: O(n), where `n` is the length of the array. Each lookup and insertion in HashMap takes O(1) on average.
- **Space Complexity**: O(n), as we might store up to `n` elements in the HashMap.

---

## Example

**Input:**\
nums = [2, 7, 11, 15], target = 9

**Output:**\
[0, 1]

**Explanation:**\
Because nums[0] + nums[1] == 9, we return [0, 1].

---

## Summary

- **Brute Force**: Check all pairs. Time: O(n^2).
- **Optimized**: Use HashMap to find complement efficiently. Time: O(n), Space: O(n).

---

## Note

- Make sure not to use the same element twice.
- You can return the indices in any order.
