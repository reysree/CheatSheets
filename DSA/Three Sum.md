
# Three Sum Problem

## Description of Problem

Given an integer array `nums`, return all the triplets `[nums[i], nums[j], nums[k]]` such that `i != j`, `i != k`, and `j != k`, and `nums[i] + nums[j] + nums[k] == 0`.

Notice that the solution set must not contain duplicate triplets.

LeetCode Link : [https://leetcode.com/problems/3sum/](https://leetcode.com/problems/3sum/)

---

## Constraints

- 3 <= nums.length <= 3000
- -10^5 <= nums[i] <= 10^5

---

## Algorithm/Data Structure to Use

- **Sorting**, **Two Pointers**

---

## Solution

### Brute Force Approach

1. The brute force way is to find the sum of each triplet of elements in the array to match the target value (0).
2. This solution takes **O(n^3)** time complexity, where `n` is the length of the array.

### Optimized Approach (Using Sorting and Two Pointers)

1. As we don't need to return indices but the elements whose sum adds up to zero, we can **sort** the array to make it easier. Sorting also helps in detecting duplicates easily.
2. Loop through the array with an index `i`:
   - Skip duplicates in outer loop by incrementing `i` if current and previous values are same that is `nums[i] == nums[i-1]` (for `i > 0`).
   - Iterate only until `nums.length - 2` because we need two more elements to form a triplet.
3. For each `i`, use two pointers:
   - `j = i + 1` (next element)
   - `k = nums.length - 1` (last element)
4. While `j < k`:
   - Calculate `sum = nums[i] + nums[j] + nums[k]`.
   - If `sum == 0`, add `[nums[i], nums[j], nums[k]]` to result list.
   - Skip duplicates for `j` and `k`:
     - Increment `j` while `nums[j] == nums[j + 1]`.
     - Decrement `k` while `nums[k] == nums[k - 1]`.
   - After handling duplicates, increment `j` and decrement `k` to continue.
5. If `sum < 0`, increment `j` to increase the sum as the j pointer value is making the sum negative.
6. If `sum > 0`, decrement `k` to decrease the sum as the k pointer value is making the sum positive.
7. Continue until `j` and `k` overlap.

---

## Complexity Analysis

- **Time Complexity**: O(n^2), where `n` is the length of the array. Sorting takes O(n log n), and the two-pointer approach takes O(n^2).
- **Space Complexity**: O(1), as no extra space is used other than the output list.

---

## Example

### Example 1:

**Input:**\
nums = [-1,0,1,2,-1,-4]

**Output:**\
[[-1,-1,2], [-1,0,1]]

**Explanation:**\
- (-1) + 0 + 1 = 0
- (-1) + (-1) + 2 = 0

### Example 2:

**Input:**\
nums = [0,1,1]

**Output:**\
[]

**Explanation:**\
No triplet adds up to 0.

### Example 3:

**Input:**\
nums = [0,0,0]

**Output:**\
[[0,0,0]]

**Explanation:**\
The only triplet that sums up to 0.

---

## Summary

- **Brute Force**: Check all triplets. Time: O(n^3).
- **Optimized**: Sort + Two Pointers. Time: O(n^2), Space: O(1).

---

## Note

- Avoid duplicates while considering elements and while adding to result.
- Only unique triplets should be returned.
