# Embedded-domain solvers

`optixde.solvers.segmented` provides CPU solvers for polygonal domains embedded
in Cartesian grids. The boundary can be divided into segments carrying
Dirichlet, Neumann, or Robin data.

These routines are distinct from the FFT-based rectangular solvers: the steady
Poisson path assembles a sparse masked-grid system, while the transient paths
use explicit finite differences.

## Grid construction

```python
build_polygon_grid(
    geometry,
    Nx,
    Ny,
    *,
    pad_ratio=0.0,
)
```

Returns a dictionary containing:

| Key | Meaning |
|---|---|
| `x`, `y` | Endpoint-inclusive coordinate vectors |
| `X`, `Y` | Cartesian coordinate arrays |
| `points` | Flattened `(Nx*Ny, 2)` coordinates |
| `hx`, `hy` | Grid spacings |
| `box_lo`, `box_hi` | Padded geometry bounds |

The default `pad_ratio=0` aligns the grid with the polygon bounding box, which
is generally preferable for masked stencils.

## Segmented Poisson problem

```python
poisson2d_polygonal_embedded(
    geometry,
    bcs=None,
    source=0.0,
    Nx=101,
    Ny=51,
    omega=0.9,
    max_iter=30000,
    tol=1e-8,
    band_width=None,
    return_metadata=False,
    **kwargs,
)
```

Solves `-Δu=f` on the polygon. `source` may be a scalar or a callable
`source(x, y)`. The current CPU implementation assembles the masked finite-
difference equations and calls `scipy.sparse.linalg.spsolve`.

`omega`, `max_iter`, and `tol` are retained for compatibility with the earlier
iterative interface. The sparse direct implementation reports one solve rather
than an iteration history.

Outside-domain values are `NaN`, which makes the result directly compatible
with the masking behavior of `optixde.post.plot_cloud`.

With `return_metadata=True`, the result is `(u, metadata)`. Important metadata
fields include:

```text
grid, inside, boundary, strict_interior, bcmap,
band_width, linear_system_size, solver
```

## Masked diffusion stepper

```python
diffusion2d_polygonal_embedded(
    u0,
    geometry,
    bcs,
    D,
    dt,
    *,
    source=0.0,
    nsteps=1,
    Nx=None,
    Ny=None,
    grid=None,
    pad_ratio=0.0,
    band_width=None,
    return_metadata=False,
)
```

Uses explicit centered finite differences at strict interior nodes and applies
the segmented boundary conditions after each step. Supply either an existing
`grid` or both `Nx` and `Ny`.

The function enforces the conservative stability estimate:

\[
\Delta t\le
0.24\,\frac{\min(h_x^2,h_y^2)}{D}.
\]

If the estimate is violated, it raises `ValueError` rather than silently
advancing an unstable scheme.

## Transient diffusion

```python
transient_diffusion_polygonal_embedded(
    geometry,
    bcs,
    u0=0.0,
    source=0.0,
    Nx=101,
    Ny=51,
    dt=0.01,
    t_final=1.0,
    D=1.0,
    pad_ratio=0.0,
    band_width=None,
    return_metadata=False,
    solver_kwargs=None,
)
```

Returns an array with shape `(Ny, Nx, n_steps)`. Scalar `u0` values are expanded
over the full grid. The metadata form returns `(U, metadata)` with the grid,
step count, timestep, final time, diffusivity, and final step metadata.

!!! warning "Time indexing"
    The current implementation uses
    `n_steps = ceil(t_final / dt) + 1` and a fixed `dt` for every update. If
    `t_final` is not an integer multiple of `dt`, the last stored state lies
    beyond the nominal final time. Choose aligned values when exact output
    timing matters.

## Transient wave equation

```python
transient_wave_polygonal_embedded(
    geometry,
    bcs,
    u0=None,
    v0=None,
    Nx=101,
    Ny=51,
    dt=0.01,
    t_final=1.0,
    c=1.0,
    return_metadata=False,
    solver_kwargs=None,
)
```

Uses an explicit leapfrog-style scheme for
`u_tt = c² Δu` and returns `(Ny, Nx, n_steps)`.

The current routine masks the exterior to zero and has limited segmented
boundary enforcement compared with the Poisson and diffusion implementations.
Treat it as an experimental CPU utility, verify the CFL condition, and validate
the intended boundary behavior for each application.

## Example with mixed conditions

```python
import numpy as np
from optixde.bc import DirichletBC, EdgeSlice, NeumannBC
from optixde.geometry import Polygon
from optixde.solvers import poisson2d_polygonal_embedded

geometry = Polygon(np.array([
    [0.0, 0.0],
    [2.0, 0.0],
    [2.0, 1.0],
    [0.0, 1.0],
]))

left = EdgeSlice("left", (0.0, 0.0), (0.0, 1.0), "dirichlet")
right = EdgeSlice("right", (2.0, 0.0), (2.0, 1.0), "dirichlet")
bottom = EdgeSlice("bottom", (0.0, 0.0), (2.0, 0.0), "neumann")
top = EdgeSlice("top", (0.0, 1.0), (2.0, 1.0), "neumann")

bcs = [
    DirichletBC(left, value=1.0),
    DirichletBC(right, value=0.0),
    NeumannBC(bottom, flux=0.0),
    NeumannBC(top, flux=0.0),
]

u, metadata = poisson2d_polygonal_embedded(
    geometry,
    bcs,
    source=0.0,
    Nx=201,
    Ny=101,
    return_metadata=True,
)
```

For convergence studies, refine both `Nx` and `Ny`, scale `band_width` with the
grid spacing, and report errors separately near corners or condition
transitions when singular behavior is expected.
