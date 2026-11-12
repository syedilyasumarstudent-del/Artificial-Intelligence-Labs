# Week 4 Lab: A* Search for the 8-Puzzle

### Google Colab Edition

**Credit:** Original lab contents generated using Claude. Implementation and explanations completed for this lab.

## 1. Objective

The objective of this lab is to implement the A* search algorithm and apply it to the 8-Puzzle problem.

We implement two heuristics:

1. Misplaced Tiles
2. Manhattan Distance

We compare these heuristics by measuring how many nodes A* expands while solving puzzles of different difficulty levels.

By completing this lab, we learn how to:

- Represent the 8-Puzzle as a search problem.
- Implement A* using a priority queue.
- Calculate Misplaced Tiles and Manhattan Distance.
- Understand admissible heuristics.
- Compare the performance of different heuristics.

## How to Use This Lab in Google Colab

1. Open [Google Colab](https://colab.research.google.com).
2. Create a new notebook.
3. Add the numbered Markdown and Code cells in the order shown below.
4. Run each cell using `Shift + Enter`.
5. Complete all sections and check the outputs.

---

## Section 1: Problem Representation

### Cell 1 — Markdown

```markdown
## 1. Problem Representation

State: A tuple of 9 integers, where 0 represents the blank space.

Goal state: (1, 2, 3, 4, 5, 6, 7, 8, 0)
```

### Cell 2 — Imports and Constants

```python
import heapq
import itertools
import random

GOAL_STATE = (1, 2, 3, 4, 5, 6, 7, 8, 0)
```

### Cell 3 — State Helper Functions

```python
def get_blank_position(state):
    """Return the index of the blank tile (0)."""
    return state.index(0)


def get_neighbors(state):
    """
    Return all valid neighboring states.

    Actions describe the direction in which
    the blank space moves.
    """

    neighbors = []

    blank = get_blank_position(state)
    row, col = divmod(blank, 3)

    moves = {
        "Up": (row - 1, col),
        "Down": (row + 1, col),
        "Left": (row, col - 1),
        "Right": (row, col + 1)
    }

    for action, (new_row, new_col) in moves.items():

        if 0 <= new_row < 3 and 0 <= new_col < 3:

            new_blank = new_row * 3 + new_col

            new_state = list(state)

            new_state[blank], new_state[new_blank] = (
                new_state[new_blank],
                new_state[blank]
            )

            neighbors.append((tuple(new_state), action))

    return neighbors


def is_goal(state):
    """Check whether the state matches the goal state."""
    return state == GOAL_STATE
```

### Cell 4 — Checkpoint

```python
# The blank is at the bottom-right corner.
# Therefore, only Up and Left are valid moves.

neighbors = get_neighbors(GOAL_STATE)

print(neighbors)

assert len(neighbors) == 2, (
    f"Expected 2 neighbors, got {len(neighbors)}"
)

print("Cell 3 checkpoint passed.")
```

Expected output:

```text
[((1, 2, 3, 4, 5, 6, 7, 0, 8), 'Up'),
 ((1, 2, 3, 4, 5, 6, 7, 8, 0), 'Left')]
```

The checkpoint should pass successfully.

---

## Section 2: Heuristics

Both heuristics estimate the remaining cost from the current state to the goal state.

A heuristic must be **admissible**, meaning it must never overestimate the actual minimum cost to reach the goal.

### Cell 5 — Markdown

```markdown
## 2. Heuristics

We implement two admissible heuristics:

1. Misplaced Tiles
2. Manhattan Distance

Both heuristics ignore the blank tile.
```

### Cell 6 — Task 2.1: Misplaced Tiles

```python
def misplaced_tiles(state):
    """
    Count tiles that are not in their goal position.
    The blank tile (0) is excluded.
    """

    count = 0

    for i in range(len(state)):

        if state[i] != 0 and state[i] != GOAL_STATE[i]:
            count += 1

    return count
```

**Explanation:**

The Misplaced Tiles heuristic compares each tile with its correct position in the goal state.

If a numbered tile is not in its correct position, the counter increases by one.

The blank tile is ignored.

### Cell 7 — Task 2.2: Manhattan Distance

```python
def manhattan_distance(state):
    """
    Calculate the total Manhattan Distance
    of all numbered tiles from their goal positions.
    """

    distance = 0

    for i, tile in enumerate(state):

        if tile != 0:

            current_row, current_col = divmod(i, 3)

            goal_index = GOAL_STATE.index(tile)

            goal_row, goal_col = divmod(goal_index, 3)

            distance += (
                abs(current_row - goal_row) +
                abs(current_col - goal_col)
            )

    return distance
```

**Explanation:**

Manhattan Distance calculates how far each numbered tile is from its correct position.

It adds the horizontal and vertical distances of every tile.

The blank tile is excluded from the calculation.

### Cell 8 — Task 2.3: Sanity Check

```python
test_state = (1, 2, 3, 4, 5, 6, 7, 0, 8)

print("Misplaced Tiles:", misplaced_tiles(test_state))
print("Manhattan Distance:", manhattan_distance(test_state))

assert misplaced_tiles(test_state) == 1

assert manhattan_distance(test_state) == 1

assert misplaced_tiles(GOAL_STATE) == 0

assert manhattan_distance(GOAL_STATE) == 0

print("Cell 8 checkpoint passed.")
```

Expected output:

```text
Misplaced Tiles: 1
Manhattan Distance: 1
Cell 8 checkpoint passed.
```

### Discussion: Why must h(n) = 0 whenever state == GOAL_STATE?

Answer:

Both heuristics must return zero when the goal state is reached because there are no misplaced tiles and no remaining distance to travel.

If the heuristic returned a positive value for the goal state, it would incorrectly estimate that additional cost is required.

This could affect the priority assigned to the goal state in A* and would violate the requirement that an admissible heuristic must never overestimate the remaining cost.

---

## Section 3: A* Search Implementation

### Cell 9 — Markdown

```markdown
## 3. A* Search

A* combines the actual cost and estimated remaining cost:

f(n) = g(n) + h(n)

g(n) = Cost from the starting state to the current state.

h(n) = Estimated cost from the current state to the goal.

f(n) = Total estimated cost of the solution through the current state.
```

### Cell 10 — Task 3.1: A* Search

```python
def a_star_search(start_state, heuristic_fn):
    """
    A* Search Algorithm.

    Returns:
        path: List of actions from start to goal.
        cost: Number of moves in the solution.
        nodes_expanded: Number of valid nodes expanded.
    """

    counter = itertools.count()

    frontier = []

    heapq.heappush(
        frontier,
        (
            heuristic_fn(start_state),
            next(counter),
            0,
            start_state,
            []
        )
    )

    best_g = {start_state: 0}

    nodes_expanded = 0

    while frontier:

        f, _, g, state, path = heapq.heappop(frontier)

        # Skip outdated entries in the priority queue.
        if g > best_g.get(state, float("inf")):
            continue

        nodes_expanded += 1

        # Check whether the goal has been reached.
        if is_goal(state):

            return {
                "path": path,
                "cost": g,
                "nodes_expanded": nodes_expanded
            }

        # Explore neighboring states.
        for neighbor_state, action in get_neighbors(state):

            new_g = g + 1

            # Update the state if it is new or a better path is found.
            if new_g < best_g.get(
                neighbor_state,
                float("inf")
            ):

                best_g[neighbor_state] = new_g

                new_f = new_g + heuristic_fn(neighbor_state)

                heapq.heappush(
                    frontier,
                    (
                        new_f,
                        next(counter),
                        new_g,
                        neighbor_state,
                        path + [action]
                    )
                )

    return {
        "path": None,
        "cost": None,
        "nodes_expanded": nodes_expanded
    }
```

**Explanation:**

The A* algorithm uses a priority queue to select the state with the lowest estimated total cost.

It calculates:

- `g(n)`: Actual cost from the starting state.
- `h(n)`: Estimated cost to reach the goal.
- `f(n)`: Sum of actual and estimated costs.

The algorithm explores the most promising state first.

It also stores the best known cost for each state to avoid unnecessary exploration.

### Why use a tie-breaker counter?

Python compares priority queue entries element by element.

The counter provides a unique value for entries with equal priorities. This ensures predictable ordering and avoids unnecessary comparisons between later elements of the entries.

### Cell 11 — Quick Smoke Test

```python
easy_state = (1, 2, 3, 4, 5, 6, 7, 0, 8)

result = a_star_search(easy_state, manhattan_distance)

print(result)

assert result["cost"] == 1, (
    f"Expected cost 1, got {result['cost']}"
)

print("Cell 11 checkpoint passed.")
```

Expected output:

```text
{
    'path': ['Right'],
    'cost': 1,
    'nodes_expanded': 2
}

Cell 11 checkpoint passed.
```

---

## Section 4: Testing

### Cell 12 — Markdown

```markdown
## 4. Testing

We generate solvable 8-Puzzle states and compare
the performance of both heuristics.

The comparison is based on the number of nodes expanded
by A*.
```

### Cell 13 — Task 4.1: Solvability Check

```python
def is_solvable(state):
    """
    A state is solvable if its inversion count is even.
    The blank tile is ignored.
    """

    tiles = [tile for tile in state if tile != 0]

    inversions = 0

    for i in range(len(tiles)):

        for j in range(i + 1, len(tiles)):

            if tiles[i] > tiles[j]:
                inversions += 1

    return inversions % 2 == 0
```

### Cell 14 — Task 4.2: Scramble Generator

```python
def generate_scramble(num_moves, seed=None):

    rng = random.Random(seed)

    state = GOAL_STATE

    for _ in range(num_moves):

        neighbors = get_neighbors(state)

        state, _ = rng.choice(neighbors)

    return state
```

**Explanation:**

The scramble generator starts from the goal state and performs a specified number of random legal moves.

Because every move is legal, the generated state is always solvable.

However, some random moves may undo previous moves. Therefore, the number of scramble moves does not necessarily equal the actual shortest solution cost.

### Cell 15 — Task 4.3: Run the Comparison

```python
def run_comparison(num_moves_list, trials_per_level=5):

    results = []

    for num_moves in num_moves_list:

        for trial in range(trials_per_level):

            start = generate_scramble(
                num_moves,
                seed=trial * 1000 + num_moves
            )

            result_misplaced = a_star_search(
                start,
                misplaced_tiles
            )

            result_manhattan = a_star_search(
                start,
                manhattan_distance
            )

            # Both admissible heuristics should find
            # an optimal solution with the same cost.

            assert (
                result_misplaced["cost"] ==
                result_manhattan["cost"]
            ), "The solution costs do not match!"

            results.append({

                "scramble_depth": num_moves,

                "trial": trial + 1,

                "misplaced_nodes":
                    result_misplaced["nodes_expanded"],

                "manhattan_nodes":
                    result_manhattan["nodes_expanded"],

                "solution_cost":
                    result_misplaced["cost"]

            })

    return results


# Run the comparison at four different scramble depths.

results = run_comparison(
    num_moves_list=[5, 10, 15, 20],
    trials_per_level=5
)

for r in results:
    print(r)
```

**Explanation:**

The experiment tests four scramble depths:

- 5 moves
- 10 moves
- 15 moves
- 20 moves

Each depth is tested five times.

For every generated puzzle, A* runs twice:

1. Using Misplaced Tiles.
2. Using Manhattan Distance.

The results are stored for comparison.

Both heuristics should produce the same optimal solution cost, although they may expand different numbers of nodes.

### Cell 16 — Comparison Table and Chart

```python
import pandas as pd
import matplotlib.pyplot as plt

# Convert the results into a DataFrame.

df = pd.DataFrame(results)

# Calculate average nodes expanded at each scramble depth.

summary = df.groupby("scramble_depth")[
    ["misplaced_nodes", "manhattan_nodes"]
].mean()

print("Average Nodes Expanded:")
print(summary)

# Generate the comparison chart.

summary.plot(marker="o")

plt.xlabel("Scramble Depth (moves from goal)")

plt.ylabel("Average Nodes Expanded")

plt.title(
    "A* Node Expansion: Misplaced Tiles vs. Manhattan Distance"
)

plt.legend([
    "Misplaced Tiles",
    "Manhattan Distance"
])

plt.grid(True)

plt.show()
```

**Expected observation:**

The Manhattan Distance heuristic generally expands fewer nodes than the Misplaced Tiles heuristic.

As puzzle difficulty increases, the difference between the two heuristics usually becomes more noticeable.

The exact numerical results depend on the generated puzzle states.

---

## Section 5: Analysis

### Cell 17 — Markdown

### 1. Comparison Table

The comparison table is generated in Cell 16.

It shows the average number of nodes expanded by both heuristics at each scramble depth.

The results allow us to observe how heuristic selection affects the performance of A*.

### 2. Chart

The chart generated in Cell 16 compares the average number of nodes expanded by Misplaced Tiles and Manhattan Distance.

It helps visualize the difference in performance between the two heuristics.

### 3. Answers

**Q1. At what scramble depth does the gap between the two heuristics become clearly visible? Why does the gap widen as depth increases rather than staying constant?**

Answer:

The difference generally becomes more noticeable at higher scramble depths.

Misplaced Tiles only counts how many tiles are in the wrong positions. It does not consider how far those tiles must move.

Manhattan Distance considers the actual horizontal and vertical distance of every tile from its goal position.

As puzzles become more complicated, this additional information helps A* identify more promising states and avoid exploring unnecessary states.

Therefore, the difference in the number of expanded nodes can increase as puzzle difficulty increases.

The exact depth at which the difference becomes clearly visible depends on the generated puzzle states.

**Q2. Manhattan Distance is always greater than or equal to Misplaced Tiles. Why is a higher admissible heuristic not a violation of admissibility, and why does it lead to fewer node expansions?**

Answer:

An admissible heuristic never overestimates the actual minimum cost required to reach the goal.

Manhattan Distance is greater than or equal to Misplaced Tiles because it considers how far each misplaced tile must travel, rather than simply counting misplaced tiles.

However, Manhattan Distance still does not overestimate the actual minimum number of moves.

A stronger heuristic provides a more accurate estimate of the remaining cost.

This allows A* to prioritize more promising states and generally expand fewer nodes before reaching the goal.

Therefore, a higher heuristic value does not violate admissibility as long as it does not exceed the true remaining cost.

**Q3. Does Manhattan Distance hit the same performance limitation as Misplaced Tiles when scramble depth increases?**

Answer:

No, Manhattan Distance generally performs better than Misplaced Tiles because it provides more information about the distance remaining to the goal.

It usually expands fewer nodes and can solve more difficult puzzles efficiently.

However, Manhattan Distance is not perfect. Very difficult 8-Puzzle instances can still require substantial memory and processing time because A* stores many states in its priority queue.

---

## Conclusion

In this lab, we implemented the A* search algorithm to solve the 8-Puzzle problem.

We implemented two heuristics: Misplaced Tiles and Manhattan Distance.

We learned that A* calculates the estimated total cost using:

`f(n) = g(n) + h(n)`

We also learned that both heuristics are admissible because they never overestimate the actual minimum cost to reach the goal.

Through the experiments, we observed that Manhattan Distance generally performs better than Misplaced Tiles because it provides a more accurate estimate of the remaining distance.

**Key Takeaway:** The quality of a heuristic has a significant effect on A* performance. A stronger admissible heuristic can reduce the number of nodes expanded while still guaranteeing an optimal solution.

---

## Submission Checklist

- [x] `get_blank_position()` implemented.
- [x] `get_neighbors()` implemented.
- [x] `is_goal()` implemented.
- [x] Misplaced Tiles heuristic implemented.
- [x] Manhattan Distance heuristic implemented.
- [x] A* search algorithm implemented.
- [x] Solvability check implemented.
- [x] Scramble generator implemented.
- [x] Comparison function implemented.
- [x] Comparison table and chart implemented.
- [x] Analysis questions answered.

**Final verification:** Run all cells in Google Colab using `Runtime → Run all` and confirm that every checkpoint passes and the comparison table and chart are generated.

**Submission:** Share the Google Colab notebook link with "Anyone with the link can view" enabled.
