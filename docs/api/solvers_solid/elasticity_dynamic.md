# `optixde.solvers.solid.elasticity_dynamic`

Periodic linear elasticity dynamics (elastic waves).

Second-order system:
    rho * u_tt = div(sigma(u)) + f = -K(u) + f

Provides a simple explicit integrator (velocity Verlet) for periodic grids.

## `apply_K_periodic(u: 'np.ndarray', lengths: 'Sequence[float]', lam: 'float', mu: 'float') -&gt; 'np.ndarray'`

*Type*: function

Dispatch periodic stiffness operator for 2D/3D.

## `elastic_wave_periodic_step(u: 'np.ndarray', v: 'np.ndarray', f: 'np.ndarray', dt: 'float', lengths: 'Sequence[float]', lam: 'float', mu: 'float', rho: 'float' = 1.0, damping: 'float' = 0.0, f_next: 'np.ndarray | None' = None) -&gt; 'Tuple[np.ndarray, np.ndarray]'`

*Type*: function

One step using velocity Verlet.

    rho*u_tt = -K(u) + f - damping*rho*v

    u, v, f: (dim, *grid)
