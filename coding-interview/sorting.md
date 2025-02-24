# Sorting

Many problems involve sorting as an essential precomputation step. Common uses:

- Finding elements with a certain condition (least, greatest, middle).
- Partitioning (separate elements to search from ones that are impossible).

Runtimes:

- General purpose:
  - `O(n log n)` for merge, heapsort.
  - GC / OO languages generally use variants of mergesort, e.g. Timsort, with special handling
    for patterns, like `O(n)` sorting for pre-sorted input.
- Quicksort:
  - Best for cache coherence, so good when comparing a contiguous block of values.
  - `O(n log n)` average case, and advanced implementations like `std::sort` have `O(n log n)`
    worst case by falling back to heapsort.
  - Classic algorithm had `O(n**2)` worst case.
- Domain-restricted:
  - `O(n)` for [Counting](#counting-sort) or Radix sort

## Counting Sort

Essentially a memory-compute tradeoff for sorting, Counting Sort allows linear-time sorting for
items with a fixed domain by pre-allocating buckets for the domain.

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
