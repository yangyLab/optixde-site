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
## Geometry Construction Tutorial

OptiXDE runs on an FFT-compatible uniform grid, so geometry is represented **implicitly** as a *mask field* on that grid (not a mesh). The recommended public-facing pipeline is:

**primitives → CSG → binary mask → smoothed mask → embedded/penalized PDE**

This keeps the API simple while allowing complex shapes, and it’s safe to share publicly because it demonstrates **usage** rather than implementation.

---

### 1) Create a geometry (primitives)
OptiXDE provides simple 2D primitives that can be composed later with CSG operations.
Here are three basic examples.

#### Example 1: Rectangle
```python
from optixde.geometry.primitives import Rectangle
from optixde.post.plotting import show_geometry

geo = Rectangle(0.0, 1.0, 0.0, 1.0)
show_geometry(geo, title="Rectangle", facecolor="skyblue", edgecolor="black")
```
![Rectangle result](image/Rectangle.png)

#### Example 2: Disk
```python
from optixde.geometry.primitives import Disk
from optixde.post.plotting import show_geometry

geo = Disk((0.5, 0.5), 0.25)
show_geometry(geo, title="Disk", facecolor="tomato", edgecolor="black")
```
![Rectangle result](image/Disk.png)

#### Example 3: Polygon
```python
from optixde.geometry.primitives import Polygon
from optixde.post.plotting import show_geometry

geo = Polygon([
    (1.0, 0.0),
    (0.5, 0.8660254),
    (-0.5, 0.8660254),
    (-1.0, 0.0),
    (-0.5, -0.8660254),
    (0.5, -0.8660254),
])
show_geometry(geo, title="Polygon", facecolor="gold", edgecolor="black")
```
![Polygon result](image/Polygon.png)

### 2) Apply boolean operations
Boolean operations combine simple primitives into more complex shapes. In OptiXDE, the most common operations are:

-  : union
-  : intersection
-  : difference

Below are three minimal examples using `Rectangle`, `Disk`, and `Polygon`.

---

#### Example 1: Union of a rectangle and two circle

```python
from optixde.geometry.primitives import Rectangle, Disk
from optixde.post.plotting import show_geometry
from optixde.geometry.boolean import Union

rect = Rectangle(0.0, 1.0, 0.0, 1.0)
left_circle  = Disk((0.0, 0.5), 0.2)
right_circle = Disk((1.0, 0.5), 0.2)
geo = Union(rect, left_circle, right_circle)
show_geometry(geo, title="Rectangle", facecolor="skyblue", edgecolor="black")
```
![Union result](image/Union.png)

#### Example: Intersection of a rectangle and a disk

```python
from optixde.geometry.primitives import Rectangle, Disk
from optixde.geometry.boolean import Intersection
from optixde.post.plotting import show_geometry

rect = Rectangle(-0.8, 0.8, -0.8, 0.8)
disk = Disk((0.3, 0.0), 0.75)

geo = Intersection(rect, disk)

show_geometry(
    geo,
    title="Rectangle ∩ Disk",
    facecolor="skyblue",
    edgecolor="black",
)
```
![Intersection result](image/Intersection.png)

#### Example: Difference of a rectangle and a disk

```python
from optixde.geometry.primitives import Rectangle, Disk
from optixde.geometry.boolean import Difference
from optixde.post.plotting import show_geometry

rect = Rectangle(-1.0, 1.0, -0.8, 0.8)
hole = Disk((0.0, 0.0), 0.35)

geo = Difference(rect, hole)

show_geometry(
    geo,
    title="Difference",
    facecolor="skyblue",
    edgecolor="black",
)
```
![Difference result](image/Difference.png)

### 3) From geometry to mask field

OptiXDE represents geometry on an FFT-compatible uniform grid as a mask field.

#### Example: geometry to binary mask
The geometry can be sampled on a uniform Cartesian grid and represented as a mask field.  
In this example, an L-shaped domain is created by subtracting the square `[0,1] × [0,1]` from the outer square `[-1,1] × [-1,1]`. The geometry is evaluated on a regular grid, and a smoothed mask is generated for visualization and for later use in embedded-domain solving.
```python
import numpy as np
import matplotlib.pyplot as plt

from optixde.geometry.primitives import Rectangle
from optixde.geometry.boolean import Difference

x = np.linspace(-1.0, 1.0, 257)
y = np.linspace(-1.0, 1.0, 257)
X, Y = np.meshgrid(x, y, indexing="xy")
P = np.c_[X.ravel(), Y.ravel()]

geo = Difference(
    Rectangle(-1.0, 1.0, -1.0, 1.0),
    Rectangle(0.0, 1.0, 0.0, 1.0),
)

eps = 2.0 * (x[1] - x[0])
mask = geo.smooth_mask(P, eps=eps).reshape(len(y), len(x))

plt.figure(figsize=(6.2, 5.2))
plt.imshow(
    mask,
    origin="lower",
    cmap="jet",
    extent=[x[0], x[-1], y[0], y[-1]],
)
plt.colorbar(label="mask")
plt.title("Smoothed mask")
plt.tight_layout()
plt.show()
```
![Mask result](image/Mask.png)

### 3) Define, solve, and visualize a PDE

Once the computational grid and source term are prepared, solving a PDE in OptiXDE is straightforward.

In this example, we consider the 2D Poisson equation on the square domain `[0, \pi] \times [0, \pi]` with homogeneous Dirichlet boundary conditions,
\[
-\Delta u = f \quad \text{in } \Omega,
\]
where the right-hand side is chosen as
\[
f(x,y)=2\sin(x)\sin(y).
\]
For this choice, the exact solution is
\[
u(x,y)=\sin(x)\sin(y),
\]
which makes the example convenient for verification. After defining the source field on the uniform grid, the Poisson solver is called directly, and the resulting solution is visualized as a color map. This illustrates the basic OptiXDE workflow: define the PDE, solve it on the FFT-compatible grid, and inspect the result immediately.

```python
import numpy as np
import matplotlib.pyplot as plt
from optixde.solvers.base.poisson import poisson2d_solve

Lx = Ly = np.pi
N = 128

dx = Lx / (N + 1)
dy = Ly / (N + 1)
x = np.linspace(dx, Lx - dx, N)
y = np.linspace(dy, Ly - dy, N)
X, Y = np.meshgrid(x, y, indexing="xy")

f = np.zeros((N + 2, N + 2))
f[1:-1, 1:-1] = 2.0 * np.sin(X) * np.sin(Y)

u = poisson2d_solve(f, Lx, Ly, bc="dirichlet", workers=8)

plt.figure(figsize=(6.2, 5.2))
plt.imshow(
    u,
    cmap="jet",
    origin="lower",
    extent=[0, Lx, 0, Ly],
)
plt.colorbar(label="u")
plt.title("2D Poisson solution")
plt.xlabel("x")
plt.ylabel("y")
plt.show()
```

