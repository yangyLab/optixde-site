# `optixde.geometry.primitives`

optixde.geometry.primitives

Geometry primitives defined by a signed-distance-like function.

Core convention (IMPORTANT)
---------------------------
All geometries here implement:

    signed_distance(X) > 0    inside Ω
    signed_distance(X) = 0    on boundary ∂Ω
    signed_distance(X) < 0    outside Ω

This is the "positive-inside" convention.

Why this matters:
- The smooth mask utilities (smooth_heaviside / smooth_dirac) assume d>0 is inside.
- The CSG module (Union/Intersection/Difference) uses max/min formulas under this convention.

Utilities provided by Geometry
------------------------------
Besides signed_distance and bbox, Geometry provides:
- inside / on_boundary predicates
- hard mask and smooth mask (for embedded-domain / penalty spectral methods)
- boundary_weight: a narrow-band weight near ∂Ω (smoothed Dirac)
- random_interior / random_boundary sampling by rejection in bbox
- project_to_boundary: simple iterative projection using finite-difference normals

Notes for API docs
------------------
- Array type: NumPy arrays, shape (N, dim) preferred. Many methods accept (..., dim)
  but are implemented via np.atleast_2d.
- bbox returns (lo, hi) with shape (dim,) arrays.
- smooth masks are typically used with eps ≈ 1~3 grid cells in physical coordinates.

## `Geometry`

*Type*: class

Abstract geometry interface based on a signed distance (positive inside).

Subclasses must implement:
- signed_distance(X): returns d(X) with positive-inside convention
- bbox(): axis-aligned bounding box of the geometry

Attributes
----------
dim : int
    Spatial dimension (2 or 3).

## `Rectangle`

*Type*: class

Axis-aligned rectangle in 2D.

Parameters
----------
xmin, xmax, ymin, ymax : float
    Rectangle bounds.

Notes
-----
signed_distance uses the positive-inside convention:
- inside:  min distance to any edge (positive)
- outside: -distance to rectangle (negative)

## `Disk`

*Type*: class

Disk (circle) in 2D.

Parameters
----------
center : tuple(float,float)
    Circle center.
radius : float
    Circle radius.

Notes
-----
The standard circle signed distance is:
    s = ||x-c|| - r
For positive-inside convention, we return:
    d = -s = r - ||x-c||

## `Box`

*Type*: class

Axis-aligned box in 3D.

Parameters
----------
xmin, xmax, ymin, ymax, zmin, zmax : float
    Box bounds.

Notes
-----
signed_distance uses the positive-inside convention:
- inside:  min distance to any face (positive)
- outside: -distance to box (negative)

## `Sphere`

*Type*: class

Sphere in 3D.

Parameters
----------
center : tuple(float,float,float)
    Sphere center.
radius : float
    Sphere radius.

Notes
-----
Standard sphere SDF:
    s = ||x-c|| - r
Positive-inside convention returns:
    d = -s = r - ||x-c||

## `Polygon`

*Type*: class

Simple polygon in 2D (vertices counter-clockwise).

Parameters
----------
vertices : Array, shape (N,2)
    Polygon vertices in counter-clockwise order.

Notes
-----
- Inside test uses ray casting.
- Distance to boundary is computed as the minimum distance to polygon edges.
- Returns +distance for inside, -distance for outside (positive-inside convention).

This is intended as a practical utility rather than a high-performance kernel.
For very large N or many queries, consider vectorizing further or using a
specialized geometry library.
