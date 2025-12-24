# `optixde.fft_backend.base`

optixde.fft_backend.base

Abstract FFT backend interface for OptiXDE.

Motivation
----------
OptiXDE supports multiple numerical backends (NumPy / CuPy / PyTorch) to run the
same spectral solvers on CPU or GPU. The solvers expect a minimal set of FFT
primitives and a consistent way to build periodic frequency grids.

This module defines:
- FreqGrids: a lightweight container holding kx/ky grids and cached K2 = KX^2+KY^2
- FFTBackend: an abstract interface implemented by concrete backends

Key design points
-----------------
1) Periodic frequency grids are cached:
   make_freq_grids(Nx, Ny, Lx, Ly, dtype) returns a cached FreqGrids object.
   The cache key includes the backend "device key" so that GPU/CPU grids do not mix.

2) Solvers can request a preferred dtype:
   Many solvers build grids using the real dtype of the input field; the backend
   should respect dtype whenever possible.

3) Memory efficiency:
   mul_inplace(X, G) is used to encourage in-place spectral multiplication (X *= G).

Coordinate / frequency conventions
----------------------------------
- Domain: [0, Lx) × [0, Ly) for periodic FFT-based solvers.
- kx, ky are the angular frequencies (2π * integer / L) or equivalent convention
  used consistently inside the backend implementation.
- KX, KY are 2D grids from meshgrid(ky, kx) in the solver's chosen indexing.
- K2 = KX^2 + KY^2 is cached since it is used frequently (diffusion/Poisson/etc.).

Notes for API docs
------------------
- This is an internal interface; end users typically call `get_backend` and do
  not instantiate FFTBackend directly.
- Concrete implementations should override: asarray, complex_dtype, fft2, ifft2,
  mul_inplace, _make_freq_grids_impl, and (optionally) _device_key / to_device.

## `FreqGrids`

*Type*: class

Frequency grids and cached K2 for periodic spectral solvers.

Attributes
----------
kx : Any, shape (Nx,)
    1D frequency array in x direction.
ky : Any, shape (Ny,)
    1D frequency array in y direction.
KX : Any, shape (Ny, Nx)
    2D frequency grid for x direction (meshgrid result).
KY : Any, shape (Ny, Nx)
    2D frequency grid for y direction (meshgrid result).
K2 : Any, shape (Ny, Nx)
    Cached squared magnitude: K2 = KX**2 + KY**2.

Notes
-----
- The exact array types depend on the backend (numpy.ndarray / cupy.ndarray / torch.Tensor).
- K2 is cached because many PDE propagators are functions of |k|^2.

## `FFTBackend`

*Type*: class

Abstract FFT backend interface.

Concrete backends (NumPy / CuPy / PyTorch) should inherit from this class and
implement the required primitives. Solvers call only methods defined here,
so the rest of OptiXDE remains backend-agnostic.

This base class also provides a frequency-grid cache for periodic problems.
