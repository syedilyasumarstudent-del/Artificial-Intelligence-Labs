# Lab 4: Hill-Climbing Search with the N-Queens Problem

**Generated using AI models and reviewed by the instructor**

## 1. Lab Objectives

The objective of this lab is to implement the Hill-Climbing algorithm in Python to solve the N-Queens problem.

By completing this lab, we will learn how to:

- Represent a problem as a state.
- Define an objective or evaluation function.
- Generate neighboring states.
- Implement basic Hill-Climbing.
- Record and plot evaluation scores over iterations.
- Understand why Hill-Climbing may terminate without finding a solution.
- Implement Random-Restart Hill-Climbing.
- Compare a single search with repeated randomized searches.

## 2. Background: Local Search and Hill-Climbing

Traditional search algorithms such as BFS, DFS, and A* commonly explore a search space to find a path from an initial state to a goal state.

In **local search**, the path is generally not important. Instead, the algorithm focuses on improving the current state until it finds a satisfactory or optimal solution.

Hill-Climbing is a greedy local search algorithm. It repeatedly examines neighboring states and moves to a better state when one is available.

The basic process is:

1. Start with an initial state.
2. Evaluate the current state.
3. Generate neighboring states.
4. Select the best neighbor.
5. Move to that neighbor if it improves the evaluation score.
6. Stop when no better neighbor is available.

### Does Hill-Climbing Always Get Stuck?

No. Hill-Climbing does not always get stuck at a local maximum.

There are two possible outcomes:

- **Successful termination:** The algorithm reaches a goal state. For N-Queens, this means that no two queens attack each other.
- **Premature termination:** The algorithm reaches a state that is better than all its immediate neighbors but is not a goal state.

The second situation is called the local maximum problem.

A local maximum is not necessarily a failure. If the local maximum is also the global maximum and represents a goal, the algorithm has succeeded.

However, basic Hill-Climbing cannot guarantee that it will find a global optimum from every starting state.

## 3. The N-Queens Problem

The N-Queens problem asks us to place N queens on an N × N chessboard so that no two queens attack each other.

Queens attack each other when they occupy:

- The same row.
- The same column.
- The same diagonal.

For example, the following arrangement is a valid solution for four queens:

```text
. Q . .
. . . Q
Q . . .
. . Q .
```

No two queens share a row, column, or diagonal.

## 4. Representing a State

We represent the board using a Python list.

For example:

```python
state = [1, 3, 0, 2]
```

The index represents the column, and the value represents the row of the queen.

Therefore:

```text
Column:  0  1  2  3
Row:     1  3  0  2
```

The corresponding board is:

```text
. . Q .
Q . . .
. . . Q
. Q . .
```

This representation automatically guarantees that each column contains exactly one queen.

### Task 1: Create and Print a State

```python
state = [1, 3, 0, 2]

print("State:", state)
```

Expected output:

```text
State: [1, 3, 0, 2]
```

## 5. Generating a Random Initial State

Hill-Climbing starts from an initial state. We generate a random row position for the queen in each column.

### Code Cell

```python
import random

def random_state(n):
    return [random.randrange(n) for _ in range(n)]


state = random_state(8)

print("Random initial state:", state)
```

Example output:

```text
Random initial state: [2, 5, 1, 7, 0, 3, 6, 4]
```

The output may differ each time because the initial state is randomly generated.

A random state is not necessarily a solution. Some queens may attack each other.

## 6. Defining the Evaluation Function

Hill-Climbing needs a way to determine whether one state is better than another.

For this problem, we count the number of pairs of queens attacking each other.

Our objective is to minimize the number of conflicts.

A valid solution has zero conflicts.

To express the problem as a maximization problem, we define:

**Evaluation Score = - Number of Conflicts**

Examples:

| Number of Conflicts | Evaluation Score |
|---:|---:|
| 5 | -5 |
| 3 | -3 |
| 1 | -1 |
| 0 | 0 |

A higher score is better. Therefore, zero is the best possible score.

### Code Cell: Count Conflicts

```python
def count_conflicts(state):

    conflicts = 0
    n = len(state)

    for i in range(n):

        for j in range(i + 1, n):

            # Queens in the same row
            if state[i] == state[j]:
                conflicts += 1

            # Queens on the same diagonal
            elif abs(state[i] - state[j]) == abs(i - j):
                conflicts += 1

    return conflicts
```

Test the function:

```python
state = [1, 3, 0, 2]

print("Conflicts:", count_conflicts(state))
```

Expected output:

```text
Conflicts: 0
```

### Code Cell: Evaluation Function

```python
def evaluation(state):
    return -count_conflicts(state)


state = [1, 3, 0, 2]

print("Evaluation score:", evaluation(state))
```

Expected output:

```text
Evaluation score: 0
```

## 7. Generating Neighboring States

A neighbor is a state that can be reached by making a small change to the current state.

In our implementation, we move one queen to another row in the same column.

For example, if the current state is:

```python
state = [1, 3, 0, 2]
```

Moving the queen in column 0 to row 0 produces:

```python
[0, 3, 0, 2]
```

Moving the same queen to row 2 produces:

```python
[2, 3, 0, 2]
```

We generate every state obtainable by moving one queen to another row in its column.

### Code Cell

```python
def get_neighbors(state):

    neighbors = []
    n = len(state)

    for col in range(n):

        for row in range(n):

            if row != state[col]:

                neighbor = state.copy()

                neighbor[col] = row

                neighbors.append(neighbor)

    return neighbors
```

Test the function:

```python
state = [1, 3, 0, 2]

neighbors = get_neighbors(state)

print("Number of neighbors:", len(neighbors))

print("First five neighbors:")

for neighbor in neighbors[:5]:
    print(neighbor)
```

For a board of size N, each state has:

\[
N \times (N-1)
\]

neighbors.

For N = 8:

\[
8 \times 7 = 56
\]

Therefore, each state has 56 neighboring states.

## 8. Implementing Basic Hill-Climbing

We now have the three main components:

1. State representation.
2. Evaluation function.
3. Neighbor-generation function.

The following algorithm starts from a random state, repeatedly selects the best neighbor, and stops when no neighbor improves the current score.

It also records the evaluation score at each iteration so that we can plot the optimization process.

### Code Cell

```python
def hill_climbing(n):

    current = random_state(n)

    scores = []

    while True:

        # Evaluate the current state
        current_score = evaluation(current)

        scores.append(current_score)

        # Generate all neighboring states
        neighbors = get_neighbors(current)

        # Find the best neighbor
        best_neighbor = max(neighbors, key=evaluation)

        best_score = evaluation(best_neighbor)

        # Stop if no neighbor improves the current score
        if best_score <= current_score:
            break

        # Move to the better neighbor
        current = best_neighbor

    return current, scores
```

### Explanation

1. The algorithm begins with a random state.
2. It calculates the current evaluation score.
3. It generates all possible neighboring states.
4. It selects the neighbor with the highest evaluation score.
5. If the best neighbor is not better, the algorithm stops.
6. Otherwise, it moves to the better neighbor and repeats.

The algorithm only accepts improvements, so the evaluation score never decreases.

## 9. Running Hill-Climbing

Run the algorithm for eight queens.

### Code Cell

```python
solution, scores = hill_climbing(8)

print("Final state:", solution)

print("Final conflicts:", count_conflicts(solution))

print("Final evaluation score:", evaluation(solution))

print("Number of iterations:", len(scores))

if count_conflicts(solution) == 0:

    print("A valid solution was found.")

else:

    print("Hill-Climbing stopped without finding a solution.")
```

Your results may differ between runs because the initial state is random.

You may find a valid solution with zero conflicts. Alternatively, the algorithm may stop at a state with remaining conflicts.

This is expected behavior, not necessarily a programming error.

### Why Can the Algorithm Stop Without Solving the Problem?

Suppose the current evaluation score is -2, and the neighboring states have the following scores:

```text
-3, -4, -2, -5
```

None of these neighbors is better than the current state.

Therefore, Hill-Climbing stops.

However, a state with a score of zero may exist elsewhere in the state space.

The algorithm has reached a local maximum with respect to its neighborhood, but it has not reached the goal.

**Important:** Hill-Climbing is not guaranteed to find a solution from every starting state. Getting stuck at a non-goal local maximum is a possibility, not an inevitable outcome.

## 10. Visualizing the Chessboard

We create a function to display the current arrangement of queens.

### Code Cell

```python
def print_board(state):

    n = len(state)

    for row in range(n):

        line = ""

        for col in range(n):

            if state[col] == row:

                line += "Q "

            else:

                line += ". "

        print(line)
```

Display the result:

```python
print_board(solution)
```

If a solution is found, no two queens should attack each other.

If the algorithm stops at a non-solution state, the board will still be displayed, but some queens may attack each other.

## 11. Plotting the Evaluation Score

A graph helps us observe how the evaluation score changes during the search.

### Code Cell

```python
import matplotlib.pyplot as plt

plt.figure(figsize=(9, 5))

plt.plot(
    range(len(scores)),
    scores,
    marker="o"
)

plt.xlabel("Iteration")

plt.ylabel("Evaluation Score")

plt.title("Hill-Climbing Optimization: N-Queens")

plt.grid(True)

plt.show()
```

### Interpret the Graph

**Q1. Does the evaluation score increase as the algorithm moves to better states?**

Answer: Yes. The algorithm accepts only neighboring states with higher evaluation scores. Therefore, the score increases or remains unchanged when the search terminates.

**Q2. Does the score decrease during a move accepted by this implementation?**

Answer: No. The algorithm only accepts a neighbor if its score is greater than the current score.

**Q3. What happens when the algorithm reaches a state with no better neighbor?**

Answer: The algorithm terminates because no neighboring state can improve the current evaluation score.

**Q4. What does an evaluation score of zero mean?**

Answer: It means that the state has zero conflicts. No two queens attack each other, so a valid N-Queens solution has been found.

**Q5. Can the algorithm stop at a score below zero? Explain why.**

Answer: Yes. It can reach a local maximum where no neighboring state has a higher score, even though conflicts still exist. In that situation, the algorithm stops without finding a valid solution.

## 12. Experimenting with Different Board Sizes

Run Hill-Climbing with different values of N.

### Code Cell

```python
for n in [4, 8, 10, 12]:

    solution, scores = hill_climbing(n)

    conflicts = count_conflicts(solution)

    print(
        f"N = {n:2d} | "
        f"Conflicts = {conflicts:2d} | "
        f"Iterations = {len(scores):3d} | "
        f"Solved = {conflicts == 0}"
    )
```

### Task 2: Record Your Observations

Run the experiment several times and record the results.

| N | Run | Final Conflicts | Iterations | Solved? |
|---:|---:|---:|---:|---|
| 4 | 1 | Run code | Run code | Run code |
| 4 | 2 | Run code | Run code | Run code |
| 8 | 1 | Run code | Run code | Run code |
| 8 | 2 | Run code | Run code | Run code |
| 8 | 3 | Run code | Run code | Run code |
| 10 | 1 | Run code | Run code | Run code |
| 10 | 2 | Run code | Run code | Run code |
| 10 | 3 | Run code | Run code | Run code |

### Observation

The results can differ between runs because each run starts with a random state.

A larger board has more possible states and generally a larger search space.

The number of iterations and the probability of finding a solution depend on the initial state and the structure of the neighboring states.

## 13. Improving the Search: Random-Restart Hill-Climbing

Basic Hill-Climbing uses a single initial state. If the search stops at a non-goal local maximum, the algorithm terminates without finding a solution.

One way to improve the chances of success is **Random-Restart Hill-Climbing**.

The algorithm repeatedly runs Hill-Climbing from new random starting states.

The process is:

1. Generate a random initial state.
2. Run Hill-Climbing until it stops.
3. Check whether a solution was found.
4. If a solution was found, return it.
5. Otherwise, start again from a new random state.
6. Stop when a solution is found or the restart limit is reached.

### What Does Probabilistically Complete Mean?

An algorithm is probabilistically complete if the probability of finding a solution approaches 1 as the search effort increases, provided that a solution exists and the algorithm's assumptions hold.

In simpler words:

If we continue giving the algorithm independent opportunities to search, its chance of finding a solution can get closer and closer to 100%.

This does not guarantee success within a fixed number of restarts.

Suppose a randomized search has a 20% probability of finding a solution in each independent run.

The probability of failure in one run is:

\[
1 - 0.20 = 0.80
\]

If each run is independent, the probability that all k runs fail is:

\[
P(\text{all fail}) = 0.80^k
\]

Therefore, the probability of finding at least one solution is:

\[
P(\text{success}) = 1 - 0.80^k
\]

For example:

| Independent Runs | Probability of Finding a Solution |
|---:|---:|
| 1 | 20.00% |
| 2 | 36.00% |
| 5 | 67.23% |
| 10 | 89.26% |
| 20 | 98.85% |

These numbers are illustrative and are not measured success rates for N-Queens.

The principle is that repeated independent attempts reduce the probability of repeated failure.

For Random-Restart Hill-Climbing, if the probability of success per independent run is p, where p is greater than zero, then:

\[
P(\text{success within } k \text{ runs}) = 1-(1-p)^k
\]

As k increases, this probability approaches 1.

In this implementation, every board state has a nonzero probability of being selected as a random starting state. Since a valid N-Queens state exists for N = 4 and N = 8 and above, repeated independent random restarts provide a positive probability of finding a solution.

## 14. Implementing Random-Restart Hill-Climbing

The following implementation keeps track of the best state found and records the best evaluation score after every restart.

It stops early if a valid solution is found.

### Code Cell

```python
def random_restart_hill_climbing(n, max_restarts=100):

    best_state = None

    best_score = float("-inf")

    best_scores = []

    for restart in range(max_restarts):

        # Start a new Hill-Climbing run
        state, scores = hill_climbing(n)

        score = evaluation(state)

        # Keep the best state found so far
        if score > best_score:

            best_state = state

            best_score = score

        # Record the best score after this restart
        best_scores.append(best_score)

        # A score of zero means a valid solution
        if best_score == 0:

            return best_state, restart + 1, best_scores

    return best_state, max_restarts, best_scores
```

### Explanation

- `best_state` stores the best board arrangement found so far.
- `best_score` stores its evaluation score.
- `best_scores` records the best score after each restart.
- `max_restarts` limits the number of attempts.
- The algorithm stops early if the best score becomes zero.

If no solution is found within the restart limit, the algorithm returns the best state encountered during those attempts.

### Code Cell: Run the Algorithm

```python
solution, restarts, best_scores = (
    random_restart_hill_climbing(8, max_restarts=100)
)

print("Final state:", solution)

print("Final conflicts:", count_conflicts(solution))

print("Restarts performed:", restarts)

if count_conflicts(solution) == 0:

    print("A valid solution was found.")

else:

    print("No solution was found within the restart limit.")

print_board(solution)
```

A valid solution should have zero conflicts.

If no solution is found within 100 restarts, the algorithm returns the best state encountered during those attempts.

The restart limit is a practical stopping condition, not a guarantee of success.

## 15. Plotting Random-Restart Progress

In the previous graph, we plotted the evaluation score during one Hill-Climbing run.

Now we plot the **best evaluation score found so far** after each restart.

### Code Cell

```python
plt.figure(figsize=(9, 5))

plt.plot(
    range(1, len(best_scores) + 1),
    best_scores,
    marker="o"
)

plt.xlabel("Restart")

plt.ylabel("Best Evaluation Score So Far")

plt.title("Random-Restart Hill-Climbing: N-Queens")

plt.grid(True)

plt.show()
```

### Interpret the Graph

The graph should be non-decreasing because the algorithm retains the best result found so far.

For example, the best score might change as follows:

```text
-5, -3, -3, -2, -1, 0
```

The algorithm improves the best-known result over several restarts.

Once the score reaches zero, a valid solution has been found, and the algorithm stops.

**Difference between the two graphs:**

- The single-run graph shows the score during one search.
- The random-restart graph shows the best score retained across repeated searches.

## 16. Comparing Basic and Random-Restart Hill-Climbing

Run both algorithms five times.

### Code Cell

```python
print("Basic Hill-Climbing")
print("-------------------")

for i in range(5):

    state, scores = hill_climbing(8)

    conflicts = count_conflicts(state)

    print(
        f"Run {i + 1}: "
        f"Conflicts = {conflicts}, "
        f"Iterations = {len(scores)}, "
        f"Solved = {conflicts == 0}"
    )


print("\nRandom-Restart Hill-Climbing")
print("----------------------------")

for i in range(5):

    state, restarts, best_scores = (
        random_restart_hill_climbing(8, max_restarts=100)
    )

    conflicts = count_conflicts(state)

    print(
        f"Run {i + 1}: "
        f"Conflicts = {conflicts}, "
        f"Restarts = {restarts}, "
        f"Solved = {conflicts == 0}"
    )
```

### Task 3: Compare the Results

**Q1. Did basic Hill-Climbing find a solution in every run?**

Answer: Not necessarily. Basic Hill-Climbing starts from a single random state and can stop at a local maximum with remaining conflicts.

**Q2. Did Random-Restart Hill-Climbing improve the results?**

Answer: Generally, yes. Random-Restart provides multiple opportunities to find a solution instead of relying on only one initial state.

**Q3. How many restarts were needed to find a solution?**

Answer: The number of restarts depends on the random starting states. It may find a solution on the first attempt or require several attempts.

**Q4. Did any random-restart experiment reach its restart limit without finding a solution?**

Answer: It is possible. Random-Restart does not guarantee success within a fixed number of attempts.

**Q5. Why can Random-Restart improve the probability of success without guaranteeing success within a fixed number of attempts?**

Answer: Each restart provides another opportunity to begin from a different random state. If each attempt has a positive probability of success, repeated independent attempts increase the overall probability of finding a solution. However, a finite number of attempts can still all fail.

## 17. Lab Questions

### Question 1: What is the purpose of the evaluation function in Hill-Climbing?

Answer:

The evaluation function measures the quality of a state.

In the N-Queens problem, it counts the number of conflicts between queens and converts that number into a score.

Hill-Climbing uses this score to identify better neighboring states.

### Question 2: Why is an evaluation score of zero the best result in our N-Queens implementation?

Answer:

The evaluation score is the negative number of conflicts.

A score of zero means there are no conflicts between queens. Therefore, no two queens attack each other, and the N-Queens problem has been solved.

### Question 3: What is the difference between a local maximum and a global maximum?

Answer:

A local maximum is a state whose evaluation score is higher than or equal to the scores of its immediate neighbors.

A global maximum is the best possible state in the entire search space.

In our N-Queens problem, a global maximum has an evaluation score of zero.

A local maximum may have a negative score, meaning that some conflicts still exist.

### Question 4: Why can basic Hill-Climbing stop without finding a solution even when a solution exists?

Answer:

Basic Hill-Climbing only examines neighboring states of the current state.

It stops when none of the neighboring states improves the current evaluation score.

A valid solution may exist elsewhere in the search space, but the algorithm cannot reach it without first making a move that does not improve the score.

### Question 5: Does reaching a local maximum always mean that the algorithm has failed? Explain.

Answer:

No. A local maximum is not necessarily a failure.

If the local maximum is also the global maximum, it represents a valid solution.

In the N-Queens problem, reaching an evaluation score of zero means that the algorithm has successfully solved the problem.

### Question 6: For N = 8, how many neighbors does each state have in our implementation?

Answer:

Each queen can move to N - 1 other rows in its column.

Therefore:

\[
N \times (N-1)
\]

For N = 8:

\[
8 \times 7 = 56
\]

Each state has 56 neighbors.

### Question 7: What does the evaluation-score graph tell you about the behavior of Hill-Climbing?

Answer:

The graph shows how the evaluation score changes during the search.

The score increases whenever the algorithm moves to a better state.

It never decreases because the algorithm only accepts improving moves.

The graph becomes flat when the algorithm reaches a state with no better neighboring state and terminates.

### Question 8: Explain probabilistic completeness in your own words.

Answer:

Probabilistic completeness means that the probability of finding a solution approaches 100% as the number of independent search attempts increases, provided that a solution exists and each attempt has a positive probability of finding it.

### Question 9: Why does Random-Restart Hill-Climbing improve the chance of finding a solution?

Answer:

Random-Restart starts the search from different random states.

This gives the algorithm additional opportunities to avoid local maxima encountered during previous attempts.

As a result, it generally has a better chance of finding a valid solution than a single Hill-Climbing run.

### Question 10: Does probabilistic completeness guarantee that a solution will be found within 100 restarts? Why or why not?

Answer:

No. Probabilistic completeness does not guarantee success within a fixed number of attempts.

Even if each attempt has a positive probability of success, all 100 attempts could still fail.

The probability of finding a solution increases with additional independent attempts, but a finite restart limit cannot guarantee success.

## 18. Extension Task

Modify the experiment to solve the N-Queens problem for N = 20.

Record the following:

- Number of conflicts in the final state.
- Number of iterations in a basic Hill-Climbing run.
- Number of restarts required by Random-Restart Hill-Climbing.
- Whether a valid solution was found.

### Code Cell

```python
# Basic Hill-Climbing for N = 20

solution, scores = hill_climbing(20)

print("Basic Hill-Climbing: N = 20")

print("Final conflicts:", count_conflicts(solution))

print("Iterations:", len(scores))

print("Solved:", count_conflicts(solution) == 0)


# Random-Restart Hill-Climbing for N = 20

solution, restarts, best_scores = (
    random_restart_hill_climbing(20, max_restarts=100)
)

print("\nRandom-Restart Hill-Climbing: N = 20")

print("Final conflicts:", count_conflicts(solution))

print("Restarts:", restarts)

print("Solved:", count_conflicts(solution) == 0)
```

### Observation

Random-Restart Hill-Climbing provides additional opportunities to find a solution.

However, the results depend on the initial states and the number of restarts.

Compare the results of basic and Random-Restart Hill-Climbing and discuss how the number of queens and random initial states affect the search process.

## 19. Conclusion

In this lab, we implemented the Hill-Climbing algorithm to solve the N-Queens problem.

We learned how to represent a chessboard as a Python list, calculate conflicts between queens, generate neighboring states, and select better states using an evaluation function.

We also observed that basic Hill-Climbing can terminate at a local maximum without finding a valid solution.

To improve the probability of success, we implemented Random-Restart Hill-Climbing, which repeatedly starts the search from new random states.

The experiments demonstrated that repeated randomized searches provide additional opportunities to find a valid solution.

**Key Takeaway:** Hill-Climbing improves the current state by selecting better neighbors, but it can become trapped at a local maximum. Random-Restart Hill-Climbing reduces this problem by providing multiple opportunities to search for a solution.

## Submission Checklist

- [x] State representation implemented.
- [x] Random initial state generator implemented.
- [x] Conflict-counting function implemented.
- [x] Evaluation function implemented.
- [x] Neighbor-generation function implemented.
- [x] Basic Hill-Climbing implemented.
- [x] Chessboard visualization implemented.
- [x] Evaluation-score graph implemented.
- [x] Different board sizes tested.
- [x] Random-Restart Hill-Climbing implemented.
- [x] Random-Restart progress graph implemented.
- [x] Comparison between both algorithms implemented.
- [x] Lab questions answered.
- [x] Extension experiment for N = 20 included.

**Final verification:** Run the notebook in Google Colab from top to bottom. Check that all code cells execute without errors, both graphs are generated, and the experimental results are recorded before submission.
