# API reference

This reference documents the current public API of the OptiXDE library.
OptiXDE provides matrix-free spectral solvers, reusable
propagation operators, embedded-domain geometry tools, and NumPy/CuPy/PyTorch
execution backends.

## Recommended import style

Import solver entry points from `optixde.solvers` and supporting components
from their owning subpackages:

```python
from optixde.solvers import poisson2d_solve, diffusion2d_solve
from optixde.fft_backend import get_backend
from optixde.geometry import Rectangle, Disk, Difference
```

The `optixde.solvers.base` namespace remains available for compatibility, but
application code should prefer `optixde.solvers`.

## API map

| Area | Main namespace | Purpose |
|---|---|---|
| Solver conventions | `optixde.solvers` | Boundary aliases, array layout, diagnostics, return contracts |
| FFT execution | `optixde.fft_backend` | NumPy, CuPy, and PyTorch FFT backends; frequency and propagator caches |
| Spectral kernels | `optixde.operators` | FFT/DCT/DST transforms, diffusion multipliers, periodic Green operators |
| Geometry | `optixde.geometry` | Positive-inside signed-distance primitives and Boolean composition |
| Boundary conditions | `optixde.bc` | Robin projection and segmented mixed-boundary specifications |
| Elliptic equations | `optixde.solvers` | Poisson and Helmholtz solvers |
| Evolution equations | `optixde.solvers` | Diffusion, wave, Allen--Cahn, Burgers, and Schrödinger solvers |
| Incompressible flow | `optixde.solvers` | Vorticity--streamfunction Navier--Stokes and cylinder-flow utilities |
| Embedded domains | `optixde.solvers.segmented` | Polygonal Poisson, diffusion, and transient solvers |
| Visualization | `optixde.post` | Publication-oriented field, geometry, time-series, and modal plots |

## Choosing an entry point

- Use a unified `*_solve` function when boundary-condition selection,
  validation, or `return_info=True` diagnostics are useful.
- Use a specialized function such as `poisson2d_periodic` when the transform
  and boundary condition are fixed and the lowest-level public solver is
  preferred.
- Use `*_step` for a single time increment and `*_solve` for a complete
  fixed-step trajectory.
- Reuse a backend and cache in repeated calls to avoid rebuilding frequency
  grids and propagation multipliers.

## Numerical scope

The rectangular spectral solvers use a tensor-product grid with the field
stored as `(Ny, Nx)`. Periodic problems are diagonalized with FFTs, homogeneous
Dirichlet problems with sine transforms, and homogeneous Neumann problems with
cosine transforms. Robin conditions are enforced by an iterative boundary
projection or penalty correction, rather than an exact Robin diagonalization.

For periodic Poisson and Neumann Poisson problems, the Laplacian has a constant
nullspace. OptiXDE can enforce the compatibility condition by subtracting the
right-hand-side mean and can fix the solution gauge by setting its zero mode.

## Pages in this reference

1. [Conventions and diagnostics](conventions.md)
2. [FFT backends](backends.md)
3. [Spectral operators](operators.md)
4. [Geometry and boundary conditions](geometry-boundaries.md)
5. [Elliptic solvers](elliptic.md)
6. [Evolution equations](evolution.md)
7. [Flow solvers](flows.md)
8. [Embedded-domain solvers](embedded.md)
9. [Post-processing](postprocessing.md)
