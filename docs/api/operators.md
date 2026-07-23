# Spectral operators

`optixde.operators` contains reusable transform and multiplier kernels. Most
users call them indirectly through a solver; they are public for custom
matrix-free algorithms.

## Real transforms

```python
dct2(a, workers=None)
idct2(A, workers=None)
dst2(a, workers=None)
idst2(A, workers=None)
```

The functions apply orthonormal two-dimensional transforms over the final two
axes.

| Pair | Typical boundary condition | Grid treatment |
|---|---|---|
| `dct2` / `idct2` | Homogeneous Neumann | Full grid |
| `dst2` / `idst2` | Homogeneous Dirichlet | Interior block |

`workers` is forwarded when the active transform library supports threaded
execution. The implementation selects SciPy, CuPy, or a compatibility path
according to the input type and installed dependencies.

```python
coefficients = dst2(f[1:-1, 1:-1], workers=-1)
interior = idst2(coefficients, workers=-1)
```

## Diffusion multiplier

```python
diffusion_propagator(
    cache: PropagatorCache,
    K2,
    *,
    D: float,
    dt: float,
    out_dtype=None,
)
```

Returns

\[
G(\mathbf{k})=\exp\!\left(-D\,\Delta t\,|\mathbf{k}|^2\right).
\]

`D` and `dt` must be non-negative. The result is obtained through the supplied
`PropagatorCache`, so repeated calls with identical parameters reuse the same
device-resident multiplier.

## Periodic Poisson Green operator

```python
poisson_green_periodic(
    cache: PropagatorCache,
    K2,
    *,
    eps: float = 1e-12,
    out_dtype=None,
)
```

Returns the regularized diagonal inverse used for `-Δu=f`:

\[
G(\mathbf{k})=
\begin{cases}
|\mathbf{k}|^{-2}, & \mathbf{k}\ne0,\\
0, & \mathbf{k}=0.
\end{cases}
\]

Setting the zero mode to zero fixes the additive-constant gauge. The forcing
must satisfy the periodic compatibility condition; the high-level Poisson
solver can enforce it automatically.

## Applying a multiplier

```python
apply_propagator(backend, u, G)
```

Computes:

```python
U = backend.fft2(u)
backend.mul_inplace(U, G)
u_new = backend.ifft2(U)
```

The result may be complex. A high-level real-valued PDE solver discards
roundoff-level imaginary components where mathematically appropriate.

## Custom periodic propagator

```python
from optixde.fft_backend import PropagatorCache, get_backend
from optixde.operators import apply_propagator

backend = get_backend("numpy")
grids = backend.make_freq_grids(Nx, Ny, Lx, Ly, dtype=u.dtype)
cache = PropagatorCache(backend)

# Exact step for u_t = D Δu
G = cache.exp_k2(grids.K2, coef=-D * dt, out_dtype=u.dtype)
u = apply_propagator(backend, u, G).real
```

For nonlinear equations, combine this linear step with a physical-space
nonlinear update using the splitting helpers documented under
[Evolution equations](evolution.md#operator-splitting-utilities).
