# Basics

This covers essential syntax, built-ins, and other non-domain-specific tricks.

## Time/Space Complexity Guide

```text
# Common time complexities
O(1)        # Constant: hash table lookups
O(log n)    # Logarithmic: binary search
O(n)        # Linear: single loop
O(n log n)  # Linearithmic: sorting
O(n²)       # Quadratic: nested loops
O(2ⁿ)       # Exponential: recursive without memoization
```

## Data Structures

```python
# Hash Maps for O(1) lookups
from collections import defaultdict, Counter
counts = Counter(['a', 'a', 'b'])  # Counter({'a': 2, 'b': 1})
graph = defaultdict(list)  # Automatic initialization of empty lists

# Heaps for priority queues
import heapq
heap = []
heapq.heappush(heap, (priority, item))
priority, item = heapq.heappop(heap)

# Deque for O(1) operations at both ends
from collections import deque
queue = deque()
queue.append(x)      # Add right
queue.appendleft(x)  # Add left
queue.pop()          # Remove right
queue.popleft()      # Remove left
```

## List Operations

The sort operations all take a `key=` parameter, a function to extract the sort key from each
element.

```python
# List operations
arr.sort()           # O(n log n), in-place
sorted(arr)          # O(n log n)
arr.reverse()        # O(n), in-place
reversed(arr)        # O(1) - returns iterator
arr.insert(i, x)     # O(n)
bisect.bisect_left(arr, x)  # O(log n) for binary search
```

## Set Operations

```python
s1 & s2              # Intersection
s1 | s2              # Union
s1 - s2              # Difference
s1 ^ s2              # Symmetric difference
```

## Math

- Exponentiation: `**`, `pow(base, exponent)`
- Sqrt: `4 ** .5`, `math.sqrt(4)`
- Complex numbers: `5 + 1j * 3`
- To Integer: `5 // 2`, `math.floor(3.5)`
- Trig: `math.sin()`, `math.cosh()`
- Combinatorics: `itertools.combinations(iterable, r)`, `math.perm(n, k)`, `math.factorial(n)`

## Bit Manipulation

These operations only work on integers.

```python
# Common operations
n & (n-1)    # Remove rightmost set bit
n & -n       # Isolate rightmost set bit
x ^ y        # XOR for finding unique elements
n >> 1       # Divide by 2
n << 1       # Multiply by 2
bin(n).count('1')  # Count set bits
```
