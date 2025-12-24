# `optixde.solvers.base.etd`

optixde.solvers.base.etd (ODE utilities)

Exponential Time Differencing (ETD) helper routines for the scalar/diagonal ODE:

    y'(t) = -lam * y(t) + f(t),

where:
- lam can be a scalar or an array/tensor (element-wise / diagonal operator),
- f(t) is an optional forcing term evaluated in physical space.

This file provides:
- phi1(z): stable evaluation of ϕ1(z) = (exp(z) - 1)/z near z=0
- etd1_step: one ETD1 time step (midpoint forcing)
- solve_ode: integrate from t0 to t1 with fixed dt and optional downsampling

Backend support:
- numpy.ndarray
- torch.Tensor (if torch is installed)
- cupy.ndarray (if cupy is installed)

Design notes (for API docs):
- All functions try to preserve the backend type of the input `y`.
- Forcing should return an array-like compatible with `y` (same shape).
- lam is interpreted element-wise; this is suitable for diagonal spectral operators.

## `phi1(z, eps)`

*Type*: function

Compute ϕ1(z) = (exp(z) - 1) / z with a stable approximation near z=0.

For |z| < eps, use the first-order Taylor expansion:
    ϕ1(z) ≈ 1 + z/2
to avoid catastrophic cancellation.

Parameters
----------
z : array-like
    Input values (numpy/torch/cupy). Typically z = -lam*dt (non-positive).
eps : float
    Small-z threshold for using the Taylor approximation.

Returns
-------
array-like
    ϕ1(z) evaluated element-wise, on the same backend as z.

Notes
-----
- This helper is used by ETD1 to integrate the forcing term.
- The numpy branch uses explicit arrays for speed and precise dtype control.

## `etd1_step(y, t, dt, lam, forcing, eps)`

*Type*: function

Take one ETD1 step for the ODE: y'(t) = -lam*y(t) + forcing(t).

The ETD1 update with midpoint forcing is:
    y_{n+1} = exp(-lam*dt) * y_n + dt * ϕ1(-lam*dt) * f(t_n + dt/2)

Parameters
----------
y : array-like
    Current state at time t. Can be scalar, vector, or tensor.
t : float
    Current time t_n.
dt : float
    Time step size.
lam : float or array-like
    Damping/decay rate. Interpreted element-wise if array-like.
    Typical usage: lam is the diagonal eigenvalue array in spectral space.
forcing : callable(t)->array-like or None
    Forcing function f(t). If None, integrates the homogeneous equation.
eps : float
    Small-z threshold for ϕ1(z) evaluation.

Returns
-------
array-like
    Updated state y_{n+1} on the same backend as y.

Notes
-----
- Forcing is evaluated at the midpoint t + dt/2.
- The code preserves the backend type/device of y (numpy/torch/cupy).

## `solve_ode(y0, t0, t1, dt, lam, forcing, eps, return_all, stack, save_stride)`

*Type*: function

Integrate y'(t) = -lam*y(t) + forcing(t) from t0 to t1 using ETD1 with fixed dt.

Parameters
----------
y0 : array-like
    Initial state at t0.
t0, t1 : float
    Start and end times. Must satisfy t1 >= t0.
dt : float
    Fixed time step size. Must be positive.
lam : float or array-like
    Damping/decay rate (scalar or element-wise array).
forcing : callable(t)->array-like or None
    Forcing function. If None, solves homogeneous system.
eps : float
    Small-z threshold for ϕ1(z) evaluation.
return_all : bool
    If True, return the saved trajectory (time array + y values).
    If False, only return final y (still returns the full time grid ts_full).
stack : bool
    Only relevant for torch/cupy when return_all=True:
    - False: return a Python list of states [y(t0), y(t0+stride*dt), ...]
    - True: stack into a single array/tensor with leading time dimension.
save_stride : int
    Save every `save_stride` steps. Must be positive.
    - save_stride=1 saves all steps (including t0).
    - save_stride=k saves t0, t0+k*dt, t0+2k*dt, ...

Returns
-------
ts : numpy.ndarray, shape (Nsaved,)
    Saved times (always returned as numpy array for simplicity).
ys :
    If return_all=True:
        - numpy: ndarray of shape (Nsaved, ...) on numpy backend
        - torch/cupy: list (or stacked tensor/array if stack=True)
    If return_all=False:
        - final state y(t1) on the same backend as y0

Notes
-----
- The number of steps M is computed by rounding (t1-t0)/dt:
      M = round((t1 - t0) / dt)
  so t1 should be aligned with dt for clean grids.
- For numpy backend, the saved trajectory is preallocated for efficiency.
- For torch/cupy, states are collected in a list to avoid costly preallocation,
  with an optional final stack.
