# `optixde.solvers.solid.postprocess`

Post-processing for periodic linear elasticity.

- displacement_gradient_periodic: âu_i/âx_j via FFT
- strain_small: eps_ij = 0.5(du_i/dx_j + du_j/dx_i)
- stress_isotropic: sigma = lam tr(eps) I + 2 mu eps (with 2D plane options)
- von_mises: scalar von Mises stress

## `displacement_gradient_periodic(u: 'np.ndarray', lengths: 'Sequence[float]') -&gt; 'np.ndarray'`

*Type*: function

Compute grad(u) on periodic grids using FFT.

    Parameters
    ----------
    u : array, shape (dim, *grid)
    lengths : (L1, L2) or (L1, L2, L3) corresponding to grid axes.

    Returns
    -------
    grad : array, shape (dim, dim, *grid)
        grad[i,j] = âu_i/âx_j, where x_j corresponds to grid axis j.

## `strain_small(u: 'np.ndarray', lengths: 'Sequence[float]') -&gt; 'np.ndarray'`

*Type*: function

Small-strain tensor eps_ij on periodic grids.

## `stress_isotropic(eps: 'np.ndarray', lam: 'float', mu: 'float', plane: 'str' = 'strain') -&gt; 'np.ndarray'`

*Type*: function

Isotropic linear elastic stress.

    plane:
      - "strain": 2D treated as plane strain (default)
      - "stress": 2D treated as plane stress (adjusts lambda)
      - "3d": force 3D formula

## `von_mises(sigma: 'np.ndarray', plane: 'str' = 'strain') -&gt; 'np.ndarray'`

*Type*: function

Von Mises stress.

    For dim==2:
      - plane="stress": uses sigma_zz=0 formula
      - plane="strain": uses plane-strain 2D formula with sigma_zz included if provided.
    For dim==3: uses 3D deviatoric stress.
