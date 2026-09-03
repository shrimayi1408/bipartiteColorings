# Graph Coloring Structure Visualizer

An interactive browser-based tool for exploring how the structure of a **coloring graph C_k(G)** changes when edges are removed from a base graph G. Built as a computational companion to research on graph coloring reconfiguration, specifically Sara Krehbiel et al.'s work on connectivity and cut-colorings in coloring graphs.

---

## What This Project Studies

Given a graph G and a number of colors k, the **coloring graph C_k(G)** is a graph where:
- Every **node** is a proper k-coloring of G
- Every **edge** connects two colorings that differ by exactly one vertex recolor

This tool lets you modify G by removing edges and immediately see how C_k(G) responds — specifically, where **bottlenecks** (high-degree nodes) appear or shift, and how the overall connectivity structure changes depending on *which* edges you remove.

The core research question: **does it matter whether you remove edges from the busiest node or a less-connected node?** Removing from a high-degree vertex tends to create vertically stacked, hub-dominated coloring graphs; removing from lower-degree vertices spreads connectivity more evenly.

---

## Background and Research Connection

This project builds on two papers:

**Ardila, Owen, Sullivant — "Geodesics in CAT(0) Cubical Complexes" (2011)**
The coloring graph C_k(G) can be given the structure of a CAT(0) cube complex, where cubes correspond to sets of vertices that can be recolored simultaneously and independently (independent sets in G with free alternate colors). In a CAT(0) space there is always a unique shortest path (geodesic) between any two points — meaning there is always a unique "smoothest" recoloring path between any two colorings.

**Krehbiel et al. — "Cut-Colorings in Coloring Graphs" (2019)**
A **cut coloring** is a proper k-coloring whose removal disconnects C_k(G). Krehbiel et al. proved that no 3-mixing graph (one where any coloring is reachable from any other via single-vertex recolors) has a cut coloring with k=3 colors. This result shapes the project's direction: since the space stays connected under modifications with k=3, any structural changes we observe are about *shape* — bottleneck formation, degree distribution, cube dimension — not fragmentation.

---

## Features

- **Multi-panel comparison** — press `+` to add panels, each starting from the original graph G with edges you can click to remove
- **Live coloring graph** — C_k(G) recomputes and re-renders in 3D after every edge removal
- **Degree-based color coding** — nodes in C_k(G) are colored green (high degree / bottleneck), yellow (below average degree), or red (isolated)
- **Analysis panel** — glass overlay showing colorings gained/lost and edges removed per panel, for direct comparison across modifications
- **3D force-directed layout** — coloring graphs render in interactive 3D (drag to rotate) so cube structure is visible rather than a tangle of intersecting lines

---

## File Structure

```
project/
├── myProj.html          # Main application — open this in Chrome
├── bipartite_study.py   # Python: systematic study of K_{m,n} graphs
│                          Runs CAT(0) flag condition check + cut-coloring
│                          detection across K_{2,3}, K_{2,4}, K_{3,3},
│                          K_{3,4}, K_{4,4} with k=3
├── cat0_coloring.py     # Python: CAT(0) flag condition checker
│                          Enumerates colorings, finds cubes (by dimension),
│                          deduplicates by corner-set, checks Gromov's flag
│                          condition via Bron-Kerbosch clique enumeration
└── README.md
```

---

## How to Run

### Browser tool (`myProj.html`)
No installation needed. Open `myProj.html` directly in Chrome.

- The first panel loads automatically with the default graph
- Click any edge in the left (base graph) panel to remove it — the coloring graph on the right updates immediately
- Press `Add Panel` to add a new modification starting fresh from the original graph
- Press `Show Analysis` to open the comparison overlay

To change the graph, edit these lines near the top of the `<script>` block in `myProj.html`:
```javascript
const vertices = ['A', 'B', 'C'];
const edges = [['A', 'B'], ['B', 'C']];
const adjacencyList = { A: ["B"], B: ["A", "C"], C: ["B"] };
const k = 3;
```

### Python scripts (`bipartite_study.py`, `cat0_coloring.py`)
Requires Python 3. No external libraries needed.

```bash
python3 bipartite_study.py    # runs full K_{m,n} family study
python3 cat0_coloring.py      # runs CAT(0) check on the graph defined in EDGES
```

To change the graph in `cat0_coloring.py`, edit the `EDGES` list near the top of the file.

For interactive exploration:
```bash
python3 -i cat0_coloring.py
>>> flippable_vertices(colorings[0])
>>> find_cubes_at(colorings[0])
```

---

## Key Functions

### JavaScript (`myProj.html`)

| Function | What it does |
|---|---|
| `enumerateColorings(vertices, edges, k)` | Generates all proper k-colorings by recursive backtracking |
| `flippableVertices(coloring, vertices, adj, k)` | Finds which vertices can be recolored and to what colors |
| `buildColoringGraphAdj(colorings, vertices, adj, k, coloringSet)` | Builds adjacency list of C_k(G) |
| `outDegree(ckAdj)` | Returns degree of each coloring node (for bottleneck detection) |
| `diffColorings(ckOld, ckNew)` | Set difference: lost, gained, preserved colorings |
| `recomputePanel(panelIndex)` | Recomputes C_k(G) after edge removal and updates 3D graph |
| `buildAnalysisHTML()` | Generates comparison summary for the glass overlay |

### Python (`cat0_coloring.py`)

| Function | What it does |
|---|---|
| `flippable_vertices(c)` | Vertices with at least one legal alternate color |
| `find_cubes_at(c)` | All valid cubes anchored at coloring c, with corner-sets for deduplication |
| `get_corners(c, S, alt_choice)` | Full set of 2^d corner colorings for one cube |
| `check_flag_condition(c, cubes)` | Gromov's flag condition via Bron-Kerbosch maximal clique enumeration |

### Python (`bipartite_study.py`)

| Function | What it does |
|---|---|
| `make_complete_bipartite(m, n)` | Builds K_{m,n} edge list |
| `tarjan_articulation_points(ck_adj)` | Detects cut colorings via Tarjan's DFS algorithm |
| `analyze(name, edges, k)` | Full pipeline: colorings → connectivity → cut-colorings → cube counts → degree distribution |

---

## Python Results (k=3, complete bipartite graphs)

| Graph | Colorings | Connected | Cut colorings | Max cube dim | Avg degree |
|---|---|---|---|---|---|
| K_{2,3} | 30 | Yes | 0 | 3 | 3.20 |
| K_{2,4} | 54 | Yes | 0 | 4 | 4.00 |
| K_{3,3} | 42 | Yes | 0 | 3 | 3.43 |
| K_{3,4} | 66 | Yes | 0 | 4 | 4.00 |
| K_{4,4} | 90 | Yes | 0 | 4 | 4.27 |

Zero cut colorings across all five graphs, confirming Krehbiel et al.'s theorem computationally. Every graph in this family with k=3 has surplus colors (k > χ(G) = 2), is 3-mixing, and has no cut colorings.

---

## Degree Color Coding

| Color | Meaning in C_k(G) |
|---|---|
| 🟢 Green | Degree ≥ average — bottleneck coloring, many recolor options |
| 🟡 Yellow | Degree < average — less connected coloring |
| 🔴 Red | Degree = 0 — isolated coloring, no legal single-vertex recolor available |

---

## Dependencies

**Browser tool**: no install. CDN-loaded via script tags in the HTML:
- [Cytoscape.js 3.28.1](https://js.cytoscape.org/) — base graph visualization
- [3d-force-graph](https://github.com/vasturiano/3d-force-graph) — 3D coloring graph visualization

**Python scripts**: standard library only (`itertools`, `collections`). No pip installs required.
