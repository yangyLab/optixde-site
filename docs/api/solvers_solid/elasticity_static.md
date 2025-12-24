# `optixde.solvers.solid.elasticity_static`

Periodic 2D/3D linear isotropic elasticity (static).

Navier equation with periodic BCs:
    -mu*Îu - (lam+mu)*â(âÂ·u) = f

2D arrays: (Ny,Nx). 3D arrays: (Nz,Ny,Nx).
Zero-frequency mode is set to 0 (zero-mean displacement).

## `apply_K_periodic_2d(u: 'np.ndarray', Lx: 'float', Ly: 'float', lam: 'float', mu: 'float') -&gt; 'np.ndarray'`

*Type*: function

Apply K(u) = -mu Îu - (lam+mu)â(âÂ·u) on periodic 2D grids.

    u: shape (2, Ny, Nx) -> returns Ku of same shape.

## `apply_K_periodic_3d(u: 'np.ndarray', Lx: 'float', Ly: 'float', Lz: 'float', lam: 'float', mu: 'float') -&gt; 'np.ndarray'`

*Type*: function

Apply K(u) = -mu Îu - (lam+mu)â(âÂ·u) on periodic 3D grids.

    u: shape (3, Nz, Ny, Nx) -> returns Ku of same shape.

## `elastic2d_periodic_solve(fx: 'np.ndarray', fy: 'np.ndarray', Lx: 'float', Ly: 'float', lam: 'float', mu: 'float', eps: 'float' = 1e-30) -&gt; 'Tuple[np.ndarray, np.ndarray]'`

*Type*: function

Solve periodic 2D Navier equation for (ux, uy).

## `elastic3d_periodic_solve(fx: 'np.ndarray', fy: 'np.ndarray', fz: 'np.ndarray', Lx: 'float', Ly: 'float', Lz: 'float', lam: 'float', mu: 'float', eps: 'float' = 1e-30) -&gt; 'Tuple[np.ndarray, np.ndarray, np.ndarray]'`

*Type*: function

Solve periodic 3D Navier equation for (ux, uy, uz).
