# Conventions and diagnostics

## Array layout and coordinates

Two-dimensional fields use the shape `(Ny, Nx)`: the first axis corresponds to
`y`, and the second axis corresponds to `x`. For a periodic domain
`[0, Lx) × [0, Ly)`, a matching grid is commonly created as:

```python
import numpy as np

Nx, Ny = 256, 192
Lx, Ly = 2.0 * np.pi, np.pi
x = np.arange(Nx) * Lx / Nx
y = np.arange(Ny) * Ly / Ny
X, Y = np.meshgrid(x, y, indexing="xy")
```

Dirichlet and Neumann transform solvers instead treat the supplied full array
as a grid including the physical boundary. In homogeneous Dirichlet mode, only
`u[1:-1, 1:-1]` is transformed and the returned boundary is zero.

## Boundary-condition names

Unified solvers accept the following canonical values and aliases:

| Canonical value | Accepted aliases | Transform or enforcement |
|---|---|---|
| `periodic` | `fourier`, `fft` | FFT |
| `dirichlet` | `d`, `dst` | DST on the interior |
| `neumann` | `n`, `dct` | DCT on the full grid |
| `robin` | `r` | Boundary projection / penalty iteration |

`mode=` is retained as a legacy alias for `bc=`. Do not pass conflicting
values to both arguments.

## Equation sign conventions

The elliptic solvers use:

\[
-\Delta u = f,
\qquad
(-\Delta+k_0^2)u=f
\]

for Poisson and Helmholtz problems, respectively. Diffusion uses

\[
\partial_t u = D\Delta u+s,
\]

and the wave solver advances

\[
\partial_{tt}u=c^2\Delta u.
\]

Always verify a manufactured solution against these signs before interpreting
an error norm.

## Backend and device preservation

The periodic FFT paths preserve the selected numerical backend:

```python
u = poisson2d_solve(
    f,
    Lx,
    Ly,
    bc="periodic",
    backend_name="torch",
    device="cuda",
)
```

NumPy is the default. CuPy and PyTorch are optional dependencies and are loaded
only when requested. DST/DCT paths depend on the real-transform implementation
and are primarily intended for NumPy arrays.

## Solver diagnostics

Unified solver and time-step functions expose `return_info=True`. When enabled,
the return value is `(result, info)` rather than only `result`.

```python
u, info = poisson2d_solve(
    f,
    Lx,
    Ly,
    bc="dirichlet",
    return_info=True,
)

print(info["solver"], info["transform"], info["output_shape"])
```

The common diagnostic fields are:

| Key | Meaning |
|---|---|
| `solver` | Public entry point used |
| `equation` | Equation family |
| `bc` | Canonical boundary-condition name |
| `transform` | Computational path, such as `fft`, `dst`, `dct`, or `robin_penalty` |
| `backend` | Array/FFT backend |
| `device` | CPU or accelerator device |
| `capabilities` | Backend capability metadata, when available |
| `input_shape`, `output_shape` | Array shapes |
| `input_dtype`, `output_dtype` | Data types |

Equation-specific values such as diffusivity, timestep, viscosity, iteration
settings, or source activation may also be included.

## Fixed-step trajectories

Time integrators generally distinguish a one-step function from a trajectory
solver:

```python
u_next = allen_cahn2d_step(u, epsilon, Lx, Ly, dt)

times, states = allen_cahn2d_solve(
    u,
    epsilon,
    Lx,
    Ly,
    dt,
    t_end=1.0,
    save_stride=10,
)
```

`save_stride` controls output sampling, not the integration timestep. A final
short step is used when necessary so the requested end time is reached without
overshoot.

## Validation and dtype guidance

- `Lx`, `Ly`, and timestep-related parameters must be finite and physically
  admissible.
- Unified two-dimensional solvers require two-dimensional inputs and validate
  minimum grid sizes.
- Use `float64`/`complex128` for convergence studies and tight conservation
  diagnostics; use single precision only after verifying that it is adequate.
- FFT efficiency is usually best for grid sizes with small prime factors.
- For repeated calls, construct the backend once and retain its caches.
