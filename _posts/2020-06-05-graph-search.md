---
layout: post
title: "Graph Search"
---
Graph Search problems are common so I wanted to write down some of my thoughts on how to implement and solve them as efficently as possible using the syntactical sugar of Python while being stranded on an island.

### Purpose
From practicing technical interview questions, I noticed that a theoretical grasp of different algorithms was helpful only to find an overall direction towards implementing these different algorithms. In most cases, the difficult part of solving a problem, personally, seemed to be:
* Creating the actual structure that allowed for an abstract algorithm to function based on the problem statement.
* Implementing these abstract algorithms efficently in different languages from which I was initially introduced to them in.

This post and others to come are personally a way to account for different approaches in implementing an algorithm based on context and different measures of increasing readability using language specific optimizations.

### Graphs

Graphs in general are an important data structure that allow for the representation of data in terms of **nodes** and **edges**. Each node in a graph has a corresponding **set of edges** that represent connections to other nodes.

![Graph Structure](/assets/images/graph_search/graph_demo.png)

Representing the abstract idea of a graph can take numerous forms but once implemented, searching algorithms on graphs become universal across representations.

Graphs are flexible data structures that have numerous complex algorithms that can be applied but here I want to mainly focus on **traversal algorithms** useful in coding challenges.

To give an example of a problem that might appear in an interview context I'll be describing a graph problem from *LeetCode*. When tackling problems of this nature, there is a system of steps that could be applied for revealing a solution.

#### Problem:
Given a 2d grid map of '1's (land) and '0's (water), count the number of islands. An island is surrounded by water and is formed by connecting adjacent lands horizontally or vertically. You may assume all four edges of the grid are all surrounded by water.

#### Samples:
```
Input:
11110
11010
11000
00000

Output: 1
```

```
Input:
11000
11000
00100
00011

Output: 3
```

### Solving
From initial observation, this problem asks the programmer to:
* Find a continous group of land tiles on the grid connected **horizontally** and **vertically** surrounded by water tiles
* Count the number of independent groups within the grid

Using this information, we can conclude that we must loop through all possible grid positions and then increment a counter if the current grid position is a part of an island that we have **not explored yet**.

It's easy to visualize this problem more concretely using the analogy of being stranded on an island. 
* If we were on one island we would search the **entire area** of the island until we there was no more **unvisited** land on the island.
    * In this case we would know we were stuck on one singular island and could conclusively add one island to our island count.
* Now if we could also magically **fly off** the island **within a bounded area**
    * We skip water tiles as they are not apart of islands
    * We land on another distinct island within the area we could repeat the above process until we flew over the entire bounded area

![Island Drawing](/assets/images/graph_search/island_analogy.png)

This process of searching for the limits of a singular island can be abstracted as a **graph traversal** problem and solved using either a *Breadth First Search (BFS)* or *Depth First Search (DFS)*. In this context both algorithms have similar complexity so I will be focusing on DFS for this problem.

However before applying the above outline algorithm to the problem we need some way to first represent the data input as a graph.

### Graph Representations
Graphs can be internally represented in any solution using numerous different techniques ranging from adjacency lists and sets to adjacency matrices. These more robust solutions extend beyond application within simple traversal problems as outlined above but representing the input as a graph is still pivotal for the program.

To achieve this we can set up functional abstractions within the problem that allow for graph operations on a simple data structure. To implement most traversal algorithms we need functional abstractions that:
* Check if a node has been visited
* Mark a node as visited
* Return adjacent nodes to specified node

If we imagine the problem case outlined above and take an input as a **2D list**, we can use this input and **destructively** modify it to **optimize** use space and accurately represent a functional representation of the algorithm.

Due to the constraints of the problem, we know that visiting a node in this problem has a similar effect to changing this node to a water node. This is due to the fact that we do not search through water nodes and therefore **water nodes** are **automatically visited**.

```python
# Problem Input (Example 1)
grid = [[1,1,1,1,0],
        [1,1,0,1,0],
        [1,1,0,0,0],
        [0,0,0,0,0]]
row_len = len(grid)
col_len = len(grid[0])
# Abstract Node as Tuple (Row, Col)
node = (1,2)
'''
Functional Abstractions
'''
# Row of Node
def row(node):
    return node[0]
# Column of Node
def col(node):
    return node[1]
# Return Node
def node(row, col):
    return (row, col)
```
With the tuple abstraction that can further be optimized later when implementing the full algorithm we are able to create a working abstraction for graph traversal without having to create a copy of the grid. With the actual implementation of the algorithm we can do away with the tuple abstraction later by using higher order functions within our abstraction methods.
``` python
# Check Visited
def visited(node):
    return grid[row(node)][col(node)] == 0
# Mark Vist
def vist(node):
    grid[row(node)][col(node)] = 0
# Return Adjacents
def edges(node):
    edge_list = []
    r, c = row(node), col(node)
    # Down
    if r < row_len - 1:
        edge_list.append(node(r + 1, c))
    # Up
    if r > 0:
        edge_list.append(node(r - 1, c))
    # Right
    if r < col_len - 1:
        edge_list.append(node(r, c + 1))
    # Left
    if r > 0:
        edge_list.append(node(r, c - 1))
    return edge_list
```
Using the idea of a graph and possible traversal points due to the problem, we know our possible movements are only up, down, left and right. This way we construct a possible edge list for a given node by only traversing nodes in those coordinate directions by manipulating the row and col values.

### Abstract Traversal
In general terms any DFS graph traversal can be abstracted using the following with obvious modifications based on the processing needed to be done on each node or additional conditions that terminate the search process:
``` python
# Iterative
def dfs_iterative(inital_nodes):
    stack = []
    stack.append(initial_nodes)
    while stack:
        node = stack.pop()
        if not visted(node):
            visit(node)
            for adj_node in edges(node):
                if not visited(adj_node):
                    stack.append(adj_node)
```
```python
# Recursive
def dfs_recursive(initial_nodes):
    def dfs(node):
        if not visited(node):
            visit(node)
            for adj_node in edges(node):
                dfs(adj_node)
    for node in initial_nodes:
        dfs(node)
```
The basic premise of both algorithms boils down to these main procedures. The algorithm loops through these steps until all nodes are visited.
1. Start with a set of initial nodes that we would like to search
    * In the case of our problem this inital node set would include a single node starting point such as the top left corner
2. Remove an arbitrary node from this set
3. Check if the node has not been visited and if this is true continue
    * In the case that the node has been visited we know that this node has already been searched therefore we do not need to do any additional proccessing and skip that node.
4. Mark the node as visited
5. Iterate through the adjacent nodes of the current node and repeat the above steps.

### The Analogy
Putting all these parts together we can see how we can solve our inital island problem. Recall that our original problem required us to find the number of islands within a certain area grid that identifies land as a `1` and water as a `0`. We developed algorithms under these assumptions.
* If we are currently on an island, we need to **traverse the entire island** until there is no more un-traversed land on our island.
    * To make it easier for us to keep track of where we have been, we mentally **change land** that we have traversed **into water** because we dont search water.
    * Once we've traversed an entire island we need to **add one** to our mental count of the number of islands within our boundry.
* If we are currently flying above water **we dont need to search it** because we only want to count the number of islands within our boundry.
* Like this we fly over the entire boundry until there is no more land.

The final algorithm is below. Analyzing the complexities of the algorithm, we can reduce the: 
* Additional space complexity to **O(1)**
* Our time complexity can be based upon the operation of iterating through only N nodes and since our dfs algorithm iterates through E edges of a particular node and this edge retrival runs in constant time we can conclude that our complexity cannot exceed O(E*N) but since our edge count for each node cannot exceed 4 our complexity becomes O(4N) to **O(N)**

```python
# Problem Input (Example 1)
grid = [[1,1,1,1,0],
        [1,1,0,1,0],
        [1,1,0,0,0],
        [0,0,0,0,0]]
row_len = len(grid)
col_len = len(grid[0])

# Row of Node
def row(node):
    return node[0]
# Column of Node
def col(node):
    return node[1]
# Return Node
def node(row, col):
    return (row, col)

# Check Visited
def visited(node):
    return grid[row(node)][col(node)] == 0
# Mark Vist
def vist(node):
    grid[row(node)][col(node)] = 0
# Return Adjacents
def edges(node):
    r, c = row(node), col(node)
    # Down
    if r < row_len - 1:
        yield node(r + 1, c)
    # Up
    if r > 0:
        yield node(r - 1, c)
    # Right
    if r < col_len - 1:
        yield node(r, c + 1)
    # Left
    if r > 0:
        yield node(r, c - 1)

# Recursive
def island_search(grid):
    def dfs(node):
        if not visited(node):
            visit(node)
            for adj_node in edges(node):
                dfs(adj_node)
    islands = 0
    for r in grid:
        for c in r:
            dfs(node(r,c))
            islands += 1
    return islands
```
The algorithm uses a generator to yield each of the edges for a constant additional space complexity. Also using the recursive structure allows for a further reduction in additional space. Overall, this approach can be extended to more complex problems by modifying the vist, visited and edges abstractions that are created making this design much more expandable than other alternatives such as hardcoding rules. We never know the next time we might be stranded on an island so hopefully this gets us prepared?












