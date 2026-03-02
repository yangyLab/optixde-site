# Quickstart

This public site can show **how to use OptiXDE** without exposing the private repository.

## Installation

If OptiXDE is private, keep installation instructions generic:

```bash
# Option A: internal/private install (example)
pip install optixde --extra-index-url <YOUR_PRIVATE_INDEX>

# Option B: request access to the private repo
# (See the "Cite & Contact" page)
```
# Geometry Construction Tutorial

OptiXDE runs on an FFT-compatible uniform grid, so geometry is represented **implicitly** as a *mask field* on that grid (not a mesh). The recommended public-facing pipeline is:

**primitives → CSG → binary mask → smoothed mask → embedded/penalized PDE**

This keeps the API simple while allowing complex shapes, and it’s safe to share publicly because it demonstrates **usage** rather than implementation.

---

## 1) Create a grid (physical + spectral)

You start by defining a rectangular embedding domain \(\tilde{\Omega}\) and a resolution \((N_x, N_y)\). Geometry will be rasterized onto this grid.

```python
import optixde as ox

grid = ox.Grid2D(
    box=[(-1.0, 1.0), (-1.0, 1.0)],
    shape=(512, 512),
)
```

---

## 2) Build geometry from primitives (CSG)

Describe geometry using parameterized primitives (box, disk, polygon, …) and combine them via CSG operations: **union** (`|`), **intersection** (`&`), and **difference** (`-`).

Example: **L-shaped domain**

\[
\Omega = (-1,1)^2 \setminus \bigl([0,1]\times[-1,0]\bigr)
\]

```python
# Primitives
outer  = ox.geom.Box2D(xmin=-1, xmax=1, ymin=-1, ymax=1)
cutout = ox.geom.Box2D(xmin=0,  xmax=1, ymin=-1, ymax=0)

# L-shape: Ω = outer \ cutout
omega = outer - cutout
```

---

## 3) Rasterize to a binary mask \(\chi(\mathbf{x})\)

The mask indicates whether a grid point is inside/outside the physical domain \(\Omega\). This mask is the core geometry object for embedded-domain solvers.

```python
chi = ox.mask.from_geometry(grid, omega)
# Convention can be either:
#   chi = 1 in Ω, 0 outside   (common)
# or
#   chi = 0 in Ω, 1 outside   (also common)
# Use the convention adopted by your public API docs.
```

---

## 4) Smooth the mask (recommended)

A sharp binary interface can cause ringing near boundaries on spectral grids. Smoothing the mask over a narrow band (e.g., 1–3 cells) often improves stability and visual quality.

```python
chi_eps = ox.mask.smooth(chi, sigma_cells=2)
```

---

## 5) Attach boundary data (embedded / penalty BC)

For non-periodic or irregular domains on an FFT grid, boundary conditions are typically enforced through an **embedded/penalty** mechanism.

```python
g = ox.bc.dirichlet_value(lambda x, y: 0.0)  # homogeneous Dirichlet
bc = ox.bc.EmbeddedDirichlet(mask=chi_eps, value=g, eta=1e-6)
```

---

## 6) Solve a PDE on the embedded geometry (Poisson example)

Below is a usage-only example: pass the grid, RHS, and embedded boundary handler. The solver remains matrix-free and FFT-based.

```python
f = ox.field.from_function(grid, lambda x, y: 1.0)  # RHS

u = ox.solve.poisson(
    grid=grid,
    rhs=f,
    bc=bc,
    backend="torch",  # or "numpy"
)
```

---

## 7) Visualize / export (public-friendly)

On the public site, prefer showing **plots / GIFs / videos** produced by the private repo, without exposing internals.

```python
ox.viz.imshow(u, title="Poisson on L-shaped domain")
ox.io.save_image("poisson_Lshape.png", u)
```

---

## Notes & Best Practices

- **Padding**: If the physical boundary is close to the embedding box boundary, enlarge \(\tilde{\Omega}\) (add padding) to reduce FFT wrap-around artifacts.
- **Penalty parameter \(\eta\)**: Smaller \(\eta\) enforces boundary conditions more strongly but may increase stiffness. Start around `1e-6` and tune per problem.
- **CSG scales well**: Complex engineering shapes can be built by composing simple primitives using `|`, `&`, and `-` before rasterization.
- **Keep the public API stable**: Share exact function signatures and small usage snippets; avoid implementation details (kernels, internal datasets, or confidential benchmarks).

