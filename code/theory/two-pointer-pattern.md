# Two Pointer DSA Pattern - Comprehensive Guide

## Table of Contents
1. [Introduction to Two Pointer Pattern](#introduction)
2. [When to Use Two Pointer Pattern](#when-to-use)
3. [Types of Two Pointer Techniques](#types)
4. [Time and Space Complexity](#complexity)
5. [10 Practical Problems with Solutions](#problems)
6. [Advanced Patterns and Variations](#advanced)
7. [Common Mistakes and Tips](#tips)

---

## Introduction to Two Pointer Pattern

The **Two Pointer** pattern is a powerful algorithmic technique that uses two pointers to traverse a data structure (typically arrays or strings) to solve problems efficiently. Instead of using nested loops (O(n²) complexity), this pattern often reduces time complexity to O(n) by intelligently moving two pointers based on certain conditions.

### Core Concept
```java
// Basic Two Pointer Structure
int left = 0;                    // Start pointer
int right = array.length - 1;   // End pointer

while (left < right) {
    // Process current window/pair
    if (someCondition) {
        left++;              // Move left pointer right
    } else {
        right--;             // Move right pointer left
    }
}
```

### Key Characteristics
- **Efficiency**: Reduces O(n²) brute force to O(n) linear time
- **Space Optimal**: Usually O(1) space complexity
- **Versatile**: Works on arrays, strings, linked lists
- **Intuitive**: Easy to understand and implement

---

## When to Use Two Pointer Pattern

### Primary Use Cases

#### 1. **Sorted Arrays**
- Finding pairs with specific sum
- Removing duplicates
- Merging sorted arrays

#### 2. **Palindrome Problems**
- Checking if string/array is palindrome
- Finding longest palindromic substring
- Validating palindrome after modifications

#### 3. **Subarray/Substring Problems**
- Finding subarrays with specific sum
- Sliding window maximum/minimum
- String matching patterns

#### 4. **Linked List Problems**
- Detecting cycles
- Finding middle element
- Removing nth node from end

#### 5. **Array Manipulation**
- Partitioning arrays
- Moving elements to specific positions
- In-place array modifications

### Problem Identification Signals
✅ **Use Two Pointer when you see:**
- "Find pair/triplet with sum X"
- "Sorted array" mentioned
- "Palindrome" in problem statement
- "Sliding window" or "subarray"
- "In-place" or "O(1) space" requirement
- "Remove duplicates"
- "Reverse" operations

❌ **Don't use Two Pointer when:**
- Unsorted data without sorting option
- Need to find all possible combinations
- Complex state tracking required
- Multiple independent conditions

---

## Types of Two Pointer Techniques

### 1. **Opposite Direction (Converging)**
Pointers start from opposite ends and move toward each other.

```java
// Example: Finding pair with target sum in sorted array
int left = 0, right = arr.length - 1;
while (left < right) {
    int sum = arr[left] + arr[right];
    if (sum == target) return new int[]{left, right};
    else if (sum < target) left++;
    else right--;
}
```

**Use Cases:**
- Pair sum problems
- Palindrome validation
- Container with most water
- Valid parentheses

### 2. **Same Direction (Fast & Slow)**
Both pointers start from same position, one moves faster.

```java
// Example: Floyd's Cycle Detection
ListNode slow = head, fast = head;
while (fast != null && fast.next != null) {
    slow = slow.next;        // Move 1 step
    fast = fast.next.next;   // Move 2 steps
    if (slow == fast) return true; // Cycle detected
}
```

**Use Cases:**
- Cycle detection in linked lists
- Finding middle of linked list
- Removing nth node from end
- Sliding window problems

### 3. **Sliding Window**
Two pointers maintain a window that expands/contracts.

```java
// Example: Maximum sum subarray of size k
int windowSum = 0, maxSum = 0;
int left = 0;
for (int right = 0; right < arr.length; right++) {
    windowSum += arr[right];
    if (right - left + 1 == k) {
        maxSum = Math.max(maxSum, windowSum);
        windowSum -= arr[left++];
    }
}
```

**Use Cases:**
- Fixed/variable size window problems
- Substring with specific properties
- Maximum/minimum in sliding window

---

## Time and Space Complexity

### Time Complexity Analysis

| Pattern Type | Typical Complexity | Explanation |
|--------------|-------------------|-------------|
| **Opposite Direction** | O(n) | Each element visited at most once |
| **Fast & Slow** | O(n) | Linear traversal with different speeds |
| **Sliding Window** | O(n) | Each element added and removed once |
| **Brute Force Alternative** | O(n²) | Nested loops for same problems |

### Space Complexity
- **Typical**: O(1) - Only using pointer variables
- **With Auxiliary Data**: O(k) - Where k is window size or additional storage
- **Recursive Variants**: O(h) - Where h is recursion depth

---

## 10 Practical Problems with Solutions

### Problem 1: Two Sum (Sorted Array)

**Problem Statement:**
Given a sorted array of integers and a target sum, find two numbers that add up to the target.

**Approach:** Use opposite direction two pointers to find the pair efficiently.

```java
/**
 * Two Sum in Sorted Array
 * Time Complexity: O(n) - Single pass through array
 * Space Complexity: O(1) - Only using two pointer variables
 */
public class TwoSum {
    
    /**
     * Finds two numbers in sorted array that add up to target
     * @param numbers Sorted array of integers
     * @param target Target sum to find
     * @return Array containing indices of the two numbers (1-based)
     */
    public int[] twoSum(int[] numbers, int target) {
        // Initialize pointers at opposite ends
        int left = 0;                           // Start from beginning
        int right = numbers.length - 1;        // Start from end
        
        // Continue until pointers meet
        while (left < right) {
            int currentSum = numbers[left] + numbers[right];
            
            if (currentSum == target) {
                // Found the pair - return 1-based indices
                return new int[]{left + 1, right + 1};
            } else if (currentSum < target) {
                // Sum is too small, need larger number
                left++;                         // Move left pointer right
            } else {
                // Sum is too large, need smaller number  
                right--;                        // Move right pointer left
            }
        }
        
        // No solution found (should not happen if solution exists)
        return new int[]{-1, -1};
    }
    
    /**
     * Alternative solution that returns actual values instead of indices
     */
    public int[] twoSumValues(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;
        
        while (left < right) {
            int sum = numbers[left] + numbers[right];
            
            if (sum == target) {
                return new int[]{numbers[left], numbers[right]};
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }
        
        return new int[]{};  // No solution found
    }
    
    // Test method
    public static void main(String[] args) {
        TwoSum solution = new TwoSum();
        
        // Test case 1: Basic case
        int[] nums1 = {2, 7, 11, 15};
        int target1 = 9;
        int[] result1 = solution.twoSum(nums1, target1);
        System.out.println("Input: " + Arrays.toString(nums1) + ", Target: " + target1);
        System.out.println("Output: " + Arrays.toString(result1)); // [1, 2]
        
        // Test case 2: Multiple solutions possible
        int[] nums2 = {2, 3, 4, 6, 8, 9};
        int target2 = 10;
        int[] result2 = solution.twoSum(nums2, target2);
        System.out.println("Input: " + Arrays.toString(nums2) + ", Target: " + target2);
        System.out.println("Output: " + Arrays.toString(result2)); // [2, 6] or [4, 6]
    }
}
```

**Key Insights:**
- **Why it works**: Sorted array allows us to make intelligent decisions about pointer movement
- **Pointer Movement Logic**: If sum is small, increase left; if sum is large, decrease right
- **Uniqueness**: Each element is visited at most once, ensuring O(n) complexity

---

### Problem 2: Valid Palindrome

**Problem Statement:**
Given a string, determine if it's a valid palindrome considering only alphanumeric characters and ignoring case.

**Approach:** Use two pointers from opposite ends, skip non-alphanumeric characters.

```java
/**
 * Valid Palindrome Checker
 * Time Complexity: O(n) - Single pass through string
 * Space Complexity: O(1) - Only using pointer variables
 */
public class ValidPalindrome {
    
    /**
     * Checks if a string is a valid palindrome
     * @param s Input string to check
     * @return true if string is palindrome, false otherwise
     */
    public boolean isPalindrome(String s) {
        // Handle edge cases
        if (s == null || s.length() <= 1) {
            return true;
        }
        
        // Convert to lowercase for case-insensitive comparison
        s = s.toLowerCase();
        
        // Initialize pointers at opposite ends
        int left = 0;                       // Start from beginning
        int right = s.length() - 1;        // Start from end
        
        while (left < right) {
            // Skip non-alphanumeric characters from left
            while (left < right && !isAlphanumeric(s.charAt(left))) {
                left++;
            }
            
            // Skip non-alphanumeric characters from right
            while (left < right && !isAlphanumeric(s.charAt(right))) {
                right--;
            }
            
            // Compare characters at current positions
            if (s.charAt(left) != s.charAt(right)) {
                return false;               // Characters don't match
            }
            
            // Move pointers toward center
            left++;
            right--;
        }
        
        return true;                        // All characters matched
    }
    
    /**
     * Helper method to check if character is alphanumeric
     * @param c Character to check
     * @return true if character is letter or digit
     */
    private boolean isAlphanumeric(char c) {
        return (c >= 'a' && c <= 'z') || 
               (c >= 'A' && c <= 'Z') || 
               (c >= '0' && c <= '9');
    }
    
    /**
     * Alternative implementation using built-in Character methods
     */
    public boolean isPalindromeBuiltIn(String s) {
        int left = 0, right = s.length() - 1;
        
        while (left < right) {
            // Skip non-alphanumeric from left
            while (left < right && !Character.isLetterOrDigit(s.charAt(left))) {
                left++;
            }
            
            // Skip non-alphanumeric from right
            while (left < right && !Character.isLetterOrDigit(s.charAt(right))) {
                right--;
            }
            
            // Compare ignoring case
            if (Character.toLowerCase(s.charAt(left)) != 
                Character.toLowerCase(s.charAt(right))) {
                return false;
            }
            
            left++;
            right--;
        }
        
        return true;
    }
    
    // Test method
    public static void main(String[] args) {
        ValidPalindrome solution = new ValidPalindrome();
        
        // Test cases
        String[] testCases = {
            "A man, a plan, a canal: Panama",  // true
            "race a car",                      // false
            " ",                               // true
            "a",                               // true
            "Madam",                           // true
            "No 'x' in Nixon"                  // true
        };
        
        for (String test : testCases) {
            boolean result = solution.isPalindrome(test);
            System.out.println("\"" + test + "\" -> " + result);
        }
    }
}
```

**Key Insights:**
- **Character Skipping**: Essential to handle non-alphanumeric characters efficiently
- **Case Handling**: Convert to lowercase or use case-insensitive comparison
- **Edge Cases**: Empty strings and single characters are palindromes

---

### Problem 3: Container With Most Water

**Problem Statement:**
Given an array of non-negative integers representing heights, find two lines that form a container holding the most water.

**Approach:** Use two pointers to explore all possible containers efficiently.

```java
/**
 * Container With Most Water
 * Time Complexity: O(n) - Single pass through array
 * Space Complexity: O(1) - Only using variables for calculation
 */
public class ContainerWithMostWater {
    
    /**
     * Finds the maximum area of water that can be contained
     * @param height Array of heights representing vertical lines
     * @return Maximum water area possible
     */
    public int maxArea(int[] height) {
        // Handle edge cases
        if (height == null || height.length < 2) {
            return 0;
        }
        
        // Initialize pointers at opposite ends
        int left = 0;                           // Left boundary
        int right = height.length - 1;         // Right boundary
        int maxWater = 0;                       // Track maximum area found
        
        while (left < right) {
            // Calculate current area
            // Area = width × min(left_height, right_height)
            int width = right - left;           // Distance between lines
            int currentHeight = Math.min(height[left], height[right]);
            int currentArea = width * currentHeight;
            
            // Update maximum if current area is larger
            maxWater = Math.max(maxWater, currentArea);
            
            // Move the pointer with smaller height
            // Why? Because keeping the shorter line won't give us better area
            if (height[left] < height[right]) {
                left++;                         // Move left pointer right
            } else {
                right--;                        // Move right pointer left
            }
        }
        
        return maxWater;
    }
    
    /**
     * Alternative implementation with detailed step tracking
     */
    public int maxAreaWithSteps(int[] height) {
        int left = 0, right = height.length - 1;
        int maxWater = 0;
        
        System.out.println("Step-by-step calculation:");
        int step = 1;
        
        while (left < right) {
            int width = right - left;
            int minHeight = Math.min(height[left], height[right]);
            int area = width * minHeight;
            
            System.out.printf("Step %d: left=%d(h=%d), right=%d(h=%d), width=%d, area=%d%n",
                step++, left, height[left], right, height[right], width, area);
            
            maxWater = Math.max(maxWater, area);
            
            // Move pointer with smaller height
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }
        
        return maxWater;
    }
    
    /**
     * Brute force solution for comparison (O(n²))
     */
    public int maxAreaBruteForce(int[] height) {
        int maxWater = 0;
        
        // Try all possible pairs
        for (int i = 0; i < height.length; i++) {
            for (int j = i + 1; j < height.length; j++) {
                int area = (j - i) * Math.min(height[i], height[j]);
                maxWater = Math.max(maxWater, area);
            }
        }
        
        return maxWater;
    }
    
    // Test method
    public static void main(String[] args) {
        ContainerWithMostWater solution = new ContainerWithMostWater();
        
        // Test case 1: Example from problem
        int[] heights1 = {1, 8, 6, 2, 5, 4, 8, 3, 7};
        System.out.println("Heights: " + Arrays.toString(heights1));
        System.out.println("Max Area (Two Pointer): " + solution.maxArea(heights1));
        System.out.println("Max Area (Brute Force): " + solution.maxAreaBruteForce(heights1));
        System.out.println();
        
        // Test case 2: Simple case
        int[] heights2 = {1, 1};
        System.out.println("Heights: " + Arrays.toString(heights2));
        System.out.println("Max Area: " + solution.maxArea(heights2));
        System.out.println();
        
        // Test case 3: With step tracking
        int[] heights3 = {2, 3, 4, 5, 18, 17, 6};
        System.out.println("Heights: " + Arrays.toString(heights3));
        System.out.println("Max Area: " + solution.maxAreaWithSteps(heights3));
    }
}
```

**Key Insights:**
- **Greedy Choice**: Always move the pointer with smaller height
- **Why Greedy Works**: Keeping shorter line can never yield better result
- **Area Calculation**: Width × minimum height of two boundaries

---

### Problem 4: Three Sum

**Problem Statement:**
Find all unique triplets in the array that sum to zero.

**Approach:** Fix one element and use two pointers for the remaining two elements.

```java
/**
 * Three Sum Problem
 * Time Complexity: O(n²) - O(n log n) for sorting + O(n²) for finding triplets
 * Space Complexity: O(1) - Not counting output space
 */
public class ThreeSum {
    
    /**
     * Finds all unique triplets that sum to zero
     * @param nums Array of integers
     * @return List of unique triplets that sum to zero
     */
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        
        // Handle edge cases
        if (nums == null || nums.length < 3) {
            return result;
        }
        
        // Sort array to enable two pointer technique
        Arrays.sort(nums);
        
        // Fix first element and find pairs for remaining two
        for (int i = 0; i < nums.length - 2; i++) {
            // Skip duplicates for first element
            if (i > 0 && nums[i] == nums[i - 1]) {
                continue;
            }
            
            // Use two pointers for remaining elements
            int left = i + 1;                   // Start after current element
            int right = nums.length - 1;       // Start from end
            int target = -nums[i];              // We need sum = 0, so target = -nums[i]
            
            while (left < right) {
                int currentSum = nums[left] + nums[right];
                
                if (currentSum == target) {
                    // Found a valid triplet
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    
                    // Skip duplicates for second element
                    while (left < right && nums[left] == nums[left + 1]) {
                        left++;
                    }
                    
                    // Skip duplicates for third element
                    while (left < right && nums[right] == nums[right - 1]) {
                        right--;
                    }
                    
                    // Move both pointers
                    left++;
                    right--;
                } else if (currentSum < target) {
                    // Sum is too small, need larger number
                    left++;
                } else {
                    // Sum is too large, need smaller number
                    right--;
                }
            }
        }
        
        return result;
    }
    
    /**
     * Alternative implementation with detailed comments for each step
     */
    public List<List<Integer>> threeSumDetailed(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        
        if (nums.length < 3) return result;
        
        Arrays.sort(nums);  // Critical: Sort for two pointer technique
        
        for (int i = 0; i < nums.length - 2; i++) {
            // Early termination: if smallest element > 0, no negative sum possible
            if (nums[i] > 0) break;
            
            // Skip duplicates for first element
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            
            int left = i + 1;
            int right = nums.length - 1;
            
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                
                if (sum == 0) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    
                    // Move both pointers and skip duplicates
                    do { left++; } while (left < right && nums[left] == nums[left - 1]);
                    do { right--; } while (left < right && nums[right] == nums[right + 1]);
                } else if (sum < 0) {
                    left++;     // Need larger sum
                } else {
                    right--;    // Need smaller sum
                }
            }
        }
        
        return result;
    }
    
    /**
     * Finds triplets with specific target sum (generalized version)
     */
    public List<List<Integer>> threeSumTarget(int[] nums, int target) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);
        
        for (int i = 0; i < nums.length - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue;
            
            int left = i + 1;
            int right = nums.length - 1;
            int twoSumTarget = target - nums[i];
            
            while (left < right) {
                int currentSum = nums[left] + nums[right];
                
                if (currentSum == twoSumTarget) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));
                    
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;
                    
                    left++;
                    right--;
                } else if (currentSum < twoSumTarget) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        
        return result;
    }
    
    // Test method
    public static void main(String[] args) {
        ThreeSum solution = new ThreeSum();
        
        // Test case 1: Basic example
        int[] nums1 = {-1, 0, 1, 2, -1, -4};
        System.out.println("Input: " + Arrays.toString(nums1));
        System.out.println("Three Sum Zero: " + solution.threeSum(nums1));
        System.out.println();
        
        // Test case 2: No solution
        int[] nums2 = {1, 2, 3};
        System.out.println("Input: " + Arrays.toString(nums2));
        System.out.println("Three Sum Zero: " + solution.threeSum(nums2));
        System.out.println();
        
        // Test case 3: With target
        int[] nums3 = {1, 0, -1, 0, -2, 2};
        System.out.println("Input: " + Arrays.toString(nums3));
        System.out.println("Three Sum Target 0: " + solution.threeSumTarget(nums3, 0));
        System.out.println("Three Sum Target 3: " + solution.threeSumTarget(nums3, 3));
    }
}
```

**Key Insights:**
- **Sorting is Essential**: Enables two pointer technique and duplicate handling
- **Duplicate Handling**: Critical to avoid duplicate triplets in result
- **Optimization**: Early termination when smallest element > 0

---

### Problem 5: Remove Duplicates from Sorted Array

**Problem Statement:**
Remove duplicates from sorted array in-place and return new length.

**Approach:** Use two pointers - one for reading, one for writing unique elements.

```java
/**
 * Remove Duplicates from Sorted Array
 * Time Complexity: O(n) - Single pass through array
 * Space Complexity: O(1) - In-place modification
 */
public class RemoveDuplicates {
    
    /**
     * Removes duplicates from sorted array in-place
     * @param nums Sorted array (may contain duplicates)
     * @return New length after removing duplicates
     */
    public int removeDuplicates(int[] nums) {
        // Handle edge cases
        if (nums == null || nums.length == 0) {
            return 0;
        }
        
        // Initialize slow pointer (write position)
        int slow = 0;  // Points to position where next unique element should be placed
        
        // Fast pointer iterates through array (read position)
        for (int fast = 1; fast < nums.length; fast++) {
            // If current element is different from last unique element
            if (nums[fast] != nums[slow]) {
                slow++;                     // Move write position forward
                nums[slow] = nums[fast];    // Copy unique element to write position
            }
            // If nums[fast] == nums[slow], skip duplicate (don't increment slow)
        }
        
        // Return length of array after removing duplicates
        return slow + 1;  // +1 because slow is 0-indexed
    }
    
    /**
     * Alternative implementation with detailed step tracking
     */
    public int removeDuplicatesWithSteps(int[] nums) {
        if (nums.length <= 1) return nums.length;
        
        int slow = 0;
        System.out.println("Initial array: " + Arrays.toString(nums));
        
        for (int fast = 1; fast < nums.length; fast++) {
            System.out.printf("Step %d: fast=%d(value=%d), slow=%d(value=%d)", 
                             fast, fast, nums[fast], slow, nums[slow]);
            
            if (nums[fast] != nums[slow]) {
                slow++;
                nums[slow] = nums[fast];
                System.out.printf(" -> Unique found, moved to position %d%n", slow);
            } else {
                System.out.println(" -> Duplicate, skipped");
            }
            
            System.out.println("Array state: " + Arrays.toString(nums));
        }
        
        return slow + 1;
    }
    
    /**
     * Removes duplicates allowing at most k duplicates
     * @param nums Sorted array
     * @param k Maximum allowed duplicates
     * @return New length
     */
    public int removeDuplicatesAllowK(int[] nums, int k) {
        if (nums.length <= k) return nums.length;
        
        int slow = k;  // Start from position k
        
        for (int fast = k; fast < nums.length; fast++) {
            // Compare with element k positions back
            if (nums[fast] != nums[slow - k]) {
                nums[slow] = nums[fast];
                slow++;
            }
        }
        
        return slow;
    }
    
    /**
     * Remove duplicates and return the actual cleaned array
     */
    public int[] removeDuplicatesReturnArray(int[] nums) {
        int newLength = removeDuplicates(nums);
        return Arrays.copyOf(nums, newLength);
    }
    
    // Test method
    public static void main(String[] args) {
        RemoveDuplicates solution = new RemoveDuplicates();
        
        // Test case 1: Basic case with duplicates
        int[] nums1 = {1, 1, 2, 2, 3, 4, 4, 5};
        System.out.println("Original: " + Arrays.toString(nums1));
        int length1 = solution.removeDuplicatesWithSteps(nums1.clone());
        System.out.println("New length: " + length1);
        System.out.println("Cleaned array: " + Arrays.toString(solution.removeDuplicatesReturnArray(nums1)));
        System.out.println();
        
        // Test case 2: No duplicates
        int[] nums2 = {1, 2, 3, 4, 5};
        System.out.println("Original: " + Arrays.toString(nums2));
        int length2 = solution.removeDuplicates(nums2);
        System.out.println("New length: " + length2);
        System.out.println();
        
        // Test case 3: All same elements
        int[] nums3 = {1, 1, 1, 1, 1};
        System.out.println("Original: " + Arrays.toString(nums3));
        int length3 = solution.removeDuplicates(nums3);
        System.out.println("New length: " + length3);
        System.out.println();
        
        // Test case 4: Allow at most 2 duplicates
        int[] nums4 = {1, 1, 1, 2, 2, 3, 3, 3, 3};
        System.out.println("Original: " + Arrays.toString(nums4));
        int length4 = solution.removeDuplicatesAllowK(nums4.clone(), 2);
        System.out.println("New length (allow 2 duplicates): " + length4);
        System.out.println("Result: " + Arrays.toString(Arrays.copyOf(nums4, length4)));
    }
}
```

**Key Insights:**
- **Two Pointer Roles**: Slow (write position), Fast (read position)
- **In-place Modification**: No extra space needed for result
- **Invariant**: Elements from 0 to slow are unique

---

### Problem 6: Linked List Cycle Detection

**Problem Statement:**
Detect if a linked list has a cycle using Floyd's Cycle Detection Algorithm.

**Approach:** Use fast and slow pointers (tortoise and hare).

```java
/**
 * Linked List Cycle Detection (Floyd's Algorithm)
 * Time Complexity: O(n) - At most 2n iterations
 * Space Complexity: O(1) - Only using two pointer variables
 */
public class LinkedListCycle {
    
    // Definition for singly-linked list
    static class ListNode {
        int val;
        ListNode next;
        
        ListNode(int x) {
            val = x;
            next = null;
        }
    }
    
    /**
     * Detects if linked list has a cycle
     * @param head Head of the linked list
     * @return true if cycle exists, false otherwise
     */
    public boolean hasCycle(ListNode head) {
        // Handle edge cases
        if (head == null || head.next == null) {
            return false;
        }
        
        // Initialize two pointers
        ListNode slow = head;       // Tortoise: moves 1 step at a time
        ListNode fast = head;       // Hare: moves 2 steps at a time
        
        // Continue until fast pointer reaches end or cycle is detected
        while (fast != null && fast.next != null) {
            slow = slow.next;           // Move slow pointer 1 step
            fast = fast.next.next;      // Move fast pointer 2 steps
            
            // If pointers meet, cycle detected
            if (slow == fast) {
                return true;
            }
        }
        
        // Fast pointer reached end, no cycle
        return false;
    }
    
    /**
     * Finds the starting node of the cycle (if exists)
     * @param head Head of the linked list
     * @return Starting node of cycle, or null if no cycle
     */
    public ListNode detectCycleStart(ListNode head) {
        if (head == null || head.next == null) {
            return null;
        }
        
        // Phase 1: Detect if cycle exists
        ListNode slow = head;
        ListNode fast = head;
        
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            if (slow == fast) {
                // Cycle detected, now find the start
                break;
            }
        }
        
        // No cycle found
        if (fast == null || fast.next == null) {
            return null;
        }
        
        // Phase 2: Find cycle start
        // Move one pointer to head, keep other at meeting point
        slow = head;
        
        // Move both pointers one step at a time until they meet
        while (slow != fast) {
            slow = slow.next;
            fast = fast.next;
        }
        
        return slow;  // This is the start of the cycle
    }
    
    /**
     * Finds the length of the cycle
     * @param head Head of the linked list
     * @return Length of cycle, or 0 if no cycle
     */
    public int cycleLength(ListNode head) {
        if (!hasCycle(head)) {
            return 0;
        }
        
        // Find meeting point first
        ListNode slow = head;
        ListNode fast = head;
        
        // Find meeting point
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;
            
            if (slow == fast) {
                break;
            }
        }
        
        // Count cycle length
        int length = 1;
        ListNode current = slow.next;
        
        while (current != slow) {
            current = current.next;
            length++;
        }
        
        return length;
    }
    
    /**
     * Creates a linked list with cycle for testing
     */
    public static ListNode createCyclicList() {
        ListNode head = new ListNode(1);
        ListNode node2 = new ListNode(2);
        ListNode node3 = new ListNode(3);
        ListNode node4 = new ListNode(4);
        ListNode node5 = new ListNode(5);
        
        // Link nodes: 1 -> 2 -> 3 -> 4 -> 5
        head.next = node2;
        node2.next = node3;
        node3.next = node4;
        node4.next = node5;
        
        // Create cycle: 5 -> 3 (cycle back to node3)
        node5.next = node3;
        
        return head;
    }
    
    /**
     * Creates a non-cyclic linked list for testing
     */
    public static ListNode createNonCyclicList() {
        ListNode head = new ListNode(1);
        head.next = new ListNode(2);
        head.next.next = new ListNode(3);
        head.next.next.next = new ListNode(4);
        
        return head;
    }
    
    // Test method
    public static void main(String[] args) {
        LinkedListCycle solution = new LinkedListCycle();
        
        // Test case 1: List with cycle
        ListNode cyclicList = createCyclicList();
        System.out.println("Cyclic list:");
        System.out.println("Has cycle: " + solution.hasCycle(cyclicList));
        System.out.println("Cycle length: " + solution.cycleLength(cyclicList));
        
        ListNode cycleStart = solution.detectCycleStart(cyclicList);
        System.out.println("Cycle starts at node with value: " + 
                          (cycleStart != null ? cycleStart.val : "null"));
        System.out.println();
        
        // Test case 2: List without cycle
        ListNode nonCyclicList = createNonCyclicList();
        System.out.println("Non-cyclic list:");
        System.out.println("Has cycle: " + solution.hasCycle(nonCyclicList));
        System.out.println("Cycle length: " + solution.cycleLength(nonCyclicList));
        
        ListNode cycleStart2 = solution.detectCycleStart(nonCyclicList);
        System.out.println("Cycle starts at: " + 
                          (cycleStart2 != null ? cycleStart2.val : "null"));
    }
}
```

**Key Insights:**
- **Floyd's Algorithm**: Fast pointer gains one position per iteration on slow pointer
- **Cycle Detection**: If there's a cycle, fast and slow pointers will eventually meet
- **Mathematical Proof**: Distance relationship ensures meeting point detection

---

### Problem 7: Trapping Rain Water

**Problem Statement:**
Given heights representing elevation map, calculate how much rainwater can be trapped.

**Approach:** Use two pointers with left and right max heights tracking.

```java
/**
 * Trapping Rain Water
 * Time Complexity: O(n) - Single pass through array
 * Space Complexity: O(1) - Only using pointer variables and max trackers
 */
public class TrappingRainWater {
    
    /**
     * Calculates trapped rainwater using two pointer technique
     * @param height Array representing elevation heights
     * @return Total units of trapped water
     */
    public int trap(int[] height) {
        // Handle edge cases
        if (height == null || height.length <= 2) {
            return 0;  // Need at least 3 heights to trap water
        }
        
        // Initialize pointers and max height trackers
        int left = 0;                           // Left pointer
        int right = height.length - 1;         // Right pointer
        int leftMax = 0;                        // Maximum height seen from left
        int rightMax = 0;                       // Maximum height seen from right
        int totalWater = 0;                     // Total trapped water
        
        while (left < right) {
            // Process the side with smaller height first
            if (height[left] < height[right]) {
                // Process left side
                if (height[left] >= leftMax) {
                    // Current height is new maximum, update leftMax
                    leftMax = height[left];
                } else {
                    // Water can be trapped here
                    // Water level = leftMax, current ground = height[left]
                    totalWater += leftMax - height[left];
                }
                left++;  // Move left pointer inward
            } else {
                // Process right side
                if (height[right] >= rightMax) {
                    // Current height is new maximum, update rightMax
                    rightMax = height[right];
                } else {
                    // Water can be trapped here
                    // Water level = rightMax, current ground = height[right]
                    totalWater += rightMax - height[right];
                }
                right--;  // Move right pointer inward
            }
        }
        
        return totalWater;
    }
    
    /**
     * Alternative implementation with detailed step tracking
     */
    public int trapWithSteps(int[] height) {
        if (height.length <= 2) return 0;
        
        int left = 0, right = height.length - 1;
        int leftMax = 0, rightMax = 0;
        int totalWater = 0;
        
        System.out.println("Heights: " + Arrays.toString(height));
        System.out.println("Step-by-step water trapping:");
        
        int step = 1;
        while (left < right) {
            System.out.printf("Step %d: left=%d(h=%d), right=%d(h=%d), leftMax=%d, rightMax=%d",
                step++, left, height[left], right, height[right], leftMax, rightMax);
            
            if (height[left] < height[right]) {
                if (height[left] >= leftMax) {
                    leftMax = height[left];
                    System.out.println(" -> New leftMax");
                } else {
                    int water = leftMax - height[left];
                    totalWater += water;
                    System.out.printf(" -> Trapped %d units at position %d%n", water, left);
                }
                left++;
            } else {
                if (height[right] >= rightMax) {
                    rightMax = height[right];
                    System.out.println(" -> New rightMax");
                } else {
                    int water = rightMax - height[right];
                    totalWater += water;
                    System.out.printf(" -> Trapped %d units at position %d%n", water, right);
                }
                right--;
            }
        }
        
        return totalWater;
    }
    
    /**
     * Brute force solution for comparison (O(n²))
     */
    public int trapBruteForce(int[] height) {
        int totalWater = 0;
        
        for (int i = 1; i < height.length - 1; i++) {
            // Find maximum height to the left
            int leftMax = 0;
            for (int j = 0; j < i; j++) {
                leftMax = Math.max(leftMax, height[j]);
            }
            
            // Find maximum height to the right
            int rightMax = 0;
            for (int j = i + 1; j < height.length; j++) {
                rightMax = Math.max(rightMax, height[j]);
            }
            
            // Calculate water at current position
            int waterLevel = Math.min(leftMax, rightMax);
            if (waterLevel > height[i]) {
                totalWater += waterLevel - height[i];
            }
        }
        
        return totalWater;
    }
    
    /**
     * Solution using pre-computed left and right max arrays (O(n) time, O(n) space)
     */
    public int trapWithArrays(int[] height) {
        if (height.length <= 2) return 0;
        
        int n = height.length;
        int[] leftMax = new int[n];
        int[] rightMax = new int[n];
        
        // Compute left max for each position
        leftMax[0] = height[0];
        for (int i = 1; i < n; i++) {
            leftMax[i] = Math.max(leftMax[i - 1], height[i]);
        }
        
        // Compute right max for each position
        rightMax[n - 1] = height[n - 1];
        for (int i = n - 2; i >= 0; i--) {
            rightMax[i] = Math.max(rightMax[i + 1], height[i]);
        }
        
        // Calculate trapped water
        int totalWater = 0;
        for (int i = 0; i < n; i++) {
            int waterLevel = Math.min(leftMax[i], rightMax[i]);
            totalWater += Math.max(0, waterLevel - height[i]);
        }
        
        return totalWater;
    }
    
    // Test method
    public static void main(String[] args) {
        TrappingRainWater solution = new TrappingRainWater();
        
        // Test case 1: Classic example
        int[] heights1 = {0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1};
        System.out.println("Test Case 1:");
        System.out.println("Two Pointer: " + solution.trap(heights1));
        System.out.println("Brute Force: " + solution.trapBruteForce(heights1));
        System.out.println("With Arrays: " + solution.trapWithArrays(heights1));
        System.out.println();
        
        // Test case 2: Simple case
        int[] heights2 = {3, 0, 2, 0, 4};
        System.out.println("Test Case 2:");
        System.out.println("Result: " + solution.trapWithSteps(heights2));
        System.out.println();
        
        // Test case 3: No water can be trapped
        int[] heights3 = {1, 2, 3, 4, 5};
        System.out.println("Test Case 3 (Ascending):");
        System.out.println("Result: " + solution.trap(heights3));
        
        int[] heights4 = {5, 4, 3, 2, 1};
        System.out.println("Test Case 4 (Descending):");
        System.out.println("Result: " + solution.trap(heights4));
    }
}
```

**Key Insights:**
- **Key Idea**: Water level at any position = min(leftMax, rightMax)
- **Two Pointer Optimization**: Process the side with smaller height first
- **Why it Works**: Smaller side determines the water level

---

### Problem 8: Longest Substring Without Repeating Characters

**Problem Statement:**
Find the length of the longest substring without repeating characters.

**Approach:** Use sliding window with two pointers and character frequency tracking.

```java
/**
 * Longest Substring Without Repeating Characters
 * Time Complexity: O(n) - Each character visited at most twice
 * Space Complexity: O(min(m,n)) - Where m is character set size
 */
public class LongestSubstringWithoutRepeating {
    
    /**
     * Finds length of longest substring without repeating characters
     * @param s Input string
     * @return Length of longest substring without repeating characters
     */
    public int lengthOfLongestSubstring(String s) {
        // Handle edge cases
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        // Use HashMap to track character positions
        Map<Character, Integer> charIndexMap = new HashMap<>();
        
        int maxLength = 0;          // Maximum length found so far
        int left = 0;               // Left pointer (start of current window)
        
        // Right pointer iterates through string (end of current window)
        for (int right = 0; right < s.length(); right++) {
            char currentChar = s.charAt(right);
            
            // If character is already in current window
            if (charIndexMap.containsKey(currentChar) && 
                charIndexMap.get(currentChar) >= left) {
                // Move left pointer to skip the repeated character
                left = charIndexMap.get(currentChar) + 1;
            }
            
            // Update character's latest position
            charIndexMap.put(currentChar, right);
            
            // Update maximum length
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
    
    /**
     * Alternative implementation using HashSet with explicit window management
     */
    public int lengthOfLongestSubstringWithSet(String s) {
        if (s.length() <= 1) return s.length();
        
        Set<Character> windowChars = new HashSet<>();
        int maxLength = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            
            // Shrink window from left until no duplicate
            while (windowChars.contains(rightChar)) {
                windowChars.remove(s.charAt(left));
                left++;
            }
            
            // Add current character to window
            windowChars.add(rightChar);
            
            // Update maximum length
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
    
    /**
     * Implementation with detailed step tracking
     */
    public int lengthOfLongestSubstringWithSteps(String s) {
        if (s.length() <= 1) return s.length();
        
        Map<Character, Integer> charIndexMap = new HashMap<>();
        int maxLength = 0;
        int left = 0;
        
        System.out.println("String: \"" + s + "\"");
        System.out.println("Step-by-step sliding window:");
        
        for (int right = 0; right < s.length(); right++) {
            char currentChar = s.charAt(right);
            
            System.out.printf("Step %d: char='%c', window=[%d,%d], ",
                right + 1, currentChar, left, right);
            
            if (charIndexMap.containsKey(currentChar) && 
                charIndexMap.get(currentChar) >= left) {
                int oldLeft = left;
                left = charIndexMap.get(currentChar) + 1;
                System.out.printf("duplicate found, moved left from %d to %d, ", oldLeft, left);
            }
            
            charIndexMap.put(currentChar, right);
            int currentLength = right - left + 1;
            maxLength = Math.max(maxLength, currentLength);
            
            System.out.printf("current_length=%d, max_length=%d%n", currentLength, maxLength);
            System.out.println("Current window: \"" + s.substring(left, right + 1) + "\"");
        }
        
        return maxLength;
    }
    
    /**
     * Optimized version for ASCII characters only
     */
    public int lengthOfLongestSubstringASCII(String s) {
        // Array to store last seen index of each ASCII character
        int[] lastIndex = new int[128];
        Arrays.fill(lastIndex, -1);
        
        int maxLength = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char currentChar = s.charAt(right);
            
            // If character was seen in current window
            if (lastIndex[currentChar] >= left) {
                left = lastIndex[currentChar] + 1;
            }
            
            lastIndex[currentChar] = right;
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
    
    /**
     * Returns the actual longest substring (not just length)
     */
    public String longestSubstringWithoutRepeating(String s) {
        if (s.length() <= 1) return s;
        
        Map<Character, Integer> charIndexMap = new HashMap<>();
        int maxLength = 0;
        int maxStart = 0;
        int left = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char currentChar = s.charAt(right);
            
            if (charIndexMap.containsKey(currentChar) && 
                charIndexMap.get(currentChar) >= left) {
                left = charIndexMap.get(currentChar) + 1;
            }
            
            charIndexMap.put(currentChar, right);
            
            int currentLength = right - left + 1;
            if (currentLength > maxLength) {
                maxLength = currentLength;
                maxStart = left;
            }
        }
        
        return s.substring(maxStart, maxStart + maxLength);
    }
    
    // Test method
    public static void main(String[] args) {
        LongestSubstringWithoutRepeating solution = new LongestSubstringWithoutRepeating();
        
        // Test case 1: Basic example
        String s1 = "abcabcbb";
        System.out.println("Test Case 1: \"" + s1 + "\"");
        System.out.println("Length: " + solution.lengthOfLongestSubstring(s1));
        System.out.println("Substring: \"" + solution.longestSubstringWithoutRepeating(s1) + "\"");
        System.out.println();
        
        // Test case 2: All same characters
        String s2 = "bbbbb";
        System.out.println("Test Case 2: \"" + s2 + "\"");
        System.out.println("Length: " + solution.lengthOfLongestSubstring(s2));
        System.out.println();
        
        // Test case 3: Complex example with steps
        String s3 = "pwwkew";
        System.out.println("Test Case 3 with steps:");
        System.out.println("Length: " + solution.lengthOfLongestSubstringWithSteps(s3));
        System.out.println();
        
        // Test case 4: No repeating characters
        String s4 = "abcdef";
        System.out.println("Test Case 4: \"" + s4 + "\"");
        System.out.println("Length: " + solution.lengthOfLongestSubstring(s4));
        
        // Performance comparison
        String largeString = "abcdefghijklmnopqrstuvwxyz".repeat(1000);
        long start = System.nanoTime();
        int result1 = solution.lengthOfLongestSubstring(largeString);
        long time1 = System.nanoTime() - start;
        
        start = System.nanoTime();
        int result2 = solution.lengthOfLongestSubstringASCII(largeString);
        long time2 = System.nanoTime() - start;
        
        System.out.printf("%nPerformance test (string length: %d):%n", largeString.length());
        System.out.printf("HashMap approach: %d (time: %.2f ms)%n", result1, time1 / 1_000_000.0);
        System.out.printf("Array approach: %d (time: %.2f ms)%n", result2, time2 / 1_000_000.0);
    }
}
```

**Key Insights:**
- **Sliding Window**: Expand right, contract left when duplicate found
- **Character Tracking**: HashMap stores last seen index of each character
- **Window Management**: Left pointer jumps to avoid duplicates

---

### Problem 9: Minimum Window Substring

**Problem Statement:**
Find the minimum window substring that contains all characters of target string.

**Approach:** Use sliding window with character frequency counting.

```java
/**
 * Minimum Window Substring
 * Time Complexity: O(|s| + |t|) - Each character in s visited at most twice
 * Space Complexity: O(|s| + |t|) - For character frequency maps
 */
public class MinimumWindowSubstring {
    
    /**
     * Finds minimum window substring containing all characters of t
     * @param s Source string to search in
     * @param t Target string containing required characters
     * @return Minimum window substring, or empty string if not found
     */
    public String minWindow(String s, String t) {
        // Handle edge cases
        if (s == null || t == null || s.length() < t.length()) {
            return "";
        }
        
        // Count frequency of characters needed
        Map<Character, Integer> targetCount = new HashMap<>();
        for (char c : t.toCharArray()) {
            targetCount.put(c, targetCount.getOrDefault(c, 0) + 1);
        }
        
        // Sliding window variables
        int left = 0;                               // Left pointer
        int minLeft = 0;                            // Start of minimum window found
        int minLength = Integer.MAX_VALUE;          // Length of minimum window
        int requiredChars = targetCount.size();     // Number of unique chars needed
        int formedChars = 0;                        // Number of unique chars satisfied
        
        // Current window character frequency
        Map<Character, Integer> windowCount = new HashMap<>();
        
        // Expand window with right pointer
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            
            // Add character to window
            windowCount.put(rightChar, windowCount.getOrDefault(rightChar, 0) + 1);
            
            // Check if this character's requirement is satisfied
            if (targetCount.containsKey(rightChar) &&
                windowCount.get(rightChar).intValue() == targetCount.get(rightChar).intValue()) {
                formedChars++;
            }
            
            // Try to shrink window from left
            while (left <= right && formedChars == requiredChars) {
                // Update minimum window if current is smaller
                if (right - left + 1 < minLength) {
                    minLength = right - left + 1;
                    minLeft = left;
                }
                
                // Remove leftmost character
                char leftChar = s.charAt(left);
                windowCount.put(leftChar, windowCount.get(leftChar) - 1);
                
                // Check if removing this character breaks a requirement
                if (targetCount.containsKey(leftChar) &&
                    windowCount.get(leftChar).intValue() < targetCount.get(leftChar).intValue()) {
                    formedChars--;
                }
                
                left++;  // Shrink window
            }
        }
        
        // Return minimum window or empty string if not found
        return minLength == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLength);
    }
    
    /**
     * Alternative implementation with detailed step tracking
     */
    public String minWindowWithSteps(String s, String t) {
        System.out.println("Finding minimum window for s=\"" + s + "\", t=\"" + t + "\"");
        
        Map<Character, Integer> targetCount = new HashMap<>();
        for (char c : t.toCharArray()) {
            targetCount.put(c, targetCount.getOrDefault(c, 0) + 1);
        }
        
        System.out.println("Target character counts: " + targetCount);
        
        int left = 0, minLeft = 0, minLength = Integer.MAX_VALUE;
        int requiredChars = targetCount.size();
        int formedChars = 0;
        Map<Character, Integer> windowCount = new HashMap<>();
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            windowCount.put(rightChar, windowCount.getOrDefault(rightChar, 0) + 1);
            
            if (targetCount.containsKey(rightChar) &&
                windowCount.get(rightChar).equals(targetCount.get(rightChar))) {
                formedChars++;
            }
            
            System.out.printf("Right=%d, char='%c', window=[%d,%d], formed=%d/%d%n",
                right, rightChar, left, right, formedChars, requiredChars);
            
            while (left <= right && formedChars == requiredChars) {
                System.out.printf("  Valid window found: \"%s\" (length=%d)%n",
                    s.substring(left, right + 1), right - left + 1);
                
                if (right - left + 1 < minLength) {
                    minLength = right - left + 1;
                    minLeft = left;
                    System.out.println("    New minimum!");
                }
                
                char leftChar = s.charAt(left);
                windowCount.put(leftChar, windowCount.get(leftChar) - 1);
                
                if (targetCount.containsKey(leftChar) &&
                    windowCount.get(leftChar) < targetCount.get(leftChar)) {
                    formedChars--;
                }
                
                left++;
                System.out.printf("  Shrunk window: left=%d, formed=%d%n", left, formedChars);
            }
        }
        
        String result = minLength == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLength);
        System.out.println("Final result: \"" + result + "\"");
        return result;
    }
    
    /**
     * Optimized version using character arrays (for ASCII characters)
     */
    public String minWindowOptimized(String s, String t) {
        if (s.length() < t.length()) return "";
        
        // Use arrays instead of HashMap for ASCII characters
        int[] targetCount = new int[128];
        int[] windowCount = new int[128];
        
        // Count target characters
        for (char c : t.toCharArray()) {
            targetCount[c]++;
        }
        
        int left = 0, minLeft = 0, minLength = Integer.MAX_VALUE;
        int requiredChars = t.length();  // Total characters needed
        int formedChars = 0;             // Characters satisfied in window
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            windowCount[rightChar]++;
            
            // If this character contributes to the requirement
            if (windowCount[rightChar] <= targetCount[rightChar]) {
                formedChars++;
            }
            
            // Try to shrink window
            while (formedChars == requiredChars) {
                if (right - left + 1 < minLength) {
                    minLength = right - left + 1;
                    minLeft = left;
                }
                
                char leftChar = s.charAt(left);
                windowCount[leftChar]--;
                
                if (windowCount[leftChar] < targetCount[leftChar]) {
                    formedChars--;
                }
                
                left++;
            }
        }
        
        return minLength == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLength);
    }
    
    /**
     * Checks if current window contains all required characters
     */
    private boolean containsAll(Map<Character, Integer> windowCount, 
                               Map<Character, Integer> targetCount) {
        for (Map.Entry<Character, Integer> entry : targetCount.entrySet()) {
            char c = entry.getKey();
            int required = entry.getValue();
            if (windowCount.getOrDefault(c, 0) < required) {
                return false;
            }
        }
        return true;
    }
    
    // Test method
    public static void main(String[] args) {
        MinimumWindowSubstring solution = new MinimumWindowSubstring();
        
        // Test case 1: Basic example
        String s1 = "ADOBECODEBANC";
        String t1 = "ABC";
        System.out.println("Test Case 1:");
        System.out.println("Result: \"" + solution.minWindow(s1, t1) + "\"");
        System.out.println();
        
        // Test case 2: No valid window
        String s2 = "a";
        String t2 = "aa";
        System.out.println("Test Case 2:");
        System.out.println("Result: \"" + solution.minWindow(s2, t2) + "\"");
        System.out.println();
        
        // Test case 3: With step tracking
        String s3 = "ADOBECODEBANC";
        String t3 = "ABC";
        System.out.println("Test Case 3 with steps:");
        solution.minWindowWithSteps(s3, t3);
        System.out.println();
        
        // Test case 4: Same length strings
        String s4 = "ab";
        String t4 = "ba";
        System.out.println("Test Case 4:");
        System.out.println("Result: \"" + solution.minWindow(s4, t4) + "\"");
        
        // Performance comparison
        String largeS = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".repeat(1000);
        String largeT = "ABC";
        
        long start = System.nanoTime();
        String result1 = solution.minWindow(largeS, largeT);
        long time1 = System.nanoTime() - start;
        
        start = System.nanoTime();
        String result2 = solution.minWindowOptimized(largeS, largeT);
        long time2 = System.nanoTime() - start;
        
        System.out.printf("%nPerformance test:%n");
        System.out.printf("HashMap approach: length=%d (time: %.2f ms)%n", 
                         result1.length(), time1 / 1_000_000.0);
        System.out.printf("Array approach: length=%d (time: %.2f ms)%n", 
                         result2.length(), time2 / 1_000_000.0);
    }
}
```

**Key Insights:**
- **Two Phase**: Expand window to include all characters, then contract to find minimum
- **Character Counting**: Track frequency of characters in both target and current window
- **Valid Window**: When all required characters are satisfied in current frequency

---

### Problem 10: Sort Colors (Dutch National Flag)

**Problem Statement:**
Sort array containing only 0s, 1s, and 2s in-place using constant space.

**Approach:** Use three pointers to partition array in single pass.

```java
/**
 * Sort Colors (Dutch National Flag Problem)
 * Time Complexity: O(n) - Single pass through array
 * Space Complexity: O(1) - In-place sorting with constant extra space
 */
public class SortColors {
    
    /**
     * Sorts array of 0s, 1s, and 2s in-place
     * @param nums Array containing only 0, 1, and 2
     */
    public void sortColors(int[] nums) {
        // Handle edge cases
        if (nums == null || nums.length <= 1) {
            return;
        }
        
        // Three pointers for three-way partitioning
        int left = 0;                    // Boundary for 0s (elements 0..left-1 are 0s)
        int right = nums.length - 1;     // Boundary for 2s (elements right+1..n-1 are 2s)
        int current = 0;                 // Current element being examined
        
        // Continue until current pointer reaches right boundary
        while (current <= right) {
            if (nums[current] == 0) {
                // Current element is 0, swap with left boundary
                swap(nums, current, left);
                left++;                  // Expand 0s region
                current++;               // Move to next element
                // Safe to increment current because nums[left] was 0 or 1
            } else if (nums[current] == 2) {
                // Current element is 2, swap with right boundary
                swap(nums, current, right);
                right--;                 // Expand 2s region
                // Don't increment current! Need to examine swapped element
            } else {
                // Current element is 1, it's in correct region
                current++;               // Move to next element
            }
        }
    }
    
    /**
     * Helper method to swap two elements in array
     */
    private void swap(int[] nums, int i, int j) {
        int temp = nums[i];
        nums[i] = nums[j];
        nums[j] = temp;
    }
    
    /**
     * Sort colors with detailed step tracking
     */
    public void sortColorsWithSteps(int[] nums) {
        System.out.println("Initial array: " + Arrays.toString(nums));
        
        if (nums.length <= 1) return;
        
        int left = 0, right = nums.length - 1, current = 0;
        int step = 1;
        
        while (current <= right) {
            System.out.printf("Step %d: left=%d, current=%d, right=%d, array=%s%n",
                step++, left, current, right, Arrays.toString(nums));
            
            if (nums[current] == 0) {
                System.out.println("  Found 0, swapping with left boundary");
                swap(nums, current, left);
                left++;
                current++;
            } else if (nums[current] == 2) {
                System.out.println("  Found 2, swapping with right boundary");
                swap(nums, current, right);
                right--;
                // Note: don't increment current
            } else {
                System.out.println("  Found 1, keeping in middle");
                current++;
            }
            
            System.out.println("  After: " + Arrays.toString(nums));
        }
        
        System.out.println("Final sorted array: " + Arrays.toString(nums));
    }
    
    /**
     * Alternative implementation using counting sort approach
     */
    public void sortColorsCountingSort(int[] nums) {
        // Count occurrences of each color
        int count0 = 0, count1 = 0, count2 = 0;
        
        for (int num : nums) {
            if (num == 0) count0++;
            else if (num == 1) count1++;
            else count2++;
        }
        
        // Fill array with counted values
        int index = 0;
        
        // Fill 0s
        for (int i = 0; i < count0; i++) {
            nums[index++] = 0;
        }
        
        // Fill 1s
        for (int i = 0; i < count1; i++) {
            nums[index++] = 1;
        }
        
        // Fill 2s
        for (int i = 0; i < count2; i++) {
            nums[index++] = 2;
        }
    }
    
    /**
     * Generalized version for sorting array with k distinct values
     */
    public void sortKColors(int[] nums, int k) {
        if (k == 3) {
            sortColors(nums);
            return;
        }
        
        // Use counting sort for general case
        int[] count = new int[k];
        
        // Count occurrences
        for (int num : nums) {
            count[num]++;
        }
        
        // Fill array
        int index = 0;
        for (int color = 0; color < k; color++) {
            for (int i = 0; i < count[color]; i++) {
                nums[index++] = color;
            }
        }
    }
    
    /**
     * Verification method to check if array is properly sorted
     */
    public boolean isSorted(int[] nums) {
        boolean foundOne = false, foundTwo = false;
        
        for (int num : nums) {
            if (num == 1) foundOne = true;
            if (num == 2) foundTwo = true;
            
            // If we find 1 after finding 2, array is not sorted
            if (foundTwo && num == 1) return false;
            // If we find 0 after finding 1, array is not sorted
            if (foundOne && num == 0) return false;
        }
        
        return true;
    }
    
    /**
     * Generates random test array with 0s, 1s, and 2s
     */
    public int[] generateTestArray(int size) {
        Random random = new Random();
        int[] nums = new int[size];
        
        for (int i = 0; i < size; i++) {
            nums[i] = random.nextInt(3);  // 0, 1, or 2
        }
        
        return nums;
    }
    
    // Test method
    public static void main(String[] args) {
        SortColors solution = new SortColors();
        
        // Test case 1: Basic example
        int[] nums1 = {2, 0, 2, 1, 1, 0};
        System.out.println("Test Case 1:");
        System.out.println("Before: " + Arrays.toString(nums1));
        solution.sortColors(nums1.clone());
        System.out.println("After:  " + Arrays.toString(nums1));
        System.out.println("Is sorted: " + solution.isSorted(nums1));
        System.out.println();
        
        // Test case 2: With step tracking
        int[] nums2 = {2, 0, 1, 2, 1, 0};
        System.out.println("Test Case 2 with steps:");
        solution.sortColorsWithSteps(nums2);
        System.out.println();
        
        // Test case 3: Edge cases
        int[] nums3 = {0};
        System.out.println("Test Case 3 (single element): " + Arrays.toString(nums3));
        solution.sortColors(nums3);
        System.out.println("After: " + Arrays.toString(nums3));
        
        int[] nums4 = {1, 1, 1};
        System.out.println("Test Case 4 (all same): " + Arrays.toString(nums4));
        solution.sortColors(nums4);
        System.out.println("After: " + Arrays.toString(nums4));
        System.out.println();
        
        // Test case 5: Comparison with counting sort
        int[] nums5 = solution.generateTestArray(10);
        int[] nums5Copy = nums5.clone();
        
        System.out.println("Test Case 5 (random array): " + Arrays.toString(nums5));
        
        solution.sortColors(nums5);
        System.out.println("Dutch flag result: " + Arrays.toString(nums5));
        
        solution.sortColorsCountingSort(nums5Copy);
        System.out.println("Counting sort result: " + Arrays.toString(nums5Copy));
        
        System.out.println("Results match: " + Arrays.equals(nums5, nums5Copy));
        
        // Performance comparison
        System.out.println("\nPerformance comparison:");
        int[] largeArray1 = solution.generateTestArray(1000000);
        int[] largeArray2 = largeArray1.clone();
        
        long start = System.nanoTime();
        solution.sortColors(largeArray1);
        long time1 = System.nanoTime() - start;
        
        start = System.nanoTime();
        solution.sortColorsCountingSort(largeArray2);
        long time2 = System.nanoTime() - start;
        
        System.out.printf("Dutch flag approach: %.2f ms%n", time1 / 1_000_000.0);
        System.out.printf("Counting sort approach: %.2f ms%n", time2 / 1_000_000.0);
        System.out.println("Results match: " + Arrays.equals(largeArray1, largeArray2));
    }
}
```

**Key Insights:**
- **Three-Way Partitioning**: Divide array into three regions (0s, 1s, 2s)
- **Pointer Management**: Left and right maintain boundaries, current explores unknown region
- **Swap Strategy**: Different handling for different values (increment current only for 0 and 1)

---

## Advanced Patterns and Variations

### Multiple Pointer Techniques

#### **Four Sum Problem**
```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    
    for (int i = 0; i < nums.length - 3; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;
        
        for (int j = i + 1; j < nums.length - 2; j++) {
            if (j > i + 1 && nums[j] == nums[j-1]) continue;
            
            int left = j + 1, right = nums.length - 1;
            long sum = (long)nums[i] + nums[j] + nums[left] + nums[right];
            
            while (left < right) {
                if (sum == target) {
                    result.add(Arrays.asList(nums[i], nums[j], nums[left], nums[right]));
                    while (left < right && nums[left] == nums[left+1]) left++;
                    while (left < right && nums[right] == nums[right-1]) right--;
                    left++; right--;
                } else if (sum < target) {
                    left++;
                } else {
                    right--;
                }
            }
        }
    }
    return result;
}
```

### Linked List Advanced Patterns

#### **Remove Nth Node from End**
```java
public ListNode removeNthFromEnd(ListNode head, int n) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    
    ListNode slow = dummy;
    ListNode fast = dummy;
    
    // Move fast pointer n+1 steps ahead
    for (int i = 0; i <= n; i++) {
        fast = fast.next;
    }
    
    // Move both pointers until fast reaches end
    while (fast != null) {
        slow = slow.next;
        fast = fast.next;
    }
    
    // Remove the nth node
    slow.next = slow.next.next;
    return dummy.next;
}
```

---

## Common Mistakes and Tips

### ❌ Common Mistakes

#### **1. Incorrect Pointer Movement**
```java
// WRONG: Both pointers moving in same direction
while (left < right) {
    if (arr[left] + arr[right] < target) {
        left++; right++;  // ❌ Both moving right
    }
}

// CORRECT: Opposite movement for sorted array
while (left < right) {
    if (arr[left] + arr[right] < target) {
        left++;          // ✅ Only left moves right
    } else {
        right--;         // ✅ Only right moves left
    }
}
```

#### **2. Off-by-One Errors**
```java
// WRONG: Missing boundary check
while (left <= right) {    // ❌ Should be < for most cases
    // Process elements
}

// CORRECT: Proper boundary
while (left < right) {     // ✅ Correct for most two-pointer problems
    // Process elements
}
```

#### **3. Forgetting Duplicate Handling**
```java
// WRONG: Will include duplicate triplets
for (int i = 0; i < nums.length; i++) {
    // Process triplets without skipping duplicates
}

// CORRECT: Skip duplicates
for (int i = 0; i < nums.length; i++) {
    if (i > 0 && nums[i] == nums[i-1]) continue;  // ✅ Skip duplicates
    // Process triplets
}
```

### ✅ Best Practices and Tips

#### **1. Always Consider Edge Cases**
- Empty arrays/strings
- Single element
- All elements same
- No solution exists

#### **2. Understand When to Move Pointers**
- **Sorted arrays**: Move based on comparison with target
- **Palindromes**: Always move both pointers inward
- **Sliding window**: Expand right, contract left when condition violated

#### **3. Choose Right Data Structures**
- **HashMap**: For character/element frequency tracking
- **HashSet**: For checking existence
- **Array**: For ASCII characters (faster than HashMap)

#### **4. Optimization Techniques**
- **Early termination**: Break when no solution possible
- **Skip duplicates**: Avoid redundant computations
- **Use appropriate data types**: Arrays vs HashMap based on character set

#### **5. Testing Strategy**
- Test with sorted and unsorted inputs
- Test edge cases (empty, single element)
- Test with duplicates
- Test with no valid solution

---

## Summary

The **Two Pointer** pattern is one of the most versatile and efficient algorithmic techniques in competitive programming and technical interviews. Key takeaways:

### **When to Use**
- Sorted arrays with pair/triplet sum problems
- Palindrome-related problems  
- Sliding window problems
- Linked list cycle detection
- In-place array modifications

### **Time Complexity Benefits**
- Reduces O(n²) brute force to O(n) in most cases
- Single pass through data structure
- Constant space complexity typically

### **Key Variations**
1. **Opposite Direction**: Start from ends, move toward center
2. **Same Direction**: Fast and slow pointers
3. **Sliding Window**: Dynamic window size based on conditions

### **Success Factors**
- Understand the problem pattern
- Choose correct pointer movement strategy
- Handle edge cases and duplicates
- Use appropriate data structures

Practice these 10 problems thoroughly to master the Two Pointer pattern and apply it confidently in coding interviews and competitive programming!
