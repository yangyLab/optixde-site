# `optixde.solvers.base.poisson_inhom`

optixde.solvers.base.poisson_inhom (Dirichlet inhomogeneous)

Inhomogeneous Dirichlet lifting for the 2D Poisson equation on a rectangle.

Goal:
Given a RHS f(x,y) and boundary values bc on ∂Ω (four edges), solve Poisson with
non-homogeneous Dirichlet boundary:

    u|∂Ω = bc|∂Ω

This file implements a robust "lifting + homogeneous solver" approach:

1) Build a lifting Tb (Coons patch) such that Tb|∂Ω = bc|∂Ω.
2) Write u = v + Tb, where v has homogeneous boundary v|∂Ω = 0.
3) Substitute into Poisson to obtain a homogeneous-BC Poisson solve for v.

Sign convention adaptation:
Different internal solvers may implement either:
    (A)  Δu = f
or
    (B) -Δu = f
To avoid hard-coding the sign, we detect the sign once per grid size
(via a known sine eigenfunction) and cache it.

Implementation choices:
- Tb is built by Coons patch blending from the four boundary curves.
- ΔTb is computed by a simple 2nd-order finite-difference Laplacian on the
  uniform grid (interior only). This is robust and independent of transforms.
- The final boundary is enforced exactly by overwriting u on the boundary nodes.

Notes for API docs:
- Input arrays f and bc are full grids (Ny,Nx) including boundary nodes.
- bc is only used on the boundary; interior values of bc are ignored.
- The lifting uses a normalized coordinate (x/Lx, y/Ly) for Coons blending.

## `detect_dirichlet_sign(Lx, Ly, Nx, Ny, workers)`

*Type*: function

Detect the sign convention used by poisson2d_dirichlet on this grid.

    We test the sine eigenfunction:
        u(x,y) = sin(pi x/Lx) sin(pi y/Ly)
which satisfies homogeneous Dirichlet BC and:
        Δu = -λ u,
where λ = (pi/Lx)^2 + (pi/Ly)^2.

We call the homogeneous Dirichlet solver twice:
- uA = solver(-λ u)  : this matches u if the solver solves Δu = f
- uB = solver(+λ u)  : this matches u if the solver solves -Δu = f

We choose whichever result best matches u (interior L2 relative error),
and return:
    sgn = +1  if solver behaves like  Δu = f
    sgn = -1  if solver behaves like -Δu = f

The result is cached per (Lx,Ly,Nx,Ny) for efficiency.

Parameters
----------
Lx, Ly : float
    Domain lengths.
Nx, Ny : int
    Grid sizes including boundary nodes.
workers : int or None
    Thread hint passed to poisson2d_dirichlet (DST path).

Returns
-------
sgn : int
    +1 means solver uses Δu=f
    -1 means solver uses -Δu=f

Notes
-----
- This detection is meant to be called rarely (once per grid), hence caching.
- Uses only numpy arrays; assumes poisson2d_dirichlet supports the same.

## `build_tb_coons(bc, Lx, Ly)`

*Type*: function

Build a Coons patch lifting Tb from boundary values.

    Tb is constructed so that:
        Tb|∂Ω = bc|∂Ω
using bilinear blending of the four boundary curves plus a corner correction.

Parameters
----------
bc : array-like, shape (Ny, Nx)
    Full grid with boundary values filled. Interior values are ignored.
Lx, Ly : float
    Domain lengths (not used directly; kept for API symmetry and future extension).

Returns
-------
Tb : numpy.ndarray, shape (Ny, Nx)
    Lifting field that matches boundary values exactly.

Notes
-----
- Uses normalized coordinates X=x/Lx, Y=y/Ly in [0,1] to keep blending generic.
- Tb is smooth enough for lifting purposes; ΔTb is computed separately by FD.

## `laplacian_fd(u, Lx, Ly)`

*Type*: function

2nd-order finite-difference Laplacian on a uniform grid.

    Computes Δu on the interior nodes only; boundary values of the returned array
    are set to 0.

Parameters
----------
u : array-like, shape (Ny, Nx)
    Field on a full grid including boundary nodes.
Lx, Ly : float
    Domain lengths.

Returns
-------
out : numpy.ndarray, shape (Ny, Nx)
    Finite-difference Laplacian, with out[1:-1,1:-1] filled and boundary = 0.

Notes
-----
- This is intentionally simple and robust; it avoids relying on spectral
  differentiation or transform-specific Laplacians for the lifting term.
- Grid spacing assumes endpoint=True coordinates: dx=Lx/(Nx-1), dy=Ly/(Ny-1).

## `poisson2d_dirichlet_inhom(f, Lx, Ly, bc, workers)`

*Type*: function

Solve Poisson with inhomogeneous Dirichlet boundary values.

    This is a "lifting" wrapper around the homogeneous Dirichlet solver
    poisson2d_dirichlet, which is assumed to solve either:
        Δu = f     (sgn=+1)
    or
       -Δu = f     (sgn=-1)

    The sign convention is detected automatically on the current grid and
    cached (see detect_dirichlet_sign).

Algorithm
---------
Let Tb be a lifting field such that Tb|∂Ω = bc|∂Ω (Coons patch).
Write:
    u = v + Tb,
where v satisfies homogeneous Dirichlet BC: v|∂Ω = 0.

If the internal solver convention is:
    sgn * Δu = f,
then
    sgn * Δ(v + Tb) = f
=>  sgn * Δv = f - sgn * ΔTb

We compute ΔTb by a 2nd-order FD Laplacian, solve for v using poisson2d_dirichlet,
then assemble u = v + Tb and enforce boundary values exactly.

Parameters
----------
f : array-like, shape (Ny, Nx)
    RHS on the full grid (boundary included).
Lx, Ly : float
    Domain lengths.
bc : array-like, shape (Ny, Nx)
    Boundary values on the full grid. Only boundary nodes are used.
workers : int or None
    Thread hint passed to poisson2d_dirichlet (DST path).

Returns
-------
u : numpy.ndarray, shape (Ny, Nx)
    Solution satisfying u|∂Ω = bc|∂Ω.

Notes
-----
- bc interior values are ignored by the lifting (only boundaries are read).
- Final boundary is hard-enforced by overwriting u on ∂Ω, which removes tiny
  numerical mismatch from the lifting/solver composition.
- This wrapper assumes uniform grid with endpoints included.
