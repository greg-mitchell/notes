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

- Runtime `O(n log (n) * m log(m))`

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
