# Sliding Window Technique: Comprehensive Guide

This guide covers the sliding window technique, a powerful algorithmic approach for solving array and string problems efficiently. It includes concepts, patterns, practical Java examples, and optimization strategies.

## Table of Contents
1. [Core Concepts](#1-core-concepts)
2. [When to Use Sliding Window](#2-when-to-use-sliding-window)
3. [Types of Sliding Window](#3-types-of-sliding-window)
4. [Java Examples with Explanations](#4-java-examples-with-explanations)
5. [Pattern Recognition](#5-pattern-recognition)
6. [Complexity Analysis](#6-complexity-analysis)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Advanced Techniques](#8-advanced-techniques)

---

## 1) Core Concepts

### 1.1 What is Sliding Window?

The sliding window technique is an algorithmic approach that maintains a subset of data (window) that slides through a larger dataset to solve problems efficiently.

```
Sliding Window Visualization:
Array: [1, 2, 3, 4, 5, 6, 7, 8]
       
Window Size = 3:
Step 1: [1, 2, 3] 4, 5, 6, 7, 8    (window at start)
Step 2:  1 [2, 3, 4] 5, 6, 7, 8    (slide right)
Step 3:  1, 2 [3, 4, 5] 6, 7, 8    (slide right)
Step 4:  1, 2, 3 [4, 5, 6] 7, 8    (slide right)
Step 5:  1, 2, 3, 4 [5, 6, 7] 8    (slide right)
Step 6:  1, 2, 3, 4, 5 [6, 7, 8]   (slide right)
```

```
Window Operations:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         Sliding Window Process                                 │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Expand    │───▶│   Process   │───▶│  Contract   │───▶│   Repeat    │     │
│  │   Window    │    │   Current   │    │   Window    │    │   Process   │     │
│  │             │    │   Window    │    │  (if needed) │    │             │     │
│  └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘     │
│        │                   │                   │                   │           │
│        ▼                   ▼                   ▼                   ▼           │
│  • Move right        • Calculate         • Move left        • Continue until   │
│    pointer             result             pointer           end of array      │
│  • Add element       • Update optimal    • Remove element                     │
│    to window           solution           from window                         │
│                                                                               │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Key Components

**Window Boundaries:**
- **Left Pointer**: Start of the current window
- **Right Pointer**: End of the current window
- **Window Size**: Can be fixed or variable

**Window State:**
- **Current Elements**: Elements within the window
- **Window Sum/Product**: Aggregate value of window elements
- **Frequency Map**: Count of elements in window

---

## 2) When to Use Sliding Window

```
Problem Pattern Recognition:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    Sliding Window Use Cases                                     │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────────┤
│  Problem Type   │   Input Type    │  Window Type    │        Examples             │
├─────────────────┼─────────────────┼─────────────────┼─────────────────────────────┤
│ Subarray Sum    │ Array/String    │ Variable Size   │ • Sum equals target         │
│                 │                 │                 │ • Maximum sum               │
│                 │                 │                 │ • Minimum length            │
├─────────────────┼─────────────────┼─────────────────┼─────────────────────────────┤
│ Substring       │ String          │ Variable Size   │ • Longest substring         │
│ Problems        │                 │                 │ • Without repeating chars   │
│                 │                 │                 │ • With k distinct chars     │
├─────────────────┼─────────────────┼─────────────────┼─────────────────────────────┤
│ Fixed Window    │ Array           │ Fixed Size      │ • Maximum in window         │
│ Analysis        │                 │                 │ • Average calculation       │
│                 │                 │                 │ • Pattern matching          │
├─────────────────┼─────────────────┼─────────────────┼─────────────────────────────┤
│ Optimization    │ Both            │ Both            │ • Replace brute force       │
│ Problems        │                 │                 │ • O(n²) to O(n)             │
│                 │                 │                 │ • Reduce time complexity    │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────────┘
```

### Use Sliding Window When:

✅ **Perfect Fit:**
- Finding subarrays/substrings with specific properties
- Optimizing O(n²) brute force solutions
- Dealing with contiguous elements
- Maintaining running calculations

❌ **Not Suitable:**
- Non-contiguous subsequences
- Problems requiring all possible combinations
- Tree or graph traversal
- Random access patterns

---

## 3) Types of Sliding Window

```
Sliding Window Types:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│  ┌─────────────────────────┐           ┌─────────────────────────┐             │
│  │     Fixed Size Window   │           │   Variable Size Window  │             │
│  │                         │           │                         │             │
│  │  • Window size constant │           │  • Window size changes  │             │
│  │  • Both pointers move   │           │  • Expand/contract      │             │
│  │    together             │           │  • Two pointer approach │             │
│  │                         │           │                         │             │
│  │  Examples:              │           │  Examples:              │             │
│  │  • Max sum of k elements│           │  • Longest substring    │             │
│  │  • Average in window    │           │  • Minimum window       │             │
│  │  • First negative in    │           │  • Sum equals target    │             │
│  │    each window          │           │                         │             │
│  └─────────────────────────┘           └─────────────────────────┘             │
│              │                                     │                           │
│              └─────────────┬───────────────────────┘                           │
│                            │                                                   │
│                            ▼                                                   │
│              ┌─────────────────────────┐                                       │
│              │  Implementation Pattern │                                       │
│              │                         │                                       │
│              │  1. Initialize window   │                                       │
│              │  2. Process first window│                                       │
│              │  3. Slide window        │                                       │
│              │  4. Update result       │                                       │
│              │  5. Repeat until end    │                                       │
│              └─────────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4) Java Examples with Explanations

### Example 1: Maximum Sum of K Elements (Fixed Window)

```java
import java.util.*;

/**
 * Problem: Find the maximum sum of any subarray of size k
 * 
 * Approach: Fixed-size sliding window
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
public class MaxSumKElements {
    
    /**
     * Finds maximum sum of k consecutive elements using sliding window
     * 
     * @param arr The input array
     * @param k The window size
     * @return Maximum sum of k consecutive elements
     */
    public static int maxSumKElements(int[] arr, int k) {
        // Validate input
        if (arr == null || arr.length < k || k <= 0) {
            throw new IllegalArgumentException("Invalid input parameters");
        }
        
        // Step 1: Calculate sum of first window
        int windowSum = 0;
        for (int i = 0; i < k; i++) {
            windowSum += arr[i];
        }
        
        // Initialize maximum sum with first window
        int maxSum = windowSum;
        
        // Step 2: Slide the window from left to right
        for (int i = k; i < arr.length; i++) {
            // Slide window: remove leftmost element, add rightmost element
            windowSum = windowSum - arr[i - k] + arr[i];
            
            // Update maximum sum if current window sum is greater
            maxSum = Math.max(maxSum, windowSum);
        }
        
        return maxSum;
    }
    
    /**
     * Alternative implementation with explicit window tracking
     */
    public static int maxSumKElementsExplicit(int[] arr, int k) {
        if (arr == null || arr.length < k || k <= 0) {
            return 0;
        }
        
        int left = 0, right = 0;
        int currentSum = 0, maxSum = Integer.MIN_VALUE;
        
        // Expand window to size k
        while (right < arr.length) {
            // Add current element to window
            currentSum += arr[right];
            
            // If window size equals k
            if (right - left + 1 == k) {
                // Update maximum sum
                maxSum = Math.max(maxSum, currentSum);
                
                // Slide window: remove leftmost element
                currentSum -= arr[left];
                left++;
            }
            
            right++;
        }
        
        return maxSum;
    }
    
    public static void main(String[] args) {
        // Test case 1: Normal case
        int[] arr1 = {2, 1, 5, 1, 3, 2};
        int k1 = 3;
        System.out.println("Array: " + Arrays.toString(arr1));
        System.out.println("k = " + k1);
        System.out.println("Maximum sum: " + maxSumKElements(arr1, k1)); // Output: 9
        
        // Explanation: Windows are [2,1,5]=8, [1,5,1]=7, [5,1,3]=9, [1,3,2]=6
        // Maximum is 9
        
        // Test case 2: All negative numbers
        int[] arr2 = {-1, -2, -3, -4, -5};
        int k2 = 2;
        System.out.println("\nArray: " + Arrays.toString(arr2));
        System.out.println("k = " + k2);
        System.out.println("Maximum sum: " + maxSumKElements(arr2, k2)); // Output: -3
    }
}
```

### Example 2: Longest Substring Without Repeating Characters (Variable Window)

```java
import java.util.*;

/**
 * Problem: Find the length of the longest substring without repeating characters
 * 
 * Approach: Variable-size sliding window with HashSet
 * Time Complexity: O(n)
 * Space Complexity: O(min(m,n)) where m is character set size
 */
public class LongestSubstringWithoutRepeating {
    
    /**
     * Finds length of longest substring without repeating characters
     * 
     * @param s Input string
     * @return Length of longest substring without repeating characters
     */
    public static int lengthOfLongestSubstring(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        // Set to track characters in current window
        Set<Character> windowChars = new HashSet<>();
        
        int left = 0, right = 0;
        int maxLength = 0;
        
        // Expand window with right pointer
        while (right < s.length()) {
            char rightChar = s.charAt(right);
            
            // If character is already in window, contract from left
            while (windowChars.contains(rightChar)) {
                char leftChar = s.charAt(left);
                windowChars.remove(leftChar);
                left++;
            }
            
            // Add current character to window
            windowChars.add(rightChar);
            
            // Update maximum length
            maxLength = Math.max(maxLength, right - left + 1);
            
            // Expand window
            right++;
        }
        
        return maxLength;
    }
    
    /**
     * Optimized version using HashMap to store character indices
     */
    public static int lengthOfLongestSubstringOptimized(String s) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        // Map to store character and its latest index
        Map<Character, Integer> charIndexMap = new HashMap<>();
        
        int left = 0;
        int maxLength = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char currentChar = s.charAt(right);
            
            // If character is found and is within current window
            if (charIndexMap.containsKey(currentChar) && 
                charIndexMap.get(currentChar) >= left) {
                // Move left pointer to next position after duplicate
                left = charIndexMap.get(currentChar) + 1;
            }
            
            // Update character's latest index
            charIndexMap.put(currentChar, right);
            
            // Update maximum length
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
    
    /**
     * Helper method to visualize the sliding window process
     */
    public static void visualizeProcess(String s) {
        System.out.println("Visualizing process for string: \"" + s + "\"");
        
        Set<Character> window = new HashSet<>();
        int left = 0, maxLen = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            
            // Contract window if duplicate found
            while (window.contains(rightChar)) {
                char leftChar = s.charAt(left);
                window.remove(leftChar);
                left++;
            }
            
            window.add(rightChar);
            maxLen = Math.max(maxLen, right - left + 1);
            
            // Print current window state
            System.out.printf("Step %d: Window [%d,%d] = \"%s\", Length = %d%n", 
                right + 1, left, right, 
                s.substring(left, right + 1), 
                right - left + 1);
        }
        
        System.out.println("Maximum length: " + maxLen);
    }
    
    public static void main(String[] args) {
        // Test case 1: String with repeating characters
        String s1 = "abcabcbb";
        System.out.println("Input: \"" + s1 + "\"");
        System.out.println("Result: " + lengthOfLongestSubstring(s1)); // Output: 3
        System.out.println("Explanation: \"abc\" is the longest substring\n");
        
        // Test case 2: All same characters
        String s2 = "bbbbb";
        System.out.println("Input: \"" + s2 + "\"");
        System.out.println("Result: " + lengthOfLongestSubstring(s2)); // Output: 1
        
        // Test case 3: No repeating characters
        String s3 = "pwwkew";
        System.out.println("\nInput: \"" + s3 + "\"");
        visualizeProcess(s3);
    }
}
```

### Example 3: Minimum Window Substring (Variable Window)

```java
import java.util.*;

/**
 * Problem: Find the minimum window substring that contains all characters of target string
 * 
 * Approach: Variable-size sliding window with frequency counting
 * Time Complexity: O(|s| + |t|)
 * Space Complexity: O(|s| + |t|)
 */
public class MinimumWindowSubstring {
    
    /**
     * Finds minimum window substring containing all characters of target
     * 
     * @param s Source string
     * @param t Target string
     * @return Minimum window substring or empty string if not found
     */
    public static String minWindow(String s, String t) {
        if (s == null || t == null || s.length() < t.length()) {
            return "";
        }
        
        // Frequency map for target string
        Map<Character, Integer> targetFreq = new HashMap<>();
        for (char c : t.toCharArray()) {
            targetFreq.put(c, targetFreq.getOrDefault(c, 0) + 1);
        }
        
        // Frequency map for current window
        Map<Character, Integer> windowFreq = new HashMap<>();
        
        int left = 0, right = 0;
        int required = targetFreq.size(); // Number of unique characters in t
        int formed = 0; // Number of unique characters in window with desired frequency
        
        // Result tracking
        int[] result = {-1, 0, 0}; // {window length, left, right}
        
        while (right < s.length()) {
            // Expand window by including character at right
            char rightChar = s.charAt(right);
            windowFreq.put(rightChar, windowFreq.getOrDefault(rightChar, 0) + 1);
            
            // Check if frequency of current character matches desired count in t
            if (targetFreq.containsKey(rightChar) && 
                windowFreq.get(rightChar).intValue() == targetFreq.get(rightChar).intValue()) {
                formed++;
            }
            
            // Contract window from left if all characters are satisfied
            while (left <= right && formed == required) {
                // Update result if current window is smaller
                if (result[0] == -1 || right - left + 1 < result[0]) {
                    result[0] = right - left + 1;
                    result[1] = left;
                    result[2] = right;
                }
                
                // Remove character at left from window
                char leftChar = s.charAt(left);
                windowFreq.put(leftChar, windowFreq.get(leftChar) - 1);
                
                if (targetFreq.containsKey(leftChar) && 
                    windowFreq.get(leftChar) < targetFreq.get(leftChar)) {
                    formed--;
                }
                
                left++;
            }
            
            right++;
        }
        
        return result[0] == -1 ? "" : s.substring(result[1], result[2] + 1);
    }
    
    /**
     * Alternative implementation with detailed tracking
     */
    public static String minWindowDetailed(String s, String t) {
        if (s.length() < t.length()) return "";
        
        // Count characters in target
        int[] targetCount = new int[128]; // ASCII character set
        int uniqueChars = 0;
        
        for (char c : t.toCharArray()) {
            if (targetCount[c] == 0) uniqueChars++;
            targetCount[c]++;
        }
        
        int[] windowCount = new int[128];
        int left = 0, right = 0;
        int matched = 0;
        
        String result = "";
        int minLen = Integer.MAX_VALUE;
        
        while (right < s.length()) {
            // Expand window
            char rightChar = s.charAt(right);
            windowCount[rightChar]++;
            
            if (targetCount[rightChar] != 0 && 
                windowCount[rightChar] == targetCount[rightChar]) {
                matched++;
            }
            
            // Contract window when all characters are matched
            while (matched == uniqueChars) {
                // Update result if current window is smaller
                if (right - left + 1 < minLen) {
                    minLen = right - left + 1;
                    result = s.substring(left, right + 1);
                }
                
                // Remove leftmost character
                char leftChar = s.charAt(left);
                windowCount[leftChar]--;
                
                if (targetCount[leftChar] != 0 && 
                    windowCount[leftChar] < targetCount[leftChar]) {
                    matched--;
                }
                
                left++;
            }
            
            right++;
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        // Test case 1: Normal case
        String s1 = "ADOBECODEBANC";
        String t1 = "ABC";
        System.out.println("Source: \"" + s1 + "\"");
        System.out.println("Target: \"" + t1 + "\"");
        System.out.println("Minimum window: \"" + minWindow(s1, t1) + "\""); // Output: "BANC"
        
        // Test case 2: Target not in source
        String s2 = "a";
        String t2 = "aa";
        System.out.println("\nSource: \"" + s2 + "\"");
        System.out.println("Target: \"" + t2 + "\"");
        System.out.println("Minimum window: \"" + minWindow(s2, t2) + "\""); // Output: ""
        
        // Test case 3: All characters same
        String s3 = "ab";
        String t3 = "b";
        System.out.println("\nSource: \"" + s3 + "\"");
        System.out.println("Target: \"" + t3 + "\"");
        System.out.println("Minimum window: \"" + minWindow(s3, t3) + "\""); // Output: "b"
    }
}
```

### Example 4: Longest Substring with At Most K Distinct Characters (Variable Window)

```java
import java.util.*;

/**
 * Problem: Find the longest substring with at most k distinct characters
 * 
 * Approach: Variable-size sliding window with frequency map
 * Time Complexity: O(n)
 * Space Complexity: O(k) for the frequency map
 */
public class LongestSubstringKDistinct {
    
    /**
     * Finds length of longest substring with at most k distinct characters
     * 
     * @param s Input string
     * @param k Maximum number of distinct characters allowed
     * @return Length of longest valid substring
     */
    public static int lengthOfLongestSubstringKDistinct(String s, int k) {
        if (s == null || s.length() == 0 || k <= 0) {
            return 0;
        }
        
        // Map to store character frequencies in current window
        Map<Character, Integer> charFrequency = new HashMap<>();
        
        int left = 0, right = 0;
        int maxLength = 0;
        
        while (right < s.length()) {
            // Expand window: add character at right
            char rightChar = s.charAt(right);
            charFrequency.put(rightChar, charFrequency.getOrDefault(rightChar, 0) + 1);
            
            // Contract window if we have more than k distinct characters
            while (charFrequency.size() > k) {
                char leftChar = s.charAt(left);
                charFrequency.put(leftChar, charFrequency.get(leftChar) - 1);
                
                // Remove character from map if frequency becomes 0
                if (charFrequency.get(leftChar) == 0) {
                    charFrequency.remove(leftChar);
                }
                
                left++;
            }
            
            // Update maximum length with current valid window
            maxLength = Math.max(maxLength, right - left + 1);
            right++;
        }
        
        return maxLength;
    }
    
    /**
     * Returns the actual longest substring with at most k distinct characters
     */
    public static String longestSubstringKDistinct(String s, int k) {
        if (s == null || s.length() == 0 || k <= 0) {
            return "";
        }
        
        Map<Character, Integer> charFreq = new HashMap<>();
        int left = 0, maxLength = 0;
        String result = "";
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            charFreq.put(rightChar, charFreq.getOrDefault(rightChar, 0) + 1);
            
            // Contract window if more than k distinct characters
            while (charFreq.size() > k) {
                char leftChar = s.charAt(left);
                charFreq.put(leftChar, charFreq.get(leftChar) - 1);
                if (charFreq.get(leftChar) == 0) {
                    charFreq.remove(leftChar);
                }
                left++;
            }
            
            // Update result if current window is longer
            if (right - left + 1 > maxLength) {
                maxLength = right - left + 1;
                result = s.substring(left, right + 1);
            }
        }
        
        return result;
    }
    
    /**
     * Optimized version using array for character counting (for ASCII)
     */
    public static int lengthOfLongestSubstringKDistinctOptimized(String s, int k) {
        if (s == null || s.length() == 0 || k <= 0) {
            return 0;
        }
        
        int[] charCount = new int[256]; // For extended ASCII
        int distinctChars = 0;
        int left = 0, maxLength = 0;
        
        for (int right = 0; right < s.length(); right++) {
            // Add character to window
            if (charCount[s.charAt(right)] == 0) {
                distinctChars++;
            }
            charCount[s.charAt(right)]++;
            
            // Contract window if too many distinct characters
            while (distinctChars > k) {
                charCount[s.charAt(left)]--;
                if (charCount[s.charAt(left)] == 0) {
                    distinctChars--;
                }
                left++;
            }
            
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
    
    /**
     * Helper method to demonstrate the sliding window process
     */
    public static void demonstrateProcess(String s, int k) {
        System.out.println("Demonstrating process for string: \"" + s + "\", k = " + k);
        
        Map<Character, Integer> freq = new HashMap<>();
        int left = 0, maxLen = 0;
        String bestSubstring = "";
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            freq.put(rightChar, freq.getOrDefault(rightChar, 0) + 1);
            
            while (freq.size() > k) {
                char leftChar = s.charAt(left);
                freq.put(leftChar, freq.get(leftChar) - 1);
                if (freq.get(leftChar) == 0) {
                    freq.remove(leftChar);
                }
                left++;
            }
            
            String currentSubstring = s.substring(left, right + 1);
            System.out.printf("Step %d: Window [%d,%d] = \"%s\", Distinct: %d, Length: %d%n",
                right + 1, left, right, currentSubstring, freq.size(), currentSubstring.length());
            
            if (currentSubstring.length() > maxLen) {
                maxLen = currentSubstring.length();
                bestSubstring = currentSubstring;
            }
        }
        
        System.out.println("Best substring: \"" + bestSubstring + "\" with length: " + maxLen);
    }
    
    public static void main(String[] args) {
        // Test case 1: Normal case
        String s1 = "eceba";
        int k1 = 2;
        System.out.println("Input: \"" + s1 + "\", k = " + k1);
        System.out.println("Length: " + lengthOfLongestSubstringKDistinct(s1, k1)); // Output: 3
        System.out.println("Substring: \"" + longestSubstringKDistinct(s1, k1) + "\""); // Output: "ece"
        
        // Test case 2: All characters distinct
        String s2 = "aa";
        int k2 = 1;
        System.out.println("\nInput: \"" + s2 + "\", k = " + k2);
        System.out.println("Length: " + lengthOfLongestSubstringKDistinct(s2, k2)); // Output: 2
        
        // Test case 3: Demonstration
        System.out.println();
        demonstrateProcess("araaci", 2);
    }
}
```

### Example 5: Find All Anagrams in String (Fixed Window)

```java
import java.util.*;

/**
 * Problem: Find all start indices of anagrams of string p in string s
 * 
 * Approach: Fixed-size sliding window with frequency comparison
 * Time Complexity: O(|s| + |p|)
 * Space Complexity: O(1) - constant space for character frequency arrays
 */
public class FindAnagrams {
    
    /**
     * Finds all starting indices where anagrams of p appear in s
     * 
     * @param s Source string
     * @param p Pattern string
     * @return List of starting indices of anagrams
     */
    public static List<Integer> findAnagrams(String s, String p) {
        List<Integer> result = new ArrayList<>();
        
        if (s == null || p == null || s.length() < p.length()) {
            return result;
        }
        
        // Frequency arrays for pattern and current window
        int[] pFreq = new int[26]; // For lowercase letters a-z
        int[] windowFreq = new int[26];
        
        // Calculate frequency of characters in pattern
        for (char c : p.toCharArray()) {
            pFreq[c - 'a']++;
        }
        
        int windowSize = p.length();
        
        // Process first window
        for (int i = 0; i < windowSize && i < s.length(); i++) {
            windowFreq[s.charAt(i) - 'a']++;
        }
        
        // Check if first window is an anagram
        if (Arrays.equals(pFreq, windowFreq)) {
            result.add(0);
        }
        
        // Slide window through rest of string
        for (int i = windowSize; i < s.length(); i++) {
            // Add new character (right side of window)
            windowFreq[s.charAt(i) - 'a']++;
            
            // Remove old character (left side of window)
            windowFreq[s.charAt(i - windowSize) - 'a']--;
            
            // Check if current window is an anagram
            if (Arrays.equals(pFreq, windowFreq)) {
                result.add(i - windowSize + 1);
            }
        }
        
        return result;
    }
    
    /**
     * Alternative implementation using HashMap for more flexibility
     */
    public static List<Integer> findAnagramsHashMap(String s, String p) {
        List<Integer> result = new ArrayList<>();
        
        if (s == null || p == null || s.length() < p.length()) {
            return result;
        }
        
        // Create frequency map for pattern
        Map<Character, Integer> pFreq = new HashMap<>();
        for (char c : p.toCharArray()) {
            pFreq.put(c, pFreq.getOrDefault(c, 0) + 1);
        }
        
        Map<Character, Integer> windowFreq = new HashMap<>();
        int left = 0, right = 0;
        int matches = 0; // Number of characters with correct frequency
        
        while (right < s.length()) {
            // Expand window
            char rightChar = s.charAt(right);
            windowFreq.put(rightChar, windowFreq.getOrDefault(rightChar, 0) + 1);
            
            // Check if right character frequency matches
            if (pFreq.containsKey(rightChar) && 
                windowFreq.get(rightChar).equals(pFreq.get(rightChar))) {
                matches++;
            }
            
            // Contract window if size exceeds pattern length
            if (right - left + 1 > p.length()) {
                char leftChar = s.charAt(left);
                
                if (pFreq.containsKey(leftChar) && 
                    windowFreq.get(leftChar).equals(pFreq.get(leftChar))) {
                    matches--;
                }
                
                windowFreq.put(leftChar, windowFreq.get(leftChar) - 1);
                if (windowFreq.get(leftChar) == 0) {
                    windowFreq.remove(leftChar);
                }
                left++;
            }
            
            // Check if current window is an anagram
            if (matches == pFreq.size()) {
                result.add(left);
            }
            
            right++;
        }
        
        return result;
    }
    
    /**
     * Helper method to verify if two strings are anagrams
     */
    private static boolean areAnagrams(String s1, String s2) {
        if (s1.length() != s2.length()) return false;
        
        int[] freq = new int[26];
        for (int i = 0; i < s1.length(); i++) {
            freq[s1.charAt(i) - 'a']++;
            freq[s2.charAt(i) - 'a']--;
        }
        
        for (int count : freq) {
            if (count != 0) return false;
        }
        
        return true;
    }
    
    /**
     * Demonstrates the sliding window process step by step
     */
    public static void demonstrateProcess(String s, String p) {
        System.out.println("Finding anagrams of \"" + p + "\" in \"" + s + "\"");
        
        if (s.length() < p.length()) {
            System.out.println("Source string too short");
            return;
        }
        
        int windowSize = p.length();
        
        for (int i = 0; i <= s.length() - windowSize; i++) {
            String window = s.substring(i, i + windowSize);
            boolean isAnagram = areAnagrams(window, p);
            
            System.out.printf("Position %d: \"%s\" -> %s%n", 
                i, window, isAnagram ? "✓ Anagram" : "✗ Not anagram");
        }
    }
    
    public static void main(String[] args) {
        // Test case 1: Multiple anagrams
        String s1 = "cbaebabacd";
        String p1 = "abc";
        System.out.println("Input: s = \"" + s1 + "\", p = \"" + p1 + "\"");
        System.out.println("Anagram indices: " + findAnagrams(s1, p1)); // Output: [1, 6]
        System.out.println();
        
        // Test case 2: No anagrams
        String s2 = "abab";
        String p2 = "ab";
        System.out.println("Input: s = \"" + s2 + "\", p = \"" + p2 + "\"");
        System.out.println("Anagram indices: " + findAnagrams(s2, p2)); // Output: [0, 2]
        System.out.println();
        
        // Demonstration
        demonstrateProcess(s1, p1);
    }
}
```

### Example 6: Subarray Sum Equals K (Variable Window with Prefix Sum)

```java
import java.util.*;

/**
 * Problem: Find number of subarrays with sum equal to k
 * 
 * Approach: Prefix sum with HashMap (not traditional sliding window)
 * Note: This problem requires prefix sum approach because we need to handle negative numbers
 * Time Complexity: O(n)
 * Space Complexity: O(n)
 */
public class SubarraySumEqualsK {
    
    /**
     * Counts number of subarrays with sum equal to k
     * Uses prefix sum technique with HashMap
     * 
     * @param nums Input array
     * @param k Target sum
     * @return Number of subarrays with sum k
     */
    public static int subarraySum(int[] nums, int k) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        
        // Map to store prefix sum and its frequency
        Map<Integer, Integer> prefixSumCount = new HashMap<>();
        prefixSumCount.put(0, 1); // Empty prefix sum
        
        int prefixSum = 0;
        int count = 0;
        
        for (int num : nums) {
            // Calculate current prefix sum
            prefixSum += num;
            
            // Check if (prefixSum - k) exists
            // If exists, it means there's a subarray ending at current index with sum k
            if (prefixSumCount.containsKey(prefixSum - k)) {
                count += prefixSumCount.get(prefixSum - k);
            }
            
            // Add current prefix sum to map
            prefixSumCount.put(prefixSum, prefixSumCount.getOrDefault(prefixSum, 0) + 1);
        }
        
        return count;
    }
    
    /**
     * Alternative approach for positive numbers only using sliding window
     * Note: This only works when all numbers are positive
     */
    public static int subarraySumPositiveNumbers(int[] nums, int k) {
        if (nums == null || nums.length == 0) {
            return 0;
        }
        
        int left = 0, right = 0;
        int currentSum = 0;
        int count = 0;
        
        while (right < nums.length) {
            // Expand window
            currentSum += nums[right];
            
            // Contract window while sum is greater than k
            while (currentSum > k && left <= right) {
                currentSum -= nums[left];
                left++;
            }
            
            // If sum equals k, increment count
            if (currentSum == k) {
                count++;
                // Continue to find more subarrays
                currentSum -= nums[left];
                left++;
            }
            
            right++;
        }
        
        return count;
    }
    
    /**
     * Finds all subarrays with sum equal to k (for demonstration)
     */
    public static List<List<Integer>> findSubarraysWithSumK(int[] nums, int k) {
        List<List<Integer>> result = new ArrayList<>();
        
        for (int i = 0; i < nums.length; i++) {
            int sum = 0;
            for (int j = i; j < nums.length; j++) {
                sum += nums[j];
                if (sum == k) {
                    List<Integer> subarray = new ArrayList<>();
                    for (int idx = i; idx <= j; idx++) {
                        subarray.add(nums[idx]);
                    }
                    result.add(subarray);
                }
            }
        }
        
        return result;
    }
    
    /**
     * Demonstrates the prefix sum approach step by step
     */
    public static void demonstratePrefixSum(int[] nums, int k) {
        System.out.println("Demonstrating prefix sum approach:");
        System.out.println("Array: " + Arrays.toString(nums) + ", k = " + k);
        
        Map<Integer, Integer> prefixSumCount = new HashMap<>();
        prefixSumCount.put(0, 1);
        
        int prefixSum = 0;
        int totalCount = 0;
        
        for (int i = 0; i < nums.length; i++) {
            prefixSum += nums[i];
            
            int needed = prefixSum - k;
            int count = prefixSumCount.getOrDefault(needed, 0);
            totalCount += count;
            
            System.out.printf("Index %d: num=%d, prefixSum=%d, needed=%d, found=%d, total=%d%n",
                i, nums[i], prefixSum, needed, count, totalCount);
            
            prefixSumCount.put(prefixSum, prefixSumCount.getOrDefault(prefixSum, 0) + 1);
        }
        
        System.out.println("Final count: " + totalCount);
    }
    
    public static void main(String[] args) {
        // Test case 1: Array with positive and negative numbers
        int[] nums1 = {1, 1, 1};
        int k1 = 2;
        System.out.println("Array: " + Arrays.toString(nums1) + ", k = " + k1);
        System.out.println("Count: " + subarraySum(nums1, k1)); // Output: 2
        System.out.println("Subarrays: " + findSubarraysWithSumK(nums1, k1));
        System.out.println();
        
        // Test case 2: Array with negative numbers
        int[] nums2 = {1, -1, 0};
        int k2 = 0;
        System.out.println("Array: " + Arrays.toString(nums2) + ", k = " + k2);
        System.out.println("Count: " + subarraySum(nums2, k2)); // Output: 3
        System.out.println();
        
        // Demonstration
        demonstratePrefixSum(nums1, k1);
    }
}
```

### Example 7: Permutation in String (Fixed Window)

```java
import java.util.*;

/**
 * Problem: Check if any permutation of s1 exists as substring in s2
 * 
 * Approach: Fixed-size sliding window with frequency matching
 * Time Complexity: O(|s1| + |s2|)
 * Space Complexity: O(1) - constant space for character frequencies
 */
public class PermutationInString {
    
    /**
     * Checks if any permutation of s1 exists as substring in s2
     * 
     * @param s1 Pattern string
     * @param s2 Source string
     * @return true if permutation exists, false otherwise
     */
    public static boolean checkInclusion(String s1, String s2) {
        if (s1 == null || s2 == null || s1.length() > s2.length()) {
            return false;
        }
        
        // Frequency arrays for pattern and current window
        int[] s1Freq = new int[26];
        int[] windowFreq = new int[26];
        
        // Calculate frequency of characters in s1
        for (char c : s1.toCharArray()) {
            s1Freq[c - 'a']++;
        }
        
        int windowSize = s1.length();
        
        // Process first window
        for (int i = 0; i < windowSize; i++) {
            windowFreq[s2.charAt(i) - 'a']++;
        }
        
        // Check if first window matches
        if (Arrays.equals(s1Freq, windowFreq)) {
            return true;
        }
        
        // Slide window through rest of string
        for (int i = windowSize; i < s2.length(); i++) {
            // Add new character (expand right)
            windowFreq[s2.charAt(i) - 'a']++;
            
            // Remove old character (contract left)
            windowFreq[s2.charAt(i - windowSize) - 'a']--;
            
            // Check if current window matches
            if (Arrays.equals(s1Freq, windowFreq)) {
                return true;
            }
        }
        
        return false;
    }
    
    /**
     * Alternative implementation using match counting for optimization
     */
    public static boolean checkInclusionOptimized(String s1, String s2) {
        if (s1 == null || s2 == null || s1.length() > s2.length()) {
            return false;
        }
        
        int[] s1Freq = new int[26];
        int[] windowFreq = new int[26];
        
        // Calculate s1 frequency
        for (char c : s1.toCharArray()) {
            s1Freq[c - 'a']++;
        }
        
        int windowSize = s1.length();
        int matches = 0; // Number of characters with matching frequency
        
        // Process first window and count initial matches
        for (int i = 0; i < windowSize; i++) {
            int index = s2.charAt(i) - 'a';
            windowFreq[index]++;
            if (windowFreq[index] == s1Freq[index]) {
                matches++;
            } else if (windowFreq[index] == s1Freq[index] + 1) {
                matches--; // We had a match but now exceeded
            }
        }
        
        if (matches == 26) return true;
        
        // Slide window
        for (int i = windowSize; i < s2.length(); i++) {
            // Add new character
            int rightIndex = s2.charAt(i) - 'a';
            windowFreq[rightIndex]++;
            if (windowFreq[rightIndex] == s1Freq[rightIndex]) {
                matches++;
            } else if (windowFreq[rightIndex] == s1Freq[rightIndex] + 1) {
                matches--;
            }
            
            // Remove old character
            int leftIndex = s2.charAt(i - windowSize) - 'a';
            windowFreq[leftIndex]--;
            if (windowFreq[leftIndex] == s1Freq[leftIndex]) {
                matches++;
            } else if (windowFreq[leftIndex] == s1Freq[leftIndex] - 1) {
                matches--;
            }
            
            if (matches == 26) return true;
        }
        
        return false;
    }
    
    /**
     * Returns all starting indices where permutations of s1 appear in s2
     */
    public static List<Integer> findAllPermutations(String s1, String s2) {
        List<Integer> result = new ArrayList<>();
        
        if (s1 == null || s2 == null || s1.length() > s2.length()) {
            return result;
        }
        
        int[] s1Freq = new int[26];
        int[] windowFreq = new int[26];
        
        for (char c : s1.toCharArray()) {
            s1Freq[c - 'a']++;
        }
        
        int windowSize = s1.length();
        
        // Process first window
        for (int i = 0; i < windowSize; i++) {
            windowFreq[s2.charAt(i) - 'a']++;
        }
        
        if (Arrays.equals(s1Freq, windowFreq)) {
            result.add(0);
        }
        
        // Slide window
        for (int i = windowSize; i < s2.length(); i++) {
            windowFreq[s2.charAt(i) - 'a']++;
            windowFreq[s2.charAt(i - windowSize) - 'a']--;
            
            if (Arrays.equals(s1Freq, windowFreq)) {
                result.add(i - windowSize + 1);
            }
        }
        
        return result;
    }
    
    /**
     * Demonstrates the sliding window process
     */
    public static void demonstrateProcess(String s1, String s2) {
        System.out.println("Checking if permutation of \"" + s1 + "\" exists in \"" + s2 + "\"");
        
        if (s1.length() > s2.length()) {
            System.out.println("s1 is longer than s2 - no permutation possible");
            return;
        }
        
        int[] s1Freq = new int[26];
        for (char c : s1.toCharArray()) {
            s1Freq[c - 'a']++;
        }
        
        int windowSize = s1.length();
        
        for (int i = 0; i <= s2.length() - windowSize; i++) {
            String window = s2.substring(i, i + windowSize);
            
            int[] windowFreq = new int[26];
            for (char c : window.toCharArray()) {
                windowFreq[c - 'a']++;
            }
            
            boolean isPermutation = Arrays.equals(s1Freq, windowFreq);
            
            System.out.printf("Position %d: \"%s\" -> %s%n", 
                i, window, isPermutation ? "✓ Permutation" : "✗ Not permutation");
        }
    }
    
    public static void main(String[] args) {
        // Test case 1: Permutation exists
        String s1_1 = "ab";
        String s2_1 = "eidbaooo";
        System.out.println("s1 = \"" + s1_1 + "\", s2 = \"" + s2_1 + "\"");
        System.out.println("Result: " + checkInclusion(s1_1, s2_1)); // Output: true
        System.out.println();
        
        // Test case 2: No permutation
        String s1_2 = "ab";
        String s2_2 = "eidboaoo";
        System.out.println("s1 = \"" + s1_2 + "\", s2 = \"" + s2_2 + "\"");
        System.out.println("Result: " + checkInclusion(s1_2, s2_2)); // Output: false
        System.out.println();
        
        // Test case 3: Find all permutations
        String s1_3 = "abc";
        String s2_3 = "abacbcab";
        System.out.println("All permutation indices: " + findAllPermutations(s1_3, s2_3));
        System.out.println();
        
        // Demonstration
        demonstrateProcess(s1_1, s2_1);
    }
}
```

### Example 8: Maximum Number of Vowels in Substring (Fixed Window)

```java
import java.util.*;

/**
 * Problem: Find maximum number of vowels in any substring of length k
 * 
 * Approach: Fixed-size sliding window with vowel counting
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
public class MaxVowelsInSubstring {
    
    private static final Set<Character> VOWELS = Set.of('a', 'e', 'i', 'o', 'u');
    
    /**
     * Finds maximum number of vowels in any substring of length k
     * 
     * @param s Input string
     * @param k Length of substring
     * @return Maximum number of vowels in k-length substring
     */
    public static int maxVowels(String s, int k) {
        if (s == null || s.length() < k || k <= 0) {
            return 0;
        }
        
        // Count vowels in first window
        int currentVowels = 0;
        for (int i = 0; i < k; i++) {
            if (VOWELS.contains(s.charAt(i))) {
                currentVowels++;
            }
        }
        
        int maxVowels = currentVowels;
        
        // Slide window through rest of string
        for (int i = k; i < s.length(); i++) {
            // Add new character (right side)
            if (VOWELS.contains(s.charAt(i))) {
                currentVowels++;
            }
            
            // Remove old character (left side)
            if (VOWELS.contains(s.charAt(i - k))) {
                currentVowels--;
            }
            
            // Update maximum
            maxVowels = Math.max(maxVowels, currentVowels);
        }
        
        return maxVowels;
    }
    
    /**
     * Returns the actual substring with maximum vowels
     */
    public static String maxVowelsSubstring(String s, int k) {
        if (s == null || s.length() < k || k <= 0) {
            return "";
        }
        
        int currentVowels = 0;
        int maxVowels = 0;
        int bestStartIndex = 0;
        
        // Process first window
        for (int i = 0; i < k; i++) {
            if (VOWELS.contains(s.charAt(i))) {
                currentVowels++;
            }
        }
        
        maxVowels = currentVowels;
        
        // Slide window
        for (int i = k; i < s.length(); i++) {
            // Update vowel count
            if (VOWELS.contains(s.charAt(i))) {
                currentVowels++;
            }
            if (VOWELS.contains(s.charAt(i - k))) {
                currentVowels--;
            }
            
            // Update best result
            if (currentVowels > maxVowels) {
                maxVowels = currentVowels;
                bestStartIndex = i - k + 1;
            }
        }
        
        return s.substring(bestStartIndex, bestStartIndex + k);
    }
    
    /**
     * Alternative implementation using character frequency array
     */
    public static int maxVowelsOptimized(String s, int k) {
        if (s == null || s.length() < k || k <= 0) {
            return 0;
        }
        
        // Pre-compute vowel status for each character
        boolean[] isVowel = new boolean[s.length()];
        for (int i = 0; i < s.length(); i++) {
            isVowel[i] = VOWELS.contains(s.charAt(i));
        }
        
        // Count vowels in first window
        int currentVowels = 0;
        for (int i = 0; i < k; i++) {
            if (isVowel[i]) currentVowels++;
        }
        
        int maxVowels = currentVowels;
        
        // Slide window
        for (int i = k; i < s.length(); i++) {
            if (isVowel[i]) currentVowels++;
            if (isVowel[i - k]) currentVowels--;
            maxVowels = Math.max(maxVowels, currentVowels);
        }
        
        return maxVowels;
    }
    
    /**
     * Finds all substrings with maximum number of vowels
     */
    public static List<String> findAllMaxVowelSubstrings(String s, int k) {
        List<String> result = new ArrayList<>();
        
        if (s == null || s.length() < k || k <= 0) {
            return result;
        }
        
        int maxVowelCount = maxVowels(s, k);
        
        int currentVowels = 0;
        
        // Count vowels in first window
        for (int i = 0; i < k; i++) {
            if (VOWELS.contains(s.charAt(i))) {
                currentVowels++;
            }
        }
        
        if (currentVowels == maxVowelCount) {
            result.add(s.substring(0, k));
        }
        
        // Slide window
        for (int i = k; i < s.length(); i++) {
            if (VOWELS.contains(s.charAt(i))) {
                currentVowels++;
            }
            if (VOWELS.contains(s.charAt(i - k))) {
                currentVowels--;
            }
            
            if (currentVowels == maxVowelCount) {
                result.add(s.substring(i - k + 1, i + 1));
            }
        }
        
        return result;
    }
    
    /**
     * Demonstrates the sliding window process
     */
    public static void demonstrateProcess(String s, int k) {
        System.out.println("Finding max vowels in substring of length " + k + " for: \"" + s + "\"");
        
        if (s.length() < k) {
            System.out.println("String too short");
            return;
        }
        
        int maxVowelCount = 0;
        String bestSubstring = "";
        
        for (int i = 0; i <= s.length() - k; i++) {
            String window = s.substring(i, i + k);
            
            int vowelCount = 0;
            for (char c : window.toCharArray()) {
                if (VOWELS.contains(c)) {
                    vowelCount++;
                }
            }
            
            String status = "";
            if (vowelCount > maxVowelCount) {
                maxVowelCount = vowelCount;
                bestSubstring = window;
                status = " ← NEW BEST";
            } else if (vowelCount == maxVowelCount && maxVowelCount > 0) {
                status = " ← TIED FOR BEST";
            }
            
            System.out.printf("Position %d: \"%s\" has %d vowels%s%n", 
                i, window, vowelCount, status);
        }
        
        System.out.println("Best substring: \"" + bestSubstring + "\" with " + maxVowelCount + " vowels");
    }
    
    public static void main(String[] args) {
        // Test case 1: Normal case
        String s1 = "abciiidef";
        int k1 = 3;
        System.out.println("Input: \"" + s1 + "\", k = " + k1);
        System.out.println("Max vowels: " + maxVowels(s1, k1)); // Output: 3
        System.out.println("Best substring: \"" + maxVowelsSubstring(s1, k1) + "\""); // Output: "iii"
        System.out.println();
        
        // Test case 2: All consonants
        String s2 = "rhythms";
        int k2 = 4;
        System.out.println("Input: \"" + s2 + "\", k = " + k2);
        System.out.println("Max vowels: " + maxVowels(s2, k2)); // Output: 0
        System.out.println();
        
        // Test case 3: Mixed vowels and consonants
        String s3 = "leetcode";
        int k3 = 3;
        System.out.println("Input: \"" + s3 + "\", k = " + k3);
        System.out.println("All max substrings: " + findAllMaxVowelSubstrings(s3, k3));
        System.out.println();
        
        // Demonstration
        demonstrateProcess(s1, k1);
    }
}
```

### Example 9: Longest Repeating Character Replacement (Variable Window)

```java
import java.util.*;

/**
 * Problem: Find length of longest substring with same character after at most k changes
 * 
 * Approach: Variable-size sliding window with frequency tracking
 * Time Complexity: O(n)
 * Space Complexity: O(1) - at most 26 characters in frequency map
 */
public class LongestRepeatingCharacterReplacement {
    
    /**
     * Finds length of longest substring with same character after at most k replacements
     * 
     * @param s Input string
     * @param k Maximum number of character replacements allowed
     * @return Length of longest valid substring
     */
    public static int characterReplacement(String s, int k) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        // Frequency map for characters in current window
        int[] charCount = new int[26];
        
        int left = 0, right = 0;
        int maxFreq = 0;  // Frequency of most frequent character in window
        int maxLength = 0;
        
        while (right < s.length()) {
            // Expand window: add character at right
            char rightChar = s.charAt(right);
            charCount[rightChar - 'A']++;
            
            // Update max frequency in current window
            maxFreq = Math.max(maxFreq, charCount[rightChar - 'A']);
            
            // Check if current window is valid
            // Window size - max frequency = number of changes needed
            int windowSize = right - left + 1;
            int changesNeeded = windowSize - maxFreq;
            
            // If changes needed > k, contract window from left
            if (changesNeeded > k) {
                char leftChar = s.charAt(left);
                charCount[leftChar - 'A']--;
                left++;
                
                // Note: We don't update maxFreq here for optimization
                // It might be slightly higher than actual, but it doesn't affect correctness
            }
            
            // Update maximum length
            maxLength = Math.max(maxLength, right - left + 1);
            right++;
        }
        
        return maxLength;
    }
    
    /**
     * Alternative implementation with accurate max frequency tracking
     */
    public static int characterReplacementAccurate(String s, int k) {
        if (s == null || s.length() == 0) {
            return 0;
        }
        
        Map<Character, Integer> charCount = new HashMap<>();
        int left = 0;
        int maxLength = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            charCount.put(rightChar, charCount.getOrDefault(rightChar, 0) + 1);
            
            // Find max frequency in current window
            int maxFreq = Collections.max(charCount.values());
            
            // Contract window if needed
            while (right - left + 1 - maxFreq > k) {
                char leftChar = s.charAt(left);
                charCount.put(leftChar, charCount.get(leftChar) - 1);
                if (charCount.get(leftChar) == 0) {
                    charCount.remove(leftChar);
                }
                left++;
            }
            
            maxLength = Math.max(maxLength, right - left + 1);
        }
        
        return maxLength;
    }
    
    /**
     * Returns the actual longest substring after character replacement
     */
    public static String longestRepeatingSubstring(String s, int k) {
        if (s == null || s.length() == 0) {
            return "";
        }
        
        int[] charCount = new int[26];
        int left = 0, maxFreq = 0;
        int bestLength = 0, bestStart = 0;
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            charCount[rightChar - 'A']++;
            maxFreq = Math.max(maxFreq, charCount[rightChar - 'A']);
            
            // Contract window if too many changes needed
            while (right - left + 1 - maxFreq > k) {
                char leftChar = s.charAt(left);
                charCount[leftChar - 'A']--;
                left++;
                
                // Recalculate max frequency
                maxFreq = 0;
                for (int count : charCount) {
                    maxFreq = Math.max(maxFreq, count);
                }
            }
            
            // Update best result
            if (right - left + 1 > bestLength) {
                bestLength = right - left + 1;
                bestStart = left;
            }
        }
        
        return s.substring(bestStart, bestStart + bestLength);
    }
    
    /**
     * Finds the character that should be kept (most frequent) in optimal solution
     */
    public static char findOptimalCharacter(String s, int k) {
        char bestChar = 'A';
        int maxLength = 0;
        
        // Try keeping each character and find which gives maximum length
        for (char targetChar = 'A'; targetChar <= 'Z'; targetChar++) {
            int left = 0, replacements = 0, currentMax = 0;
            
            for (int right = 0; right < s.length(); right++) {
                if (s.charAt(right) != targetChar) {
                    replacements++;
                }
                
                while (replacements > k) {
                    if (s.charAt(left) != targetChar) {
                        replacements--;
                    }
                    left++;
                }
                
                currentMax = Math.max(currentMax, right - left + 1);
            }
            
            if (currentMax > maxLength) {
                maxLength = currentMax;
                bestChar = targetChar;
            }
        }
        
        return bestChar;
    }
    
    /**
     * Demonstrates the sliding window process
     */
    public static void demonstrateProcess(String s, int k) {
        System.out.println("Finding longest repeating character replacement for: \"" + s + "\", k = " + k);
        
        int[] charCount = new int[26];
        int left = 0, maxFreq = 0, maxLength = 0;
        String bestWindow = "";
        
        for (int right = 0; right < s.length(); right++) {
            char rightChar = s.charAt(right);
            charCount[rightChar - 'A']++;
            maxFreq = Math.max(maxFreq, charCount[rightChar - 'A']);
            
            int windowSize = right - left + 1;
            int changesNeeded = windowSize - maxFreq;
            
            while (changesNeeded > k) {
                char leftChar = s.charAt(left);
                charCount[leftChar - 'A']--;
                left++;
                
                // Recalculate max frequency
                maxFreq = 0;
                for (int count : charCount) {
                    maxFreq = Math.max(maxFreq, count);
                }
                
                windowSize = right - left + 1;
                changesNeeded = windowSize - maxFreq;
            }
            
            String currentWindow = s.substring(left, right + 1);
            System.out.printf("Step %d: Window [%d,%d] = \"%s\", MaxFreq=%d, Changes=%d, Length=%d%n",
                right + 1, left, right, currentWindow, maxFreq, changesNeeded, windowSize);
            
            if (windowSize > maxLength) {
                maxLength = windowSize;
                bestWindow = currentWindow;
            }
        }
        
        System.out.println("Best window: \"" + bestWindow + "\" with length: " + maxLength);
    }
    
    public static void main(String[] args) {
        // Test case 1: Normal case
        String s1 = "ABAB";
        int k1 = 2;
        System.out.println("Input: \"" + s1 + "\", k = " + k1);
        System.out.println("Max length: " + characterReplacement(s1, k1)); // Output: 4
        System.out.println("Best substring: \"" + longestRepeatingSubstring(s1, k1) + "\"");
        System.out.println("Optimal character to keep: " + findOptimalCharacter(s1, k1));
        System.out.println();
        
        // Test case 2: More complex case
        String s2 = "AABABBA";
        int k2 = 1;
        System.out.println("Input: \"" + s2 + "\", k = " + k2);
        System.out.println("Max length: " + characterReplacement(s2, k2)); // Output: 4
        System.out.println();
        
        // Demonstration
        demonstrateProcess(s1, k1);
    }
}
```

### Example 10: Sliding Window Maximum (Fixed Window with Deque)

```java
import java.util.*;

/**
 * Problem: Find maximum element in each sliding window of size k
 * 
 * Approach: Fixed-size sliding window with deque for efficient maximum tracking
 * Time Complexity: O(n)
 * Space Complexity: O(k) for the deque
 */
public class SlidingWindowMaximum {
    
    /**
     * Finds maximum element in each sliding window of size k
     * Uses deque to maintain elements in decreasing order
     * 
     * @param nums Input array
     * @param k Window size
     * @return Array of maximum elements in each window
     */
    public static int[] maxSlidingWindow(int[] nums, int k) {
        if (nums == null || nums.length == 0 || k <= 0) {
            return new int[0];
        }
        
        int n = nums.length;
        int[] result = new int[n - k + 1];
        
        // Deque to store indices of array elements
        // Elements are stored in decreasing order of their values
        Deque<Integer> deque = new ArrayDeque<>();
        
        // Process first window
        for (int i = 0; i < k; i++) {
            // Remove all smaller elements from rear
            while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
                deque.pollLast();
            }
            deque.offerLast(i);
        }
        
        // The front of deque contains index of maximum element
        result[0] = nums[deque.peekFirst()];
        
        // Process remaining windows
        for (int i = k; i < n; i++) {
            // Remove indices that are out of current window
            while (!deque.isEmpty() && deque.peekFirst() <= i - k) {
                deque.pollFirst();
            }
            
            // Remove all smaller elements from rear
            while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
                deque.pollLast();
            }
            
            // Add current element
            deque.offerLast(i);
            
            // The front contains index of maximum element of current window
            result[i - k + 1] = nums[deque.peekFirst()];
        }
        
        return result;
    }
    
    /**
     * Brute force approach for comparison (O(n*k) time complexity)
     */
    public static int[] maxSlidingWindowBruteForce(int[] nums, int k) {
        if (nums == null || nums.length == 0 || k <= 0) {
            return new int[0];
        }
        
        int n = nums.length;
        int[] result = new int[n - k + 1];
        
        for (int i = 0; i <= n - k; i++) {
            int max = nums[i];
            for (int j = i + 1; j < i + k; j++) {
                max = Math.max(max, nums[j]);
            }
            result[i] = max;
        }
        
        return result;
    }
    
    /**
     * Alternative implementation using TreeMap for different approach
     */
    public static int[] maxSlidingWindowTreeMap(int[] nums, int k) {
        if (nums == null || nums.length == 0 || k <= 0) {
            return new int[0];
        }
        
        int n = nums.length;
        int[] result = new int[n - k + 1];
        
        // TreeMap to maintain elements in sorted order with counts
        TreeMap<Integer, Integer> window = new TreeMap<>();
        
        // Process first window
        for (int i = 0; i < k; i++) {
            window.put(nums[i], window.getOrDefault(nums[i], 0) + 1);
        }
        result[0] = window.lastKey(); // Maximum element
        
        // Process remaining windows
        for (int i = k; i < n; i++) {
            // Remove element going out of window
            int outgoing = nums[i - k];
            window.put(outgoing, window.get(outgoing) - 1);
            if (window.get(outgoing) == 0) {
                window.remove(outgoing);
            }
            
            // Add new element
            window.put(nums[i], window.getOrDefault(nums[i], 0) + 1);
            
            // Maximum is the last key
            result[i - k + 1] = window.lastKey();
        }
        
        return result;
    }
    
    /**
     * Helper class to demonstrate the deque state during processing
     */
    public static class DequeTracker {
        private Deque<Integer> deque = new ArrayDeque<>();
        private int[] nums;
        
        public DequeTracker(int[] nums) {
            this.nums = nums;
        }
        
        public void processWindow(int start, int end) {
            System.out.printf("Processing window [%d, %d]: ", start, end);
            for (int i = start; i <= end; i++) {
                System.out.print(nums[i] + " ");
            }
            System.out.println();
            
            // Clear deque for new window demonstration
            deque.clear();
            
            // Build deque for this window
            for (int i = start; i <= end; i++) {
                while (!deque.isEmpty() && nums[deque.peekLast()] <= nums[i]) {
                    deque.pollLast();
                }
                deque.offerLast(i);
                
                System.out.printf("  After adding nums[%d]=%d: deque indices=", i, nums[i]);
                printDeque();
            }
            
            System.out.println("  Maximum: " + nums[deque.peekFirst()]);
            System.out.println();
        }
        
        private void printDeque() {
            System.out.print("[");
            Iterator<Integer> it = deque.iterator();
            boolean first = true;
            while (it.hasNext()) {
                if (!first) System.out.print(", ");
                int idx = it.next();
                System.out.print(idx + "(" + nums[idx] + ")");
                first = false;
            }
            System.out.println("]");
        }
    }
    
    /**
     * Demonstrates the sliding window maximum process
     */
    public static void demonstrateProcess(int[] nums, int k) {
        System.out.println("Demonstrating sliding window maximum:");
        System.out.println("Array: " + Arrays.toString(nums) + ", k = " + k);
        System.out.println();
        
        DequeTracker tracker = new DequeTracker(nums);
        
        // Show first few windows
        int windowsToShow = Math.min(4, nums.length - k + 1);
        for (int i = 0; i < windowsToShow; i++) {
            tracker.processWindow(i, i + k - 1);
        }
        
        // Show final result
        int[] result = maxSlidingWindow(nums, k);
        System.out.println("Final result: " + Arrays.toString(result));
    }
    
    /**
     * Performance comparison between different approaches
     */
    public static void performanceComparison(int[] nums, int k) {
        System.out.println("Performance Comparison:");
        System.out.println("Array size: " + nums.length + ", Window size: " + k);
        
        long startTime, endTime;
        
        // Deque approach
        startTime = System.nanoTime();
        int[] result1 = maxSlidingWindow(nums, k);
        endTime = System.nanoTime();
        System.out.println("Deque approach: " + (endTime - startTime) / 1000 + " microseconds");
        
        // TreeMap approach
        startTime = System.nanoTime();
        int[] result2 = maxSlidingWindowTreeMap(nums, k);
        endTime = System.nanoTime();
        System.out.println("TreeMap approach: " + (endTime - startTime) / 1000 + " microseconds");
        
        // Verify results are the same
        System.out.println("Results match: " + Arrays.equals(result1, result2));
    }
    
    public static void main(String[] args) {
        // Test case 1: Normal case
        int[] nums1 = {1, 3, -1, -3, 5, 3, 6, 7};
        int k1 = 3;
        System.out.println("Input: " + Arrays.toString(nums1) + ", k = " + k1);
        System.out.println("Result: " + Arrays.toString(maxSlidingWindow(nums1, k1)));
        // Output: [3, 3, 5, 5, 6, 7]
        System.out.println();
        
        // Test case 2: Single element window
        int[] nums2 = {1};
        int k2 = 1;
        System.out.println("Input: " + Arrays.toString(nums2) + ", k = " + k2);
        System.out.println("Result: " + Arrays.toString(maxSlidingWindow(nums2, k2)));
        System.out.println();
        
        // Demonstration
        demonstrateProcess(nums1, k1);
        
        // Performance comparison with larger array
        System.out.println();
        int[] largeArray = new int[10000];
        Random random = new Random(42);
        for (int i = 0; i < largeArray.length; i++) {
            largeArray[i] = random.nextInt(1000);
        }
        performanceComparison(largeArray, 100);
    }
}
```

---

## 5) Pattern Recognition

```
Sliding Window Problem Patterns:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          Pattern Recognition Guide                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────┐           ┌─────────────────────────┐             │
│  │    Fixed Window         │           │   Variable Window       │             │
│  │                         │           │                         │             │
│  │  Key Indicators:        │           │  Key Indicators:        │             │
│  │  • "exactly k"          │           │  • "at most k"          │             │
│  │  • "window of size k"   │           │  • "longest/shortest"   │             │
│  │  • "every k elements"   │           │  • "maximum/minimum"    │             │
│  │  • "in each subarray"   │           │  • "equal to target"    │             │
│  │                         │           │  • "distinct characters"│             │
│  │  Examples:              │           │                         │             │
│  │  • Max sum of k elements│           │  Examples:              │             │
│  │  • Average of k elements│           │  • Longest substring    │             │
│  │  • First negative in    │           │  • Minimum window       │             │
│  │    each window          │           │  • Sum equals target    │             │
│  │  • Anagrams in string   │           │  • At most k distinct   │             │
│  └─────────────────────────┘           └─────────────────────────┘             │
│              │                                     │                           │
│              └─────────────┬───────────────────────┘                           │
│                            │                                                   │
│                            ▼                                                   │
│              ┌─────────────────────────┐                                       │
│              │  Decision Framework     │                                       │
│              │                         │                                       │
│              │  1. Identify window type│                                       │
│              │  2. Choose pointers     │                                       │
│              │  3. Define window state │                                       │
│              │  4. Handle edge cases   │                                       │
│              │  5. Optimize for space  │                                       │
│              └─────────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Problem Identification Checklist

**Use Fixed Window When:**
- ✅ Problem mentions exact window size (k)
- ✅ Need to process every possible k-length subarray
- ✅ Window size remains constant
- ✅ Examples: "maximum sum of k elements", "anagrams of length k"

**Use Variable Window When:**
- ✅ Problem asks for longest/shortest subarray
- ✅ Window size changes based on conditions
- ✅ Uses phrases like "at most", "at least", "exactly"
- ✅ Examples: "longest substring without repeating", "minimum window"

---

## 6) Complexity Analysis

```
Complexity Comparison:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                      Brute Force vs Sliding Window                             │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  Brute Force Approach:                                                         │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  for i = 0 to n-k:                    Time: O(n * k)                   │   │
│  │      for j = i to i+k-1:              Space: O(1)                      │   │
│  │          process nums[j]                                                │   │
│  │      calculate result for window                                        │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                    │                                           │
│                                    ▼                                           │
│  Sliding Window Approach:                                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐   │
│  │  initialize first window               Time: O(n)                      │   │
│  │  for i = k to n-1:                    Space: O(1) to O(k)             │   │
│  │      remove nums[i-k]                                                  │   │
│  │      add nums[i]                                                       │   │
│  │      update result                                                     │   │
│  └─────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  Improvement: O(n*k) → O(n)                                                    │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Time Complexity Analysis

**Fixed Window:**
- **Initialization**: O(k) for first window
- **Sliding**: O(n-k) iterations, O(1) per iteration
- **Total**: O(k) + O(n-k) = O(n)

**Variable Window:**
- **Two Pointers**: Each element visited at most twice
- **Total**: O(n)

**Space Complexity:**
- **Basic**: O(1) for counters
- **With HashMap**: O(k) for distinct elements
- **With Deque**: O(k) for window elements

---

## 7) Common Pitfalls

### 7.1 Off-by-One Errors

```java
// ❌ Wrong: Missing boundary check
while (right < s.length() && left <= right) {
    // Process without checking left boundary
}

// ✅ Correct: Proper boundary checks
while (right < s.length()) {
    // Expand window
    while (left <= right && needToContract) {
        // Contract window
        left++;
    }
    right++;
}
```

### 7.2 Window State Management

```java
// ❌ Wrong: Forgetting to update state when contracting
while (windowSize > k) {
    left++;
    // Forgot to remove element from frequency map
}

// ✅ Correct: Always update state
while (windowSize > k) {
    charFreq[s.charAt(left)]--;
    if (charFreq[s.charAt(left)] == 0) {
        distinctChars--;
    }
    left++;
}
```

### 7.3 Edge Cases

```java
// ❌ Wrong: Not handling edge cases
public int solve(int[] nums, int k) {
    // Missing validation
    for (int i = k; i < nums.length; i++) {
        // Process
    }
}

// ✅ Correct: Handle edge cases
public int solve(int[] nums, int k) {
    if (nums == null || nums.length < k || k <= 0) {
        return 0; // or appropriate default
    }
    // Rest of implementation
}
```

---

## 8) Advanced Techniques

### 8.1 Multiple Sliding Windows

```java
/**
 * Example: Find intervals where both conditions are met
 */
public static boolean hasValidIntervals(int[] nums, int k1, int k2) {
    // Maintain two sliding windows simultaneously
    int sum1 = 0, sum2 = 0;
    
    // Initialize first windows
    for (int i = 0; i < k1; i++) sum1 += nums[i];
    for (int i = 0; i < k2; i++) sum2 += nums[i];
    
    boolean found = false;
    
    for (int i = Math.max(k1, k2); i < nums.length; i++) {
        // Update first window
        if (i >= k1) {
            sum1 = sum1 - nums[i - k1] + nums[i];
        }
        
        // Update second window  
        if (i >= k2) {
            sum2 = sum2 - nums[i - k2] + nums[i];
        }
        
        // Check conditions
        if (sum1 > threshold1 && sum2 > threshold2) {
            found = true;
            break;
        }
    }
    
    return found;
}
```

### 8.2 Sliding Window with Binary Search

```java
/**
 * Example: Find minimum window size that satisfies condition
 */
public static int minWindowSize(int[] nums, int target) {
    int left = 1, right = nums.length;
    int result = -1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;
        
        if (canAchieveWithWindowSize(nums, target, mid)) {
            result = mid;
            right = mid - 1; // Try smaller window
        } else {
            left = mid + 1; // Need larger window
        }
    }
    
    return result;
}

private static boolean canAchieveWithWindowSize(int[] nums, int target, int windowSize) {
    // Use sliding window to check if any window of given size meets target
    int sum = 0;
    
    // First window
    for (int i = 0; i < windowSize; i++) {
        sum += nums[i];
    }
    
    if (sum >= target) return true;
    
    // Slide window
    for (int i = windowSize; i < nums.length; i++) {
        sum = sum - nums[i - windowSize] + nums[i];
        if (sum >= target) return true;
    }
    
    return false;
}
```

---


