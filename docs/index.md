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

## A minimal diffusion solving using 20 Lines

```python
import numpy as np
import matplotlib.pyplot as plt
from optixde.solvers.base.diffusion import diffusion2d_solve
Lx=Ly=np.pi
N=128
D=1.0
dt=0.5 
T=2.0
steps=int(round(T/dt))
dx=Lx/(N+1)
dy=Ly/(N+1)
x=np.linspace(dx,Lx-dx,N)
y=np.linspace(dy,Ly-dy,N)
X,Y=np.meshgrid(x,y,indexing="xy")
u=np.zeros((N+2,N+2))
u[1:-1,1:-1]=10*np.sin(X)*np.sin(Y)
for _ in range(steps): u = diffusion2d_solve(u, D, Lx, Ly, dt, bc="dirichlet")
plt.imshow(u, cmap="jet", origin="lower", extent=[0,Lx,0,Ly])
plt.colorbar()
plt.show()
```

---

## Results snapshot

Add a few **representative** figures:
- error vs resolution
- runtime vs resolution
- a complex geometry demo (L-shape, hole, union/intersection)

Create these images from your private repo results and copy only the exported figures into `docs/assets/figures/`.
