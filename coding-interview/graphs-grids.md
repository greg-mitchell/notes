# Graphs, Grids, and Trees

Grids and Trees are special cases of Graphs.

## Graphs

Graphs are common for problems about relationships between entities, and for networks (like roads).
Generally, graphs have a set of Vertices **V** and Edges **E** between those vertices, or
equivalently nodes and arcs. Common properties of graphs:

- Undirected vs Directed
- Weighted
- Acyclic. Implies sparsity: `|E| <= |V|`
- Dense or complete. Implies cycles.
  - Edges approach upper bound: `|E| <= |V|**2`

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

## Binary Trees

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

### Inorder / Preorder / Postorder Traversal

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

### Level order traversal

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

## Grids

Many problems involve parsing and operating on 2D grids. For data modeling, see the example in [Types](coding-interview/python-types.md#grid-example). Generally these grids easy to work
with as an `m x n` matrix, where `m` is the number of rows, and `n` is the number of columns.
There are two reasonable ways to represent a grid.

- **Dense Matrix**: A multidimensional array, with a slot per cell.
  Consumes `O(m * n)` space. Best when:
  - The entire matrix can fit in memory. It's particularly efficient if it fits in cache.
  - There is no good default value, or each cell has a unique value.
- **Sparse Matrix**: A mapping from coordinate to value. Consumes `O(m * n)` space worst case,
  but given problem assumptions, `O(values)` is more accurate. Best when:
  - The matrix is mostly empty or default.
  - The matrix is too large to fit in memory.

### Dense Matrix

The most common representation is a row-major 2D array, which in Python is implemented as
a jagged array, i.e. each row is a list. Assuming the grid is a multiline string `s`
with possible whitespace between cells, and each cell is a number:

```python
[[int(c) for c in row.split()] for row in s.strip().splitlines()]
```

### Sparse Matrix

The `collections.defaultdict` container is extremely helpful. Python containers default to value
equality, so you can construct a tuple or list as the key like `(row, col)` to look up a value.

#### Complex Number Representation

A representation that's well suited to sparse matrices, especially when you have to
examine neighbors, is to represent the matrix as a dictionary of complex numbers to
value, where the complex number is a position. This simplifies finding neighbors: 
add a basis vector to the position.

Note that Python complex number literals are written as `real + 1j * imag`, where `j` is the
imaginary number.

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
