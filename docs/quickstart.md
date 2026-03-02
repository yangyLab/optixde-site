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
## Geometry Construction Tutorial

OptiXDE runs on an FFT-compatible uniform grid, so geometry is represented **implicitly** as a *mask field* on that grid (not a mesh). The recommended public-facing pipeline is:

**primitives → CSG → binary mask → smoothed mask → embedded/penalized PDE**

This keeps the API simple while allowing complex shapes, and it’s safe to share publicly because it demonstrates **usage** rather than implementation.

---

### 1) Create a grid (physical + spectral)

You start by defining a rectangular embedding domain \(\tilde{\Omega}\) and a resolution \((N_x, N_y)\). Geometry will be rasterized onto this grid.

```python
import optixde as ox

grid = ox.Grid2D(
    box=[(-1.0, 1.0), (-1.0, 1.0)],
    shape=(512, 512),
)


## What to show publicly (recommended)

- Exact **function signatures** (stable API)
- Small snippets that demonstrate **usage**, not implementation
- Plots / GIFs / videos produced by the private repo

## What NOT to show

- Private kernels / implementation details
- Internal datasets
- Confidential benchmarks from third-party projects
