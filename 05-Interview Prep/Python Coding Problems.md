# Python Coding Problems

> Data structures and algorithms practice following NeetCode roadmap

## Practice Platforms

| Platform | Focus | Link |
|----------|-------|------|
| **NeetCode** | Structed DSA roadmap | [neetcode.io](https://neetcode.io/) |
| **LeetCode** | Problem database | [leetcode.com](https://leetcode.com/) |
| **LeetCode 75** | Curated study plan | [LeetCode 75](https://leetcode.com/studyplan/leetcode-75/) |
| **LeetCode Top 150** | Comprehensive list | [Top Interview 150](https://leetcode.com/studyplan/top-interview-150/) |
| **Blind 75** | Essential problems | [Blind 75 List](https://neetcode.io/list?list=blind75) |

## YouTube Solution Channels

| Channel | Content | Link |
|---------|---------|------|
| **NeetCode** | Full roadmap solutions | [NeetCode](https://www.youtube.com/@NeetCode) |
| **NeetCode 150 Playlist** | All 150 problems | [NeetCode 150](https://www.youtube.com/playlist?list=PLot-Xpze53ldVwtstag2TL4HQhAnC8m) |
| **Tech With Tim** | Python DSA | [Tech With Tim](https://www.youtube.com/@TechWithTim) |
| **Kevin Naughton Jr.** | Interview prep | [Kevin Naughton Jr.](https://www.youtube.com/@KevinNaughtonJr) |

---

## Pattern 1: Arrays & Hashing

### Key Concepts

```python
# Common Hash Map Patterns
from collections import defaultdict, Counter

# Frequency counter
freq = Counter(nums)

# Default dictionary for grouping
groups = defaultdict(list)

# Set for O(1) lookup
seen = set()
```

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Contains Duplicate | [NC](https://neetcode.io/problems/duplicate-integer) | [LC 217](https://leetcode.com/problems/contains-duplicate/) | Easy |
| 2 | Valid Anagram | [NC](https://neetcode.io/problems/is-anagram) | [LC 242](https://leetcode.com/problems/valid-anagram/) | Easy |
| 3 | Two Sum | [NC](https://neetcode.io/problems/two-sum) | [LC 1](https://leetcode.com/problems/two-sum/) | Easy |
| 4 | Plus One | [NC](https://neetcode.io/problems/plus-one) | [LC 66](https://leetcode.com/problems/plus-one/) | Easy |
| 5 | Contains Duplicate II | - | [LC 219](https://leetcode.com/problems/contains-duplicate-ii/) | Easy |
| 6 | Majority Element | [NC](https://neetcode.io/problems/majority-element) | [LC 169](https://leetcode.com/problems/majority-element/) | Easy |
| 7 | Find Pivot Index | - | [LC 724](https://leetcode.com/problems/find-pivot-index/) | Easy |
| 8 | Intersection of Two Arrays | - | [LC 349](https://leetcode.com/problems/intersection-of-two-arrays/) | Easy |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Group Anagrams | [NC](https://neetcode.io/problems/group-anagrams) | [LC 49](https://leetcode.com/problems/group-anagrams/) | Medium |
| 2 | Top K Frequent Elements | [NC](https://neetcode.io/problems/top-k-elements-in-list) | [LC 347](https://leetcode.com/problems/top-k-frequent-elements/) | Medium |
| 3 | Product of Array Except Self | [NC](https://neetcode.io/problems/products-of-array-discluding-self) | [LC 238](https://leetcode.com/problems/product-of-array-except-self/) | Medium |
| 4 | Valid Sudoku | [NC](https://neetcode.io/problems/valid-sudoku) | [LC 36](https://leetcode.com/problems/valid-sudoku/) | Medium |
| 5 | Encode and Decode Strings | [NC](https://neetcode.io/problems/string-encode-and-decode) | [LC 271](https://leetcode.com/problems/encode-and-decode-strings/) | Medium |
| 6 | Longest Consecutive Sequence | [NC](https://neetcode.io/problems/longest-consecutive-sequence) | [LC 128](https://leetcode.com/problems/longest-consecutive-sequence/) | Medium |
| 7 | Sort Colors | - | [LC 75](https://leetcode.com/problems/sort-colors/) | Medium |
| 8 | Subarray Sum Equals K | - | [LC 560](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium |
| 9 | Continuous Subarray Sum | - | [LC 523](https://leetcode.com/problems/continuous-subarray-sum/) | Medium |

### Hard Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Valid Anagram (Follow-up) | - | [LC 242](https://leetcode.com/problems/valid-anagram/) | Hard |

### Sample Solutions

#### Contains Duplicate
```python
def containsDuplicate(nums: list[int]) -> bool:
    """O(n) time, O(n) space"""
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False

# Alternative: O(n log n) time, O(1) space
def containsDuplicate_sorted(nums: list[int]) -> bool:
    nums.sort()
    for i in range(1, len(nums)):
        if nums[i] == nums[i-1]:
            return True
    return False
```

#### Valid Anagram
```python
def isAnagram(s: str, t: str) -> bool:
    """O(n) time, O(1) space (26 letters)"""
    if len(s) != len(t):
        return False
    return Counter(s) == Counter(t)

# Array-based (faster in practice)
def isAnagram_array(s: str, t: str) -> bool:
    if len(s) != len(t):
        return False
    count = [0] * 26
    for i in range(len(s)):
        count[ord(s[i]) - ord('a')] += 1
        count[ord(t[i]) - ord('a')] -= 1
    return all(c == 0 for c in count)
```

#### Two Sum
```python
def twoSum(nums: list[int], target: int) -> list[int]:
    """O(n) time, O(n) space"""
    seen = {}  # value -> index
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i
    return []
```

#### Group Anagrams
```python
def groupAnagrams(strs: list[str]) -> list[list[str]]:
    """O(n * k log k) time, O(n * k) space"""
    groups = defaultdict(list)
    for s in strs:
        # Sort string as key
        key = ''.join(sorted(s))
        groups[key].append(s)
    return list(groups.values())

# Alternative: Character count as key (O(n * k) time)
def groupAnagrams_count(strs: list[str]) -> list[list[str]]:
    groups = defaultdict(list)
    for s in strs:
        count = [0] * 26
        for c in s:
            count[ord(c) - ord('a')] += 1
        groups[tuple(count)].append(s)
    return list(groups.values())
```

#### Top K Frequent Elements
```python
def topKFrequent(nums: list[int], k: int) -> list[int]:
    """O(n) time using bucket sort"""
    count = Counter(nums)
    # Bucket: index = frequency
    freq = [[] for _ in range(len(nums) + 1)]
    for num, cnt in count.items():
        freq[cnt].append(num)

    result = []
    for i in range(len(freq) - 1, 0, -1):
        for num in freq[i]:
            result.append(num)
            if len(result) == k:
                return result
    return result

# Using heap: O(n log k) time
import heapq
def topKFrequent_heap(nums: list[int], k: int) -> list[int]:
    count = Counter(nums)
    return [num for num, _ in heapq.nlargest(k, count.items(), key=lambda x: x[1])]
```

#### Product of Array Except Self
```python
def productExceptSelf(nums: list[int]) -> list[int]:
    """O(n) time, O(1) extra space (output array doesn't count)"""
    n = len(nums)
    result = [1] * n

    # Prefix products
    prefix = 1
    for i in range(n):
        result[i] = prefix
        prefix *= nums[i]

    # Suffix products
    suffix = 1
    for i in range(n - 1, -1, -1):
        result[i] *= suffix
        suffix *= nums[i]

    return result
```

#### Longest Consecutive Sequence
```python
def longestConsecutive(nums: list[int]) -> int:
    """O(n) time, O(n) space"""
    num_set = set(nums)
    longest = 0

    for num in num_set:
        # Only start counting if num is the start of a sequence
        if num - 1 not in num_set:
            current_num = num
            current_streak = 1

            while current_num + 1 in num_set:
                current_num += 1
                current_streak += 1

            longest = max(longest, current_streak)

    return longest
```

---

## Pattern 2: Two Pointers

### Key Concepts

```python
# Opposite direction (from both ends)
left, right = 0, len(arr) - 1
while left < right:
    # Process
    left += 1
    right -= 1

# Same direction (fast/slow)
slow = fast = 0
while fast < len(arr):
    # Process
    slow += 1
    fast += 2  # or some increment
```

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Valid Palindrome | [NC](https://neetcode.io/problems/is-palindrome) | [LC 125](https://leetcode.com/problems/valid-palindrome/) | Easy |
| 2 | Move Zeroes | [NC](https://neetcode.io/problems/move-zeroes) | [LC 283](https://leetcode.com/problems/move-zeroes/) | Easy |
| 3 | Merge Sorted Array | - | [LC 88](https://leetcode.com/problems/merge-sorted-array/) | Easy |
| 4 | Squares of Sorted Array | - | [LC 977](https://leetcode.com/problems/squares-of-a-sorted-array/) | Easy |
| 5 | Reverse String | - | [LC 344](https://leetcode.com/problems/reverse-string/) | Easy |
| 6 | Reverse Vowels of String | - | [LC 345](https://leetcode.com/problems/reverse-vowels-of-a-string/) | Easy |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Two Sum II - Input Sorted | [NC](https://neetcode.io/problems/two-sum-input-array-sorted) | [LC 167](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Medium |
| 2 | 3Sum | [NC](https://neetcode.io/problems/three-integer-sum) | [LC 15](https://leetcode.com/problems/3sum/) | Medium |
| 3 | Container With Most Water | [NC](https://neetcode.io/problems/max-area-water) | [LC 11](https://leetcode.com/problems/container-with-most-water/) | Medium |
| 4 | Remove Duplicates from Sorted Array II | - | [LC 80](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) | Medium |
| 5 | 3Sum Closest | - | [LC 16](https://leetcode.com/problems/3sum-closest/) | Medium |
| 6 | 4Sum | - | [LC 18](https://leetcode.com/problems/4sum/) | Medium |
| 7 | Sort Colors (Dutch National Flag) | - | [LC 75](https://leetcode.com/problems/sort-colors/) | Medium |
| 8 | Trapping Rain Water | [NC](https://neetcode.io/problems/trapping-rain-water) | [LC 42](https://leetcode.com/problems/trapping-rain-water/) | Hard |

### Sample Solutions

#### Valid Palindrome
```python
def isPalindrome(s: str) -> bool:
    """O(n) time, O(1) space"""
    left, right = 0, len(s) - 1

    while left < right:
        # Skip non-alphanumeric
        while left < right and not s[left].isalnum():
            left += 1
        while left < right and not s[right].isalnum():
            right -= 1

        if s[left].lower() != s[right].lower():
            return False

        left += 1
        right -= 1

    return True
```

#### Two Sum II (Sorted Input)
```python
def twoSum(numbers: list[int], target: int) -> list[int]:
    """O(n) time, O(1) space"""
    left, right = 0, len(numbers) - 1

    while left < right:
        current_sum = numbers[left] + numbers[right]
        if current_sum == target:
            return [left + 1, right + 1]  # 1-indexed
        elif current_sum < target:
            left += 1
        else:
            right -= 1

    return []
```

#### 3Sum
```python
def threeSum(nums: list[int]) -> list[list[int]]:
    """O(n^2) time, O(1) space (excluding output)"""
    nums.sort()
    result = []
    n = len(nums)

    for i in range(n - 2):
        # Skip duplicates
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        # Early termination
        if nums[i] > 0:
            break

        left, right = i + 1, n - 1
        while left < right:
            total = nums[i] + nums[left] + nums[right]

            if total == 0:
                result.append([nums[i], nums[left], nums[right]])
                # Skip duplicates
                while left < right and nums[left] == nums[left + 1]:
                    left += 1
                while left < right and nums[right] == nums[right - 1]:
                    right -= 1
                left += 1
                right -= 1
            elif total < 0:
                left += 1
            else:
                right -= 1

    return result
```

#### Container With Most Water
```python
def maxArea(height: list[int]) -> int:
    """O(n) time, O(1) space"""
    left, right = 0, len(height) - 1
    max_area = 0

    while left < right:
        # Calculate area
        h = min(height[left], height[right])
        w = right - left
        max_area = max(max_area, h * w)

        # Move pointer with smaller height
        if height[left] < height[right]:
            left += 1
        else:
            right -= 1

    return max_area
```

#### Trapping Rain Water
```python
def trap(height: list[int]) -> int:
    """Two pointer approach - O(n) time, O(1) space"""
    if not height:
        return 0

    left, right = 0, len(height) - 1
    left_max, right_max = height[left], height[right]
    water = 0

    while left < right:
        if left_max < right_max:
            left += 1
            left_max = max(left_max, height[left])
            water += left_max - height[left]
        else:
            right -= 1
            right_max = max(right_max, height[right])
            water += right_max - height[right]

    return water

# Alternative: Dynamic Programming - O(n) time, O(n) space
def trap_dp(height: list[int]) -> int:
    if not height:
        return 0

    n = len(height)
    left_max = [0] * n
    right_max = [0] * n

    left_max[0] = height[0]
    for i in range(1, n):
        left_max[i] = max(left_max[i-1], height[i])

    right_max[n-1] = height[n-1]
    for i in range(n-2, -1, -1):
        right_max[i] = max(right_max[i+1], height[i])

    water = 0
    for i in range(n):
        water += min(left_max[i], right_max[i]) - height[i]

    return water
```

#### Move Zeroes
```python
def moveZeroes(nums: list[int]) -> None:
    """O(n) time, O(1) space - modify in-place"""
    # Position to place next non-zero
    insert_pos = 0

    # Move all non-zeroes to front
    for num in nums:
        if num != 0:
            nums[insert_pos] = num
            insert_pos += 1

    # Fill remaining with zeroes
    while insert_pos < len(nums):
        nums[insert_pos] = 0
        insert_pos += 1

# Alternative: Swap approach
def moveZeroes_swap(nums: list[int]) -> None:
    snowball = 0  # Count of zeros
    for i in range(len(nums)):
        if nums[i] == 0:
            snowball += 1
        elif snowball > 0:
            nums[i - snowball] = nums[i]
            nums[i] = 0
```

---

## Pattern 3: Sliding Window

### Key Concepts

```python
# Fixed size window
def fixed_window(arr, k):
    window_sum = sum(arr[:k])
    max_sum = window_sum

    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i-k]  # Add new, remove old
        max_sum = max(max_sum, window_sum)
    return max_sum

# Variable size window
def variable_window(s, target):
    left = 0
    window_state = {}  # Track window contents

    for right in range(len(s)):
        # Expand window
        # Update window_state

        # While condition not met, shrink
        while condition_not_met:
            # Update window_state
            left += 1
```

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Best Time to Buy Sell Stock | [NC](https://neetcode.io/problems/best-time-to-buy-and-sell-stocks) | [LC 121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Easy |
| 2 | Find All Anagrams in String | - | [LC 438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Easy |
| 3 | Minimum Size Subarray Sum | - | [LC 209](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Longest Substring Without Repeating | [NC](https://neetcode.io/problems/longest-substring-without-duplicates) | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium |
| 2 | Longest Repeating Character Replacement | [NC](https://neetcode.io/problems/longest-repeating-substring-with-replacement) | [LC 424](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium |
| 3 | Permutation in String | [NC](https://neetcode.io/problems/permutation-string) | [LC 567](https://leetcode.com/problems/permutation-in-string/) | Medium |
| 4 | Max Consecutive Ones III | - | [LC 1004](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium |
| 5 | Fruit Into Baskets | [NC](https://neetcode.io/problems/fruits-into-baskets) | [LC 904](https://leetcode.com/problems/fruit-into-baskets/) | Medium |
| 6 | Subarray Product Less Than K | - | [LC 713](https://leetcode.com/problems/subarray-product-less-than-k/) | Medium |
| 7 | Minimum Window Substring | [NC](https://neetcode.io/problems/minimum-window-substring) | [LC 76](https://leetcode.com/problems/minimum-window-substring/) | Hard |

### Hard Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Minimum Window Substring | [NC](https://neetcode.io/problems/minimum-window-substring) | [LC 76](https://leetcode.com/problems/minimum-window-substring/) | Hard |
| 2 | Sliding Window Maximum | [NC](https://neetcode.io/problems/sliding-window-maximum) | [LC 239](https://leetcode.com/problems/sliding-window-maximum/) | Hard |
| 3 | Substring with Concatenation of All Words | - | [LC 30](https://leetcode.com/problems/substring-with-concatenation-of-all-words/) | Hard |

### Sample Solutions

#### Best Time to Buy and Sell Stock
```python
def maxProfit(prices: list[int]) -> int:
    """Track minimum price seen so far"""
    min_price = float('inf')
    max_profit = 0

    for price in prices:
        min_price = min(min_price, price)
        max_profit = max(max_profit, price - min_price)

    return max_profit
```

#### Longest Substring Without Repeating Characters
```python
def lengthOfLongestSubstring(s: str) -> int:
    """O(n) time, O(min(m,n)) space where m is charset size"""
    char_index = {}  # Character -> last index
    left = 0
    max_length = 0

    for right, char in enumerate(s):
        # If char seen and is within current window
        if char in char_index and char_index[char] >= left:
            left = char_index[char] + 1

        char_index[char] = right
        max_length = max(max_length, right - left + 1)

    return max_length
```

#### Longest Repeating Character Replacement
```python
def characterReplacement(s: str, k: int) -> int:
    """O(n) time, O(1) space (26 letters)"""
    count = {}
    left = 0
    max_count = 0
    max_length = 0

    for right in range(len(s)):
        count[s[right]] = count.get(s[right], 0) + 1
        max_count = max(max_count, count[s[right]])

        # If window size - max_count > k, shrink
        while (right - left + 1) - max_count > k:
            count[s[left]] -= 1
            left += 1

        max_length = max(max_length, right - left + 1)

    return max_length
```

#### Permutation in String
```python
def checkInclusion(s1: str, s2: str) -> bool:
    """Check if s2 contains any permutation of s1"""
    if len(s1) > len(s2):
        return False

    s1_count = [0] * 26
    s2_count = [0] * 26

    for i in range(len(s1)):
        s1_count[ord(s1[i]) - ord('a')] += 1
        s2_count[ord(s2[i]) - ord('a')] += 1

    matches = sum(1 for i in range(26) if s1_count[i] == s2_count[i])

    left = 0
    for right in range(len(s1), len(s2)):
        if matches == 26:
            return True

        index = ord(s2[right]) - ord('a')
        s2_count[index] += 1
        if s2_count[index] == s1_count[index]:
            matches += 1
        elif s2_count[index] == s1_count[index] + 1:
            matches -= 1

        index = ord(s2[left]) - ord('a')
        s2_count[index] -= 1
        if s2_count[index] == s1_count[index]:
            matches += 1
        elif s2_count[index] == s1_count[index] - 1:
            matches -= 1

        left += 1

    return matches == 26
```

#### Minimum Window Substring
```python
def minWindow(s: str, t: str) -> str:
    """O(|s| + |t|) time"""
    if not s or not t or len(s) < len(t):
        return ""

    count_t = Counter(t)
    required = len(count_t)

    left = 0
    formed = 0
    window_counts = {}

    # (window length, left, right)
    result = (float('inf'), 0, 0)

    for right, char in enumerate(s):
        window_counts[char] = window_counts.get(char, 0) + 1

        if char in count_t and window_counts[char] == count_t[char]:
            formed += 1

        # Try to contract window
        while left <= right and formed == required:
            char = s[left]

            # Save smallest window
            if right - left + 1 < result[0]:
                result = (right - left + 1, left, right)

            window_counts[char] -= 1
            if char in count_t and window_counts[char] < count_t[char]:
                formed -= 1

            left += 1

    return "" if result[0] == float('inf') else s[result[1]:result[2]+1]
```

---

## Pattern 4: Stack

### Key Concepts

```python
# Monotonic Stack (increasing)
stack = []
for num in nums:
    while stack and stack[-1] > num:
        stack.pop()
    stack.append(num)

# Monotonic Stack (decreasing)
stack = []
for num in nums:
    while stack and stack[-1] < num:
        # Process pop
        stack.pop()
    stack.append(num)

# Common patterns
# - Next Greater Element
# - Previous Smaller Element
# - Valid Parentheses
# - Evaluate Expression
```

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Valid Parentheses | [NC](https://neetcode.io/problems/validate-parentheses) | [LC 20](https://leetcode.com/problems/valid-parentheses/) | Easy |
| 2 | Baseball Game | - | [LC 682](https://leetcode.com/problems/baseball-game/) | Easy |
| 3 | Backspace String Compare | - | [LC 844](https://leetcode.com/problems/backspace-string-compare/) | Easy |
| 4 | Implement Stack using Queues | - | [LC 225](https://leetcode.com/problems/implement-stack-using-queues/) | Easy |
| 5 | Min Stack | [NC](https://neetcode.io/problems/minimum-stack) | [LC 155](https://leetcode.com/problems/min-stack/) | Medium |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Evaluate Reverse Polish Notation | [NC](https://neetcode.io/problems/evaluate-reverse-polish-notation) | [LC 150](https://leetcode.com/problems/evaluate-reverse-polish-notation/) | Medium |
| 2 | Daily Temperatures | [NC](https://neetcode.io/problems/daily-temperatures) | [LC 739](https://leetcode.com/problems/daily-temperatures/) | Medium |
| 3 | Car Fleet | [NC](https://neetcode.io/problems/car-fleet) | [LC 853](https://leetcode.com/problems/car-fleet/) | Medium |
| 4 | Next Greater Element I | - | [LC 496](https://leetcode.com/problems/next-greater-element-i/) | Easy |
| 5 | Next Greater Element II | - | [LC 503](https://leetcode.com/problems/next-greater-element-ii/) | Medium |
| 6 | Asteroid Collision | - | [LC 735](https://leetcode.com/problems/asteroid-collision/) | Medium |
| 7 | Online Stock Span | - | [LC 901](https://leetcode.com/problems/online-stock-span/) | Medium |

### Hard Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Largest Rectangle in Histogram | [NC](https://neetcode.io/problems/largest-rectangle-in-histogram) | [LC 84](https://leetcode.com/problems/largest-rectangle-in-histogram/) | Hard |
| 2 | Trapping Rain Water (Stack approach) | [NC](https://neetcode.io/problems/trapping-rain-water) | [LC 42](https://leetcode.com/problems/trapping-rain-water/) | Hard |
| 3 | Basic Calculator | - | [LC 224](https://leetcode.com/problems/basic-calculator/) | Hard |
| 4 | Basic Calculator II | - | [LC 227](https://leetcode.com/problems/basic-calculator-ii/) | Medium |
| 5 | Maximal Rectangle | - | [LC 85](https://leetcode.com/problems/maximal-rectangle/) | Hard |

### Sample Solutions

#### Valid Parentheses
```python
def isValid(s: str) -> bool:
    """O(n) time, O(n) space"""
    mapping = {')': '(', '}': '{', ']': '['}
    stack = []

    for char in s:
        if char in mapping:
            if not stack or stack.pop() != mapping[char]:
                return False
        else:
            stack.append(char)

    return not stack
```

#### Min Stack
```python
class MinStack:
    """O(1) for all operations"""
    def __init__(self):
        self.stack = []
        self.min_stack = []  # Tracks minimum at each state

    def push(self, val: int) -> None:
        self.stack.append(val)
        if not self.min_stack or val <= self.min_stack[-1]:
            self.min_stack.append(val)

    def pop(self) -> None:
        if self.stack.pop() == self.min_stack[-1]:
            self.min_stack.pop()

    def top(self) -> int:
        return self.stack[-1]

    def getMin(self) -> int:
        return self.min_stack[-1]
```

#### Evaluate Reverse Polish Notation
```python
def evalRPN(tokens: list[str]) -> int:
    """O(n) time, O(n) space"""
    stack = []

    for token in tokens:
        if token in '+-*/':
            b = stack.pop()
            a = stack.pop()
            if token == '+':
                stack.append(a + b)
            elif token == '-':
                stack.append(a - b)
            elif token == '*':
                stack.append(a * b)
            else:  # Division truncates toward zero
                stack.append(int(a / b))
        else:
            stack.append(int(token))

    return stack[0]
```

#### Daily Temperatures
```python
def dailyTemperatures(temperatures: list[int]) -> list[int]:
    """Monotonic decreasing stack - O(n) time"""
    n = len(temperatures)
    result = [0] * n
    stack = []  # Store indices

    for i, temp in enumerate(temperatures):
        # Pop all cooler days
        while stack and temperatures[stack[-1]] < temp:
            prev_index = stack.pop()
            result[prev_index] = i - prev_index
        stack.append(i)

    return result
```

#### Car Fleet
```python
def carFleet(target: int, position: list[int], speed: list[int]) -> int:
    """O(n log n) time for sorting"""
    # Sort by position (descending)
    cars = sorted(zip(position, speed), reverse=True)
    stack = []

    for pos, spd in cars:
        time = (target - pos) / spd

        # If this car takes longer than car ahead, it joins that fleet
        if not stack or time > stack[-1]:
            stack.append(time)

    return len(stack)
```

#### Largest Rectangle in Histogram
```python
def largestRectangleArea(heights: list[int]) -> int:
    """Monotonic stack - O(n) time"""
    stack = []  # Store indices
    max_area = 0

    for i, h in enumerate(heights + [0]):  # Add sentinel
        while stack and heights[stack[-1]] > h:
            height = heights[stack.pop()]
            width = i if not stack else i - stack[-1] - 1
            max_area = max(max_area, height * width)
        stack.append(i)

    return max_area
```

---

## Pattern 5: Binary Search

### Key Concepts

```python
# Standard binary search
def binary_search(arr, target):
    left, right = 0, len(arr) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1

# Find leftmost position (lower bound)
def lower_bound(arr, target):
    left, right = 0, len(arr)

    while left < right:
        mid = left + (right - left) // 2
        if arr[mid] < target:
            left = mid + 1
        else:
            right = mid

    return left

# Find rightmost position (upper bound)
def upper_bound(arr, target):
    left, right = 0, len(arr)

    while left < right:
        mid = left + (right - left) // 2
        if arr[mid] <= target:
            left = mid + 1
        else:
            right = mid

    return left
```

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Binary Search | [NC](https://neetcode.io/problems/binary-search) | [LC 704](https://leetcode.com/problems/binary-search/) | Easy |
| 2 | Search Insert Position | - | [LC 35](https://leetcode.com/problems/search-insert-position/) | Easy |
| 3 | First Bad Version | - | [LC 278](https://leetcode.com/problems/first-bad-version/) | Easy |
| 4 | Sqrt(x) | - | [LC 69](https://leetcode.com/problems/sqrtx/) | Easy |
| 5 | Guess Number Higher or Lower | - | [LC 374](https://leetcode.com/problems/guess-number-higher-or-lower/) | Easy |
| 6 | Arranging Coins | - | [LC 441](https://leetcode.com/problems/arranging-coins/) | Easy |
| 7 | Ceiling of a Number | - | [Coding Pattern] | Easy |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Search in Rotated Sorted Array | [NC](https://neetcode.io/problems/search-in-rotated-sorted-array) | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium |
| 2 | Find Minimum in Rotated Sorted Array | [NC](https://neetcode.io/problems/find-minimum-in-rotated-sorted-array) | [LC 153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium |
| 3 | Search a 2D Matrix | [NC](https://neetcode.io/problems/search-2d-matrix) | [LC 74](https://leetcode.com/problems/search-a-2d-matrix/) | Medium |
| 4 | Search in Rotated Sorted Array II | - | [LC 81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) | Medium |
| 5 | Find Peak Element | - | [LC 162](https://leetcode.com/problems/find-peak-element/) | Medium |
| 6 | Find First and Last Position | - | [LC 34](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium |
| 7 | Single Element in Sorted Array | - | [LC 540](https://leetcode.com/problems/single-element-in-a-sorted-array/) | Medium |
| 8 | Time Based Key-Value Store | [NC](https://neetcode.io/problems/time-based-key-value-store) | [LC 981](https://leetcode.com/problems/time-based-key-value-store/) | Medium |
| 9 | Koko Eating Bananas | [NC](https://neetcode.io/problems/koko-eating-bananas) | [LC 875](https://leetcode.com/problems/koko-eating-bananas/) | Medium |
| 10 | Search Suggestions System | - | [LC 1268](https://leetcode.com/problems/search-suggestions-system/) | Medium |

### Hard Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Median of Two Sorted Arrays | [NC](https://neetcode.io/problems/median-of-two-sorted-arrays) | [LC 4](https://leetcode.com/problems/median-of-two-sorted-arrays/) | Hard |
| 2 | Find Minimum in Rotated Sorted Array II | - | [LC 154](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) | Hard |
| 3 | Split Array Largest Sum | - | [LC 410](https://leetcode.com/problems/split-array-largest-sum/) | Hard |
| 4 | Dungeon Game | - | [LC 174](https://leetcode.com/problems/dungeon-game/) | Hard |

### Sample Solutions

#### Binary Search
```python
def search(nums: list[int], target: int) -> int:
    """Standard binary search - O(log n) time"""
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2  # Prevents overflow

        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1
        else:
            right = mid - 1

    return -1
```

#### Search in Rotated Sorted Array
```python
def search_rotated(nums: list[int], target: int) -> int:
    """O(log n) time"""
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2

        if nums[mid] == target:
            return mid

        # Left half is sorted
        if nums[left] <= nums[mid]:
            if nums[left] <= target < nums[mid]:
                right = mid - 1
            else:
                left = mid + 1
        # Right half is sorted
        else:
            if nums[mid] < target <= nums[right]:
                left = mid + 1
            else:
                right = mid - 1

    return -1
```

#### Find Minimum in Rotated Sorted Array
```python
def findMin(nums: list[int]) -> int:
    """O(log n) time"""
    left, right = 0, len(nums) - 1

    while left < right:
        mid = left + (right - left) // 2

        if nums[mid] > nums[right]:
            # Minimum is in right half
            left = mid + 1
        else:
            # Minimum is in left half (including mid)
            right = mid

    return nums[left]
```

#### Find First and Last Position
```python
def searchRange(nums: list[int], target: int) -> list[int]:
    """O(log n) time"""

    def find_left():
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] < target:
                left = mid + 1
            else:
                right = mid - 1
        return left

    def find_right():
        left, right = 0, len(nums) - 1
        while left <= right:
            mid = left + (right - left) // 2
            if nums[mid] <= target:
                left = mid + 1
            else:
                right = mid - 1
        return right

    left_idx = find_left()
    right_idx = find_right()

    if left_idx <= right_idx < len(nums) and nums[left_idx] == target:
        return [left_idx, right_idx]
    return [-1, -1]
```

#### Koko Eating Bananas (Binary Search on Answer)
```python
import math

def minEatingSpeed(piles: list[int], h: int) -> int:
    """Binary search on answer space"""
    left, right = 1, max(piles)

    def can_finish(k):
        return sum(math.ceil(pile / k) for pile in piles) <= h

    while left < right:
        mid = left + (right - left) // 2

        if can_finish(mid):
            right = mid
        else:
            left = mid + 1

    return left
```

#### Search a 2D Matrix
```python
def searchMatrix(matrix: list[list[int]], target: int) -> bool:
    """O(log(m*n)) time - treat as 1D sorted array"""
    if not matrix or not matrix[0]:
        return False

    rows, cols = len(matrix), len(matrix[0])
    left, right = 0, rows * cols - 1

    while left <= right:
        mid = left + (right - left) // 2
        mid_val = matrix[mid // cols][mid % cols]

        if mid_val == target:
            return True
        elif mid_val < target:
            left = mid + 1
        else:
            right = mid - 1

    return False
```

#### Time Based Key-Value Store
```python
import bisect

class TimeMap:
    def __init__(self):
        self.store = {}  # key -> list of (timestamp, value)

    def set(self, key: str, value: str, timestamp: int) -> None:
        if key not in self.store:
            self.store[key] = []
        self.store[key].append((timestamp, value))

    def get(self, key: str, timestamp: int) -> str:
        if key not in self.store:
            return ""

        entries = self.store[key]

        # Binary search for largest timestamp <= target
        left, right = 0, len(entries) - 1
        while left <= right:
            mid = left + (right - left) // 2
            if entries[mid][0] <= timestamp:
                left = mid + 1
            else:
                right = mid - 1

        return entries[right][1] if right >= 0 else ""
```

---

## Pattern 6: Linked List (Bonus)

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Reverse Linked List | [NC](https://neetcode.io/problems/reverse-linked-list) | [LC 206](https://leetcode.com/problems/reverse-linked-list/) | Easy |
| 2 | Merge Two Sorted Lists | [NC](https://neetcode.io/problems/merge-two-sorted-linked-lists) | [LC 21](https://leetcode.com/problems/merge-two-sorted-lists/) | Easy |
| 3 | Linked List Cycle | [NC](https://neetcode.io/problems/linked-list-cycle-detection) | [LC 141](https://leetcode.com/problems/linked-list-cycle/) | Easy |
| 4 | Remove Duplicates from Sorted List | - | [LC 83](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) | Easy |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Reorder List | [NC](https://neetcode.io/problems/reorder-linked-list) | [LC 143](https://leetcode.com/problems/reorder-list/) | Medium |
| 2 | Remove Nth Node From End | [NC](https://neetcode.io/problems/remove-node-from-end-of-linked-list) | [LC 19](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) | Medium |
| 3 | Copy List with Random Pointer | [NC](https://neetcode.io/problems/copy-linked-list-with-random-pointer) | [LC 138](https://leetcode.com/problems/copy-list-with-random-pointer/) | Medium |
| 4 | Add Two Numbers | [NC](https://neetcode.io/problems/add-two-numbers) | [LC 2](https://leetcode.com/problems/add-two-numbers/) | Medium |
| 5 | Linked List Cycle II | [NC](https://neetcode.io/problems/linked-list-cycle) | [LC 142](https://leetcode.com/problems/linked-list-cycle-ii/) | Medium |

---

## Pattern 7: Trees (Bonus)

### Easy Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Invert Binary Tree | [NC](https://neetcode.io/problems/invert-a-binary-tree) | [LC 226](https://leetcode.com/problems/invert-binary-tree/) | Easy |
| 2 | Maximum Depth of Binary Tree | [NC](https://neetcode.io/problems/depth-of-binary-tree) | [LC 104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Easy |
| 3 | Same Tree | [NC](https://neetcode.io/problems/same-binary-tree) | [LC 100](https://leetcode.com/problems/same-tree/) | Easy |
| 4 | Subtree of Another Tree | - | [LC 572](https://leetcode.com/problems/subtree-of-another-tree/) | Easy |
| 5 | Balanced Binary Tree | - | [LC 110](https://leetcode.com/problems/balanced-binary-tree/) | Easy |

### Medium Problems

| # | Problem | NeetCode | LeetCode | Difficulty |
|---|---------|----------|----------|------------|
| 1 | Binary Tree Level Order Traversal | [NC](https://neetcode.io/problems/level-order-traversal-of-binary-tree) | [LC 102](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Medium |
| 2 | Validate Binary Search Tree | [NC](https://neetcode.io/problems/validate-bst) | [LC 98](https://leetcode.com/problems/validate-binary-search-tree/) | Medium |
| 3 | Lowest Common Ancestor of BST | [NC](https://neetcode.io/problems/lowest-common-ancestor-in-binary-search-tree) | [LC 235](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | Medium |
| 4 | Construct Binary Tree from Preorder and Inorder | - | [LC 105](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | Medium |
| 5 | Binary Tree Right Side View | - | [LC 199](https://leetcode.com/problems/binary-tree-right-side-view/) | Medium |
| 6 | Count Good Nodes in Binary Tree | - | [LC 1448](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) | Medium |

---

## Quick Reference: Time Complexities

| Pattern | Typical Time | Typical Space |
|---------|--------------|---------------|
| Array & Hashing | O(n) | O(n) |
| Two Pointers | O(n) | O(1) |
| Sliding Window | O(n) | O(k) or O(1) |
| Stack | O(n) | O(n) |
| Binary Search | O(log n) | O(1) |
| Linked List | O(n) | O(1) |
| Trees (DFS/BFS) | O(n) | O(h) or O(w) |

## Study Order Recommendation

1. **Week 1-2**: Arrays & Hashing (foundation for all patterns)
2. **Week 3**: Two Pointers
3. **Week 4**: Sliding Window
4. **Week 5**: Stack
5. **Week 6**: Binary Search

Each week:
- Learn 2-3 problems per day
- Understand the pattern, not just the solution
- Practice variations of each problem
- Time yourself after understanding the pattern

---

## Related Topics

- [[05-Interview Prep/SQL Coding Problems]]
- [[05-Interview Prep/System Design Framework]]
- [[02-Areas/Python/Python Basics]]
