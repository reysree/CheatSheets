
# Valid Palindrome

## Description of Problem ✨📚📝

A phrase is a palindrome if, after converting all uppercase letters into lowercase letters and removing all non-alphanumeric characters, it reads the same forward and backward. Alphanumeric characters include letters and numbers. ✨📚📝

Given a string `s`, return `true` if it is a palindrome, or `false` otherwise. ✨📚📝

**LeetCode Link**: [https://leetcode.com/problems/valid-palindrome/](https://leetcode.com/problems/valid-palindrome/)

---

## Constraints 📏🔢📊

- 1 <= s.length <= 2 * 10^5
- `s` consists only of printable ASCII characters.

---

## Algorithm/Data Structure to Use ⚙️🧠🔍

- **Two Pointer**

---

## Solution 💡🧩🚀

### Brute Force Approach 💭🛠️🧱

1. The brute force way is to create a reverse of the string and compare using `string1.equals(string2)`.
2. This solution takes **O(n)** time and **O(n)** space because of the additional string created.

### Optimized Approach (Using Two Pointer) ⚡🏹🔗

1. First, filter out all non-alphanumeric characters from the input string and convert everything to lowercase.
2. A palindrome reads the same from both directions. So, an optimal way to check is using the **Two Pointer** technique. Essentially, if we divide the string in the middle, both halves should mirror each other.
3. For the filtered string, initialize two pointers:
   - `start` index at the beginning of the string.
   - `end` index at the end of the string.
4. Compare characters at `start` and `end` positions while `start < end`:
   - If characters don't match, return `false`.
   - If they match, move `start` forward and `end` backward.
5. If the loop completes without returning `false`, return `true` as it is a palindrome.

---

## Complexity Analysis 📈⏱️🧮

- **Time Complexity**: O(n), where `n` is the length of the input string. We traverse the string once to filter and once to check palindrome.
- **Space Complexity**: O(n) for the filtered string.

---

## Example 🔍📝🎯

### Example 1:

**Input:**\
s = "A man, a plan, a canal: Panama"

**Output:**\
true

**Explanation:**\
"amanaplanacanalpanama" is a palindrome.

### Example 2:

**Input:**\
s = "race a car"

**Output:**\
false

**Explanation:**\
"raceacar" is not a palindrome.

### Example 3:

**Input:**\
s = " "

**Output:**\
true

**Explanation:**\
After removing non-alphanumeric characters, it's an empty string, which is considered a palindrome.

---

## Summary 📝✅📚

- **Brute Force**: Reverse and compare. Time: O(n), Space: O(n).
- **Optimized**: Two Pointer check. Time: O(n), Space: O(n).

---

## Note 🧠⚠️💡

- Ensure to remove non-alphanumeric characters and convert to lowercase before checking.
- Handle edge cases like empty string or string with only non-alphanumeric characters gracefully.
