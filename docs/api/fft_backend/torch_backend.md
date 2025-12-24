# `optixde.fft_backend.torch_backend`

optixde.fft_backend.torch_backend

PyTorch FFT backend implementation.

This backend provides FFT primitives and periodic frequency grids using torch.fft,
supporting both CPU and CUDA devices. It conforms to the abstract interface defined
in optixde.fft_backend.base.

Key behaviors:
- Device-aware: backend runs on a specified torch.device (cpu / cuda[:i]).
- dtype handling:
  - real input is promoted to a complex dtype for FFT (complex64/complex128).
  - grid dtype can be requested via make_freq_grids(..., dtype=...).
- Frequency convention:
  - kx, ky are angular frequencies (rad/unit length):
        k = 2π * fftfreq(N, d=Δx)
    where Δx = Lx/N for periodic grid on [0,Lx).

Notes for API docs:
- Users typically do not instantiate this class directly; they call get_backend("torch", device="cuda:0").
- Frequency grids are cached in FFTBackend.make_freq_grids; the cache key includes `str(self.device)`.

## `TorchBackend`

*Type*: class

PyTorch FFT backend for CPU/GPU execution.

Parameters
----------
device : str or None
    Torch device string:
    - "cpu"
    - "cuda"
    - "cuda:0", "cuda:1", ...
    If None, defaults to "cuda" when available, otherwise "cpu".

Attributes
----------
device : torch.device
    The device used by this backend instance.

Notes
-----
- All arrays produced by this backend are placed on `self.device`.
- Frequency grids are cached per device via FFTBackend._freq_cache and _device_key.
