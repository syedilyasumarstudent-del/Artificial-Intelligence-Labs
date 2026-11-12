# Lab 2: Uninformed Search — BFS and DFS on a 2D Maze

## 1. Objective

The objective of this lab is to understand and implement two uninformed search algorithms:

- **Breadth-First Search (BFS)**
- **Depth-First Search (DFS)**

Both algorithms will be used to find a path from a starting position to a goal position in a 2D maze.

By completing this lab, we will understand how BFS and DFS explore a search space, how their frontier structures differ, and how their performance changes depending on the maze and the order in which neighboring cells are explored.

## 2. Maze Representation

We represent the maze using a 5×5 grid.

- `0` represents an open cell that can be visited.
- `1` represents a wall that cannot be crossed.
- Start position: `(0, 0)`
- Goal position: `(4, 4)`

### Maze

```text
S  0  0  0  0
1  1  0  1  0
0  0  0  0  0
0  1  1  1  0
0  0  0  0  G
```

The Python representation is:

```python
maze = [
    [0, 0, 0, 0, 0],
    [1, 1, 0, 1, 0],
    [0, 0, 0, 0, 0],
    [0, 1, 1, 1, 0],
    [0, 0, 0, 0, 0]
]

start = (0, 0)
goal = (4, 4)
```

## 3. Define Movement Rules

A position is valid if:

1. It is inside the maze boundaries.
2. It is not a wall.

We use four possible movement directions:

- Up
- Down
- Left
- Right

```python
directions = [
    (-1, 0),  # Up
    (1, 0),   # Down
    (0, -1),  # Left
    (0, 1)    # Right
]

def valid_move(position, maze):
    row, col = position

    if row < 0 or row >= len(maze):
        return False

    if col < 0 or col >= len(maze[0]):
        return False

    if maze[row][col] == 1:
        return False

    return True
```

## 4. Breadth-First Search (BFS)

Breadth-First Search explores the maze level by level.

It uses a **queue (FIFO)** data structure, meaning the first position added to the queue is the first one explored.

BFS is guaranteed to find the shortest path in an unweighted maze if a path exists.

```python
from collections import deque

def bfs(maze, start, goal, directions):

    queue = deque([start])

    visited = {start}
    parent = {}

    max_frontier = 1
    expanded_nodes = 0

    while queue:

        current = queue.popleft()
        expanded_nodes += 1

        if current == goal:

            path = [current]

            while current in parent:
                current = parent[current]
                path.append(current)

            path.reverse()

            return path, expanded_nodes, max_frontier

        for dr, dc in directions:

            neighbor = (
                current[0] + dr,
                current[1] + dc
            )

            if valid_move(neighbor, maze) and neighbor not in visited:

                visited.add(neighbor)
                parent[neighbor] = current
                queue.append(neighbor)

        max_frontier = max(max_frontier, len(queue))

    return None, expanded_nodes, max_frontier
```

## 5. Depth-First Search (DFS)

Depth-First Search explores one path as deeply as possible before backtracking.

It uses a **stack (LIFO)** data structure, meaning the last position added to the stack is the first one explored.

Unlike BFS, DFS does not guarantee the shortest path.

```python
def dfs(maze, start, goal, directions):

    stack = [start]

    visited = {start}
    parent = {}

    max_frontier = 1
    expanded_nodes = 0

    while stack:

        current = stack.pop()
        expanded_nodes += 1

        if current == goal:

            path = [current]

            while current in parent:
                current = parent[current]
                path.append(current)

            path.reverse()

            return path, expanded_nodes, max_frontier

        for dr, dc in directions:

            neighbor = (
                current[0] + dr,
                current[1] + dc
            )

            if valid_move(neighbor, maze) and neighbor not in visited:

                visited.add(neighbor)
                parent[neighbor] = current
                stack.append(neighbor)

        max_frontier = max(max_frontier, len(stack))

    return None, expanded_nodes, max_frontier
```

## 6. Visualize the Solution

The following function displays the maze and highlights the path found by the search algorithm.

```python
def print_maze(maze, path):

    path = set(path or [])

    for row in range(len(maze)):

        for col in range(len(maze[0])):

            position = (row, col)

            if position == start:
                print("S", end="  ")

            elif position == goal:
                print("G", end="  ")

            elif maze[row][col] == 1:
                print("#", end="  ")

            elif position in path:
                print("*", end="  ")

            else:
                print(".", end="  ")

        print()
```

Symbols:

- `S` = Starting position
- `G` = Goal position
- `#` = Wall
- `*` = Path found by the algorithm
- `.` = Open cell

## 7. Run Both Algorithms

Now we run BFS and DFS on the same maze and compare their results.

```python
bfs_path, bfs_expanded, bfs_frontier = bfs(
    maze, start, goal, directions
)

dfs_path, dfs_expanded, dfs_frontier = dfs(
    maze, start, goal, directions
)

print("BFS Results")
print("Path:", bfs_path)
print("Path Length:", len(bfs_path) - 1 if bfs_path else "No path")
print("Expanded Nodes:", bfs_expanded)
print("Maximum Frontier Size:", bfs_frontier)

print("\nBFS Maze:")
print_maze(maze, bfs_path)

print("\nDFS Results")
print("Path:", dfs_path)
print("Path Length:", len(dfs_path) - 1 if dfs_path else "No path")
print("Expanded Nodes:", dfs_expanded)
print("Maximum Frontier Size:", dfs_frontier)

print("\nDFS Maze:")
print_maze(maze, dfs_path)
```

### Expected Results

**BFS:**

```text
Path:
[(0, 0), (0, 1), (0, 2), (1, 2),
 (2, 2), (2, 3), (2, 4), (3, 4), (4, 4)]

Path Length: 8
Expanded Nodes: 16
Maximum Frontier Size: 3
```

**DFS:**

```text
Path:
[(0, 0), (0, 1), (0, 2), (0, 3),
 (0, 4), (1, 4), (2, 4), (2, 3),
 (2, 2), (2, 1), (2, 0), (3, 0),
 (4, 0), (4, 1), (4, 2), (4, 3), (4, 4)]

Path Length: 16
Expanded Nodes: 17
Maximum Frontier Size: 3
```

*Note: The number of expanded nodes and maximum frontier size depend on the maze and neighbor exploration order. These results correspond to the maze and code above.*

## 8. Comparison Between BFS and DFS

| Feature | BFS | DFS |
|---|---|---|
| Data Structure | Queue | Stack |
| Exploration | Level by level | Deep path first |
| Shortest Path | Guaranteed in an unweighted maze | Not guaranteed |
| Memory Usage | Usually higher | Often lower |
| Completeness | Complete for a finite maze | Complete here with visited tracking |
| Main Advantage | Finds shortest path | Can explore deep paths quickly |

### Questions

**Q1. Which algorithm finds the shortest path?**

Answer: BFS finds the shortest path because it explores all positions at one distance before moving to the next distance.

**Q2. Which algorithm uses a stack?**

Answer: DFS uses a stack because it follows the Last In, First Out (LIFO) principle.

**Q3. Why does DFS sometimes find a longer path?**

Answer: DFS explores one direction deeply without checking whether another route is shorter. Therefore, it can reach the goal through a longer path.

**Q4. Which algorithm generally requires more memory?**

Answer: BFS generally requires more memory because it stores multiple positions in its queue while exploring each level.

## 9. Experiment 1: Change Neighbor Exploration Order

In this experiment, we change the order in which neighboring cells are explored.

Original order:

```python
directions = [
    (-1, 0),  # Up
    (1, 0),   # Down
    (0, -1),  # Left
    (0, 1)    # Right
]
```

New order:

```python
directions = [
    (0, 1),   # Right
    (1, 0),   # Down
    (0, -1),  # Left
    (-1, 0)   # Up
]
```

Run DFS again:

```python
new_dfs_path, new_dfs_expanded, new_dfs_frontier = dfs(
    maze, start, goal, directions
)

print("DFS with new direction order:")
print("Path:", new_dfs_path)
print("Path Length:", len(new_dfs_path) - 1 if new_dfs_path else "No path")
print("Expanded Nodes:", new_dfs_expanded)
print("Maximum Frontier Size:", new_dfs_frontier)

print_maze(maze, new_dfs_path)
```

### Observation

With the new direction order, DFS produces a different path.

The original DFS path length was 16, while the new DFS path length is 12.

This happens because DFS uses a stack. The order in which neighbors are added affects which neighbor is explored first.

### Questions

**Q1. Did the DFS path change after changing the direction order?**

Answer: Yes. DFS explored a different route because the neighbor exploration order changed.

**Q2. Does BFS also depend on the direction order?**

Answer: Yes. Changing the direction order can change the particular shortest path selected by BFS when multiple shortest paths exist. However, BFS still guarantees a shortest path in an unweighted maze.

**Q3. Does changing the direction order affect the shortest path length found by BFS?**

Answer: No, not when the maze and start and goal positions remain unchanged and a path exists. BFS still finds a shortest path, although the actual route may be different.

## 10. Experiment 2: Change Start and Goal Positions

In this experiment, we test different starting and goal positions to observe how the algorithms behave.

Use the original direction order again:

```python
directions = [
    (-1, 0),  # Up
    (1, 0),   # Down
    (0, -1),  # Left
    (0, 1)    # Right
]
```

Run the following code:

```python
test_cases = [
    ((0, 0), (4, 4)),
    ((0, 4), (4, 0)),
    ((2, 0), (4, 4))
]

for test_start, test_goal in test_cases:

    print("\nStart:", test_start)
    print("Goal:", test_goal)

    bfs_result, bfs_nodes, bfs_max = bfs(
        maze, test_start, test_goal, directions
    )

    dfs_result, dfs_nodes, dfs_max = dfs(
        maze, test_start, test_goal, directions
    )

    bfs_length = len(bfs_result) - 1 if bfs_result else "No path"
    dfs_length = len(dfs_result) - 1 if dfs_result else "No path"

    print("\nBFS:")
    print("Path Length:", bfs_length)
    print("Expanded Nodes:", bfs_nodes)
    print("Maximum Frontier:", bfs_max)

    print("\nDFS:")
    print("Path Length:", dfs_length)
    print("Expanded Nodes:", dfs_nodes)
    print("Maximum Frontier:", dfs_max)
```

### Experiment Results

| Start | Goal | BFS Path Length | DFS Path Length |
|---|---|---:|---:|
| (0, 0) | (4, 4) | 8 | 16 |
| (0, 4) | (4, 0) | 8 | 12 |
| (2, 0) | (4, 4) | 6 | 6 |

*The path lengths are measured in movements between cells, not the total number of cells in the path.*

### Questions

**Q1. Does BFS always find the same path?**

Answer: Not necessarily. If multiple shortest paths exist, BFS can select different paths depending on the neighbor exploration order.

**Q2. Does DFS always find the same path?**

Answer: DFS follows the same path when the maze, start position, goal position, and neighbor exploration order remain unchanged. Changing any of these can change its result.

**Q3. Why can the frontier size change?**

Answer: The frontier size changes because different positions have different numbers of valid neighboring cells. The search algorithm may also explore different routes depending on the starting and goal positions.

**Q4. What happens if the goal is unreachable?**

Answer: If the goal cannot be reached, both algorithms eventually explore all reachable cells and return `None` for the path.

We can test an unreachable goal by placing the goal position on a wall:

```python
unreachable_goal = (1, 0)

bfs_result, _, _ = bfs(
    maze, start, unreachable_goal, directions
)

dfs_result, _, _ = dfs(
    maze, start, unreachable_goal, directions
)

print("BFS Result:", bfs_result)
print("DFS Result:", dfs_result)
```

Expected output:

```text
BFS Result: None
DFS Result: None
```

## 11. Real-World Applications

### Applications of BFS

1. Finding the shortest route in an unweighted map.
2. Finding the minimum number of connections between people in a social network.
3. Searching nearby locations in a network.
4. Solving puzzles where the shortest sequence of moves is required.

### Applications of DFS

1. Exploring files and folders in a computer system.
2. Detecting cycles in graphs.
3. Solving mazes and puzzles.
4. Exploring connected components in a graph.
5. Backtracking problems such as Sudoku.

## 12. Conclusion

In this lab, we implemented Breadth-First Search (BFS) and Depth-First Search (DFS) to solve a 2D maze.

We learned that BFS uses a queue and explores the maze level by level, which allows it to find the shortest path in an unweighted maze.

DFS uses a stack and explores one path deeply before backtracking. It may find a solution quickly, but it does not guarantee the shortest path.

We also observed that changing the neighbor exploration order can affect the path selected by both algorithms. However, BFS continues to guarantee the shortest path as long as all movements have equal cost.

**Key Takeaway:** BFS is generally preferred when finding the shortest path is important, while DFS is useful when exploring possibilities deeply and memory usage is a concern.
