# API Reference

OptiXDE is organized around a small set of building blocks:

- **FFT backends** (NumPy / Torch / CuPy) that provide a uniform FFT interface and cached frequency grids.

- **Spectral operators** (propagators / Green's functions) built on top of the backend.

- **Geometry** primitives and boolean composition for embedded/penalty domains.

- **Solvers** for Poisson / Helmholtz / diffusion and (optionally) linear elasticity.

- **Post-processing** helpers for plotting.


## Modules

### FFT backends

- [`optixde.fft_backend.base`](api/fft_backend/base.md)
- [`optixde.fft_backend.numpy_backend`](api/fft_backend/numpy_backend.md)
- [`optixde.fft_backend.torch_backend`](api/fft_backend/torch_backend.md)

### Operators

- [`optixde.operators.transforms`](api/operators/transforms.md)
- [`optixde.operators.spectral`](api/operators/spectral.md)

### Geometry

- [`optixde.geometry.primitives`](api/geometry/primitives.md)
- [`optixde.geometry.boolean`](api/geometry/boolean.md)

### Solvers (base)

- [`optixde.solvers.base.poisson`](api/solvers/poisson.md)
- [`optixde.solvers.base.poisson_inhom`](api/solvers/poisson_inhom.md)
- [`optixde.solvers.base.helmholtz`](api/solvers/helmholtz.md)
- [`optixde.solvers.base.diffusion`](api/solvers/diffusion.md)
- [`optixde.solvers.base.etd`](api/solvers/etd.md)


### Post-processing

- [`optixde.post.plotting`](api/post/plotting.md)

## Installation (editable)

For local development:

```bash
pip install -e .
```
