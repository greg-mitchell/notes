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