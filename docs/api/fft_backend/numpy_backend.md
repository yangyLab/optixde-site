# `optixde.fft_backend.numpy_backend`

optixde.fft_backend.numpy_backend

NumPy FFT backend implementation (CPU, periodic).

This backend provides FFT primitives and periodic frequency grids using NumPy.
It conforms to the abstract interface defined in optixde.fft_backend.base.

Key behaviors:
- CPU-only, no device concept (to_device is a no-op in the base class).
- dtype handling:
  - real input is promoted to a complex dtype for FFT (complex64/complex128).
  - grid dtype can be requested via make_freq_grids(..., dtype=...).
- Frequency convention:
  - kx, ky are angular frequencies (rad/unit length):
        k = 2π * fftfreq(N, d=Δx)
    where Δx = Lx/N for periodic grid on [0,Lx).

Notes for API docs:
- Users typically do not instantiate this class directly; they call get_backend("numpy").
- Frequency grids are cached in FFTBackend.make_freq_grids to avoid repeated allocation.

## `NumpyBackend`

*Type*: class

NumPy FFT backend (periodic) on CPU.

This backend implements the FFTBackend interface using numpy.fft.

Notes
-----
- This backend assumes periodic problems on [0,Lx)×[0,Ly) for frequency grids.
- For large grids, using the cache in FFTBackend.make_freq_grids is important
  to avoid repeatedly constructing K2.
