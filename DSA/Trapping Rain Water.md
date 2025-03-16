
# Trapping Rain Water

## Description of Problem 🌧️🌊📐

Given `n` non-negative integers representing an elevation map where the width of each bar is `1`, compute how much water it can trap after raining.

LeetCode Link: [https://leetcode.com/problems/trapping-rain-water/](https://leetcode.com/problems/trapping-rain-water/)

---

## Constraints 📏🚧🧱

- n == height.length
- 1 <= n <= 2 * 10^4
- 0 <= height[i] <= 10^5

---

## Algorithm/Data Structure to Use 🛠️🧠📈

- **Two Pointer**, **Prefix and Suffix Sum**

---

## Solution 💡🚀✨

### Brute Force Approach ⏳🔨🧱

1. The brute force way is to iterate through every index and calculate the left and right maximum heights separately, then find the water stored at each index.
2. This approach takes **O(n²)** time complexity.

### Optimized Approach (Using Two Pointer & Prefix-Suffix Max Arrays) ⚡📈🏹

1. We are given an array of heights representing walls.
2. At any index `i`, the amount of water it can contain depends on the tallest walls on its left and right sides. Specifically, the maximum water level at any position is limited by the shorter of the two surrounding walls.
3. For efficient calculation, we utilize two arrays, `leftMax` and `rightMax`:
   - `leftMax[i]`: Maximum height to the left of the index `i` (inclusive).
   - `rightMax[i]`: Maximum height to the right of the index `i` (inclusive).
4. To calculate `leftMax`, iterate from left to right, updating each position with the maximum wall height encountered so far.
5. To calculate `rightMax`, iterate from right to left, again updating with the maximum encountered height.
6. Now for each index `i`, the trapped water is calculated by taking the smaller height between `leftMax[i]` and `rightMax[i]` and subtracting the height of the current wall `height[i]`. Formally:
   - `trappedWaterAt[i] = min(leftMax[i], rightMax[i]) - height[i]`
7. The sum of these individual trapped water values provides the total trapped rainwater.

---

## Complexity Analysis 📊⏱️🚦

- **Time Complexity**: O(n), as it requires three separate passes through the array.
- **Space Complexity**: O(n), due to the two additional arrays `leftMax` and `rightMax`.

---

## Examples 🧩🔎🎯

### Example 1:

**Input:**height = [0,1,0,2,1,0,1,3,2,1,2,1]

**Output:**6

**Explanation:**The elevation map traps 6 units of rainwater.

### Example 2:

**Input:**height = [4,2,0,3,2,5]

**Output:**9

**Explanation:**The elevation map traps 9 units of rainwater.

---

## Summary 📌✅📘

- **Brute Force**: Calculate left-right max at each position. Time: O(n²).
- **Optimized**: Prefix and Suffix max arrays using Two pointers. Time: O(n), Space: O(n).

---

## Note 📚⚠️💡

- Always compute maximum height from both left and right before calculating trapped water at each index.
- Remember to handle cases with zero heights and edges properly.
