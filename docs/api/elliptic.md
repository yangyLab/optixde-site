# Elliptic solvers

The rectangular elliptic solvers diagonalize constant-coefficient operators in
a basis matched to the boundary condition. Periodic problems use the selectable
FFT backend; homogeneous Dirichlet and Neumann problems use DST and DCT,
respectively.

## Poisson equation

OptiXDE solves:

\[
-\Delta u=f.
\]

### Unified interface

```python
poisson2d_solve(
    f,
    Lx,
    Ly,
    bc="periodic",
    mode=None,
    robin=None,
    robin_max_iter=200,
    robin_tol=1e-10,
    robin_relax=0.02,
    robin_penalty=10.0,
    enforce_zero_mean_rhs=True,
    fix_mean=True,
    eps=1e-14,
    backend=None,
    backend_name="numpy",
    workers=None,
    return_info=False,
    **backend_kwargs,
)
```

Parameters:

| Parameter | Description |
|---|---|
| `f` | Right-hand side with shape `(Ny, Nx)` |
| `Lx`, `Ly` | Positive physical domain lengths |
| `bc` | `periodic`, `dirichlet`, `neumann`, or `robin` |
| `robin` | Global or per-side `(alpha, beta, g)` specification |
| `enforce_zero_mean_rhs` | Subtract `mean(f)` in singular periodic/Neumann cases |
| `fix_mean` | Fix the Neumann solution gauge to zero mean |
| `eps` | Non-negative denominator regularization |
| `backend`, `backend_name` | Periodic FFT execution backend |
| `workers` | Thread hint for DST/DCT paths |
| `return_info` | Return `(u, diagnostics)` |

The result has the same shape as `f`. In Dirichlet mode the input must include
boundary nodes and the returned boundary is zero.

```python
u, info = poisson2d_solve(
    f,
    Lx,
    Ly,
    bc="periodic",
    backend_name="numpy",
    return_info=True,
)
```

### Specialized Poisson entry points

```python
poisson2d_periodic(
    f, Lx, Ly,
    enforce_zero_mean_rhs=True,
    eps=1e-14,
    backend=None,
    backend_name="numpy",
    **backend_kwargs,
)
```

For nonzero Fourier modes,

\[
\widehat u(\mathbf{k})
=\frac{\widehat f(\mathbf{k})}{|\mathbf{k}|^2}.
\]

The zero mode of `f` is cleared, and the zero mode of `u` is set to zero.

```python
poisson2d_dirichlet(f, Lx, Ly, eps=1e-14, workers=None)
poisson2d_neumann(
    f, Lx, Ly,
    enforce_zero_mean_rhs=True,
    fix_mean=True,
    eps=1e-14,
    workers=None,
)
```

The Dirichlet solver transforms only the interior array. The Neumann solver
includes the constant cosine mode and therefore exposes compatibility and gauge
controls.

```python
poisson2d_robin_penalty(
    f, Lx, Ly,
    robin=None,
    max_iter=200,
    tol=1e-10,
    relax=0.2,
    penalty=10.0,
    enforce_zero_mean_rhs=True,
    fix_mean=True,
    eps=1e-14,
    backend=None,
    backend_name="numpy",
    **backend_kwargs,
)
```

This is a projected Richardson method with a fast periodic Poisson inverse as
preconditioner. It is practical for prototypes and moderate tolerances, but it
is not an exact Robin spectral diagonalization. Increasing `penalty` strengthens
boundary enforcement and may require decreasing `relax`.

### Inhomogeneous Dirichlet lifting

The lower-level module `optixde.solvers.base.poisson_inhom` provides:

```python
poisson2d_dirichlet_inhom(f, Lx, Ly, bc, workers=None)
build_tb_coons(bc, Lx, Ly)
laplacian_fd(u, Lx, Ly)
```

`poisson2d_dirichlet_inhom` builds a Coons-patch lifting from the four boundary
curves, solves a homogeneous Dirichlet correction, and overwrites the boundary
exactly. `bc` is a full `(Ny, Nx)` array; only its boundary values are used.

## Shifted Helmholtz equation

OptiXDE uses the screened-Poisson convention:

\[
(-\Delta+k_0^2)u=f.
\]

### Unified interface

```python
helmholtz2d_solve(
    f,
    Lx,
    Ly,
    k0,
    bc="fourier",
    mode=None,
    eps=1e-14,
    backend=None,
    backend_name="numpy",
    workers=None,
    return_info=False,
    **backend_kwargs,
)
```

The spectral denominator is

\[
|\mathbf{k}|^2+k_0^2.
\]

For `k0 > 0`, the constant mode is nonsingular. When `k0 = 0`, the periodic and
Neumann cases reduce to a Laplacian problem; prefer `poisson2d_solve` when you
need explicit compatibility and gauge handling.

### Specialized Helmholtz entry points

```python
helmholtz2d_fourier(
    f, Lx, Ly, k0,
    eps=1e-14,
    backend=None,
    backend_name="numpy",
    **backend_kwargs,
)

helmholtz2d_dirichlet(
    f, Lx, Ly, k0,
    eps=1e-14,
    workers=None,
)

helmholtz2d_neumann(
    f, Lx, Ly, k0,
    eps=1e-14,
    workers=None,
)
```

All return a real array with the same shape as `f`. The periodic path supports
NumPy, CuPy, and PyTorch backends. The real-transform paths are intended for
NumPy/CuPy-compatible inputs and return the corresponding array type.

## Manufactured-solution check

```python
import numpy as np
from optixde.solvers import poisson2d_solve

Nx = Ny = 129
Lx = Ly = 1.0
x = np.linspace(0.0, Lx, Nx)
y = np.linspace(0.0, Ly, Ny)
X, Y = np.meshgrid(x, y, indexing="xy")

u_exact = np.sin(np.pi * X) * np.sin(np.pi * Y)
f = 2.0 * np.pi**2 * u_exact
u = poisson2d_solve(f, Lx, Ly, bc="dirichlet")

relative_error = np.linalg.norm(u - u_exact) / np.linalg.norm(u_exact)
```

This example respects the library sign convention and includes boundary nodes,
which are required by the DST path.
