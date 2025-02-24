# Techniques

Collection of problem-solving techniques that don't fit neatly in the larger topics.

## Dynamic Programming

Signs of a DP problem:

- Without memoization, runtime is exponential.
- The problem at `n` is reducible to `step + solution(n-1)`. THIS IS CRUCIAL. Expanding on that:
  - There exists a base case.
  - Each non-base case step relies on a previous value (smaller problem).
  - There's no backtracking. E.g. if the intermediate value could be inapplicable because of a
    condition that's only discovered later, then DP is inapplicable.

```python
# Top-down with memoization
def fibonacci(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fibonacci(n-1, memo) + fibonacci(n-2, memo)
    return memo[n]

# Bottom-up tabulation
def fibonacci_bottom_up(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[1] = 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n]
```

DP in more than one dimension is possible. Common problems might involve going from the top-left
to bottom-right of an array, counting something along the way.

## Sum of Pairs / Two Pointers Technique

A classic interview problem involving some condition about a pair of elements in an array.
Naively, considering every pair of elements would take `O(n**2)` time. A more efficient approach
is to define pointers at opposite ends of a sorted array, and move one or the other pointer inwards
using the result of comparing the pair. This avoids considering impossible pairs, and has `O(n)`
runtime.

```python
def two_sum_sorted(numbers, target):
    left, right = 0, len(numbers) - 1
    while left < right:
        curr_sum = numbers[left] + numbers[right]
        if curr_sum == target:
            return [left, right]
        elif curr_sum < target:
            left += 1
        else:
            right -= 1
```

Common twists on it include:

- Find the closest to target: keep track of best so far.
- Compare pairs from two arrays: combine into one sorted array, plus track the source for each
  element.
