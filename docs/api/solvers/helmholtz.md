# `optixde.solvers.base.helmholtz`

optixde.solvers.base.helmholtz

2D Helmholtz-type solver on a rectangular domain.

We solve the screened Poisson / shifted Laplacian problem:

    (-Δ + k0^2) u = f

on [0, Lx] × [0, Ly], with boundary condition modes:
- periodic/fourier/fft: periodic BC, FFT-based diagonal solve in Fourier space
- dirichlet/d/dst: homogeneous Dirichlet BC (u=0 on boundary), DST-based solve
- neumann/n/dct: homogeneous Neumann BC (∂u/∂n=0), DCT-based solve

Spectral diagonalization:
- Periodic:
      Û(k) = F̂(k) / (|k|^2 + k0^2)
- Dirichlet (interior sine basis):
      U_mn = F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 + k0^2 )
- Neumann (cosine basis):
      U_mn = F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 + k0^2 )

Numerical notes:
- `eps` is used to clamp denominators away from 0 to avoid division-by-zero
  and to make the solver robust when k0≈0 and low-frequency modes exist.
- Outputs are sanitized by nan_to_num to avoid propagating NaNs/Infs if the
  input contains invalid values.

Backend support:
- Periodic/Fourier path uses optixde.fft_backend (numpy/cupy/torch).
- Dirichlet/Neumann path uses dst2/dct2 transforms from optixde.operators.transforms
  and supports numpy (and cupy if transforms support it).

## `helmholtz2d_fourier(f, Lx, Ly, k0, eps, backend, backend_name, **backend_kwargs)`

*Type*: function

Solve (-Δ + k0^2) u = f on a periodic rectangle using FFT.

In Fourier space:
    Û(k) = F̂(k) / (|k|^2 + k0^2)

Parameters
----------
f : array-like, shape (Ny, Nx)
    Right-hand-side in physical space.
Lx, Ly : float
    Domain lengths.
k0 : float
    Helmholtz shift parameter (screening wavenumber).
eps : float
    Minimum denominator clamp to avoid division by zero.
backend : object or None
    FFT backend instance. If None, constructed by get_backend(backend_name).
backend_name : str
    Backend name passed to get_backend (e.g., "numpy", "cupy", "torch").
**backend_kwargs :
    Extra kwargs passed to get_backend.

Returns
-------
u : array-like, shape (Ny, Nx)
    Solution field in physical space (real-valued output).

Notes
-----
- For torch/cupy backends, k0^2 is cast onto the same device/dtype as K2.
- Output is `ifft2(...).real` and then sanitized by nan_to_num.

## `helmholtz2d_dirichlet(f, Lx, Ly, k0, eps, workers)`

*Type*: function

Solve (-Δ + k0^2) u = f with homogeneous Dirichlet BC using DST.

Boundary condition:
    u = 0 on ∂Ω

Implementation details:
- Input f is assumed to be on a full grid including boundary nodes.
- Only the interior f[1:-1,1:-1] is transformed with DST.
- After solving, the returned u has boundary values equal to 0.

Spectral diagonal:
    U_mn = F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 + k0^2 )

Parameters
----------
f : array-like, shape (Ny, Nx)
    Right-hand-side on the full grid (boundary included).
Lx, Ly : float
    Domain lengths.
k0 : float
    Helmholtz shift parameter.
eps : float
    Denominator clamp.
workers : int or None
    Optional thread count passed to dst2/idst2 if supported.

Returns
-------
out : array-like, shape (Ny, Nx)
    Solution on full grid with zero boundary.

Notes
-----
- This routine uses numpy/cupy grids (via _xp). The DST implementation
  must support the corresponding backend if you want GPU end-to-end.

## `helmholtz2d_neumann(f, Lx, Ly, k0, eps, workers)`

*Type*: function

Solve (-Δ + k0^2) u = f with homogeneous Neumann BC using DCT.

Boundary condition:
    ∂u/∂n = 0 on ∂Ω

Spectral diagonal:
    U_mn = F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 + k0^2 )

Parameters
----------
f : array-like, shape (Ny, Nx)
    Right-hand-side on the full grid.
Lx, Ly : float
    Domain lengths.
k0 : float
    Helmholtz shift parameter.
eps : float
    Denominator clamp.
workers : int or None
    Optional thread count passed to dct2/idct2 if supported.

Returns
-------
u : array-like, shape (Ny, Nx)
    Solution field.

Notes
-----
- For k0=0, the Neumann Laplacian has a zero mode; eps-clamping avoids
  division by zero and effectively regularizes the mean mode.

## `helmholtz2d_solve(f, Lx, Ly, k0, bc, eps, backend, backend_name, workers, **backend_kwargs)`

*Type*: function

Unified 2D Helmholtz solver interface.

Solves:
    (-Δ + k0^2) u = f

Parameters
----------
f : array-like, shape (Ny, Nx)
    Right-hand-side field.
Lx, Ly : float
    Domain lengths.
k0 : float
    Helmholtz shift parameter.
bc : str
    Boundary condition / transform mode:
    - "fourier" / "periodic" / "fft"
    - "dirichlet" / "d" / "dst"
    - "neumann" / "n" / "dct"
eps : float
    Denominator clamp.
backend, backend_name, **backend_kwargs :
    Only used in periodic/Fourier mode.
workers : int or None
    Only used in DST/DCT modes.

Returns
-------
u : array-like
    Solution field.

Raises
------
ValueError
    If `bc` is not recognized.
