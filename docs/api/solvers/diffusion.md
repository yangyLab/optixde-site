# `optixde.solvers.base.diffusion`

optixde.solvers.base.diffusion

2D diffusion (heat) equation time stepping on a rectangular domain.

We solve the diffusion/heat equation in 2D:

    ∂u/∂t = D Δu + s(x,y,t)

on a rectangle [0, Lx] × [0, Ly], with different boundary condition (bc) modes:
- periodic/fourier/fft: periodic boundary, FFT-based spectral update
- dirichlet/d/dst: homogeneous Dirichlet boundary (u=0 on boundary), DST-based update
- neumann/n/dct: homogeneous Neumann boundary (∂u/∂n=0), DCT-based update

Time stepping:
- Periodic (no source): exact linear propagator in Fourier space:
      U^{n+1}(k) = exp(-D dt |k|^2) U^n(k)
- Periodic (with source): ETD1 (exponential time differencing, first order)
  using midpoint source evaluation s(t+dt/2) and ϕ1(z) = (e^z - 1)/z:
      U^{n+1} = E U^n + dt ϕ1(-D dt |k|^2) S_mid
  where E = exp(-D dt |k|^2), S_mid is FFT(source at midpoint).

Notes:
- For Dirichlet/Neumann modes, this module uses real sine/cosine transforms
  implemented in optixde.operators.transforms (dst2/dct2 and inverses).
- For periodic mode, this module delegates FFT and frequency-grid creation to
  optixde.fft_backend, allowing numpy/cupy/torch backends and caching.

API design notes (for documentation):
- All solvers accept array-like u and return a new array (same shape).
- Periodic backend path returns real part (numerical noise may create tiny imag).

## `diffusion2d_periodic(u, D, Lx, Ly, dt, backend, backend_name, cache, **backend_kwargs)`

*Type*: function

One diffusion step with periodic boundary conditions (FFT / Fourier mode).

This performs the exact linear update in Fourier space for the PDE:
    ∂u/∂t = D Δu
on a periodic rectangle [0,Lx]×[0,Ly]:

    U^{n+1}(k) = exp(-D dt |k|^2) U^n(k)

where U = FFT(u), and k-grid is created by the selected backend.

Parameters
----------
u : array-like, shape (Ny, Nx)
    Current solution field (real-valued recommended).
D : float
    Diffusivity coefficient.
Lx, Ly : float
    Domain lengths in x and y directions.
dt : float
    Time step size.
backend : object or None
    Backend instance implementing fft2/ifft2 and grid generation.
    If None, constructed via get_backend(backend_name, **backend_kwargs).
backend_name : str
    Backend name passed to get_backend (e.g., "numpy", "cupy", "torch").
cache : PropagatorCache or None
    Cache object for reusing exp(-D dt K2) factors across calls.
    If None, a new PropagatorCache is created.
**backend_kwargs :
    Additional kwargs forwarded to get_backend.

Returns
-------
out : array-like, shape (Ny, Nx)
    Updated field u^{n+1} in physical space (real part is returned).

Notes
-----
- The returned value is `out.real` to suppress tiny imaginary parts from FFT
  roundoff when backend uses complex transforms.
- For performance, the exponential multiplier is cached by PropagatorCache.

## `diffusion2d_periodic_etd1(u, D, Lx, Ly, dt, t, source, backend, backend_name, cache, eps, **backend_kwargs)`

*Type*: function

One diffusion step with periodic BC using ETD1 for a time-dependent source.

Solves (semi-discrete in time):
    ∂u/∂t = D Δu + s(x,y,t)

using first-order exponential time differencing (ETD1) with midpoint source:
    U^{n+1} = E U^n + dt * ϕ1(-D dt K2) * S_mid
where
    E = exp(-D dt K2)
    S_mid = FFT(source(t + dt/2))
    ϕ1(z) = (exp(z) - 1)/z  (stable near z=0)

Parameters
----------
u : array-like, shape (Ny, Nx)
    Current solution.
D : float
    Diffusivity coefficient.
Lx, Ly : float
    Domain lengths.
dt : float
    Time step.
t : float
    Current time (used for evaluating midpoint source).
source : callable or None
    Source function s(t)->array in physical space. Must return same shape as u.
    If None, falls back to diffusion2d_periodic (no-source exact propagator).
backend, backend_name, cache, **backend_kwargs :
    Same as diffusion2d_periodic.
eps : float
    Small-z threshold used in ϕ1 evaluation.

Returns
-------
out : array-like, shape (Ny, Nx)
    Updated field (real part returned).

Notes
-----
- The source is evaluated at t + 0.5*dt (midpoint) for improved stability.
- For a constant source, this is a standard ETD1 scheme.

## `diffusion2d_dirichlet(u, D, Lx, Ly, dt, workers)`

*Type*: function

One diffusion step with homogeneous Dirichlet BC using DST.

PDE:
    ∂u/∂t = D Δu
with boundary:
    u = 0 on ∂Ω

Implementation details:
- Assumes input u includes boundary nodes (Ny×Nx grid).
- Only the interior block u[1:-1,1:-1] is evolved.
- Uses DST on the interior (sine basis satisfies zero boundary).
- Spectral update:
      Ui_new = exp(-D dt K2) * Ui
  where K2 = (mπ/Lx)^2 + (nπ/Ly)^2 for interior modes.

Parameters
----------
u : array-like, shape (Ny, Nx)
    Field including boundary nodes. Boundary values are expected to be 0.
D : float
    Diffusivity.
Lx, Ly : float
    Domain lengths.
dt : float
    Time step.
workers : int or None
    Optional thread count passed to dst2/idst2 if supported.

Returns
-------
out : array-like, shape (Ny, Nx)
    Updated field with boundary unchanged.

Notes
-----
- Works with numpy arrays; if u is cupy.ndarray, uses cupy for k-grids and exp.
  (Transforms dst2/idst2 must support cupy if you expect full cupy pipeline.)

## `diffusion2d_neumann(u, D, Lx, Ly, dt, workers)`

*Type*: function

One diffusion step with homogeneous Neumann BC using DCT.

PDE:
    ∂u/∂t = D Δu
with boundary:
    ∂u/∂n = 0 on ∂Ω

Implementation:
- Uses DCT basis which naturally corresponds to even extension and zero
  normal derivative at boundaries.
- Spectral update:
      Uc_new = exp(-D dt K2) * Uc
  where kx = mπ/Lx (m=0..Nx-1), ky = nπ/Ly (n=0..Ny-1).

Parameters
----------
u : array-like, shape (Ny, Nx)
    Field including boundary nodes.
D : float
    Diffusivity.
Lx, Ly : float
    Domain lengths.
dt : float
    Time step.
workers : int or None
    Optional thread count passed to dct2/idct2 if supported.

Returns
-------
u_new : array-like, shape (Ny, Nx)
    Updated field.

Notes
-----
- Neumann Laplacian has a zero eigenvalue at (0,0) mode. This is not a
  problem for diffusion, since exp(0)=1 preserves the mean value.

## `diffusion2d_solve(u, D, Lx, Ly, dt, bc, source, t, etd_eps, backend, backend_name, workers, cache, **backend_kwargs)`

*Type*: function

Unified 2D diffusion solver interface: one time step update.

Parameters
----------
u : array-like, shape (Ny, Nx)
    Current field. For Dirichlet mode, u is expected to include boundary nodes.
D : float
    Diffusivity.
Lx, Ly : float
    Domain lengths.
dt : float
    Time step size.
bc : str
    Boundary condition / transform mode:
    - "fourier" / "periodic" / "fft"
    - "dirichlet" / "d" / "dst"
    - "neumann" / "n" / "dct"
source : callable or None
    Source term s(t)->array, only supported for periodic mode in this version.
t : float
    Current time for source evaluation (periodic ETD1 only).
etd_eps : float
    Small-z threshold for ϕ1 in ETD1 (periodic with source).
backend, backend_name, cache, **backend_kwargs :
    Backend controls for periodic mode. Ignored for dirichlet/neumann.
workers : int or None
    Optional thread count for DST/DCT transforms.

Returns
-------
array-like
    Updated field after one step.

Raises
------
NotImplementedError
    If `source` is provided for dirichlet/neumann modes (sealed version).
ValueError
    If `bc` is not recognized.

Examples
--------
Periodic diffusion (FFT backend):
    u = diffusion2d_solve(u, D=1.0, Lx=2*np.pi, Ly=2*np.pi, dt=1e-3, bc="periodic")

Periodic diffusion with source (ETD1):
    def s(t):
        return np.sin(X) * np.cos(Y) * np.exp(-t)
    u = diffusion2d_solve(u, 1.0, Lx, Ly, dt, bc="fft", source=s, t=t)

Homogeneous Dirichlet diffusion (DST on interior):
    u_full = pad_dirichlet(u_interior)   # boundary is zero
    u_full = diffusion2d_solve(u_full, 1.0, Lx, Ly, dt, bc="dirichlet")

Homogeneous Neumann diffusion (DCT on full grid):
    u = diffusion2d_solve(u, 1.0, Lx, Ly, dt, bc="neumann")
