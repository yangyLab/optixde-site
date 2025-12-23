# Quickstart

This public site can show **how to use OptiXDE** without exposing the private repository.

## Installation

If OptiXDE is private, keep installation instructions generic:

```bash
# Option A: internal/private install (example)
pip install optixde --extra-index-url <YOUR_PRIVATE_INDEX>

# Option B: request access to the private repo
# (See the "Cite & Contact" page)
```

## Heat Conduction in 20 Lines: A Minimal Diffusion Solver

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

## What to show publicly (recommended)

- Exact **function signatures** (stable API)
- Small snippets that demonstrate **usage**, not implementation
- Plots / GIFs / videos produced by the private repo

## What NOT to show

- Private kernels / implementation details
- Internal datasets
- Confidential benchmarks from third-party projects
