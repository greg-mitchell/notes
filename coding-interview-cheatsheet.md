# Coding Interview Cheatsheet

These are my notes on common tricks and gotchas for solving coding problems, with implementations in Python.

## Strings

Common subproblems, with a snippet solution and complexity analysis.

### ASCII Value

`ord('a')` => `97`

`chr(97)` => `a`

### Substrings

Generate all substrings for a string `s` with length `n`:

- Runtime `O(n^2)`
- Memory `O(n^2)`

```python
[s[i:j] for i in range(0, len(s)) for j in range(i+1, len(s)+1)]
```

Note the `+1` for the `j` range because both Python slice _and_ `range` have exclusive upper bounds.

#### Generator

Constant runtime using generators.

```python
from collections.abc import Iterator

def generate_substrings(s: str) -> Iterator[str]:
    for i in range(0, len(s)):
        for j in range(i+1, len(s)+1):
            yield s[i:j]

```

### Anagrams

Anagrams are strings that can be rearranged to each other. The simplest solution is to make a
canonical order by sorting lexographically.

A common problem is to identify anagrams for a sequence of strings, length n, where each string
is of at most length m.

- Runtime `O(n log (n) + n * m log(m))`. The problem may have assumptions like `n >> m` that would allow simplification.

```python
ss = ['abc', 'bca', 'lol']

from itertools import groupby

anagrams = [list(g) for _canonical, g in groupby(sorted(ss, key=sorted), key=sorted)]
```

This implementation is simple but inefficient: it sorts each string twice, and often strings
are amenable to a linear sort algorithm, [Counting Sort](#counting-sort), for example if limited
to ASCII.

## Grids

Many problems involve parsing and operating on 2D grids. For data modeling, see the example in [Types](python-types.md#grid-example).

### Jagged Grid Parsing

The most common representation is a row-major jagged array, i.e. each row is a list.
Assuming the grid is a multiline string `s` with possible whitespace between cells,
and each cell is a number.

This representation consumes `O(m * n)` space, where `m` is the number of rows, and `n` is the
number of columns.

```python
[[int(c) for c in row.split()] for row in s.strip().splitlines()]
```

### Complex Number Representation

An alternative that's well suited to sparse matrices, especially when you have to examine neighbors,
is to represent the matrix as a dictionary of complex numbers to value, where the complex number
is a position.

Finding neighbors simple: add a basis vector to the position.

This consumes `O(v)` space, where `v` is the number of non-default values in the `m x n` matrix.

```python
from collections import defaultdict
from collections.abc import Mapping

type Pos = complex
type Val = str

# Uses None as the default value
g: Mapping[Pos, Val] = defaultdict()

# some 2D grid where '.' represents empty
s = """
a..
.bc
"""

dim = (2,3)

for i, row in enumerate(s.strip().splitlines()):
    for j, v in enumerate(row):
        if v == '.':
            continue
        g[i + 1j * j] = v

nondiag_neighbor_basis_vecs = [
    -1j,    # left
    +1j,    # right
    -1+0j,  # up
    +1+0j,  # down
]

count_items_with_neighbors = 0
for pos, val in g.items():
    neighbors = [g[pos + basis]
        for basis in nondiag_neighbor_basis_vecs
        if (pos + basis) in g
    ]
    print(pos, neighbors)
    count_items_with_neighbors += len(neighbors)

print(count_items_with_neighbors)
```

## Sorting

### Counting Sort

Essentially a memory-compute tradeoff for sorting, Counting Sort allows linear-time sorting for
items with a fixed domain.

For ASCII strings, for example:

```python
def sort_ascii(s: str) -> str:
    # standard ascii is (0, 127), with printable chars starting at 32
    lower = 0
    upper = 128
    char_buckets = [0 for i in range(lower, upper)]
    result = []
    for c in s:
        char_buckets[ord(c)] += 1
    for c_ord, count in enumerate(char_buckets):
        c = chr(c_ord)
        result.extend([c for i in range(count)])
    return ''.join(result)
```

Counting sort is a special case of Radix sort. Radix sort would move items into buckets by
iterating through the input sequence.

## Math

- Exponentiation: `**`, `pow(base, exponent)`
- Sqrt: `4 ** .5`, `math.sqrt(4)`
- Complex numbers: `5 + 1j * 3`
- To Integer: `5 // 2`, `math.floor(3.5)`
- Trig: `math.sin()`, `math.cosh()`
- Combinatorics: `itertools.combinations(iterable, r)`, `math.perm(n, k)`, `math.factorial(n)`

## Unit Tests

Many coding interviews like to see basic unit testing.

```python
import unittest

class TestFoo(unittest.TestCase):
    def test_condition_one(self):
        # code under test
        actual = (lambda: 'a')()
        self.assertEqual(actual, 'a')

if __name__ == '__main__':
    unittest.main()
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

## Graphs

Graphs are common for problems about relationships between entities, and for networks (like roads).
Generally, graphs have a set of Vertices **V** and Edges **E** between those vertices, or
equivalently nodes and arcs. Common properties of graphs:

- Undirected vs Directed
- Weighted
- Acyclic. Implies sparsity.
- Dense or complete. Implies cycles.

### Adjacency List & Matrix

The most common representation is an adjacency list, which is space efficient for sparse
graphs and supports efficient iteration through neighbors. This requires that Node be hashable.
If it's not, you can define an ordering and use the index.

- Space `O(V + E)`

```python
type Node = any
type Graph = dict[Node, list[Node]]
```

Adjacency Matrices are more space efficient for dense graphs, since `|E| <= |V|**2`. Typically
you define an ordering over the vertices, then build a `V x V` boolean matrix, where `True` marks
an edge.

The following algorithms assume we're using the above Adjacency List representation.

### Search

This is the problem of finding a path from source to target in the graph, used as the basis for
many algorithms which build orderings from graphs. The complexity of BFS and DFS are the same,
but they have different applications.

- Runtime: `O(V + E)`
- Space: `O(V)`

#### BFS

- Processes nodes in level order and guarantees finding the shortest path if one exists.
- Better for infinite graphs with bounded degree, like games.

```python
def bfs(graph: Graph, source: Node, target: Node) -> bool:
    queue: deque[Node] = deque([source])
    seen: set[Node] = {source}
    while queue:
        node = queue.popleft()  # only difference from DFS
        if node == target:
            return True
        for neighbor in graph[node]:
            if neighbor not in seen:
                seen.add(neighbor)
                queue.append(neighbor)
    return False
```

#### DFS

- Processes each path to the end.
- Basis for heuristic-based approaches to speed up graph exploration, like A*.
- Basis for finding a in-, pre-, post-, and toplogical ordering (reverse postorder).

```python
def dfs(graph: Graph, source: Node, target: Node) -> bool:
    stack: deque[Node] = deque([source])
    seen: set[Node] = {source}
    while stack:
        node = stack.pop()  # only difference from BFS
        if node == target:
            return True
        for neighbor in graph[node]:
            if neighbor not in seen:
                seen.add(neighbor)
                stack.append(neighbor)
    return False
```

It's common to see DFS implemented recursively, but since Python (and most non-functional languages)
don't have tail-call recursion and have limited stack size and depth, that approach is less 
preferred due to consuming stack frames, which could result in OOMs for large graphs even when
there is plenty of system memory.

### Binary Tree Operations

Binary trees are DAGs that have 0 to 2 out edges (children), and each interior node is complete.
The most common is a Binary Search Tree, a Binary tree where an inorder traversal of nodes visits
them in sorted order. Equivalently, each left child is `<=` to its parent, and each right child is
`>` its parent.

The typical recursive definition is:

```python
from __future__ import annotations  # required for self-referential types
from dataclasses import dataclass, field

@dataclass
class TreeNode:
    val: any
    left: TreeNode | None = None
    right: TreeNode | None = None
```

A common operation is to define an ordering on a tree.

#### Inorder / Preorder / Postorder Traversal

This uses recursive DFS for simplicity, but nonrecursive works too.

```python
from collections.abc import Callable

type Processor = Callable[[Node], None]

def visit(
    root: TreeNode | None,
    preorder: Processor,
    inorder: Processor,
    postorder: Processor,
) -> None:
    if not root:
        return
    preorder(root)
    visit(root.left, preorder, inorder, postorder)
    inorder(root)
    visit(root.right, preorder, inorder, postorder)
    postorder(root.right)
```

#### Level order traversal

Basically BFS, where you return a list of each level.

```python
def level_order(root: TreeNode | None) -> list[list[Node]]:
    if not root:
        return []
    result: list[list[Node]] = []
    queue = deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):
            node = queue.popleft()
            level.append(node.val)
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
        result.append(level)
    return result
```

## Dynamic Programming

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

## Common Built-in Functions and their Complexity

```python
# List operations
arr.sort()           # O(n log n)
sorted(arr)          # O(n log n)
arr.reverse()        # O(n)
reversed(arr)        # O(1) - returns iterator
arr.insert(i, x)     # O(n)
bisect.bisect_left(arr, x)  # O(log n) for binary search

# Set operations
s1 & s2              # Intersection
s1 | s2              # Union
s1 - s2              # Difference
s1 ^ s2              # Symmetric difference
```

The sort operations all take a `key=` parameter, a function to extract the sort key from each
element.

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
