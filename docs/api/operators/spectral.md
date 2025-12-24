# `optixde.operators.spectral`

optixde.solvers.base.propagators (helpers)

High-level helpers that build/apply common spectral propagators.

These helpers sit on top of:
- FFTBackend: fft2 / ifft2 / mul_inplace
- PropagatorCache: cached exp(K2) and inv(K2) style diagonal operators

They provide:
- diffusion_propagator:   G = exp(-D*dt*K2)
- poisson_green_periodic: G = -1/(K2+eps) with zero-mode fixed to 0
- apply_propagator:       u <- F^{-1}( G * F(u) )

Notes for API docs
------------------
- These are small utilities used by diffusion/poisson/helmholtz solvers.
- `K2` is typically obtained from backend.make_freq_grids(...).K2 (periodic grids).
- For Poisson periodic, the zero-frequency mode is singular; we fix it by setting
  G[0,0]=0 and optionally enforcing zero-mean RHS elsewhere.

## `diffusion_propagator(cache, K2, D, dt, out_dtype)`

*Type*: function

Build (cached) diffusion propagator in spectral space.

Diffusion equation (periodic FFT form):
    u_t = D Δu
Over a single timestep dt, the exact linear propagator is:
    Û(t+dt) = exp(-D*dt*K2) * Û(t)

Parameters
----------
cache : PropagatorCache
    Cache object tied to a backend (numpy/cupy/torch). Avoids recomputing exp arrays.
K2 : array-like, shape (Ny, Nx)
    Squared frequency magnitude grid (KX^2 + KY^2).
D : float
    Diffusion coefficient.
dt : float
    Time step.
out_dtype : optional
    If provided, cast the propagator to this dtype (commonly complex dtype).

Returns
-------
G : array-like
    Spectral propagator exp(-D*dt*K2).

## `poisson_green_periodic(cache, K2, eps, out_dtype)`

*Type*: function

Build (cached) periodic Poisson Green operator in spectral space.

For periodic Poisson under the library sign convention:
    -Δu = f
In Fourier space:
    K2 * Û = -F̂   =>   Û = -(1/K2) * F̂

The k=0 mode is singular for periodic Poisson; we fix the gauge by setting:
    (1/K2)[0,0] = 0

Parameters
----------
cache : PropagatorCache
    Cache object tied to a backend.
K2 : array-like, shape (Ny, Nx)
    Squared frequency magnitude grid.
eps : float
    Small number added to K2 to avoid division by zero and stabilize tiny modes.
out_dtype : optional
    If provided, cast the operator to this dtype.

Returns
-------
G : array-like
    Spectral Green operator:  G = -1/(K2+eps), with G[0,0]=0.

Notes
-----
- Many solvers also enforce zero-mean RHS: f <- f - mean(f), which is consistent
  with the existence condition for periodic Poisson.

## `apply_propagator(backend, u, G)`

*Type*: function

Apply a diagonal propagator in frequency space:
    u_out = F^{-1}( G * F(u) )

Parameters
----------
backend : FFTBackend-like
    Backend providing fft2 / ifft2 / mul_inplace.
u : array-like, shape (Ny, Nx)
    Physical-space field.
G : array-like, shape (Ny, Nx)
    Spectral-space diagonal operator (propagator/Green function).

Returns
-------
u_out : array-like, shape (Ny, Nx), complex
    Result in physical space (often caller takes .real).

Notes
-----
- mul_inplace is preferred to reduce memory allocation: Û *= G.
- This helper does not enforce realness; many PDE solvers call `.real` after ifft2.
