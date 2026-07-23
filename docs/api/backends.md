# FFT backends

`optixde.fft_backend` isolates array creation, FFT execution, frequency-grid
construction, device placement, and propagator caching from the PDE solvers.
The same periodic solver can therefore run on NumPy, CuPy, or PyTorch without
changing its numerical formulation.

## Backend selection

```python
from optixde.fft_backend import get_backend

cpu = get_backend("numpy")
gpu_torch = get_backend("torch", device="cuda")
gpu_cupy = get_backend("cupy", device=0)
```

```python
get_backend(name: str = "numpy", **kwargs) -> FFTBackend
```

Accepted names are `numpy`/`np`, `torch`/`pytorch`, and
`cupy`/`cp`/`cufft`. Backend instances are shared by normalized constructor
arguments, so repeated calls reuse cached arrays.

```python
clear_backend_cache()
```

Clears shared backend instances, frequency grids, and propagator arrays. This is
mainly useful in long-running interactive processes or when releasing GPU
memory between unrelated simulations.

!!! note "Optional dependencies"
    Requesting `torch` or `cupy` without the corresponding package raises a
    targeted `ImportError`. Importing OptiXDE itself does not require either
    accelerator library.

## `FFTBackend`

```python
class FFTBackend
```

Abstract execution contract used by periodic spectral solvers. Application code
normally obtains a concrete backend with `get_backend()` rather than
instantiating this class.

### Core properties and methods

| Member | Contract |
|---|---|
| `name` | Stable backend identifier |
| `device` | Execution device label |
| `capabilities` | Backend feature dictionary |
| `supports(capability)` | Query a capability flag |
| `asarray(x, dtype=None)` | Convert or transfer an input |
| `to_device(x)` | Move an array to the backend device |
| `complex_dtype(real_dtype)` | Select matching complex precision |
| `fft2(x)`, `ifft2(X)` | Two-dimensional complex FFT pair |
| `mul_inplace(X, G)` | Prefer in-place spectral multiplication |
| `make_freq_grids(...)` | Create or reuse angular-frequency grids |
| `clear_caches()` | Release backend-owned cached arrays |

```python
make_freq_grids(
    Nx: int,
    Ny: int,
    Lx: float,
    Ly: float,
    *,
    dtype=None,
) -> FreqGrids
```

The cache key includes the grid dimensions, physical lengths, dtype, and device,
preventing accidental reuse across incompatible simulations.

## `FreqGrids`

```python
FreqGrids(kx, ky, KX, KY, K2)
```

Container returned by `make_freq_grids`.

| Attribute | Shape | Meaning |
|---|---:|---|
| `kx` | `(Nx,)` | Angular frequencies in the x direction |
| `ky` | `(Ny,)` | Angular frequencies in the y direction |
| `KX` | `(Ny, Nx)` | x-frequency mesh |
| `KY` | `(Ny, Nx)` | y-frequency mesh |
| `K2` | `(Ny, Nx)` | `KX**2 + KY**2` |

The periodic convention is compatible with FFT ordering and the domain
`[0, Lx) × [0, Ly)`.

## Concrete backends

### `NumpyBackend()`

CPU backend based on `numpy.fft`. It is the default and requires no optional
dependency.

### `TorchBackend(device=None)`

PyTorch backend using `torch.fft`. If `device` is omitted, the implementation
selects its default device; pass an explicit value such as `"cpu"`, `"cuda"`,
or `"mps"` when reproducible placement matters.

### `CuPyBackend(device=None)`

GPU backend based on CuPy/cuFFT. `device` is an integer CUDA device index.

## `PropagatorCache`

```python
PropagatorCache(backend)
```

Stores time-invariant spectral multipliers on the backend device.

```python
cache.exp_k2(K2, *, coef: float, out_dtype=None)
cache.inv_k2(
    K2,
    *,
    eps: float = 1e-12,
    out_dtype=None,
    zero_mode_to_zero: bool = True,
)
cache.clear()
```

`exp_k2` caches arrays of the form `exp(coef * K2)`, used by diffusion and
splitting methods. `inv_k2` caches a regularized inverse of `K2` for periodic
Green operators. Cache keys account for the coefficient, dtype, device, array
shape, and array identity.

## Reusing a backend

```python
from optixde.fft_backend import PropagatorCache, get_backend
from optixde.solvers import diffusion2d_periodic

backend = get_backend("numpy")
cache = PropagatorCache(backend)

for _ in range(1000):
    u = diffusion2d_periodic(
        u, D, Lx, Ly, dt,
        backend=backend,
        cache=cache,
    )
```

Keeping the backend, frequency grids, and multipliers resident is the preferred
pattern for production time integration.
