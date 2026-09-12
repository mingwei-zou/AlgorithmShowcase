# Greedy Algorithm: Triangle Stripification

[All showcases](../README.md) · [Divide and Conquer](divide-and-conquer.md) · [Dynamic Programming](dynamic-programming.md)

**Computational geometry · Graph modeling · Greedy heuristics · Interactive visualization**

## Watch the demonstration

> **Demo video — awaiting recording.** This space is reserved for a recording of the mesh viewer and an explanation of the resulting triangle strips.

<!-- DEMO:GREEDY-ALGORITHM:START -->
<!-- Insert the actual video attachment URL here, on its own line. -->
<!-- DEMO:GREEDY-ALGORITHM:END -->

The supplied program computes the strips before its interactive result display. A recording of this version shows the completed solution and viewer interactions; it does not show the algorithm growing the strips frame by frame.

## The problem

A triangle mesh consists of triangular faces connected along shared edges. In this project's coursework formulation, a **triangle strip** is a sequence of distinct, edge-adjacent triangles.

The objective is to partition the mesh into as few strips as possible, subject to two constraints:

- Every triangle belongs to exactly one strip.
- Consecutive triangles in a strip share an edge.

This becomes a graph problem: each triangle is a node, and each shared edge creates an adjacency. A feasible output covers all nodes with disjoint paths. Longer paths generally mean fewer strips, but choosing a locally attractive continuation does not guarantee the minimum possible strip count.

## What the video shows

| Visual element | How to interpret it |
| --- | --- |
| Filled triangular faces | The input mesh |
| A shared color along connected triangles | Membership in the same computed strip; colors may look similar across strips |
| Segments joining triangle centers | Stored links between consecutive triangles in a strip |
| A dot in a triangle | A singleton strip in this implementation |
| Console output | Input vertex and triangle counts, followed by the generated strip count |

The explanation should connect the visible paths to a simple intuition: **prefer triangles with fewer remaining choices**, reducing the chance of leaving difficult-to-connect triangles behind.

## Algorithm and implementation

The core routine is [`buildTristrips()`](../Greedy%20Algorithm/tristrips.py). It uses two related heuristics:

1. **Initial seed ordering.** Sort the triangles once by their initial number of available neighbors. Consider possible strip starts in that fixed order.
2. **Local continuation.** At the current strip endpoint, choose an available adjacent triangle with the fewest available neighbors of its own.

The second rule is evaluated as the strip grows. The initial seed ordering is **not recomputed** before each new strip. Equal scores follow the existing list order.

```text
Order all triangles once by initial available-neighbor count.
For each triangle in that order:
    If it is eligible to start a strip:
        Start a new strip there.
        While the endpoint has an available neighbor:
            Choose the neighbor with the fewest available neighbors.
            Link it to the endpoint and advance to it.
```

The implementation stores strip structure in `nextTri` and `prevTri` pointers. Its eligibility checks use those pointers; although it sets `isOnStrip`, that flag is not used to select candidates. Adjacency is constructed by matching shared edges through sorted vertex-index pairs.

## Results and validation

The following results were obtained by executing the original mesh reader and greedy routine on the repository datasets, with OpenGL rendering excluded. These are structural results, not timing measurements.

| Dataset | Vertices | Triangles | Generated strips | Coverage and link checks |
| --- | ---: | ---: | ---: | --- |
| `data/200` | 200 | 384 | 15 | Passed |
| `data/1000` | 1,000 | 1,978 | 82 | Passed |
| `data/10000` | 10,000 | 19,969 | 872 | Passed |

Dataset filenames identify the vertex count, not the triangle count.

For all three inputs, validation followed every strip from its first triangle and checked that all triangles were visited exactly once, no cycle occurred, each successive pair was adjacent, and each forward link had the matching backward link. The number of reconstructed strips agreed with the routine's printed count.

<details>
<summary>Reference figures and measurement scope</summary>

The existing [`results.txt`](../Greedy%20Algorithm/results.txt) describes approximate expected results: 15, 43, and 466 strips for the three datasets. These reference figures are distinct from the outputs above; the current implementation produces more strips on the two larger inputs.

That file also lists 45 strips without optimization and 15 with both heuristics on `data/200`. The reduction `(45 - 15) / 45` is approximately 67%, but the 45-strip baseline has not been reproduced for this documentation. The percentage is therefore a comparison between reference figures, not a newly measured improvement by this implementation.

Similarly, the listed 1.83 seconds is a reference timing on an Intel i5-6600 at 3.30 GHz. This documentation makes no measured runtime claim.

Validation used the exact `Colour`, `Triangle`, `readTriangles`, and `buildTristrips` definitions from the supplied source, executed without the rendering imports or GUI loop. It establishes properties of these three outputs; it is not a proof for every possible input or a verification of the graphical application.

Source snapshot: repository commit `c76e0428b691596d6d511e439e88c90fab6676d1`. The uploaded `tristrips.py`, `A2.txt`, and `results.txt` are byte-identical to that snapshot. Validation was performed on 2026-09-12 with Python 3.12.14 on Linux x86_64.

</details>

## Complexity and limitations

For a valid manifold triangle mesh, each triangle has at most three edge-neighbors. With `V` vertices and `T` triangles, shared-edge lookup has expected linear construction cost, initial sorting costs `O(T log T)`, and bounded-degree strip extension costs `O(T)`. The resulting bound is **expected `O(V + T log T)` time and `O(V + T)` memory**, excluding rendering.

The heuristic optimizes local choices and does not certify a globally minimum strip count. Its outcome can depend on ordering when scores tie. The mesh reader assumes suitably formed input; malformed and non-manifold meshes need additional validation.

The viewer draws individual triangles and overlays the strip links. This project demonstrates adjacency-based strip construction; it does not benchmark GPU rendering acceleration or emit a validated GPU triangle-strip index buffer.

## Run the demonstration

Requires Python 3, PyOpenGL, GLFW, and a desktop environment capable of opening an OpenGL window. From the repository root:

```bash
python -m pip install PyOpenGL glfw
python "Greedy Algorithm/tristrips.py" "Greedy Algorithm/data/200"
```

To examine the larger inputs, replace the final path with `Greedy Algorithm/data/1000` or `Greedy Algorithm/data/10000`. On systems where the interpreter is named `python3`, use that name for both commands.

If your local folder is named `Greedy-Algorithm`, substitute that spelling in the commands and source links. The command uses `tristrips.py`, the actual supplied filename.

| Input | Function during the interactive display |
| --- | --- |
| `B` | Toggle colored triangle backgrounds |
| `O` | Toggle triangle outlines |
| `F` | Switch which stored pointers are drawn; undirected segments may look identical |
| Mouse click | Print the selected triangle and its neighbors; toggle debug highlight flags |
| `P` | Leave the interactive display stage |
| `Esc` | Exit |

## Project context

This project is based on **CMPE/CISC 365 Assignment 2**. The [assignment specification](../Greedy%20Algorithm/A2.txt) supplies the problem and a visualization framework, and directs students to implement `buildTristrips()` and helper functions. The portfolio focus is the greedy strip-building logic and its evaluation within that framework.

The project connects mathematical modeling to implementation: translating a geometric objective into a graph structure, encoding local decision rules, and checking the feasibility and quality of the resulting solution.

---

[Back to Algorithm Showcase](../README.md)
