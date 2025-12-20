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

## Minimal example (public)

```python
import numpy as np

# --- placeholder API ---
# from optixde.solvers.diffusion import diffusion2d_solve

N = 128
u0 = np.zeros((N, N))
# u = diffusion2d_solve(u0, dt=..., steps=..., mask=...)
```

## What to show publicly (recommended)

- Exact **function signatures** (stable API)
- Small snippets that demonstrate **usage**, not implementation
- Plots / GIFs / videos produced by the private repo

## What NOT to show

- Private kernels / implementation details
- Internal datasets
- Confidential benchmarks from third-party projects
