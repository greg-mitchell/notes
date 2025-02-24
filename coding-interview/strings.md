
# Strings

## ASCII Value

`ord('a')` => `97`

`chr(97)` => `a`

## Substrings

Generate all substrings for a string `s` with length `n`:

- Runtime `O(n^2)`
- Memory `O(n^2)`

```python
[s[i:j] for i in range(0, len(s)) for j in range(i+1, len(s)+1)]
```

Note the `+1` for the `j` range because both Python slice _and_ `range` have exclusive upper bounds.

### Generator

Constant runtime using generators.

```python
from collections.abc import Iterator

def generate_substrings(s: str) -> Iterator[str]:
    for i in range(0, len(s)):
        for j in range(i+1, len(s)+1):
            yield s[i:j]

```

## Anagrams

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
are amenable to a linear sort algorithm, [Counting Sort](coding-interview/sorting.md#counting-sort),
for example if limited to ASCII.
