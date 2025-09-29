# Dynamic Programming: Comprehensive Guide

This comprehensive guide covers Dynamic Programming (DP), one of the most powerful algorithmic techniques for solving optimization problems. It includes core concepts, patterns, visual diagrams, and 10 famous Java examples with detailed explanations.

## Table of Contents
1. [What is Dynamic Programming](#1-what-is-dynamic-programming)
2. [When to Use Dynamic Programming](#2-when-to-use-dynamic-programming)
3. [DP Patterns and Approaches](#3-dp-patterns-and-approaches)
4. [Visual Diagrams and Examples](#4-visual-diagrams-and-examples)
5. [10 Famous DP Problems in Java](#5-10-famous-dp-problems-in-java)
6. [Optimization Techniques](#6-optimization-techniques)
7. [Common Pitfalls](#7-common-pitfalls)
8. [Advanced DP Patterns](#8-advanced-dp-patterns)

---

## 1) What is Dynamic Programming

### 1.1 Definition and Core Concept

Dynamic Programming is an algorithmic technique that solves complex problems by breaking them down into simpler subproblems and storing the results to avoid redundant calculations.

```
Dynamic Programming Essence:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    "Remember the past to predict the future"                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────────────┐           ┌─────────────────────────┐             │
│  │    Complex Problem      │           │    Optimal Solution     │             │
│  │                         │           │                         │             │
│  │         ↓               │           │         ↑               │             │
│  │  Break into subproblems │    ═══    │  Combine subproblem     │             │
│  │         ↓               │           │    solutions            │             │
│  │   Solve subproblems     │           │         ↑               │             │
│  │         ↓               │           │                         │             │
│  │    Store results        │           │                         │             │
│  │    (Memoization)        │           │                         │             │
│  └─────────────────────────┘           └─────────────────────────┘             │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 Key Principles

**1. Optimal Substructure:**
```
Problem(n) = function(Problem(n-1), Problem(n-2), ..., Problem(0))

Example: Fibonacci
F(n) = F(n-1) + F(n-2)
```

**2. Overlapping Subproblems:**
```
Without DP: Recalculate same subproblems multiple times
With DP: Calculate once, store result, reuse when needed
```

### 1.3 DP vs Other Techniques

```
Algorithm Comparison:
┌─────────────────┬─────────────────┬─────────────────┬─────────────────────┐
│   Technique     │  Time Approach  │   Space Usage   │    Best For         │
├─────────────────┼─────────────────┼─────────────────┼─────────────────────┤
│ Brute Force     │ Exponential     │ O(depth)        │ Small inputs        │
│ Greedy          │ O(n log n)      │ O(1)           │ Local optimum       │
│ Divide&Conquer  │ O(n log n)      │ O(log n)       │ Independent subprob │
│ Dynamic Prog    │ O(n²) typical   │ O(n)           │ Overlapping subprob │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────┘
```

---

## 2) When to Use Dynamic Programming

### 2.1 Problem Identification

```
DP Problem Indicators:
┌────────────────────────────────────────────────────────────────────────────────┐
│                          DP Decision Framework                                 │
├────────────────────────────────────────────────────────────────────────────────┤
│                                                                                │
│  ┌─────────────────────────┐    ┌─────────────────────────┐                    │
│  │  Optimal Substructure   │    │ Overlapping Subproblems │                    │
│  │                         │    │                         │                    │
│  │  ✓ Solution depends on  │    │  ✓ Same subproblems     │                    │
│  │    smaller subproblems  │    │    solved multiple times│                    │
│  │  ✓ Optimal solution     │ +  │  ✓ Recursive calls with │  =  Use DP         │
│  │    contains optimal     │    │    same parameters      │                    │
│  │    subproblems          │    │  ✓ Redundant computation│                    │
│  │  ✓ Can build solution   │    │  ✓ Exponential time     │                    │
│  │    bottom-up            │    │    without memoization  │                    │
│  └─────────────────────────┘    └─────────────────────────┘                    │
└────────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 Common DP Problem Types

**1. Optimization Problems:**
- Maximum/Minimum path sum
- Longest/Shortest subsequence
- Maximum profit scenarios

**2. Counting Problems:**
- Number of ways to reach target
- Count unique paths
- Decode variations

**3. Decision Problems:**
- Can we achieve target?
- Is sequence possible?
- Partition feasibility

### 2.3 Keywords to Look For

```
🚨 DP Signal Words:
• "optimal" / "maximum" / "minimum"
• "longest" / "shortest"
• "count number of ways"
• "is it possible"
• "find all solutions"
• Problems with choices at each step
• Problems with constraints
```

---

## 3) DP Patterns and Approaches

### 3.1 Top-Down vs Bottom-Up

```
DP Approaches Comparison:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                            Memoization vs Tabulation                           │
├─────────────────┬─────────────────────────────────┬─────────────────────────────┤
│    Approach     │         Top-Down                │        Bottom-Up            │
│                 │        (Memoization)            │       (Tabulation)          │
├─────────────────┼─────────────────────────────────┼─────────────────────────────┤
│ Strategy        │ Recursion + Cache               │ Iteration + Table           │
│ Direction       │ Problem → Base Case             │ Base Case → Problem         │
│ Memory          │ Implicit call stack             │ Explicit DP table           │
│ Space           │ O(recursion depth)              │ O(problem size)             │
│ Implementation  │ Natural, intuitive              │ Requires careful ordering   │
│ Stack Overflow  │ Possible for deep recursion     │ No risk                     │
│ Performance     │ Function call overhead          │ Faster iteration            │
└─────────────────┴─────────────────────────────────┴─────────────────────────────┘
```

### 3.2 State Definition Patterns

```
Common DP State Patterns:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│  1D DP: dp[i] = optimal value at position i                                    │
│  ┌───┬───┬───┬───┬───┬───┐                                                     │
│  │ 0 │ 1 │ 2 │ 3 │ 4 │ 5 │  Example: Fibonacci, Climbing Stairs               │
│  └───┴───┴───┴───┴───┴───┘                                                     │
│                                                                                 │
│  2D DP: dp[i][j] = optimal value at position (i,j)                            │
│  ┌───┬───┬───┬───┐                                                             │
│  │0,0│0,1│0,2│0,3│  Example: Grid paths, Edit distance                        │
│  ├───┼───┼───┼───┤                                                             │
│  │1,0│1,1│1,2│1,3│                                                             │
│  ├───┼───┼───┼───┤                                                             │
│  │2,0│2,1│2,2│2,3│                                                             │
│  └───┴───┴───┴───┘                                                             │
│                                                                                 │
│  3D DP: dp[i][j][k] = optimal value with 3 dimensions                         │
│  Example: Knapsack with constraints, Game theory with states                   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

---

## 4) Visual Diagrams and Examples

### 4.1 Fibonacci Sequence (Classic DP Example)

```
Problem: F(n) = F(n-1) + F(n-2), F(0)=0, F(1)=1

Without DP (Exponential Time):
                    F(5)
                  /      \
               F(4)        F(3)
              /    \      /    \
           F(3)    F(2) F(2)   F(1)
          /   \   /   \  /  \
       F(2)  F(1) F(1) F(0) F(1) F(0)
      /   \
    F(1)  F(0)

Redundant calculations: F(3) computed 2 times, F(2) computed 3 times, etc.

With DP (Linear Time):
Cache: [0, 1, 1, 2, 3, 5]
Index:  0  1  2  3  4  5

F(0) = 0                    ← Base case
F(1) = 1                    ← Base case  
F(2) = F(1) + F(0) = 1 + 0 = 1
F(3) = F(2) + F(1) = 1 + 1 = 2
F(4) = F(3) + F(2) = 2 + 1 = 3
F(5) = F(4) + F(3) = 3 + 2 = 5
```

### 4.2 Grid Path Problem

```
Problem: Count unique paths from top-left to bottom-right (only right/down moves)

Grid (3x3):
┌───┬───┬───┐
│ S │   │   │
├───┼───┼───┤
│   │   │   │
├───┼───┼───┤
│   │   │ E │
└───┴───┴───┘

DP Solution:
┌───┬───┬───┐
│ 1 │ 1 │ 1 │  ← First row: only one way (all right)
├───┼───┼───┤
│ 1 │ 2 │ 3 │  ← dp[i][j] = dp[i-1][j] + dp[i][j-1]
├───┼───┼───┤
│ 1 │ 3 │ 6 │  ← Answer: 6 unique paths
└───┴───┴───┘

State Transition:
dp[i][j] = dp[i-1][j] + dp[i][j-1]
            ↑             ↑
         from top    from left
```

---

## 5) 10 Famous DP Problems in Java

### Example 1: Fibonacci Numbers (1D DP)

```java
/**
 * Problem: Calculate nth Fibonacci number
 * F(n) = F(n-1) + F(n-2), where F(0)=0, F(1)=1
 * 
 * Time Complexity: O(n)
 * Space Complexity: O(n) for memoization, O(1) for optimized
 */
public class FibonacciDP {
    
    /**
     * Top-Down Approach (Memoization)
     */
    public static long fibonacciMemo(int n) {
        if (n < 0) return -1;
        
        Long[] memo = new Long[n + 1];
        return fibMemoHelper(n, memo);
    }
    
    private static long fibMemoHelper(int n, Long[] memo) {
        // Base cases
        if (n == 0) return 0;
        if (n == 1) return 1;
        
        // Check if already computed
        if (memo[n] != null) {
            return memo[n];
        }
        
        // Compute and store result
        memo[n] = fibMemoHelper(n - 1, memo) + fibMemoHelper(n - 2, memo);
        return memo[n];
    }
    
    /**
     * Bottom-Up Approach (Tabulation)
     */
    public static long fibonacciTabulation(int n) {
        if (n < 0) return -1;
        if (n == 0) return 0;
        if (n == 1) return 1;
        
        // DP table to store computed values
        long[] dp = new long[n + 1];
        
        // Base cases
        dp[0] = 0;
        dp[1] = 1;
        
        // Fill table bottom-up
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    /**
     * Space-Optimized Approach (O(1) space)
     */
    public static long fibonacciOptimized(int n) {
        if (n < 0) return -1;
        if (n == 0) return 0;
        if (n == 1) return 1;
        
        long prev2 = 0;  // F(n-2)
        long prev1 = 1;  // F(n-1)
        long current = 0; // F(n)
        
        for (int i = 2; i <= n; i++) {
            current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }
        
        return current;
    }
    
    /**
     * Demonstrate all approaches with timing
     */
    public static void demonstrateApproaches(int n) {
        System.out.println("Calculating Fibonacci(" + n + "):");
        
        // Memoization
        long start = System.nanoTime();
        long resultMemo = fibonacciMemo(n);
        long timeMemo = System.nanoTime() - start;
        
        // Tabulation
        start = System.nanoTime();
        long resultTab = fibonacciTabulation(n);
        long timeTab = System.nanoTime() - start;
        
        // Optimized
        start = System.nanoTime();
        long resultOpt = fibonacciOptimized(n);
        long timeOpt = System.nanoTime() - start;
        
        System.out.println("Memoization: " + resultMemo + " (Time: " + timeMemo/1000 + " μs)");
        System.out.println("Tabulation:  " + resultTab + " (Time: " + timeTab/1000 + " μs)");
        System.out.println("Optimized:   " + resultOpt + " (Time: " + timeOpt/1000 + " μs)");
    }
    
    public static void main(String[] args) {
        demonstrateApproaches(40);
        
        // Show sequence
        System.out.println("\nFirst 15 Fibonacci numbers:");
        for (int i = 0; i < 15; i++) {
            System.out.print(fibonacciOptimized(i) + " ");
        }
        System.out.println();
    }
}
```

### Example 2: Climbing Stairs (1D DP)

```java
/**
 * Problem: Count ways to climb n stairs (1 or 2 steps at a time)
 * 
 * This is essentially Fibonacci: ways(n) = ways(n-1) + ways(n-2)
 * Time Complexity: O(n)
 * Space Complexity: O(n) for memoization, O(1) for optimized
 */
public class ClimbingStairs {
    
    /**
     * Top-Down with Memoization
     */
    public static int climbStairsMemo(int n) {
        if (n <= 1) return 1;
        
        Integer[] memo = new Integer[n + 1];
        return climbHelper(n, memo);
    }
    
    private static int climbHelper(int n, Integer[] memo) {
        // Base cases
        if (n <= 1) return 1;
        
        // Check memo
        if (memo[n] != null) return memo[n];
        
        // Recurrence: ways to reach step n = ways to reach (n-1) + ways to reach (n-2)
        memo[n] = climbHelper(n - 1, memo) + climbHelper(n - 2, memo);
        return memo[n];
    }
    
    /**
     * Bottom-Up Tabulation
     */
    public static int climbStairsTabulation(int n) {
        if (n <= 1) return 1;
        
        int[] dp = new int[n + 1];
        
        // Base cases
        dp[0] = 1; // One way to stay at ground (do nothing)
        dp[1] = 1; // One way to reach first step
        
        // Fill table
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        
        return dp[n];
    }
    
    /**
     * Space-Optimized O(1)
     */
    public static int climbStairsOptimized(int n) {
        if (n <= 1) return 1;
        
        int prev2 = 1; // ways to reach (i-2)
        int prev1 = 1; // ways to reach (i-1)
        
        for (int i = 2; i <= n; i++) {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }
        
        return prev1;
    }
    
    /**
     * Extension: k steps allowed
     */
    public static int climbStairsKSteps(int n, int k) {
        if (n == 0) return 1;
        if (n < 0) return 0;
        
        int[] dp = new int[n + 1];
        dp[0] = 1;
        
        for (int i = 1; i <= n; i++) {
            for (int step = 1; step <= k && step <= i; step++) {
                dp[i] += dp[i - step];
            }
        }
        
        return dp[n];
    }
    
    /**
     * Show step-by-step calculation
     */
    public static void demonstrateStepByStep(int n) {
        System.out.println("Climbing " + n + " stairs step by step:");
        
        if (n <= 1) {
            System.out.println("Result: 1 way");
            return;
        }
        
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;
        
        System.out.println("dp[0] = 1 (base case)");
        System.out.println("dp[1] = 1 (base case)");
        
        for (int i = 2; i <= n; i++) {
            dp[i] = dp[i - 1] + dp[i - 2];
            System.out.printf("dp[%d] = dp[%d] + dp[%d] = %d + %d = %d%n", 
                i, i-1, i-2, dp[i-1], dp[i-2], dp[i]);
        }
        
        System.out.println("Total ways: " + dp[n]);
    }
    
    public static void main(String[] args) {
        int n = 5;
        
        System.out.println("Ways to climb " + n + " stairs:");
        System.out.println("Memoization: " + climbStairsMemo(n));
        System.out.println("Tabulation:  " + climbStairsTabulation(n));
        System.out.println("Optimized:   " + climbStairsOptimized(n));
        
        System.out.println("\nStep-by-step:");
        demonstrateStepByStep(n);
        
        System.out.println("\nWith k=3 steps allowed:");
        System.out.println("Ways: " + climbStairsKSteps(n, 3));
    }
}
```

### Example 3: Longest Common Subsequence (2D DP)

```java
/**
 * Problem: Find length of longest common subsequence between two strings
 * 
 * LCS(i,j) = LCS(i-1,j-1) + 1               if text1[i] == text2[j]
 *          = max(LCS(i-1,j), LCS(i,j-1))    otherwise
 * 
 * Time Complexity: O(m*n)
 * Space Complexity: O(m*n)
 */
public class LongestCommonSubsequence {
    
    /**
     * Top-Down Memoization
     */
    public static int lcsLengthMemo(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        Integer[][] memo = new Integer[m][n];
        
        return lcsHelper(text1, text2, 0, 0, memo);
    }
    
    private static int lcsHelper(String text1, String text2, int i, int j, Integer[][] memo) {
        // Base case: reached end of either string
        if (i >= text1.length() || j >= text2.length()) {
            return 0;
        }
        
        // Check memo
        if (memo[i][j] != null) {
            return memo[i][j];
        }
        
        int result;
        if (text1.charAt(i) == text2.charAt(j)) {
            // Characters match: include in LCS
            result = 1 + lcsHelper(text1, text2, i + 1, j + 1, memo);
        } else {
            // Characters don't match: try both possibilities
            int skipFirst = lcsHelper(text1, text2, i + 1, j, memo);
            int skipSecond = lcsHelper(text1, text2, i, j + 1, memo);
            result = Math.max(skipFirst, skipSecond);
        }
        
        memo[i][j] = result;
        return result;
    }
    
    /**
     * Bottom-Up Tabulation
     */
    public static int lcsLengthTabulation(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        
        // DP table: dp[i][j] = LCS length for text1[0..i-1] and text2[0..j-1]
        int[][] dp = new int[m + 1][n + 1];
        
        // Base case: empty string has LCS length 0 (already initialized)
        
        // Fill table
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    // Characters match
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    // Characters don't match
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        
        return dp[m][n];
    }
    
    /**
     * Get actual LCS string (not just length)
     */
    public static String getLCS(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1];
        
        // Fill DP table
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        
        // Backtrack to construct LCS
        StringBuilder lcs = new StringBuilder();
        int i = m, j = n;
        
        while (i > 0 && j > 0) {
            if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                // Character is part of LCS
                lcs.append(text1.charAt(i - 1));
                i--;
                j--;
            } else if (dp[i - 1][j] > dp[i][j - 1]) {
                // Move up
                i--;
            } else {
                // Move left
                j--;
            }
        }
        
        return lcs.reverse().toString();
    }
    
    /**
     * Space-Optimized version (O(min(m,n)) space)
     */
    public static int lcsLengthOptimized(String text1, String text2) {
        // Ensure text1 is shorter for space optimization
        if (text1.length() > text2.length()) {
            return lcsLengthOptimized(text2, text1);
        }
        
        int m = text1.length();
        int n = text2.length();
        
        // Only need current and previous row
        int[] prev = new int[m + 1];
        int[] curr = new int[m + 1];
        
        for (int j = 1; j <= n; j++) {
            for (int i = 1; i <= m; i++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    curr[i] = prev[i - 1] + 1;
                } else {
                    curr[i] = Math.max(prev[i], curr[i - 1]);
                }
            }
            // Swap arrays
            int[] temp = prev;
            prev = curr;
            curr = temp;
        }
        
        return prev[m];
    }
    
    /**
     * Visualize DP table construction
     */
    public static void visualizeDPTable(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1];
        
        System.out.println("LCS DP Table for \"" + text1 + "\" and \"" + text2 + "\":");
        
        // Print header
        System.out.print("    ");
        for (int j = 0; j <= n; j++) {
            if (j == 0) System.out.print("  ");
            else System.out.printf("%2c", text2.charAt(j - 1));
        }
        System.out.println();
        
        // Fill and print table
        for (int i = 0; i <= m; i++) {
            if (i == 0) System.out.print("  ");
            else System.out.printf("%2c", text1.charAt(i - 1));
            
            for (int j = 0; j <= n; j++) {
                if (i == 0 || j == 0) {
                    dp[i][j] = 0;
                } else if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
                System.out.printf("%3d", dp[i][j]);
            }
            System.out.println();
        }
        
        System.out.println("LCS Length: " + dp[m][n]);
        System.out.println("LCS String: \"" + getLCS(text1, text2) + "\"");
    }
    
    public static void main(String[] args) {
        String text1 = "ABCDGH";
        String text2 = "AEDFHR";
        
        System.out.println("Text 1: " + text1);
        System.out.println("Text 2: " + text2);
        
        System.out.println("\nLCS Length (Memoization): " + lcsLengthMemo(text1, text2));
        System.out.println("LCS Length (Tabulation): " + lcsLengthTabulation(text1, text2));
        System.out.println("LCS Length (Optimized): " + lcsLengthOptimized(text1, text2));
        System.out.println("LCS String: \"" + getLCS(text1, text2) + "\"");
        
        System.out.println();
        visualizeDPTable(text1, text2);
    }
}
```

### Example 4: 0/1 Knapsack Problem (2D DP)

```java
import java.util.*;

/**
 * Problem: Given items with weights and values, maximize value in knapsack of capacity W
 * Each item can be taken at most once (0/1 constraint)
 * 
 * DP[i][w] = maximum value using first i items with weight limit w
 * 
 * Time Complexity: O(n*W)
 * Space Complexity: O(n*W)
 */
public class KnapsackProblem {
    
    static class Item {
        int weight;
        int value;
        String name;
        
        Item(int weight, int value, String name) {
            this.weight = weight;
            this.value = value;
            this.name = name;
        }
        
        @Override
        public String toString() {
            return name + "(w:" + weight + ", v:" + value + ")";
        }
    }
    
    /**
     * Top-Down Memoization
     */
    public static int knapsackMemo(Item[] items, int capacity) {
        Integer[][] memo = new Integer[items.length][capacity + 1];
        return knapsackHelper(items, capacity, 0, memo);
    }
    
    private static int knapsackHelper(Item[] items, int capacity, int index, Integer[][] memo) {
        // Base case: no items left or no capacity
        if (index >= items.length || capacity <= 0) {
            return 0;
        }
        
        // Check memo
        if (memo[index][capacity] != null) {
            return memo[index][capacity];
        }
        
        // Option 1: Don't include current item
        int exclude = knapsackHelper(items, capacity, index + 1, memo);
        
        // Option 2: Include current item (if it fits)
        int include = 0;
        if (items[index].weight <= capacity) {
            include = items[index].value + 
                     knapsackHelper(items, capacity - items[index].weight, index + 1, memo);
        }
        
        // Take maximum
        memo[index][capacity] = Math.max(exclude, include);
        return memo[index][capacity];
    }
    
    /**
     * Bottom-Up Tabulation
     */
    public static int knapsackTabulation(Item[] items, int capacity) {
        int n = items.length;
        
        // DP table: dp[i][w] = max value with first i items and capacity w
        int[][] dp = new int[n + 1][capacity + 1];
        
        // Base case: 0 items or 0 capacity = 0 value (already initialized)
        
        // Fill table
        for (int i = 1; i <= n; i++) {
            for (int w = 1; w <= capacity; w++) {
                Item currentItem = items[i - 1];
                
                // Option 1: Don't take current item
                dp[i][w] = dp[i - 1][w];
                
                // Option 2: Take current item (if it fits)
                if (currentItem.weight <= w) {
                    int includeValue = currentItem.value + dp[i - 1][w - currentItem.weight];
                    dp[i][w] = Math.max(dp[i][w], includeValue);
                }
            }
        }
        
        return dp[n][capacity];
    }
    
    /**
     * Get actual items selected (backtracking)
     */
    public static List<Item> getSelectedItems(Item[] items, int capacity) {
        int n = items.length;
        int[][] dp = new int[n + 1][capacity + 1];
        
        // Fill DP table
        for (int i = 1; i <= n; i++) {
            for (int w = 1; w <= capacity; w++) {
                Item currentItem = items[i - 1];
                dp[i][w] = dp[i - 1][w]; // Don't take
                
                if (currentItem.weight <= w) {
                    int includeValue = currentItem.value + dp[i - 1][w - currentItem.weight];
                    dp[i][w] = Math.max(dp[i][w], includeValue);
                }
            }
        }
        
        // Backtrack to find selected items
        List<Item> selected = new ArrayList<>();
        int i = n, w = capacity;
        
        while (i > 0 && w > 0) {
            // If value didn't come from above, item was included
            if (dp[i][w] != dp[i - 1][w]) {
                selected.add(items[i - 1]);
                w -= items[i - 1].weight;
            }
            i--;
        }
        
        Collections.reverse(selected);
        return selected;
    }
    
    /**
     * Space-Optimized version (O(capacity) space)
     */
    public static int knapsackOptimized(Item[] items, int capacity) {
        // Only need current and previous row
        int[] dp = new int[capacity + 1];
        
        for (Item item : items) {
            // Traverse backwards to avoid using updated values
            for (int w = capacity; w >= item.weight; w--) {
                dp[w] = Math.max(dp[w], dp[w - item.weight] + item.value);
            }
        }
        
        return dp[capacity];
    }
    
    /**
     * Visualize DP table construction
     */
    public static void visualizeKnapsack(Item[] items, int capacity) {
        int n = items.length;
        int[][] dp = new int[n + 1][capacity + 1];
        
        System.out.println("Knapsack DP Table (capacity = " + capacity + "):");
        
        // Print header
        System.out.printf("%15s", "Item\\Capacity");
        for (int w = 0; w <= capacity; w++) {
            System.out.printf("%4d", w);
        }
        System.out.println();
        
        // Fill and print table
        for (int i = 0; i <= n; i++) {
            if (i == 0) {
                System.out.printf("%15s", "None");
            } else {
                System.out.printf("%15s", items[i - 1].name);
            }
            
            for (int w = 0; w <= capacity; w++) {
                if (i == 0 || w == 0) {
                    dp[i][w] = 0;
                } else {
                    Item item = items[i - 1];
                    dp[i][w] = dp[i - 1][w]; // Don't take
                    
                    if (item.weight <= w) {
                        int includeValue = item.value + dp[i - 1][w - item.weight];
                        dp[i][w] = Math.max(dp[i][w], includeValue);
                    }
                }
                System.out.printf("%4d", dp[i][w]);
            }
            System.out.println();
        }
        
        System.out.println("\nMaximum value: " + dp[n][capacity]);
        
        List<Item> selected = getSelectedItems(items, capacity);
        System.out.println("Selected items: " + selected);
        
        int totalWeight = selected.stream().mapToInt(item -> item.weight).sum();
        int totalValue = selected.stream().mapToInt(item -> item.value).sum();
        System.out.println("Total weight: " + totalWeight + "/" + capacity);
        System.out.println("Total value: " + totalValue);
    }
    
    public static void main(String[] args) {
        Item[] items = {
            new Item(1, 1, "Item1"),
            new Item(3, 4, "Item2"), 
            new Item(4, 5, "Item3"),
            new Item(5, 7, "Item4")
        };
        int capacity = 7;
        
        System.out.println("Items available:");
        for (Item item : items) {
            System.out.println("  " + item);
        }
        System.out.println("Knapsack capacity: " + capacity);
        
        System.out.println("\nResults:");
        System.out.println("Memoization: " + knapsackMemo(items, capacity));
        System.out.println("Tabulation: " + knapsackTabulation(items, capacity));
        System.out.println("Optimized: " + knapsackOptimized(items, capacity));
        
        System.out.println();
        visualizeKnapsack(items, capacity);
    }
}
```

### Example 5: Coin Change Problem (1D DP)

```java
import java.util.*;

/**
 * Problem: Find minimum number of coins to make amount (or count ways)
 * 
 * Two variations:
 * 1. Minimum coins needed
 * 2. Number of ways to make amount
 * 
 * Time Complexity: O(amount * coins.length)
 * Space Complexity: O(amount)
 */
public class CoinChangeProblems {
    
    /**
     * Problem 1: Minimum coins to make amount
     */
    public static int coinChangeMinCoins(int[] coins, int amount) {
        if (amount == 0) return 0;
        
        // dp[i] = minimum coins needed to make amount i
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1); // Initialize with impossible value
        dp[0] = 0; // Base case: 0 coins for amount 0
        
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }
        
        return dp[amount] > amount ? -1 : dp[amount];
    }
    
    /**
     * Problem 2: Number of ways to make amount
     */
    public static int coinChangeWays(int[] coins, int amount) {
        // dp[i] = number of ways to make amount i
        int[] dp = new int[amount + 1];
        dp[0] = 1; // One way to make 0: use no coins
        
        // For each coin
        for (int coin : coins) {
            // Update all amounts that can use this coin
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }
        
        return dp[amount];
    }
    
    /**
     * Get actual coin combination for minimum coins
     */
    public static List<Integer> getCoinCombination(int[] coins, int amount) {
        if (amount == 0) return new ArrayList<>();
        
        int[] dp = new int[amount + 1];
        int[] parent = new int[amount + 1]; // To track which coin was used
        Arrays.fill(dp, amount + 1);
        Arrays.fill(parent, -1);
        dp[0] = 0;
        
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i && dp[i - coin] + 1 < dp[i]) {
                    dp[i] = dp[i - coin] + 1;
                    parent[i] = coin;
                }
            }
        }
        
        if (dp[amount] > amount) return null; // Impossible
        
        // Reconstruct solution
        List<Integer> result = new ArrayList<>();
        int current = amount;
        while (current > 0) {
            int coinUsed = parent[current];
            result.add(coinUsed);
            current -= coinUsed;
        }
        
        return result;
    }
    
    /**
     * Top-Down memoization for minimum coins
     */
    public static int coinChangeMinCoinsMemo(int[] coins, int amount) {
        Integer[] memo = new Integer[amount + 1];
        int result = coinHelper(coins, amount, memo);
        return result == Integer.MAX_VALUE ? -1 : result;
    }
    
    private static int coinHelper(int[] coins, int amount, Integer[] memo) {
        if (amount == 0) return 0;
        if (amount < 0) return Integer.MAX_VALUE;
        
        if (memo[amount] != null) return memo[amount];
        
        int minCoins = Integer.MAX_VALUE;
        for (int coin : coins) {
            int subResult = coinHelper(coins, amount - coin, memo);
            if (subResult != Integer.MAX_VALUE) {
                minCoins = Math.min(minCoins, subResult + 1);
            }
        }
        
        memo[amount] = minCoins;
        return minCoins;
    }
    
    /**
     * Visualize DP table for minimum coins
     */
    public static void visualizeMinCoins(int[] coins, int amount) {
        System.out.println("Minimum Coins DP Table:");
        System.out.println("Coins: " + Arrays.toString(coins));
        System.out.println("Target amount: " + amount);
        
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);
        dp[0] = 0;
        
        System.out.printf("%8s", "Amount");
        for (int i = 0; i <= amount; i++) {
            System.out.printf("%4d", i);
        }
        System.out.println();
        
        System.out.printf("%8s", "MinCoins");
        System.out.printf("%4d", 0);
        
        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
            System.out.printf("%4s", dp[i] > amount ? "∞" : String.valueOf(dp[i]));
        }
        System.out.println();
        
        if (dp[amount] <= amount) {
            List<Integer> combination = getCoinCombination(coins, amount);
            System.out.println("Optimal combination: " + combination);
        } else {
            System.out.println("Amount cannot be made with given coins");
        }
    }
    
    /**
     * Visualize DP table for counting ways
     */
    public static void visualizeCountWays(int[] coins, int amount) {
        System.out.println("\nCounting Ways DP Table:");
        
        int[] dp = new int[amount + 1];
        dp[0] = 1;
        
        System.out.printf("%12s", "Amount");
        for (int i = 0; i <= amount; i++) {
            System.out.printf("%4d", i);
        }
        System.out.println();
        
        System.out.printf("%12s", "Initial");
        for (int i = 0; i <= amount; i++) {
            System.out.printf("%4d", dp[i]);
        }
        System.out.println();
        
        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
            
            System.out.printf("%12s", "After coin " + coin);
            for (int i = 0; i <= amount; i++) {
                System.out.printf("%4d", dp[i]);
            }
            System.out.println();
        }
        
        System.out.println("Total ways: " + dp[amount]);
    }
    
    public static void main(String[] args) {
        int[] coins = {1, 3, 4};
        int amount = 6;
        
        System.out.println("=== Coin Change Problems ===");
        System.out.println("Coins: " + Arrays.toString(coins));
        System.out.println("Amount: " + amount);
        
        // Minimum coins
        System.out.println("\n1. Minimum coins needed:");
        int minCoins = coinChangeMinCoins(coins, amount);
        System.out.println("Tabulation: " + minCoins);
        System.out.println("Memoization: " + coinChangeMinCoinsMemo(coins, amount));
        
        List<Integer> combination = getCoinCombination(coins, amount);
        System.out.println("Coin combination: " + combination);
        
        // Count ways
        System.out.println("\n2. Number of ways:");
        int ways = coinChangeWays(coins, amount);
        System.out.println("Total ways: " + ways);
        
        // Visualizations
        visualizeMinCoins(coins, amount);
        visualizeCountWays(coins, amount);
    }
}
```

### Example 6: House Robber (1D DP)

```java
/**
 * Problem: Rob houses to get maximum money without robbing adjacent houses
 * 
 * DP[i] = maximum money that can be robbed up to house i
 * DP[i] = max(DP[i-1], DP[i-2] + nums[i])
 * 
 * Time Complexity: O(n)
 * Space Complexity: O(1) optimized
 */
public class HouseRobber {
    
    /**
     * Bottom-Up Tabulation
     */
    public static int robTabulation(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        if (nums.length == 1) return nums[0];
        
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        dp[1] = Math.max(nums[0], nums[1]);
        
        for (int i = 2; i < nums.length; i++) {
            // Either rob current house + max from i-2, or don't rob (take i-1)
            dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
        }
        
        return dp[nums.length - 1];
    }
    
    /**
     * Space-Optimized O(1)
     */
    public static int robOptimized(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        if (nums.length == 1) return nums[0];
        
        int prev2 = nums[0];                    // Maximum up to i-2
        int prev1 = Math.max(nums[0], nums[1]); // Maximum up to i-1
        
        for (int i = 2; i < nums.length; i++) {
            int current = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = current;
        }
        
        return prev1;
    }
    
    /**
     * Top-Down Memoization
     */
    public static int robMemo(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        Integer[] memo = new Integer[nums.length];
        return robHelper(nums, 0, memo);
    }
    
    private static int robHelper(int[] nums, int index, Integer[] memo) {
        if (index >= nums.length) return 0;
        
        if (memo[index] != null) return memo[index];
        
        // Choice 1: Rob current house + optimal from index+2
        int robCurrent = nums[index] + robHelper(nums, index + 2, memo);
        
        // Choice 2: Skip current house, optimal from index+1
        int skipCurrent = robHelper(nums, index + 1, memo);
        
        memo[index] = Math.max(robCurrent, skipCurrent);
        return memo[index];
    }
    
    /**
     * House Robber II: Houses are in a circle (first and last are adjacent)
     */
    public static int robCircular(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        if (nums.length == 1) return nums[0];
        if (nums.length == 2) return Math.max(nums[0], nums[1]);
        
        // Case 1: Rob houses 0 to n-2 (exclude last)
        int robWithFirst = robRange(nums, 0, nums.length - 2);
        
        // Case 2: Rob houses 1 to n-1 (exclude first)
        int robWithoutFirst = robRange(nums, 1, nums.length - 1);
        
        return Math.max(robWithFirst, robWithoutFirst);
    }
    
    private static int robRange(int[] nums, int start, int end) {
        int prev2 = 0;
        int prev1 = 0;
        
        for (int i = start; i <= end; i++) {
            int current = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = current;
        }
        
        return prev1;
    }
    
    /**
     * Show which houses to rob for maximum profit
     */
    public static List<Integer> getHousesToRob(int[] nums) {
        if (nums == null || nums.length == 0) return new ArrayList<>();
        if (nums.length == 1) return Arrays.asList(0);
        
        int[] dp = new int[nums.length];
        boolean[] robbed = new boolean[nums.length];
        
        dp[0] = nums[0];
        robbed[0] = true;
        
        if (nums[1] > nums[0]) {
            dp[1] = nums[1];
            robbed[1] = true;
        } else {
            dp[1] = nums[0];
            robbed[1] = false;
        }
        
        for (int i = 2; i < nums.length; i++) {
            if (dp[i - 2] + nums[i] > dp[i - 1]) {
                dp[i] = dp[i - 2] + nums[i];
                robbed[i] = true;
            } else {
                dp[i] = dp[i - 1];
                robbed[i] = false;
            }
        }
        
        // Reconstruct solution
        List<Integer> result = new ArrayList<>();
        for (int i = nums.length - 1; i >= 0; i--) {
            if (robbed[i]) {
                result.add(i);
                i--; // Skip next house
            }
        }
        
        Collections.reverse(result);
        return result;
    }
    
    public static void main(String[] args) {
        int[] houses = {2, 7, 9, 3, 1};
        
        System.out.println("Houses: " + Arrays.toString(houses));
        System.out.println("Max money (Tabulation): " + robTabulation(houses));
        System.out.println("Max money (Optimized): " + robOptimized(houses));
        System.out.println("Max money (Memoization): " + robMemo(houses));
        
        List<Integer> robbedHouses = getHousesToRob(houses);
        System.out.println("Houses to rob: " + robbedHouses);
        
        int totalRobbed = robbedHouses.stream().mapToInt(i -> houses[i]).sum();
        System.out.println("Total money: " + totalRobbed);
        
        // Circular houses
        int[] circularHouses = {2, 3, 2};
        System.out.println("\nCircular houses: " + Arrays.toString(circularHouses));
        System.out.println("Max money (Circular): " + robCircular(circularHouses));
    }
}
```

### Example 7: Edit Distance (2D DP)

```java
/**
 * Problem: Find minimum operations to convert string1 to string2
 * Operations: Insert, Delete, Replace
 * 
 * DP[i][j] = minimum operations to convert string1[0..i-1] to string2[0..j-1]
 * 
 * Time Complexity: O(m*n)
 * Space Complexity: O(m*n)
 */
public class EditDistance {
    
    /**
     * Bottom-Up Tabulation
     */
    public static int minDistance(String word1, String word2) {
        int m = word1.length();
        int n = word2.length();
        
        // dp[i][j] = min operations to convert word1[0..i-1] to word2[0..j-1]
        int[][] dp = new int[m + 1][n + 1];
        
        // Base cases
        for (int i = 0; i <= m; i++) {
            dp[i][0] = i; // Delete all characters from word1
        }
        for (int j = 0; j <= n; j++) {
            dp[0][j] = j; // Insert all characters of word2
        }
        
        // Fill table
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    // Characters match, no operation needed
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    // Take minimum of three operations
                    int replace = dp[i - 1][j - 1] + 1; // Replace
                    int delete = dp[i - 1][j] + 1;      // Delete
                    int insert = dp[i][j - 1] + 1;      // Insert
                    
                    dp[i][j] = Math.min(Math.min(replace, delete), insert);
                }
            }
        }
        
        return dp[m][n];
    }
    
    /**
     * Top-Down Memoization
     */
    public static int minDistanceMemo(String word1, String word2) {
        Integer[][] memo = new Integer[word1.length()][word2.length()];
        return editHelper(word1, word2, 0, 0, memo);
    }
    
    private static int editHelper(String word1, String word2, int i, int j, Integer[][] memo) {
        // Base cases
        if (i == word1.length()) return word2.length() - j; // Insert remaining
        if (j == word2.length()) return word1.length() - i; // Delete remaining
        
        if (memo[i][j] != null) return memo[i][j];
        
        int result;
        if (word1.charAt(i) == word2.charAt(j)) {
            // Characters match, move to next
            result = editHelper(word1, word2, i + 1, j + 1, memo);
        } else {
            // Try all three operations
            int replace = 1 + editHelper(word1, word2, i + 1, j + 1, memo);
            int delete = 1 + editHelper(word1, word2, i + 1, j, memo);
            int insert = 1 + editHelper(word1, word2, i, j + 1, memo);
            
            result = Math.min(Math.min(replace, delete), insert);
        }
        
        memo[i][j] = result;
        return result;
    }
    
    /**
     * Space-Optimized version (O(min(m,n)) space)
     */
    public static int minDistanceOptimized(String word1, String word2) {
        // Ensure word1 is shorter for space optimization
        if (word1.length() > word2.length()) {
            return minDistanceOptimized(word2, word1);
        }
        
        int m = word1.length();
        int n = word2.length();
        
        int[] prev = new int[m + 1];
        int[] curr = new int[m + 1];
        
        // Initialize first row
        for (int i = 0; i <= m; i++) {
            prev[i] = i;
        }
        
        for (int j = 1; j <= n; j++) {
            curr[0] = j; // Insert j characters
            
            for (int i = 1; i <= m; i++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    curr[i] = prev[i - 1];
                } else {
                    curr[i] = Math.min(Math.min(prev[i - 1] + 1, prev[i] + 1), curr[i - 1] + 1);
                }
            }
            
            // Swap arrays
            int[] temp = prev;
            prev = curr;
            curr = temp;
        }
        
        return prev[m];
    }
    
    /**
     * Get actual sequence of operations
     */
    public static List<String> getEditSequence(String word1, String word2) {
        int m = word1.length();
        int n = word2.length();
        int[][] dp = new int[m + 1][n + 1];
        
        // Fill DP table
        for (int i = 0; i <= m; i++) dp[i][0] = i;
        for (int j = 0; j <= n; j++) dp[0][j] = j;
        
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1] + 1, dp[i - 1][j] + 1), dp[i][j - 1] + 1);
                }
            }
        }
        
        // Backtrack to find operations
        List<String> operations = new ArrayList<>();
        int i = m, j = n;
        
        while (i > 0 || j > 0) {
            if (i > 0 && j > 0 && word1.charAt(i - 1) == word2.charAt(j - 1)) {
                // Characters match, no operation
                i--;
                j--;
            } else if (i > 0 && j > 0 && dp[i][j] == dp[i - 1][j - 1] + 1) {
                // Replace operation
                operations.add("Replace '" + word1.charAt(i - 1) + "' with '" + word2.charAt(j - 1) + "' at position " + (i - 1));
                i--;
                j--;
            } else if (i > 0 && dp[i][j] == dp[i - 1][j] + 1) {
                // Delete operation
                operations.add("Delete '" + word1.charAt(i - 1) + "' at position " + (i - 1));
                i--;
            } else {
                // Insert operation
                operations.add("Insert '" + word2.charAt(j - 1) + "' at position " + i);
                j--;
            }
        }
        
        Collections.reverse(operations);
        return operations;
    }
    
    /**
     * Visualize DP table
     */
    public static void visualizeEditDistance(String word1, String word2) {
        int m = word1.length();
        int n = word2.length();
        int[][] dp = new int[m + 1][n + 1];
        
        System.out.println("Edit Distance DP Table:");
        System.out.println("Word1: \"" + word1 + "\" → Word2: \"" + word2 + "\"");
        
        // Fill table
        for (int i = 0; i <= m; i++) dp[i][0] = i;
        for (int j = 0; j <= n; j++) dp[0][j] = j;
        
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.min(Math.min(dp[i - 1][j - 1] + 1, dp[i - 1][j] + 1), dp[i][j - 1] + 1);
                }
            }
        }
        
        // Print table
        System.out.print("     ");
        for (int j = 0; j <= n; j++) {
            if (j == 0) System.out.print("  ");
            else System.out.printf("%2c", word2.charAt(j - 1));
        }
        System.out.println();
        
        for (int i = 0; i <= m; i++) {
            if (i == 0) System.out.print("  ");
            else System.out.printf("%2c", word1.charAt(i - 1));
            
            for (int j = 0; j <= n; j++) {
                System.out.printf("%3d", dp[i][j]);
            }
            System.out.println();
        }
        
        System.out.println("Minimum edit distance: " + dp[m][n]);
        
        List<String> operations = getEditSequence(word1, word2);
        System.out.println("Operations sequence:");
        for (int i = 0; i < operations.size(); i++) {
            System.out.println((i + 1) + ". " + operations.get(i));
        }
    }
    
    public static void main(String[] args) {
        String word1 = "horse";
        String word2 = "ros";
        
        System.out.println("Converting \"" + word1 + "\" to \"" + word2 + "\":");
        System.out.println("Tabulation: " + minDistance(word1, word2));
        System.out.println("Memoization: " + minDistanceMemo(word1, word2));
        System.out.println("Optimized: " + minDistanceOptimized(word1, word2));
        
        System.out.println();
        visualizeEditDistance(word1, word2);
    }
}
```

### Example 8: Maximum Subarray (Kadane's Algorithm - DP Variant)

```java
/**
 * Problem: Find maximum sum of contiguous subarray
 * 
 * Kadane's Algorithm: DP approach
 * maxSoFar[i] = max(nums[i], maxSoFar[i-1] + nums[i])
 * 
 * Time Complexity: O(n)
 * Space Complexity: O(1)
 */
public class MaximumSubarray {
    
    /**
     * Classic Kadane's Algorithm
     */
    public static int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        int maxSoFar = nums[0];    // Maximum sum ending at current position
        int maxEndingHere = nums[0]; // Maximum sum found so far
        
        for (int i = 1; i < nums.length; i++) {
            // Either start new subarray or extend existing one
            maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
            maxSoFar = Math.max(maxSoFar, maxEndingHere);
        }
        
        return maxSoFar;
    }
    
    /**
     * DP Table approach (for visualization)
     */
    public static int maxSubArrayDP(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        int[] dp = new int[nums.length];
        dp[0] = nums[0];
        int maxSum = dp[0];
        
        for (int i = 1; i < nums.length; i++) {
            dp[i] = Math.max(nums[i], dp[i - 1] + nums[i]);
            maxSum = Math.max(maxSum, dp[i]);
        }
        
        return maxSum;
    }
    
    /**
     * Find actual subarray indices with maximum sum
     */
    public static int[] maxSubArrayIndices(int[] nums) {
        if (nums == null || nums.length == 0) return new int[]{-1, -1};
        
        int maxSum = nums[0];
        int currentSum = nums[0];
        int start = 0, end = 0, tempStart = 0;
        
        for (int i = 1; i < nums.length; i++) {
            if (currentSum < 0) {
                currentSum = nums[i];
                tempStart = i;
            } else {
                currentSum += nums[i];
            }
            
            if (currentSum > maxSum) {
                maxSum = currentSum;
                start = tempStart;
                end = i;
            }
        }
        
        return new int[]{start, end, maxSum};
    }
    
    /**
     * Handle all negative numbers case
     */
    public static int maxSubArrayAllNegative(int[] nums) {
        if (nums == null || nums.length == 0) return 0;
        
        int maxElement = nums[0];
        for (int num : nums) {
            maxElement = Math.max(maxElement, num);
        }
        
        // If all negative, return the largest element
        if (maxElement < 0) return maxElement;
        
        // Otherwise use Kadane's algorithm
        return maxSubArray(nums);
    }
    
    /**
     * Maximum subarray with at most k negations allowed
     */
    public static int maxSubArrayWithNegations(int[] nums, int k) {
        int n = nums.length;
        
        // dp[i][j][0] = max sum ending at i with j negations, last element not negated
        // dp[i][j][1] = max sum ending at i with j negations, last element negated
        int[][][] dp = new int[n][k + 1][2];
        
        // Initialize first element
        dp[0][0][0] = nums[0];
        dp[0][1][1] = -nums[0];
        
        int result = Math.max(dp[0][0][0], dp[0][1][1]);
        
        for (int i = 1; i < n; i++) {
            for (int j = 0; j <= k; j++) {
                // Don't negate current element
                dp[i][j][0] = Math.max(nums[i], 
                    Math.max(dp[i-1][j][0] + nums[i], dp[i-1][j][1] + nums[i]));
                
                // Negate current element (if we have negations left)
                if (j > 0) {
                    dp[i][j][1] = Math.max(-nums[i], 
                        Math.max(dp[i-1][j-1][0] - nums[i], dp[i-1][j-1][1] - nums[i]));
                }
                
                result = Math.max(result, Math.max(dp[i][j][0], dp[i][j][1]));
            }
        }
        
        return result;
    }
    
    /**
     * Visualize Kadane's algorithm step by step
     */
    public static void visualizeKadane(int[] nums) {
        System.out.println("Kadane's Algorithm Visualization:");
        System.out.println("Array: " + Arrays.toString(nums));
        System.out.println();
        
        int maxSoFar = nums[0];
        int maxEndingHere = nums[0];
        
        System.out.printf("%5s %10s %15s %12s %10s%n", 
            "Index", "Element", "MaxEndingHere", "MaxSoFar", "Decision");
        System.out.println("─".repeat(65));
        
        System.out.printf("%5d %10d %15d %12d %10s%n", 
            0, nums[0], maxEndingHere, maxSoFar, "Initial");
        
        for (int i = 1; i < nums.length; i++) {
            int prevMaxEndingHere = maxEndingHere;
            maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
            maxSoFar = Math.max(maxSoFar, maxEndingHere);
            
            String decision = (maxEndingHere == nums[i]) ? "Start new" : "Extend";
            
            System.out.printf("%5d %10d %15d %12d %10s%n", 
                i, nums[i], maxEndingHere, maxSoFar, decision);
        }
        
        int[] indices = maxSubArrayIndices(nums);
        System.out.println("\nMaximum subarray: [" + indices[0] + ", " + indices[1] + "]");
        System.out.print("Elements: ");
        for (int i = indices[0]; i <= indices[1]; i++) {
            System.out.print(nums[i] + " ");
        }
        System.out.println("\nSum: " + indices[2]);
    }
    
    public static void main(String[] args) {
        int[] nums1 = {-2, 1, -3, 4, -1, 2, 1, -5, 4};
        
        System.out.println("Array: " + Arrays.toString(nums1));
        System.out.println("Maximum subarray sum: " + maxSubArray(nums1));
        
        int[] indices = maxSubArrayIndices(nums1);
        System.out.println("Subarray indices: [" + indices[0] + ", " + indices[1] + "]");
        System.out.println("Maximum sum: " + indices[2]);
        
        System.out.println();
        visualizeKadane(nums1);
        
        // All negative case
        int[] nums2 = {-3, -2, -5, -1};
        System.out.println("\nAll negative array: " + Arrays.toString(nums2));
        System.out.println("Maximum subarray sum: " + maxSubArrayAllNegative(nums2));
    }
}
```

### Example 9: Unique Paths (2D DP)

```java
/**
 * Problem: Count unique paths from top-left to bottom-right in grid
 * Can only move right or down
 * 
 * DP[i][j] = number of unique paths to reach cell (i,j)
 * DP[i][j] = DP[i-1][j] + DP[i][j-1]
 * 
 * Time Complexity: O(m*n)
 * Space Complexity: O(m*n) or O(min(m,n)) optimized
 */
public class UniquePaths {
    
    /**
     * Basic 2D DP approach
     */
    public static int uniquePaths(int m, int n) {
        int[][] dp = new int[m][n];
        
        // Base cases: first row and first column have only one path
        for (int i = 0; i < m; i++) {
            dp[i][0] = 1;
        }
        for (int j = 0; j < n; j++) {
            dp[0][j] = 1;
        }
        
        // Fill the rest of the table
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        
        return dp[m - 1][n - 1];
    }
    
    /**
     * Space-optimized O(min(m,n)) approach
     */
    public static int uniquePathsOptimized(int m, int n) {
        // Use smaller dimension for the array
        if (m > n) return uniquePathsOptimized(n, m);
        
        int[] dp = new int[m];
        Arrays.fill(dp, 1); // First row is all 1s
        
        for (int j = 1; j < n; j++) {
            for (int i = 1; i < m; i++) {
                dp[i] = dp[i] + dp[i - 1]; // dp[i] + dp[i-1]
            }
        }
        
        return dp[m - 1];
    }
    
    /**
     * Mathematical approach using combinations
     */
    public static int uniquePathsMath(int m, int n) {
        // Total moves needed: (m-1) down + (n-1) right = m+n-2
        // Choose (m-1) positions for down moves: C(m+n-2, m-1)
        
        int totalMoves = m + n - 2;
        int downMoves = m - 1;
        
        // Calculate C(totalMoves, downMoves)
        long result = 1;
        for (int i = 0; i < downMoves; i++) {
            result = result * (totalMoves - i) / (i + 1);
        }
        
        return (int) result;
    }
    
    /**
     * Unique Paths II: With obstacles
     */
    public static int uniquePathsWithObstacles(int[][] obstacleGrid) {
        int m = obstacleGrid.length;
        int n = obstacleGrid[0].length;
        
        // If start or end is blocked
        if (obstacleGrid[0][0] == 1 || obstacleGrid[m - 1][n - 1] == 1) {
            return 0;
        }
        
        int[][] dp = new int[m][n];
        dp[0][0] = 1;
        
        // Fill first row
        for (int j = 1; j < n; j++) {
            dp[0][j] = (obstacleGrid[0][j] == 1) ? 0 : dp[0][j - 1];
        }
        
        // Fill first column
        for (int i = 1; i < m; i++) {
            dp[i][0] = (obstacleGrid[i][0] == 1) ? 0 : dp[i - 1][0];
        }
        
        // Fill rest of the table
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (obstacleGrid[i][j] == 1) {
                    dp[i][j] = 0;
                } else {
                    dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
                }
            }
        }
        
        return dp[m - 1][n - 1];
    }
    
    /**
     * Unique Paths III: Visit all empty squares exactly once
     */
    public static int uniquePathsIII(int[][] grid) {
        int m = grid.length, n = grid[0].length;
        int startX = 0, startY = 0, empty = 1; // Count starting cell as empty
        
        // Find start position and count empty cells
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (grid[i][j] == 1) {
                    startX = i;
                    startY = j;
                } else if (grid[i][j] == 0) {
                    empty++;
                }
            }
        }
        
        return dfsAllPaths(grid, startX, startY, empty);
    }
    
    private static int dfsAllPaths(int[][] grid, int x, int y, int empty) {
        if (x < 0 || x >= grid.length || y < 0 || y >= grid[0].length || grid[x][y] < 0) {
            return 0;
        }
        
        if (grid[x][y] == 2) {
            return empty == 0 ? 1 : 0; // Reached end, check if all cells visited
        }
        
        grid[x][y] = -2; // Mark as visited
        empty--;
        
        int paths = dfsAllPaths(grid, x + 1, y, empty) +
                   dfsAllPaths(grid, x - 1, y, empty) +
                   dfsAllPaths(grid, x, y + 1, empty) +
                   dfsAllPaths(grid, x, y - 1, empty);
        
        grid[x][y] = 0; // Backtrack
        empty++;
        
        return paths;
    }
    
    /**
     * Visualize path counting
     */
    public static void visualizeUniquePaths(int m, int n) {
        System.out.println("Unique Paths Visualization (" + m + "x" + n + " grid):");
        
        int[][] dp = new int[m][n];
        
        // Initialize first row and column
        for (int i = 0; i < m; i++) dp[i][0] = 1;
        for (int j = 0; j < n; j++) dp[0][j] = 1;
        
        // Fill table and visualize
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        
        // Print grid
        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                System.out.printf("%4d", dp[i][j]);
            }
            System.out.println();
        }
        
        System.out.println("Total unique paths: " + dp[m - 1][n - 1]);
        
        // Show mathematical verification
        int mathResult = uniquePathsMath(m, n);
        System.out.println("Mathematical result: " + mathResult);
        System.out.println("Results match: " + (dp[m - 1][n - 1] == mathResult));
    }
    
    /**
     * Show some actual paths (for small grids)
     */
    public static void showSamplePaths(int m, int n) {
        if (m > 3 || n > 3) {
            System.out.println("Grid too large to show all paths");
            return;
        }
        
        System.out.println("Sample paths for " + m + "x" + n + " grid:");
        List<String> paths = new ArrayList<>();
        generateAllPaths(0, 0, m - 1, n - 1, "", paths);
        
        for (int i = 0; i < paths.size(); i++) {
            System.out.println("Path " + (i + 1) + ": " + paths.get(i));
        }
    }
    
    private static void generateAllPaths(int x, int y, int targetX, int targetY, 
                                       String path, List<String> paths) {
        if (x == targetX && y == targetY) {
            paths.add(path);
            return;
        }
        
        if (x < targetX) {
            generateAllPaths(x + 1, y, targetX, targetY, path + "D", paths);
        }
        if (y < targetY) {
            generateAllPaths(x, y + 1, targetX, targetY, path + "R", paths);
        }
    }
    
    public static void main(String[] args) {
        int m = 3, n = 3;
        
        System.out.println("Grid size: " + m + "x" + n);
        System.out.println("Unique paths (DP): " + uniquePaths(m, n));
        System.out.println("Unique paths (Optimized): " + uniquePathsOptimized(m, n));
        System.out.println("Unique paths (Math): " + uniquePathsMath(m, n));
        
        System.out.println();
        visualizeUniquePaths(m, n);
        
        System.out.println();
        showSamplePaths(3, 2);
        
        // With obstacles
        int[][] obstacles = {{0, 0, 0}, {0, 1, 0}, {0, 0, 0}};
        System.out.println("\nWith obstacles:");
        System.out.println("Paths: " + uniquePathsWithObstacles(obstacles));
    }
}
```

### Example 10: Palindrome Partitioning (2D DP)

```java
import java.util.*;

/**
 * Problem: Find minimum cuts needed to partition string into palindromes
 * 
 * DP approach:
 * 1. Precompute palindrome table: isPalindrome[i][j]
 * 2. Calculate minimum cuts: minCuts[i] = min cuts for substring[0..i]
 * 
 * Time Complexity: O(n²)
 * Space Complexity: O(n²)
 */
public class PalindromePartitioning {
    
    /**
     * Find minimum cuts for palindrome partitioning
     */
    public static int minCut(String s) {
        int n = s.length();
        if (n <= 1) return 0;
        
        // Step 1: Precompute palindrome table
        boolean[][] isPalindrome = new boolean[n][n];
        computePalindromeTable(s, isPalindrome);
        
        // Step 2: DP for minimum cuts
        int[] minCuts = new int[n];
        
        for (int i = 0; i < n; i++) {
            if (isPalindrome[0][i]) {
                minCuts[i] = 0; // Whole substring is palindrome
            } else {
                minCuts[i] = i; // Worst case: cut after each character
                
                for (int j = 1; j <= i; j++) {
                    if (isPalindrome[j][i]) {
                        minCuts[i] = Math.min(minCuts[i], minCuts[j - 1] + 1);
                    }
                }
            }
        }
        
        return minCuts[n - 1];
    }
    
    /**
     * Compute palindrome table using DP
     */
    private static void computePalindromeTable(String s, boolean[][] isPalindrome) {
        int n = s.length();
        
        // Single characters are palindromes
        for (int i = 0; i < n; i++) {
            isPalindrome[i][i] = true;
        }
        
        // Check for palindromes of length 2
        for (int i = 0; i < n - 1; i++) {
            isPalindrome[i][i + 1] = (s.charAt(i) == s.charAt(i + 1));
        }
        
        // Check for palindromes of length 3 and more
        for (int len = 3; len <= n; len++) {
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                isPalindrome[i][j] = (s.charAt(i) == s.charAt(j)) && isPalindrome[i + 1][j - 1];
            }
        }
    }
    
    /**
     * Find all possible palindrome partitions
     */
    public static List<List<String>> partition(String s) {
        List<List<String>> result = new ArrayList<>();
        List<String> current = new ArrayList<>();
        
        boolean[][] isPalindrome = new boolean[s.length()][s.length()];
        computePalindromeTable(s, isPalindrome);
        
        backtrackPartitions(s, 0, current, result, isPalindrome);
        return result;
    }
    
    private static void backtrackPartitions(String s, int start, List<String> current, 
                                           List<List<String>> result, boolean[][] isPalindrome) {
        if (start >= s.length()) {
            result.add(new ArrayList<>(current));
            return;
        }
        
        for (int end = start; end < s.length(); end++) {
            if (isPalindrome[start][end]) {
                current.add(s.substring(start, end + 1));
                backtrackPartitions(s, end + 1, current, result, isPalindrome);
                current.remove(current.size() - 1);
            }
        }
    }
    
    /**
     * Get one optimal partitioning with minimum cuts
     */
    public static List<String> getOptimalPartition(String s) {
        int n = s.length();
        if (n <= 1) return Arrays.asList(s);
        
        boolean[][] isPalindrome = new boolean[n][n];
        computePalindromeTable(s, isPalindrome);
        
        int[] minCuts = new int[n];
        String[] partition = new String[n]; // To store the actual partitions
        
        for (int i = 0; i < n; i++) {
            if (isPalindrome[0][i]) {
                minCuts[i] = 0;
                partition[i] = s.substring(0, i + 1);
            } else {
                minCuts[i] = i;
                partition[i] = s.substring(i, i + 1);
                
                for (int j = 1; j <= i; j++) {
                    if (isPalindrome[j][i] && minCuts[j - 1] + 1 < minCuts[i]) {
                        minCuts[i] = minCuts[j - 1] + 1;
                        partition[i] = s.substring(j, i + 1);
                    }
                }
            }
        }
        
        // Reconstruct the partition
        List<String> result = new ArrayList<>();
        int i = n - 1;
        while (i >= 0) {
            String part = partition[i];
            result.add(part);
            i -= part.length();
        }
        
        Collections.reverse(result);
        return result;
    }
    
    /**
     * Optimized space version
     */
    public static int minCutOptimized(String s) {
        int n = s.length();
        if (n <= 1) return 0;
        
        int[] minCuts = new int[n];
        
        for (int i = 0; i < n; i++) {
            minCuts[i] = i; // Worst case initialization
            
            // Check for odd length palindromes centered at i
            for (int left = i, right = i; left >= 0 && right < n && s.charAt(left) == s.charAt(right); left--, right++) {
                if (left == 0) {
                    minCuts[right] = 0;
                } else {
                    minCuts[right] = Math.min(minCuts[right], minCuts[left - 1] + 1);
                }
            }
            
            // Check for even length palindromes centered between i and i+1
            for (int left = i, right = i + 1; left >= 0 && right < n && s.charAt(left) == s.charAt(right); left--, right++) {
                if (left == 0) {
                    minCuts[right] = 0;
                } else {
                    minCuts[right] = Math.min(minCuts[right], minCuts[left - 1] + 1);
                }
            }
        }
        
        return minCuts[n - 1];
    }
    
    /**
     * Visualize palindrome table and cuts
     */
    public static void visualizePalindromePartitioning(String s) {
        int n = s.length();
        boolean[][] isPalindrome = new boolean[n][n];
        computePalindromeTable(s, isPalindrome);
        
        System.out.println("Palindrome Partitioning Analysis for: \"" + s + "\"");
        System.out.println();
        
        // Show palindrome table
        System.out.println("Palindrome Table (T = palindrome, F = not palindrome):");
        System.out.print("   ");
        for (int j = 0; j < n; j++) {
            System.out.printf("%2c", s.charAt(j));
        }
        System.out.println();
        
        for (int i = 0; i < n; i++) {
            System.out.printf("%2c ", s.charAt(i));
            for (int j = 0; j < n; j++) {
                if (j < i) {
                    System.out.print("  ");
                } else {
                    System.out.print(isPalindrome[i][j] ? " T" : " F");
                }
            }
            System.out.println();
        }
        
        // Show DP process
        System.out.println("\nMinimum Cuts DP Process:");
        int[] minCuts = new int[n];
        
        System.out.printf("%8s %10s %20s%n", "Position", "MinCuts", "Reasoning");
        System.out.println("─".repeat(50));
        
        for (int i = 0; i < n; i++) {
            if (isPalindrome[0][i]) {
                minCuts[i] = 0;
                System.out.printf("%8d %10d %20s%n", i, minCuts[i], "Whole substring is palindrome");
            } else {
                minCuts[i] = i;
                
                for (int j = 1; j <= i; j++) {
                    if (isPalindrome[j][i]) {
                        minCuts[i] = Math.min(minCuts[i], minCuts[j - 1] + 1);
                    }
                }
                
                System.out.printf("%8d %10d %20s%n", i, minCuts[i], "Optimal partitioning");
            }
        }
        
        System.out.println("\nMinimum cuts needed: " + minCuts[n - 1]);
        
        List<String> optimalPartition = getOptimalPartition(s);
        System.out.println("Optimal partition: " + optimalPartition);
        
        // Show all possible partitions for small strings
        if (n <= 6) {
            List<List<String>> allPartitions = partition(s);
            System.out.println("\nAll possible palindrome partitions:");
            for (int i = 0; i < allPartitions.size(); i++) {
                System.out.println((i + 1) + ". " + allPartitions.get(i) + " (cuts: " + (allPartitions.get(i).size() - 1) + ")");
            }
        }
    }
    
    public static void main(String[] args) {
        String s1 = "aab";
        System.out.println("String: \"" + s1 + "\"");
        System.out.println("Minimum cuts: " + minCut(s1));
        System.out.println("Minimum cuts (optimized): " + minCutOptimized(s1));
        
        System.out.println();
        visualizePalindromePartitioning(s1);
        
        // Another example
        String s2 = "racecar";
        System.out.println("\n" + "=".repeat(60));
        System.out.println("String: \"" + s2 + "\"");
        System.out.println("Minimum cuts: " + minCut(s2));
        System.out.println("Optimal partition: " + getOptimalPartition(s2));
    }
}
```

---

## 6) Optimization Techniques

### 6.1 Space Optimization Patterns

```
Space Optimization Strategies:
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│  1. Rolling Array (1D DP):                                                     │
│     dp[i] = f(dp[i-1], dp[i-2])  →  Use two variables instead of array        │
│                                                                                 │
│  2. Dimension Reduction (2D DP):                                               │
│     dp[i][j] = f(dp[i-1][j], dp[i][j-1])  →  Use two 1D arrays                │
│                                                                                 │
│  3. In-place Modification:                                                     │
│     Modify input array instead of creating new DP table                        │
│                                                                                 │
│  4. State Compression:                                                         │
│     Use bit manipulation for boolean states                                    │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 6.2 Time Optimization Techniques

```java
/**
 * Matrix Chain Multiplication - Optimization example
 */
public class MatrixChainOptimizations {
    
    // Standard O(n³) DP
    public static int matrixChainOrder(int[] p) {
        int n = p.length - 1;
        int[][] dp = new int[n][n];
        
        for (int len = 2; len <= n; len++) {
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                dp[i][j] = Integer.MAX_VALUE;
                
                for (int k = i; k < j; k++) {
                    int cost = dp[i][k] + dp[k+1][j] + p[i]*p[k+1]*p[j+1];
                    dp[i][j] = Math.min(dp[i][j], cost);
                }
            }
        }
        
        return dp[0][n-1];
    }
    
    // Memoized version to avoid redundant calculations
    public static int matrixChainMemo(int[] p) {
        int n = p.length - 1;
        Integer[][] memo = new Integer[n][n];
        return mcHelper(p, 0, n-1, memo);
    }
    
    private static int mcHelper(int[] p, int i, int j, Integer[][] memo) {
        if (i >= j) return 0;
        
        if (memo[i][j] != null) return memo[i][j];
        
        int min = Integer.MAX_VALUE;
        for (int k = i; k < j; k++) {
            int cost = mcHelper(p, i, k, memo) + 
                      mcHelper(p, k+1, j, memo) + 
                      p[i] * p[k+1] * p[j+1];
            min = Math.min(min, cost);
        }
        
        memo[i][j] = min;
        return min;
    }
}
```

---

## 7) Common Pitfalls

### 7.1 State Definition Errors

```java
// ❌ Wrong: Incorrect state definition
// Problem: Longest increasing subsequence
int[] dp = new int[n]; // dp[i] = LIS ending exactly at i
// This doesn't give us the overall LIS length directly

// ✅ Correct: Proper state definition  
int[] dp = new int[n]; // dp[i] = length of LIS ending at position i
int maxLength = 0;
for (int i = 0; i < n; i++) {
    maxLength = Math.max(maxLength, dp[i]); // Find maximum
}
```

### 7.2 Base Case Initialization

```java
// ❌ Wrong: Incorrect base case
int[][] dp = new int[m][n];
// Forgot to initialize base cases

// ✅ Correct: Proper base case initialization
int[][] dp = new int[m][n];
for (int i = 0; i < m; i++) dp[i][0] = 1; // First column
for (int j = 0; j < n; j++) dp[0][j] = 1; // First row
```

### 7.3 Recurrence Relation Mistakes

```java
// ❌ Wrong: Incorrect recurrence
// Knapsack problem
dp[i][w] = Math.max(dp[i-1][w], dp[i-1][w] + values[i]); // Missing weight check

// ✅ Correct: Proper recurrence with weight check
if (weights[i-1] <= w) {
    dp[i][w] = Math.max(dp[i-1][w], dp[i-1][w-weights[i-1]] + values[i-1]);
} else {
    dp[i][w] = dp[i-1][w];
}
```

---

## 8) Advanced DP Patterns

### 8.1 Bitmask DP

```java
/**
 * Traveling Salesman Problem using Bitmask DP
 */
public class TSPBitmaskDP {
    
    public static int tsp(int[][] dist) {
        int n = dist.length;
        int VISITED_ALL = (1 << n) - 1;
        
        // dp[mask][i] = minimum cost to visit all cities in mask ending at city i
        int[][] dp = new int[1 << n][n];
        
        // Initialize with maximum values
        for (int i = 0; i < (1 << n); i++) {
            Arrays.fill(dp[i], Integer.MAX_VALUE);
        }
        
        // Starting at city 0
        dp[1][0] = 0; // Only city 0 visited, cost = 0
        
        for (int mask = 0; mask < (1 << n); mask++) {
            for (int u = 0; u < n; u++) {
                if ((mask & (1 << u)) == 0) continue; // u not in mask
                if (dp[mask][u] == Integer.MAX_VALUE) continue;
                
                for (int v = 0; v < n; v++) {
                    if (mask & (1 << v)) continue; // v already visited
                    
                    int newMask = mask | (1 << v);
                    dp[newMask][v] = Math.min(dp[newMask][v], dp[mask][u] + dist[u][v]);
                }
            }
        }
        
        // Find minimum cost to return to starting city
        int result = Integer.MAX_VALUE;
        for (int i = 1; i < n; i++) {
            if (dp[VISITED_ALL][i] != Integer.MAX_VALUE) {
                result = Math.min(result, dp[VISITED_ALL][i] + dist[i][0]);
            }
        }
        
        return result;
    }
}
```

### 8.2 Digit DP

```java
/**
 * Count numbers with specific digit properties
 */
public class DigitDP {
    
    public static int countNumbersWithSumDigits(int n, int targetSum) {
        String num = String.valueOf(n);
        Integer[][][] memo = new Integer[num.length()][targetSum + 1][2];
        
        return solve(num, 0, targetSum, true, memo);
    }
    
    private static int solve(String num, int pos, int sum, boolean tight, Integer[][][] memo) {
        if (pos == num.length()) {
            return sum == 0 ? 1 : 0;
        }
        
        int tightInt = tight ? 1 : 0;
        if (memo[pos][sum][tightInt] != null) {
            return memo[pos][sum][tightInt];
        }
        
        int limit = tight ? (num.charAt(pos) - '0') : 9;
        int result = 0;
        
        for (int digit = 0; digit <= limit; digit++) {
            if (sum - digit >= 0) {
                result += solve(num, pos + 1, sum - digit, 
                               tight && (digit == limit), memo);
            }
        }
        
        memo[pos][sum][tightInt] = result;
        return result;
    }
}
```

---

This comprehensive Dynamic Programming guide provides you with:

✅ **Core Concepts** - What DP is and when to use it  
✅ **Visual Diagrams** - Understanding through visualization  
✅ **10 Famous Problems** - Complete Java implementations with explanations  
✅ **Optimization Techniques** - Space and time optimizations  
✅ **Common Pitfalls** - Mistakes to avoid  
✅ **Advanced Patterns** - Bitmask DP, Digit DP, and more  

Each example includes multiple approaches (memoization, tabulation, space-optimized), detailed explanations, visualization methods, and practical applications. The guide serves as both a learning resource and a quick reference for implementing dynamic programming solutions efficiently.

