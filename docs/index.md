# OptiXDE
![OptiXDE](assets/optixde_logo_horizontal.svg)
**OptiXDE** is a fast optical-inspired solver for differential equations.

<div style="display:flex; gap:12px; flex-wrap:wrap; margin: 16px 0;">
  <a class="md-button md-button--primary" href="quickstart/">Quickstart</a>
  <a class="md-button" href="method/">Method</a>
  <a class="md-button" href="benchmarks/">Benchmarks</a>
  <a class="md-button" href="cite/">Cite & Contact</a>
</div>

---

## One-line pitch

A **matrix-free spectral propagation-operator** approach (FFT → propagate → iFFT) for rapid PDE solves on uniform grids, with support for **embedded domains** and **penalty-based** boundary handling.

> Tip: Keep this page “light” and visual—put deeper explanations into the Method/Benchmarks pages.

---

## Highlights

- **Matrix-free**: no stiffness-matrix assembly; GPU-friendly building blocks.
- **Fast**: FFT-based operators with typical complexity **O(N log N)**.
- **Unified**: diffusion / Poisson / wave share a consistent operator view.
- **Embedded geometry**: work with irregular domains via mask + penalty.
- **Reproducible**: clean benchmarks, fixed seeds, and documented configs.

---

## The big picture

![pipeline](assets/pipeline.svg){ width="900" }

---

## 20-line teaser (pseudo-code)

```python
# PSEUDO-CODE (for the public site)
u = u0
for n in range(steps):
    U = fft2(u)
    U = G_kdt * U          # propagation / Green's operator in k-space
    u = ifft2(U).real
    u = apply_penalty(u, mask, boundary_data)  # for embedded domains
```

---

## Results snapshot

Add a few **representative** figures:
- error vs resolution
- runtime vs resolution
- a complex geometry demo (L-shape, hole, union/intersection)

Create these images from your private repo results and copy only the exported figures into `docs/assets/figures/`.
