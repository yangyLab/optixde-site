# Flow solvers

OptiXDE exposes two distinct incompressible-flow formulations:

- a fully periodic pseudo-spectral vorticity--streamfunction solver;
- an inlet/outlet finite-difference projection solver with Brinkman
  penalization for flow past a circular cylinder.

They target different boundary conditions and should not be interchanged
without reconsidering the physical model.

## Periodic vorticity--streamfunction solver

The solver advances:

\[
\partial_t\omega+\mathbf{u}\cdot\nabla\omega
=\nu\Delta\omega,\qquad
\nabla\cdot\mathbf{u}=0.
\]

Velocity is reconstructed through a zero-mean streamfunction in Fourier space.

### Velocity reconstruction

```python
vorticity_to_velocity(
    omega,
    Lx,
    Ly,
    *,
    backend_name="numpy",
    device=None,
    return_streamfunction=False,
)
```

Returns `(u, v)` or `(u, v, psi)`. The mean streamfunction mode is set to zero,
which fixes its additive gauge.

```python
velocity_divergence(
    u, v, Lx, Ly,
    *,
    backend_name="numpy",
    device=None,
)
```

Computes the divergence with Fourier differentiation. It is useful for
verifying that velocity reconstruction remains divergence free to numerical
precision.

### One-step solver

```python
navier_stokes2d_vorticity_step(
    omega,
    viscosity,
    Lx,
    Ly,
    dt,
    *,
    nonlinear_order=2,
    dealias=True,
    backend_name="numpy",
    device=None,
    return_velocity=False,
    return_info=False,
)
```

The method applies an exact half diffusion step, explicit nonlinear advection,
and a second half diffusion step. Nonlinear advection is evaluated in
conservative flux form. `nonlinear_order=1` uses Euler and `2` uses midpoint
RK2. The two-thirds filter is enabled by default.

Return forms:

| Flags | Return value |
|---|---|
| default | `omega_next` |
| `return_info=True` | `(omega_next, info)` |
| `return_velocity=True` | `(omega_next, u, v)` |
| both | `(omega_next, u, v, info)` |

### Trajectory solver

```python
navier_stokes2d_vorticity_solve(
    omega0,
    viscosity,
    Lx,
    Ly,
    dt,
    t_end,
    *,
    t0=0.0,
    save_stride=1,
    nonlinear_order=2,
    dealias=True,
    backend_name="numpy",
    device=None,
    return_info=False,
)
```

Returns `(times, snapshots)` or `(times, snapshots, info)`. The implementation
supports NumPy and PyTorch arrays. The nominal `dt` is clipped on the final
step to reach `t_end` exactly.

```python
enstrophy(omega, Lx, Ly)
```

Computes

\[
\mathcal{Z}=\frac12\int_\Omega\omega^2\,d\Omega.
\]

Monitor enstrophy, resolved spectra, and timestep sensitivity when studying
high-Reynolds-number flows.

## Inlet/outlet cylinder flow

This solver uses a collocated finite-difference channel grid, periodicity in
`y`, prescribed inlet velocity, a zero-gradient outlet velocity, an incremental
pressure projection, and Brinkman penalization inside a stationary circular
cylinder. SciPy sparse linear algebra is required.

### Grid and pressure factorization

```python
make_cylinder_flow_grid(
    nx,
    ny,
    *,
    xlim=(-15.0, 25.0),
    ylim=(-8.0, 8.0),
    cylinder_center=(0.0, 0.0),
    cylinder_diameter=1.0,
) -> CylinderFlowGrid
```

`CylinderFlowGrid` stores:

| Attribute | Meaning |
|---|---|
| `x`, `y` | One-dimensional coordinates |
| `X`, `Y` | Grid-coordinate arrays |
| `dx`, `dy` | Grid spacings |
| `cylinder_mask` | Boolean solid mask |
| `pressure_solve` | Cached sparse pressure factorization |

The x grid includes both endpoints; the y grid is periodic and excludes its
upper endpoint.

### One projection step

```python
cylinder_flow2d_step(
    u,
    v,
    grid,
    dt,
    *,
    viscosity=0.01,
    inflow_speed=1.0,
    penalty_eta=0.01,
    cylinder_diameter=1.0,
    return_info=False,
)
```

Returns `(u_next, v_next, pressure)`. With diagnostics enabled it returns
`(u_next, v_next, pressure, info)`.

The diagnostic dictionary includes:

| Key | Meaning |
|---|---|
| `max_divergence`, `rms_divergence` | Projection quality in the fluid |
| `solid_speed` | Maximum residual speed in the cylinder |
| `drag`, `lift` | Momentum-exchange force estimates |
| `cd`, `cl` | Dimensionless force coefficients |
| `cfl` | Grid-based advective CFL estimate |

```python
velocity_divergence_channel(u, v, grid)
```

Computes centered divergence on the channel grid.

```python
cylinder_force_coefficients(
    u_before_penalty,
    v_before_penalty,
    u_after_penalty,
    v_after_penalty,
    grid,
    dt,
    inflow_speed,
    diameter,
)
```

Returns `(drag, lift, cd, cl)` from penalization momentum exchange.

### Complete simulation

```python
cylinder_flow2d_solve(
    grid,
    dt,
    t_end,
    *,
    viscosity=0.01,
    inflow_speed=1.0,
    penalty_eta=0.01,
    cylinder_diameter=1.0,
    t0=0.0,
    initial_u=None,
    initial_v=None,
    save_stride=100,
    callback=None,
    return_info=False,
)
```

Returns:

```text
times, u_states, v_states, pressure_states, history
```

and appends `info` when `return_info=True`. `history` contains one diagnostic
record per integration step, while field arrays are stored only according to
`save_stride`.

`callback(step_count, time, info)` is invoked after every completed step and is
appropriate for progress reporting or custom monitoring. Avoid expensive
visualization inside the callback unless the overhead is acceptable.

### Parameter guidance

- The Reynolds number reported in the final diagnostics is
  `inflow_speed * cylinder_diameter / viscosity`.
- `penalty_eta` controls no-slip enforcement; a smaller value strengthens the
  solid constraint but increases stiffness.
- Choose `dt` using the reported CFL value and verify temporal convergence.
- Resolve both the cylinder diameter and near-wake shear layers. Force
  coefficients are especially sensitive to grid and penalization refinement.
