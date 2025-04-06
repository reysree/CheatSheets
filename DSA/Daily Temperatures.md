
# Daily Temperatures

## Description of Problem 🌡️📅🔍

You are given an array of integers `temperatures` where `temperatures[i]` represents the daily temperature on the `i-th` day.

Return an array `result` such that `result[i]` is the number of days after the `i-th` day until a warmer temperature appears. If there is no future day for a warmer temperature, set `result[i]` to 0.

LeetCode Link: [https://leetcode.com/problems/daily-temperatures/description/](https://leetcode.com/problems/daily-temperatures/description/)

---

## Constraints 📏📌

- 1 <= temperatures.length <= 1000
- 1 <= temperatures[i] <= 100

---

## Algorithm/Data Structure to Use 🛠️📊📚

- **Monotonic Stack**

---

## Solution 💡🚀✨

### Brute Force Approach ⏳🔨

1. The brute force way is to iterate for each day and then look ahead until we find a day with a higher temperature.
2. For each index `i`, iterate through all future indices `j > i` and check if `temperatures[j] > temperatures[i]`. Return `j - i` if found.
3. This has a **Time Complexity of O(n^2)** which is inefficient for large input.

### Optimized Way (Monotonic Stack) ⚡📈📦

1. If we look at the problem carefully, it’s really just about finding out how many days you need to wait to see a warmer temperature. That means for each day's temperature, we want to find the next day that has a higher temperature and count how many days are in between.
2. A monotonic stack is well suited here since we want to keep track of decreasing temperatures and pop when we encounter a warmer one.
3. Create a stack that stores **pairs** of `{temperature, index}` using something like `Stack<int[]>` in Java. For example, if you have `temperatures = [30, 25, 27]`, you first push `new int[]{30, 0}`. Then when you see 25 (index 1), it's less than 30, so you push `new int[]{25, 1}`. Now when you see 27 (index 2), it's warmer than 25, so you pop `new int[]{25, 1}` from the stack and calculate the difference `2 - 1 = 1`. This means it takes 1 day from day 1 to get a warmer temperature. You continue this process for each temperature.
4. For each current temperature:
   - While the stack is not empty **and** the current temperature is greater than the temperature at the index at the top of the stack:
     - Pop the index from the stack.
     - Compute the difference between the current index and the popped index. Store this in the result array.
5. Push the current index to the stack.
6. Continue this until the end of the array. The remaining indices in the stack don’t have a warmer temperature ahead, so their result stays 0.

---

## Example 🧩💬📘

### Example 1:
**Input:**
```
temperatures = [30,38,30,36,35,40,28]
```
**Output:**
```
[1,4,1,2,1,0,0]
```

### Example 2:
**Input:**
```
temperatures = [22,21,20]
```
**Output:**
```
[0,0,0]
```

---

## Complexity Analysis 📊🧠🧮

- **Time Complexity**: O(n), each temperature index is pushed and popped at most once.
- **Space Complexity**: O(n), for the stack and result array.

---

## Summary 📌✅🗂️

- **Brute Force**: Check future temperatures for each day → Time: O(n^2)
- **Optimized**: Monotonic Stack for next greater element pattern → Time: O(n), Space: O(n)

---

## Note 🧾⚠️💡

- Using the index instead of storing the actual temperature in the stack helps easily compute the difference in days.
- Monotonic decreasing stack ensures that every temperature is compared with future warmer ones only once.
