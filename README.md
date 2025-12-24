# OptiXDE

**OptiXDE** is a fast, matrix-free solver for partial differential equations,
inspired by optical wave propagation and spectral operator theory.

The core idea of OptiXDE is to reformulate elliptic and parabolic PDEs
(e.g. Poisson, Helmholtz, diffusion) as **spectral propagation problems**,
leading to highly efficient solvers based on FFT/DCT/DST operators,
without assembling stiffness matrices.

---

## Key features

- **Optical-inspired formulation**  
  PDE operators are treated as spectral propagators, analogous to
  angular-spectrum wave propagation.

- **Matrix-free and FFT-based**  
  No sparse matrix assembly; all operators are applied via FFT/DCT/DST.

- **Multiple boundary conditions**  
  Periodic, Dirichlet, Neumann, and inhomogeneous Dirichlet problems are supported.

- **Backend-agnostic**  
  Unified support for NumPy (CPU), CuPy (GPU), and PyTorch backends.

- **Embedded and complex geometries**  
  Built-in geometry primitives and Boolean operations enable embedded-domain
  formulations via smooth masks and penalty methods.

---

## Scope of the library

OptiXDE currently focuses on:

- Poisson and Helmholtz equations
- Diffusion equations and exponential time differencing (ETD)
- Spectral Green’s operators and propagators
- Geometry construction and embedded-domain handling
- Post-processing and visualization utilities

Extensions to solid mechanics and time-dependent problems are under active development.

---

## Documentation

- **Quickstart**: installation and minimal examples  
- **Method**: mathematical formulation and operator interpretation  
- **Benchmarks**: representative numerical tests  
- **API Reference**: detailed documentation of all modules and functions

Use the navigation bar above to explore the documentation.

---

## Project status

OptiXDE is an **active research codebase**.
The website is public for documentation and reproducibility,
while the core solver implementation may remain private during ongoing research.

---

## Citation

If you use OptiXDE in academic work, please cite the corresponding paper
(to be announced).

