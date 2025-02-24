# Python Types

Brief notes about common solutions and gotchas with Python typing, relevant for coding interviews.

## Built-In Types

Literals:

```python
int, float, str, dict, list, set, tuple
```

## Custom Types

Problems often involve some amount of data modeling. The main options are:

- `namedtuple`: for simple, immutable data types, especially when tuple-like unpacking is helpful.
- `@dataclass`: for mutable value-equality types, especially when you want methods or custom
  constructors / validators.
- Type Alias: for shortening long generic types

### Grid Example

In this example, the classes:

- `namedtuple` for a 2d vector in cartesian coordinates. The fields `x` and `y` can be referred
    by name or index.
- Type Alias for the grid repr.
- `@dataclass` holding a grid. Since the class and its fields all use value-equality, instances
    are comparable (`==`).

```python
# namedtuple
from collections import namedtuple

Vec2 = namedtuple('Vec2', ['x', 'y'])

p = Vec2(1, 2)
p[0] == p.x == 1
x, y = p
print(x, y)

# type alias
type Grid2Repr = list[list[int]]

# dataclass
from dataclasses import dataclass

@dataclass
class Grid():
    repr: Grid2Repr
    dim: Vec2


def grid_from_str(s: str) -> Grid:
    repr = [[int(c) for c in row.split()] for row in s.strip().splitlines()]
    return Grid(repr, Vec2(len(repr), len(repr[0])))


g1_str = """
0 0 1
1 2 0
3 0 1 
"""

g2_str = "1 2 3"

assert grid_from_str(g1_str) != grid_from_str(g2_str)
assert grid_from_str(g1_str) == grid_from_str(g1_str)
```

## Self-referential types

This requires `from __future__ import annotations` at the top of the file.

Example:

```python
from __future__ import annotations
from dataclasses import dataclass

@dataclass
class Grid():
    repr: Grid2Repr
    dim: Vec2

    # this method could not refer to the Grid type without the annotations import.
    def from_str(s: str) -> Grid:
        pass

```

## ABC types

These are generic abstract base classes used to describe traits that other types can have. Some
Python style guides recommend annotating input types with `abc` traits and output types with
the type literal to avoid overly constraining the consumer of your code.

Collections:

```python
from collections.abc import Iterable

def generic_sorted[T](collection: Iterable[T]) -> list[T]:
    return sorted(list(collection))

from collections.abc import Mapping

def incr_val_any_mapping(d: Mapping[str, int]) -> dict[str, int]:
    return {k: v+1 for k, v in d.items()}
```

Higher-Order Functions:

```python
from collections.abc import Callable

# a callable that take arguments of type int and float, and returns a string.
callable: Callable[[int, float], str]
```
