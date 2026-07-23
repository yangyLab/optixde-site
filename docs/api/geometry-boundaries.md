# Geometry and boundary conditions

## Signed-distance convention

All OptiXDE geometries use a **positive-inside** signed-distance-like function:

\[
d(\mathbf{x})>0 \text{ in }\Omega,\qquad
d(\mathbf{x})=0 \text{ on }\partial\Omega,\qquad
d(\mathbf{x})<0 \text{ outside }\Omega.
\]

This convention controls masks, smooth interface weights, Boolean operations,
and embedded-domain enforcement.

## `Geometry`

`Geometry` is the abstract base class. Subclasses implement:

```python
signed_distance(X)
bbox()
```

where `X` has shape `(..., dim)` and `bbox()` returns `(lower, upper)`.

The base class also provides:

```python
geometry.mask(X)
geometry.smooth_mask(X, eps)
geometry.boundary_weight(X, eps)
geometry.inside(X, tol=0.0)
geometry.on_boundary(X, tol=1e-6)
geometry.random_interior(n, rng=None)
geometry.random_boundary(n, tol=1e-4, rng=None)
geometry.project_to_boundary(X, iters=20)
geometry.build_grid(Nx, Ny)
geometry.build_mesh(Nx, Ny)
```

`smooth_mask` uses a regularized Heaviside profile; `boundary_weight` uses a
regularized Dirac profile concentrated near the zero level set. Choose `eps`
in physical units, usually on the order of one to three grid spacings.

## Primitive geometries

```python
Rectangle(xmin, xmax, ymin, ymax)
Disk(center: tuple[float, float], radius: float)
Sphere(center: tuple[float, float, float], radius: float)
Polygon(vertices)
```

`Polygon` expects an `(N, 2)` vertex array in counter-clockwise order. It also
provides edge metadata and projection methods used by segmented boundary
conditions.

!!! note
    A three-dimensional `Box` implementation exists in
    `optixde.geometry.primitives`, but it is not currently exported from the
    top-level `optixde.geometry` namespace. Import it from its defining module
    only if you intentionally depend on that lower-level API.

## Boolean composition

```python
Union(*geometries)
Intersection(*geometries)
Difference(geometry_a, geometry_b)
```

For the positive-inside convention:

\[
d_{A\cup B}=\max(d_A,d_B),\qquad
d_{A\cap B}=\min(d_A,d_B),\qquad
d_{A\setminus B}=\min(d_A,-d_B).
\]

Example: an L-shaped domain.

```python
from optixde.geometry import Difference, Rectangle

outer = Rectangle(-1.0, 1.0, -1.0, 1.0)
cutout = Rectangle(0.0, 1.0, -1.0, 0.0)
domain = Difference(outer, cutout)
```

## Robin boundary specification

The rectangular Poisson and diffusion solvers accept either a global tuple or
a side-specific dictionary:

```python
# α u + β ∂u/∂n = g on every side
robin = (alpha, beta, g)

# Side-specific values; omitted sides use "default"
robin = {
    "left": (1.0, 0.0, 0.0),
    "right": (0.0, 1.0, flux_right),
    "default": (1.0, 1.0, 0.0),
}
```

`g` may be a scalar or a one-dimensional array matching the relevant side.

```python
normalize_robin_spec(robin) -> dict
apply_robin_fd_projection(u, Lx, Ly, robin)
```

`normalize_robin_spec` expands the input into `left`, `right`, `bottom`, and
`top` entries. `apply_robin_fd_projection` updates boundary values using
one-sided finite differences. It preserves NumPy, CuPy, or PyTorch array
placement.

## Segmented polygon boundaries

Use `EdgeSlice` to identify part of a polygon edge:

```python
EdgeSlice(
    name: str,
    start: tuple[float, float],
    end: tuple[float, float],
    kind: str,
    value=0.0,
)
```

Boundary objects attach a condition to a segment:

```python
DirichletBC(segment, priority=0, value=0.0)
NeumannBC(segment, priority=0, flux=0.0)
RobinBC(
    segment,
    priority=0,
    alpha=1.0,
    beta=1.0,
    value=0.0,
)
```

Scalar data and callables of `(x, y)` are accepted. Higher `priority` resolves
overlap where multiple boundary definitions rasterize to the same grid point.

```python
split_bcs_by_type(bcs)
rasterize_segmented_bcs(geometry, X, bcs, band_width)
```

`rasterize_segmented_bcs` evaluates the boundary definitions on a narrow band
around the polygon boundary and produces arrays used by the embedded solvers.
