# `optixde.operators.transforms`

optixde.operators.transforms

Fast 2D DCT/DST transforms used by OptiXDE's separable spectral solvers.

What is provided
----------------
This module defines 2D transforms on the last two axes:
- dct2 / idct2 : DCT-II with orthonormal normalization
- dst2 / idst2 : DST-I  with orthonormal normalization

Backend selection
-----------------
- CPU (NumPy arrays):
    Prefer SciPy FFT implementations: scipy.fft.{dct,idct,dst,idst}
    Supports `workers=` for parallel execution (SciPy >= 1.6 typically).
- GPU (CuPy arrays):
    Use cupyx.scipy.fft.{dct,idct,dst,idst}
    CuPy path ignores `workers` (handled by GPU runtime).
- Fallback:
    If SciPy is not available, use slow, explicit O(N^2) kernel-based transforms
    to keep minimal functionality (useful for tiny grids / environments without SciPy).

Important conventions
---------------------
- DCT-II (type=2, norm="ortho") is self-inverse up to using idct with the same type/norm.
- DST-I  (type=1, norm="ortho") is its own inverse under the same type/norm.

These conventions match the discrete eigen-bases for:
- Neumann BC (cosine basis)    -> DCT
- Dirichlet BC (sine basis)    -> DST

Notes for API docs
------------------
- These functions accept NumPy or CuPy arrays (2D fields or batched tensors).
- Torch tensors are explicitly rejected to avoid accidental np.asarray() copies.

## `dct2(a, workers)`

*Type*: function

2D DCT-II (orthonormal) on the last two axes.

Parameters
----------
a : np.ndarray or cupy.ndarray
    Input array. Can be 2D (Ny,Nx) or batched (...,Ny,Nx).
workers : int or None
    Number of worker threads for SciPy CPU transforms. Ignored for CuPy.

Returns
-------
A : array-like
    DCT-II coefficients of the same shape as `a`.

Notes
-----
Uses:
    dct(type=2, norm="ortho", axis=-1) then axis=-2.

## `idct2(A, workers)`

*Type*: function

2D inverse DCT corresponding to dct2 (type=2, norm="ortho").

Parameters
----------
A : np.ndarray or cupy.ndarray
    DCT coefficients, shape (...,Ny,Nx).
workers : int or None
    SciPy workers for CPU path. Ignored for CuPy.

Returns
-------
a : array-like
    Reconstructed array (same shape).

Notes
-----
Uses:
    idct(type=2, norm="ortho", axis=-1) then axis=-2.

## `dst2(a, workers)`

*Type*: function

2D DST-I (orthonormal) on the last two axes.

Parameters
----------
a : np.ndarray or cupy.ndarray
    Input array, shape (...,Ny,Nx).
workers : int or None
    SciPy workers for CPU path. Ignored for CuPy.

Returns
-------
A : array-like
    DST-I coefficients.

Notes
-----
Uses:
    dst(type=1, norm="ortho", axis=-1) then axis=-2.
DST-I is commonly used for homogeneous Dirichlet BC discretizations.

## `idst2(A, workers)`

*Type*: function

2D inverse DST corresponding to dst2 (type=1, norm="ortho").

Parameters
----------
A : np.ndarray or cupy.ndarray
    DST coefficients, shape (...,Ny,Nx).
workers : int or None
    SciPy workers for CPU path. Ignored for CuPy.

Returns
-------
a : array-like
    Reconstructed array.

Notes
-----
Uses:
    idst(type=1, norm="ortho", axis=-1) then axis=-2.
