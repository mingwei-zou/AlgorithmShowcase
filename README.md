[Divide and Conquer](#divide-and-conquer) · [Dynamic Programming](#dynamic-programming) · [Greedy Algorithm](#greedy-algorithm)

# AlgorithmShowcase-main

**Mingwei Zou | Mathematical modeling · Algorithm design · Visual explanation**

A visual portfolio of three fundamental algorithmic paradigms. Each showcase connects a problem formulation to its implementation through a demonstration video, a technical explanation, and an evaluation of the results.

The portfolio focuses on translating mathematical ideas into computational procedures and assessing their correctness, efficiency, and limitations.

---

## Divide and Conquer

<!-- DIVIDE-AND-CONQUER:CONTENT:START -->

**2D Convex Hull | Computational Geometry · Recursive Decomposition · Geometric Predicates**

This project constructs the convex hull of a planar point set through recursive decomposition and geometric merging. It connects a mathematical description of an enclosing boundary with orientation tests, linked data structures, and a visual account of how smaller solutions combine into a larger one.

The technical focus is translating geometric reasoning into an executable algorithm whose intermediate states and final output can be inspected.

### Algorithm Visualization

<p align="center">
  <img src="./DandC.gif"
       alt="Divide and Conquer: Convex Hull Construction"
       width="800">
</p>

<!-- Replace the placeholder with the actual recording URL. -->

The viewer pauses before and after recursive merges, making it possible to examine the two component hulls and their combined boundary. Press **P** in the graphics window to advance through the paused stages.

| Visual element | Meaning in the implementation |
| --- | --- |
| Outlined circles | Input points |
| Yellow highlighting | Points in the current recursive subproblem, or points selected for inspection |
| Blue arrows | Links stored in `ccwPoint` |
| Red arrows | Links stored in `cwPoint` |

The arrows expose the underlying data structure: a hull is represented by neighboring-point links in both directions. Intermediate links can remain visible on points excluded from the final hull, as described under implementation limitations below.

### Problem and objective

Given a finite set of points in two dimensions, the **convex hull** is the smallest convex set containing them. For a non-collinear input, its boundary is a polygon whose vertices are drawn from the input points; the two-point case reduces to a line segment.

The computational task is to identify these boundary vertices and connect them in order. Interior points remain part of the original dataset but do not belong to the final boundary.

The recursive strategy uses the identity:

```text
conv(P_L ∪ P_R) = conv(conv(P_L) ∪ conv(P_R))
```

This means that the two subproblem hulls contain enough information to construct the combined hull. Points strictly inside either subproblem hull cannot become new extreme vertices of the union.

### Algorithm and implementation

The implementation in [`main.py`](Divide%20and%20Conquer/main.py) follows three stages:

1. **Divide.** Sort points by increasing `x`, using `y` to break ties. Split the ordered list into two approximately equal halves.
2. **Conquer.** Recursively construct each half's hull. Two-point and non-collinear three-point cases are handled directly by assigning clockwise and counterclockwise links.
3. **Combine.** Start at the rightmost point of the left hull and the leftmost point of the right hull. Walk along their boundaries using orientation tests to locate the two connecting tangents, reconnect the surviving boundary chains, and traverse the merged hull.

The geometric decision is based on the signed determinant:

```text
orient(a, b, c) = (a_x - c_x)(b_y - c_y) - (b_x - c_x)(a_y - c_y)
```

Its sign identifies a left turn, a right turn, or collinearity in the coordinate system. The `turn()` function supplies this predicate to the base cases and tangent searches.

| Component | Role |
| --- | --- |
| `buildHull()` | Handle base cases, divide the point list, and combine recursive results |
| `turn()` | Convert geometric orientation into a numerical decision |
| `merge()` | Search for connecting tangents and splice the hull boundaries |
| `cwPoint` / `ccwPoint` | Store each boundary point's two neighbors |
| `display()` | Expose intermediate structures before and after merges |

A useful correctness argument follows the same recursive structure: establish the small base cases, assume that the two subproblem hulls are correct, and show that the tangent connections preserve the outer boundary of their union. The visualization supports inspection of these steps; validation of the implementation provides a separate empirical check.

### Results and validation

The uploaded implementation was evaluated on the repository's five supplied point files. The original geometry functions were executed with rendering disabled, and their returned hull vertex sets were compared with an independently implemented monotone-chain reference algorithm.

| Input | Input points | Returned hull vertices | Vertex-set comparison | Reciprocal hull links |
| --- | ---: | ---: | --- | --- |
| `points1.txt` | 2 | 2 | Matched | Passed |
| `points2.txt` | 3 | 3 | Matched | Passed |
| `points3.txt` | 10 | 7 | Matched | Passed |
| `points4.txt` | 66 | 12 | Matched | Passed |
| `points5.txt` | 50 | 14 | Matched | Passed |

The same comparisons passed with `discardPoints` both disabled and enabled. Reciprocal-link checks confirmed that following a final hull vertex's clockwise link and then its counterclockwise link returns to that vertex, and vice versa.

These results support the returned boundary vertices and link consistency on the supplied inputs. They are not a claim of correctness for every possible point configuration, nor a runtime benchmark.

### Complexity and limitations

For balanced division and linear-time tangent merging on valid, non-degenerate hulls, the geometric computation follows:

```text
T(n) = T(⌊n/2⌋) + T(⌈n/2⌉) + O(n) = O(n log n)
```

Initial sorting also costs `O(n log n)`. Point records, temporary lists, and hull representations use `O(n)` peak storage under this model, with `O(log n)` recursive depth.

**Visualization has a separate cost.** Each call to `display()` redraws every input point. Across the recursion, this can contribute `O(n²)` drawing work, before additional refreshes and user-controlled pauses. The geometric bound therefore should not be interpreted as the elapsed complexity of the interactive demonstration.

<details>
<summary>Current implementation boundaries</summary>

- The code has no explicit zero-point or one-point base case, and its three-point case does not construct links for collinear points. Duplicate points and other degenerate configurations need additional handling.
- Orientation uses floating-point arithmetic and an exact zero comparison, so nearly collinear configurations require more robust numerical treatment.
- The optional `-d` mode attempts to remove obsolete links during merging. Checks on `points3.txt`, `points4.txt`, and `points5.txt` found that some excluded points still retain a link, even though the returned hull vertex sets and reciprocal links pass the checks above. Complete cleanup would require clearing both pointers of every excluded point while preserving the final boundary.

</details>

These distinctions make the project useful for studying the relationship between a mathematical algorithm, its representation in code, and the additional work needed for reliable numerical and visual behavior.

### Run the demonstration

Requires Python 3, PyOpenGL, GLFW, and a desktop environment that supports an OpenGL window. From the repository root:

```bash
python -m pip install PyOpenGL glfw
python "Divide and Conquer/main.py" "Divide and Conquer/points3.txt"
```

Use `points1.txt` and `points2.txt` to inspect the base cases, or `points4.txt` and `points5.txt` for larger recursive examples. Input files contain one `x y` coordinate pair per line.

| Control or option | Behavior |
| --- | --- |
| `P` in the graphics window | Advance past the current pause |
| Click a point | Print its coordinates and toggle its highlight |
| `Esc` | Exit |
| `-np` before the input filename | Disable instructional pauses |
| `-d` before the input filename | Enable the existing link-discarding logic, subject to the limitation above |

*Project context:* Based on [CMPE/CISC 365 Assignment 1](Divide%20and%20Conquer/A1.txt), using its supplied visualization framework. The algorithmic work centers on convex-hull base cases, recursive construction, and merging through geometric predicates and pointer updates.

<!-- DIVIDE-AND-CONQUER:CONTENT:END -->

[Back to overview](#algorithmshowcase-main)

---


## Dynamic Programming

<!-- DYNAMIC-PROGRAMMING:CONTENT:START -->

**3D Surface Reconstruction | Geometric Modeling · Discrete Optimization · Solution Reconstruction**

This project connects ordered 3D contours into a triangular surface using dynamic programming. For each adjacent pair of contours, it represents alternative connections as paths through a two-dimensional state space and minimizes their accumulated triangle area.

The technical focus is turning a geometric objective into a recurrence, reconstructing the decisions behind the minimum cost, and checking how the resulting mesh relates to the mathematical model.

### Algorithm Visualization

<p align="center">
  <img src="./dp.gif"
       alt="Dynamic Programming: Surface Reconstruction from 3D Contours"
       width="800">
</p>

The viewer provides two complementary views: the input contours and the surface constructed between them. Computing one contour pair isolates the connection problem; computing all adjacent pairs builds the lateral surface of the supplied femur dataset. Rotation and zoom support inspection of the resulting geometry.

| Visual element | Meaning in the implementation |
| --- | --- |
| Contour curves before computation | Ordered input vertices connected around each slice |
| Shaded triangles after computation | Faces reconstructed by backtracking through the DP decisions |
| One-pair mode | Two adjacent contours selected for a smaller reconstruction problem |
| Red, green, and blue axes | The `x`, `y`, and `z` coordinate directions |

### Problem and objective

The input is a sequence of closed contours, each represented by an ordered list of 3D vertices. Adjacent contours can contain different numbers of vertices. The task is to connect them with triangles while preserving their cyclic vertex order.

Each triangle advances one edge on either the upper or lower contour and connects that edge to a vertex on the other contour. The objective is to minimize the sum of triangle areas within this connection model.

The implementation first selects the closest cross-contour vertex pair as a starting connection. It then rotates both vertex lists to begin at those vertices and repeats each first vertex at the end to represent a complete circuit. The DP optimum is therefore conditional on this chosen starting connection and the permitted order-preserving steps.

### Algorithm and implementation

The algorithm is implemented in [`buildTriangles()` in slices.py](Dynamic%20Programming/slices.py).

1. **Choose the starting connection.** Examine all cross-contour vertex pairs and select the closest pair by Euclidean distance.
2. **Define the state.** Let `U[0] ... U[m]` and `L[0] ... L[n]` denote the rotated upper and lower contours, with `U[m] = U[0]` and `L[n] = L[0]`. The state `D[r, c]` stores the minimum accumulated area of a partial triangulation ending at the connection between `L[r]` and `U[c]`.
3. **Compare two predecessor states.** Advance along the lower contour from the previous row, or along the upper contour from the previous column. Each transition adds exactly one triangle.
4. **Recover the triangles.** Store the selected predecessor in `minDir`, then backtrack from `D[n, m]` to `D[0, 0]` to reconstruct the chosen connections.

Writing `A(a, b, c)` for a triangle's area, the computation is:

```text
A(a, b, c) = 0.5 * ||(b - a) 脳 (c - a)||

D[0, 0] = 0
D[0, c] = D[0, c-1] + A(L[0], U[c-1], U[c])
D[r, 0] = D[r-1, 0] + A(U[0], L[r-1], L[r])

D[r, c] = min(
    D[r-1, c] + A(U[c], L[r-1], L[r]),
    D[r, c-1] + A(L[r], U[c-1], U[c])
)
```

The boundary formulas apply where only one predecessor exists; the final formula applies when both indices are positive. The row index refers to the lower contour and the column index to the upper contour. Equal candidate costs select the previous column in the current implementation.

This is also a shortest-path problem on a directed acyclic grid: each transition carries a triangle-area cost. Any allowed path reaching a state must arrive through one of its two predecessors. Replacing a non-optimal prefix with a cheaper prefix leaves the same frontier connection, which gives the recurrence its optimal-substructure justification.

| Component | Role |
| --- | --- |
| `minArea[r][c]` | Store the minimum accumulated area for a state |
| `minDir[r][c]` | Record `PREV_ROW` or `PREV_COL` for reconstruction |
| `triangleArea()` | Calculate area from the magnitude of a cross product |
| `Triangle` | Store the reconstructed vertices and a normal for rendering |
| `readSlices()` | Read contours and connect each contour's vertices cyclically |

### Results and validation

Headless checks executed the uploaded geometry code on both supplied datasets, with graphical initialization omitted and diagnostic printing suppressed. The test harness observed the final DP cost without changing the recurrence or returned triangle list.

| Input | Contours | Input vertices | DP states across adjacent pairs | Triangle output: non-zero + zero area |
| --- | ---: | ---: | ---: | ---: |
| `testSlices.dat` | 2 | 8 | 25 | 8 + 1 |
| `femurSlices.dat` | 61 | 17,960 | 5,839,087 | 35,672 + 60 |

For the small test, all **70 monotone paths** between the chosen endpoints were enumerated independently. Their minimum agreed with the DP result of **960 square coordinate units**, within floating-point tolerance.

Across the test pair and all **60 femur contour pairs**, the reconstructed triangle areas summed to the final DP costs within numerical tolerance. After excluding zero-area faces, each pair produced the expected `m + n` triangles, covered every contour edge once, and used every interior edge twice.

The zero-area counts expose an implementation issue: after backtracking, the code appends one extra triangle whose first and third vertices are identical. Thus, the original function returns 9 triangles for the small test and 35,732 across the femur dataset. These extra faces do not change the area sum. Directed-edge checks also found inconsistent triangle winding, as explained below.

The approximately 5.84 million states are accumulated over separate contour-pair problems; they are not all stored simultaneously. These checks establish the reported numerical and edge-incidence results on the supplied data, rather than a runtime benchmark or a guarantee of a valid surface for arbitrary inputs.

### Complexity and limitations

For two contours containing `m` and `n` vertices, the closest-pair search and DP table construction each take `O(mn)` time. Backtracking takes `O(m + n)` time. The cost and predecessor tables require `O(mn)` space.

For a sequence of contours, total computational work is proportional to the sum of the products of adjacent contour sizes. Pairs are processed sequentially, so peak DP storage is determined by the largest adjacent pair, alongside storage for input vertices and accumulated triangles.

<details>
<summary>Optimization scope and current implementation boundaries</summary>

- **Fixed starting connection.** The closest-pair choice is a heuristic. The DP finds a minimum among the monotone paths represented by that choice; it does not search every possible seam or every possible 3D triangulation.
- **Mesh construction details.** The extra zero-area triangle should be removed because repeating the starting vertices already closes the path. The two backtracking branches also produce inconsistent vertex winding. Shared-edge orientation needs to be made consistent before the mesh can be treated as having coherent outward-facing normals.
- **Geometric quality.** Minimizing area does not enforce well-shaped triangles or prevent self-intersections. The code connects adjacent contours without constructing end caps or handling contour branching, so it does not guarantee a watertight solid.
- **Diagnostic overhead.** The current code prints every DP table entry, which adds substantial terminal output on the femur data. The displayed costs are truncated to integers for printing; the optimization itself uses floating-point values.

</details>

The project illustrates how state design and backtracking make a geometric optimization problem computationally tractable, while numerical and structural checks distinguish the optimized objective from the quality of its mesh representation.

### Run the demonstration

Requires Python 3, PyOpenGL, GLFW, and a desktop environment that supports an OpenGL window. From the repository root, start with the small test:

```bash
python -m pip install PyOpenGL glfw
python "Dynamic Programming/slices.py" "Dynamic Programming/testSlices.dat"
```

For the femur reconstruction:

```bash
python "Dynamic Programming/slices.py" "Dynamic Programming/femurSlices.dat"
```

Press **C** in the graphics window to compute the triangulation. To inspect an individual contour pair, enter one-pair mode with **S**, choose a pair with **,** or **.**, then press **C**. Changing the selection does not rebuild an existing mesh; press **C** again to recompute it.

| Control | Behavior |
| --- | --- |
| `C` | Compute the selected pair or all adjacent pairs, according to the current mode |
| `S` | Toggle between one-pair and all-contour mode |
| `,` / `.` | Move to the previous or next contour pair; the help text labels these `<` / `>` |
| Left-button drag | Rotate the view |
| Right-button drag up/down | Zoom |
| `/` or `?` key | Print the controls in the terminal |
| `V` / `E` / `T` | Toggle vertex, edge, or triangle labels when optional GLUT font support is enabled |
| `Esc` | Exit |

Input files begin with a contour count, followed by a vertex count and `x y z` coordinate rows for each contour. GLUT labels are optional; the supplied code leaves `haveGlutForFonts = False`.

*Project context:* Based on [CMPE/CISC 365 Assignment 3](Dynamic%20Programming/A3.txt), using its supplied datasets and visualization framework. The algorithmic work centers on choosing a starting connection, defining and filling the DP tables, and reconstructing triangles from predecessor decisions.

<!-- DYNAMIC-PROGRAMMING:CONTENT:END -->

[Back to overview](#algorithmshowcase-main)

---

## Greedy Algorithm

<!-- GREEDY-ALGORITHM:CONTENT:START -->

**Triangle Stripification | Computational Geometry · Graph Modeling · Discrete Optimization**

This project explores how local algorithmic decisions shape a global geometric solution. It converts a triangle mesh into edge-adjacent strips, connecting a mathematical optimization objective with explicit data structures, an executable heuristic, and checks on the resulting solution.

The work forms part of my broader interest in translating mathematical formulations into reliable computational methods.

### Algorithm Visualization

<p align="center">
  <img src="./Greedy_video.gif" alt="Greedy Algorithm Demonstration" width="800">
</p>

<!-- Paste the actual recording URL on its own line, replacing the placeholder above. -->

**Example output: 200 vertices · 384 triangles · 15 triangle strips**

In the original mesh viewer, black edges outline the triangles, colored regions indicate strip membership, and white segments connect the centers of consecutive triangles. Following these segments reveals how local connections combine into paths across the mesh. Colors are visual aids and may look similar across different strips.

### Problem and objective

A triangle mesh represents a surface through triangular faces. In this project's formulation, a **triangle strip** is a sequence of distinct triangles in which consecutive triangles share an edge.

The objective is to partition the mesh into as few strips as possible while ensuring that:

- Every triangle is included exactly once.
- Each consecutive pair of triangles shares an edge.
- Each strip forms a path without repeated triangles or cycles.

The geometric problem can be expressed as a graph problem: triangles become nodes, and shared edges define adjacency. The task is then to find a vertex-disjoint path cover with a small number of paths. This representation makes both the optimization objective and the feasibility constraints explicit.

### Algorithm and implementation

The greedy heuristic prioritizes triangles with fewer available connection options. The intuition is that these constrained triangles may become harder to incorporate if their neighbors are assigned elsewhere first.

The implementation applies this principle in two stages:

1. **Order the starting triangles.** Sort triangles once by their initial available-neighbor count, then consider eligible strip starts in that fixed order.
2. **Extend each strip.** From the current endpoint, choose an available adjacent triangle that itself has the fewest available neighbors. Connect it to the strip and continue until no extension is possible.

For endpoint `c`, the extension rule is:

$$
t^* = \arg\min_{t \in N_{\mathrm{avail}}(c)} |N_{\mathrm{avail}}(t)|.
$$

Here, `N_avail` denotes neighbors eligible under the implementation's pointer-based checks. Candidate scores are evaluated as the strip grows; the initial seed order is not recomputed between strips. Ties follow the existing list order.

| Implementation component | Purpose |
| --- | --- |
| Shared-edge lookup | Match sorted vertex-index pairs to construct mesh adjacency |
| `adjTris` | Store the neighboring triangles used in local decisions |
| `nextTri` and `prevTri` | Represent each strip as a doubly linked sequence |
| Strip colors and center-to-center links | Make the computed structure visually inspectable |

The core logic is implemented in [`buildTristrips()`](Greedy%20Algorithm/tristrips.py). Each extension commits to one local choice, without backtracking or enumerating complete strip arrangements.

### Results and validation

The supplied desktop run confirms that `data/200` contains **200 vertices and 384 triangles**, producing **15 strips**. Core-function checks on all three repository datasets yielded:

| Dataset | Vertices | Triangles | Generated strips | Structural validation |
| --- | ---: | ---: | ---: | --- |
| `data/200` | 200 | 384 | 15 | Passed |
| `data/1000` | 1,000 | 1,978 | 82 | Passed |
| `data/10000` | 10,000 | 19,969 | 872 | Passed |

Validation traversed every resulting strip and checked complete coverage, absence of repeated triangles and cycles, adjacency of consecutive triangles, and reciprocal forward/backward pointers. Reconstructed strip counts agreed with the routine's printed output.

These checks establish feasible solutions on the supplied inputs. They distinguish structural correctness from the separate question of how close the strip count is to the global minimum.

<details>
<summary>Evaluation scope</summary>

The results above come from executing the original mesh reader and greedy routine without the OpenGL interface. The checked source and datasets match repository commit `c76e0428b691596d6d511e439e88c90fab6676d1`.

The course's [`results.txt`](Greedy%20Algorithm/results.txt) contains reference figures, including different strip counts on the larger inputs. Its baseline reduction and machine-specific timings are not measurements of this implementation. No runtime or rendering-speed improvement is claimed here.

</details>

### Complexity and limitations

For a valid manifold triangle mesh, each triangle has at most three edge-neighbors. With `n` triangles and `v` vertices:

- Reading the mesh and constructing adjacency takes expected `O(v + n)` time using shared-edge hashing.
- Sorting the initial starting order takes `O(n log n)` time.
- Strip extension takes `O(n)` time because each local candidate search has bounded size.

The resulting bound is **expected `O(v + n log n)` time and `O(v + n)` memory**, excluding visualization.

The heuristic produces a feasible path cover but does not guarantee the minimum number of strips. Initial ordering and tie-breaking can affect solution quality. A natural extension would compare fixed seed ordering with dynamically updated priorities or limited look-ahead, measuring improvements in strip count against additional computational cost.

This distinction between objective, heuristic, and verified outcome is central to the project's technical value: translating a mathematical problem into code while evaluating what the implementation actually establishes.

### Run the demonstration

Requires Python 3, PyOpenGL, GLFW, and a desktop environment that supports an OpenGL window. From the repository root:

```bash
python -m pip install PyOpenGL glfw
python "Greedy Algorithm/tristrips.py" "Greedy Algorithm/data/200"
```

Replace the final dataset path with `Greedy Algorithm/data/1000` or `Greedy Algorithm/data/10000` to inspect larger inputs. If the local directory is named `Greedy-Algorithm`, use that spelling in both paths.

*Project context:* Based on [CMPE/CISC 365 Assignment 2](Greedy%20Algorithm/A2.txt), using its supplied visualization framework. The algorithmic focus is the greedy strip-building routine and the interpretation of its results.

<!-- GREEDY-ALGORITHM:CONTENT:END -->

[Back to overview](#algorithmshowcase-main)
