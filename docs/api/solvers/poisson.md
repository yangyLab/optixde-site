# `optixde.solvers.base.poisson`

optixde.solvers.base.poisson

2D Poisson solver on a rectangular domain.

We solve the Poisson equation in the library-wide sign convention:

    -Δu = f

on [0, Lx] × [0, Ly], with boundary condition (bc) modes:
- periodic/fourier/fft: periodic BC, FFT-based diagonal solve in Fourier space
- dirichlet/d/dst: homogeneous Dirichlet BC (u=0 on boundary), DST-based solve
- neumann/n/dct: homogeneous Neumann BC (∂u/∂n=0), DCT-based solve

Solvability / gauge issues:
- Periodic and Neumann Laplacians have a nullspace at the zero-frequency mode.
  Therefore a solution exists only if the RHS has zero mean:
      mean(f) = 0
  This module provides `enforce_zero_mean_rhs=True` to automatically subtract
  the mean of f.
- The solution is not unique up to an additive constant. This module fixes the
  gauge by setting the mean/zero mode of u to 0 (fix_mean / Uhat(0,0)=0).

Spectral diagonalization:
- Periodic:
      Û(k) = - F̂(k) / |k|^2   (with zero mode handled separately)
- Dirichlet (sine basis on interior):
      U_mn = - F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 )
- Neumann (cosine basis):
      U_mn = - F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 )  (with (0,0) handled)

Numerical notes:
- eps is used to avoid division-by-zero and to stabilize extremely small modes.
- Outputs are sanitized by nan_to_num in periodic/neumann modes for robustness.

Backend support:
- Periodic/Fourier path uses optixde.fft_backend (numpy/cupy/torch).
- Dirichlet/Neumann path uses dst2/dct2 from optixde.operators.transforms
  and supports numpy (and cupy if transforms support it).

## `poisson2d_periodic(f, Lx, Ly, enforce_zero_mean_rhs, eps, backend, backend_name, **backend_kwargs)`

*Type*: function

Solve -Δu = f on a periodic rectangle using FFT.

In Fourier space (for nonzero modes):
    Û(k) = - F̂(k) / |k|^2

The (0,0) mode is singular because |k|^2=0. This function:
- optionally enforces mean(f)=0 by subtracting the mean,
- sets F̂(0,0)=0 (clean solvability),
- sets Û(0,0)=0 to fix the additive-constant gauge.

Parameters
----------
f : array-like, shape (Ny, Nx)
    Right-hand-side in physical space.
Lx, Ly : float
    Domain lengths.
enforce_zero_mean_rhs : bool
    If True, replace f by f - mean(f) to satisfy solvability.
eps : float
    Small positive number added to the denominator for robustness.
    (The zero mode is handled separately regardless.)
backend : object or None
    FFT backend instance. If None, constructed via get_backend(backend_name).
backend_name : str
    Backend name for get_backend (e.g., "numpy", "cupy", "torch").
**backend_kwargs :
    Extra kwargs passed to get_backend.

Returns
-------
u : array-like, shape (Ny, Nx)
    Periodic solution with zero-mean gauge (û(0,0)=0).

Notes
-----
- For periodic Poisson, the solution is only defined up to a constant.
  The gauge û(0,0)=0 is a common choice.

## `poisson2d_dirichlet(f, Lx, Ly, eps, workers)`

*Type*: function

Solve -Δu = f with homogeneous Dirichlet BC using DST.

Boundary condition:
    u = 0 on ∂Ω

Implementation details:
- Input f is assumed to be on a full grid including boundary nodes.
- Unknowns live on the interior: u[1:-1,1:-1].
- DST diagonalizes the Dirichlet Laplacian on the interior.

Spectral diagonal:
    U_mn = - F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 )

Parameters
----------
f : array-like, shape (Ny, Nx)
    RHS on full grid (boundary included).
Lx, Ly : float
    Domain lengths.
eps : float
    Minimum eigenvalue clamp (stability safeguard).
workers : int or None
    Optional thread count passed to dst2/idst2 if supported.

Returns
-------
u : array-like, shape (Ny, Nx)
    Solution on full grid, with u=0 on boundary.

Notes
-----
- Dirichlet Poisson has a unique solution; no mean-fixing is required.

## `poisson2d_neumann(f, Lx, Ly, enforce_zero_mean_rhs, fix_mean, eps, workers)`

*Type*: function

Solve -Δu = f with homogeneous Neumann BC using DCT.

Boundary condition:
    ∂u/∂n = 0 on ∂Ω

Solvability / gauge:
- Neumann Laplacian has a nullspace (constant mode). A solution exists only
  if mean(f)=0. This function can enforce it by subtracting mean(f).
- The solution is defined up to an additive constant. If fix_mean=True, this
  function sets the zero-frequency coefficient Udct(0,0)=0 (zero-mean gauge).

Spectral diagonal (nonzero modes):
    U_mn = - F_mn / ( (mπ/Lx)^2 + (nπ/Ly)^2 )

Parameters
----------
f : array-like, shape (Ny, Nx)
    RHS on full grid.
Lx, Ly : float
    Domain lengths.
enforce_zero_mean_rhs : bool
    If True, replace f by f - mean(f) to satisfy solvability.
fix_mean : bool
    If True, set the solution mean to zero (Udct(0,0)=0).
eps : float
    Small positive regularization added to denom after setting (0,0)=1.
workers : int or None
    Optional thread count passed to dct2/idct2 if supported.

Returns
-------
u : array-like, shape (Ny, Nx)
    Neumann solution (optionally zero-mean if fix_mean=True).

Notes
-----
- If you want a different gauge (e.g., specify u at one point), you can
  post-process the returned u by adding a constant.

## `poisson2d_solve(f, Lx, Ly, bc, enforce_zero_mean_rhs, fix_mean, eps, backend, backend_name, workers, **backend_kwargs)`

*Type*: function

Unified 2D Poisson solver interface.

Solves the library-wide sign convention:
    -Δu = f

Parameters
----------
f : array-like, shape (Ny, Nx)
    Right-hand-side field.
Lx, Ly : float
    Domain lengths.
bc : str
    Boundary condition / transform mode:
    - "periodic" / "fourier" / "fft"
    - "dirichlet" / "d" / "dst"
    - "neumann" / "n" / "dct"
enforce_zero_mean_rhs : bool
    For periodic and neumann modes: enforce mean(f)=0 by subtracting mean.
    Ignored by dirichlet mode.
fix_mean : bool
    For neumann mode: fix solution gauge by enforcing zero-mean u.
    For periodic mode, the solver always enforces Uhat(0,0)=0 internally.
eps : float
    Small regularization for denominators.
backend, backend_name, **backend_kwargs :
    Only used by periodic/Fourier mode.
workers : int or None
    Only used by DST/DCT modes.

Returns
-------
u : array-like
    Solution field.

Raises
------
ValueError
    If `bc` is not recognized.

Examples
--------
Periodic Poisson (FFT backend):
    u = poisson2d_solve(f, Lx, Ly, bc="periodic")

Homogeneous Dirichlet Poisson (DST):
    # f includes boundary nodes; solution boundary is zero
    u = poisson2d_solve(f, Lx, Ly, bc="dirichlet")

Homogeneous Neumann Poisson (DCT):
    u = poisson2d_solve(f, Lx, Ly, bc="neumann", fix_mean=True)
