# Evolution equations

OptiXDE advances constant-coefficient linear terms in transform space and
evaluates nonlinear or local terms in physical space. This section covers the
diffusion, wave, Allen--Cahn, Burgers, and Schrödinger APIs.

## Diffusion equation

\[
\partial_tu=D\Delta u+s(t).
\]

### Unified one-step interface

```python
diffusion2d_solve(
    u,
    D,
    Lx,
    Ly,
    dt,
    bc="fourier",
    mode=None,
    robin=None,
    robin_iters=20,
    robin_tol=1e-10,
    robin_relax=0.5,
    robin_penalty=10.0,
    source=None,
    t=0.0,
    etd_eps=None,
    backend=None,
    backend_name="numpy",
    workers=None,
    cache=None,
    return_info=False,
    **backend_kwargs,
)
```

For a periodic homogeneous problem, the update is exact for the spatially
discrete linear system:

\[
\widehat u^{\,n+1}
=\exp(-D\Delta t|\mathbf{k}|^2)\widehat u^{\,n}.
\]

When `source` is provided in periodic mode, it is evaluated at
`t + dt/2` and incorporated with an ETD1 phi function. Source terms are not
currently supported in the Dirichlet, Neumann, or Robin paths.

Specialized entry points:

```python
diffusion2d_periodic(
    u, D, Lx, Ly, dt,
    backend=None,
    backend_name="numpy",
    cache=None,
    **backend_kwargs,
)

diffusion2d_periodic_etd1(
    u, D, Lx, Ly, dt,
    t=0.0,
    source=None,
    backend=None,
    backend_name="numpy",
    cache=None,
    eps=None,
    **backend_kwargs,
)

diffusion2d_dirichlet(u, D, Lx, Ly, dt, workers=None)
diffusion2d_neumann(u, D, Lx, Ly, dt, workers=None)

diffusion2d_robin_penalty(
    u, D, Lx, Ly, dt,
    robin=None,
    iters=20,
    tol=1e-10,
    relax=0.5,
    penalty=10.0,
    backend=None,
    backend_name="numpy",
    cache=None,
    **backend_kwargs,
)
```

The Robin routine performs a periodic spectral prediction followed by repeated
one-sided finite-difference boundary projections.

## Wave equation

\[
\partial_{tt}u=c^2\Delta u,\qquad v=\partial_tu.
\]

```python
wave2d_solve(
    u,
    v,
    c,
    Lx,
    Ly,
    dt,
    bc="fourier",
    mode=None,
    backend=None,
    backend_name="numpy",
    workers=None,
    return_info=False,
    **backend_kwargs,
)
```

The function returns `(u_next, v_next)`, or
`(u_next, v_next, info)` when diagnostics are requested. Each spectral mode is
advanced analytically:

\[
\begin{aligned}
U^{n+1} &= \cos(\omega\Delta t)U^n
 + \frac{\sin(\omega\Delta t)}{\omega}V^n,\\
V^{n+1} &= -\omega\sin(\omega\Delta t)U^n
 + \cos(\omega\Delta t)V^n.
\end{aligned}
\]

The zero-frequency limit uses
`\sin(\omega dt)/\omega → dt`.

```python
wave2d_periodic(u, v, c, Lx, Ly, dt, backend=None, backend_name="numpy", **kwargs)
wave2d_dirichlet(u, v, c, Lx, Ly, dt, workers=None)
wave2d_neumann(u, v, c, Lx, Ly, dt, workers=None)
```

## Allen--Cahn equation

The two-dimensional solver advances:

\[
\partial_tu=\varepsilon^2\Delta u+u-u^3.
\]

The diffusion subproblem is solved spectrally and the reaction subproblem is
integrated exactly pointwise. `method="strang"` is second-order symmetric
splitting; `method="lie"` is first order.

```python
allen_cahn2d_step(
    u,
    epsilon,
    Lx,
    Ly,
    dt,
    *,
    bc="periodic",
    mode=None,
    method="strang",
    backend=None,
    backend_name="numpy",
    workers=None,
    cache=None,
    return_info=False,
    **backend_kwargs,
)

allen_cahn2d_solve(
    u0,
    epsilon,
    Lx,
    Ly,
    dt,
    t_end,
    *,
    t0=0.0,
    bc="periodic",
    mode=None,
    method="strang",
    backend=None,
    backend_name="numpy",
    workers=None,
    cache=None,
    save_stride=1,
    return_all=True,
    return_info=False,
    **backend_kwargs,
)
```

Supported two-dimensional boundary conditions are periodic, homogeneous
Dirichlet, and homogeneous Neumann. The trajectory return contract is:

- `(times, states)` normally;
- `(times, states, info)` with `return_info=True`;
- if `return_all=False`, `states` is the final state.

One-dimensional periodic interfaces:

```python
allen_cahn_reaction_step(u, dt, reaction=1.0)

allen_cahn1d_step(
    u, diffusion, reaction, L, dt,
    *,
    bc="periodic",
    method="strang",
    backend=None,
    backend_name="numpy",
    device=None,
    return_info=False,
    **backend_kwargs,
)

allen_cahn1d_solve(
    u0, diffusion, reaction, L, dt, t_end,
    *,
    t0=0.0,
    bc="periodic",
    method="strang",
    backend=None,
    backend_name="numpy",
    device=None,
    save_stride=1,
    return_all=True,
    return_info=False,
    **backend_kwargs,
)
```

Energy diagnostics:

```python
allen_cahn1d_energy_periodic(u, diffusion, reaction, L)
allen_cahn_energy_periodic(u, epsilon, Lx, Ly)
```

For an adequately resolved dissipative simulation, the discrete free energy
should show the expected non-increasing trend up to splitting and roundoff
errors.

## Viscous Burgers equation

\[
\partial_tu+u\,\partial_xu=\nu\partial_{xx}u.
\]

### Periodic spectral solver

```python
burgers1d_step(
    u,
    nu,
    L,
    dt,
    *,
    nonlinear_order=2,
    dealias=True,
    conservative=True,
    backend_name="numpy",
    device=None,
    return_info=False,
)

burgers1d_solve(
    u0,
    nu,
    L,
    dt,
    t_end,
    *,
    t0=0.0,
    save_stride=1,
    nonlinear_order=2,
    dealias=True,
    conservative=True,
    backend_name="numpy",
    device=None,
    return_info=False,
)
```

The linear diffusion part is applied exactly in Fourier space. The nonlinear
substep uses explicit Euler (`nonlinear_order=1`) or midpoint RK2
(`nonlinear_order=2`) within a symmetric diffusion--convection--diffusion
composition.

`dealias=True` applies the two-thirds rule. `conservative=True` evaluates
`-∂x(u²/2)`, which is usually preferred for mass behavior. Supported backends
are NumPy and PyTorch.

Aliases:

```python
solve_burgers1d = burgers1d_solve
solve_burgers1d_periodic = burgers1d_solve
```

Utilities:

```python
spectral_dx_1d(u, L, *, backend_name="numpy", device=None)
burgers1d_mass(u, L)
```

### Homogeneous Dirichlet solver

```python
burgers1d_dirichlet_step(u, nu, L, dt, *, return_info=False)

burgers1d_dirichlet_solve(
    u0, nu, L, dt, t_end,
    *,
    t0=0.0,
    save_stride=1,
    return_info=False,
)
```

This path uses an endpoint-inclusive NumPy grid, a second-order conservative
finite-difference spatial discretization, and explicit midpoint RK2. The first
and last values are set to zero at every stage. Select `dt` according to both
convective and diffusive stability restrictions.

## Schrödinger equations

### One-dimensional cubic NLS

\[
i\partial_t\psi
+a\,\partial_{xx}\psi
+b|\psi|^2\psi=0.
\]

```python
schrodinger1d_cubic_step(
    psi,
    L,
    dt,
    *,
    dispersion=0.5,
    nonlinearity=1.0,
    bc="periodic",
    mode=None,
    backend=None,
    backend_name="numpy",
    device=None,
    return_info=False,
    **backend_kwargs,
)

schrodinger1d_cubic_solve(
    psi0,
    L,
    dt,
    t_end,
    *,
    t0=0.0,
    dispersion=0.5,
    nonlinearity=1.0,
    bc="periodic",
    mode=None,
    backend=None,
    backend_name="numpy",
    device=None,
    save_stride=1,
    return_info=False,
    **backend_kwargs,
)
```

The implementation uses the split-step Fourier method with half linear phases
around the exact local nonlinear phase. Only periodic boundaries are supported.

### Two-dimensional free Schrödinger equation

\[
\partial_t\psi=i\,a\,\Delta\psi.
\]

```python
schrodinger2d_step(
    psi, Lx, Ly, dt,
    *,
    coefficient=0.5,
    bc="periodic",
    mode=None,
    backend=None,
    backend_name="numpy",
    device=None,
    return_info=False,
    **backend_kwargs,
)

schrodinger2d_solve(
    psi0, Lx, Ly, dt, t_end,
    *,
    t0=0.0,
    coefficient=0.5,
    bc="periodic",
    mode=None,
    backend=None,
    backend_name="numpy",
    device=None,
    save_stride=1,
    return_info=False,
    **backend_kwargs,
)
```

Each Fourier mode is advanced by the exact phase
`exp(-1j * coefficient * K2 * dt)`. The input and output are complex.

Mass diagnostics:

```python
schrodinger1d_mass(psi, L)
schrodinger_mass(psi, Lx, Ly)
```

These compute the periodic integral of `|psi|²` and transfer only the final
scalar to the host when the field resides on an accelerator.

## Operator-splitting utilities

```python
lie_step(state, dt, first_step, second_step)
strang_step(state, dt, linear_step, nonlinear_step)

advance_fixed_step(
    state,
    t0,
    t1,
    dt,
    step,
    *,
    save_stride=1,
    return_all=True,
    stack=None,
)
```

`lie_step` applies two full substeps in sequence. `strang_step` applies
`L(dt/2) → N(dt) → L(dt/2)`. `advance_fixed_step` is backend-agnostic: the
supplied callable has signature `step(state, step_dt, time)`, and an optional
`stack` function controls how saved states are assembled.
