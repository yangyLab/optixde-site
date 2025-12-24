# `optixde.geometry.boolean`

optixde.geometry.csg

Constructive Solid Geometry (CSG) Boolean operators based on signed distance fields.

This module defines Boolean combinations of primitive geometries that implement
the same `Geometry` interface (signed_distance + bbox):

- Union:         Ω = Ω1 ∪ Ω2 ∪ ...
- Intersection:  Ω = Ω1 ∩ Ω2 ∩ ...
- Difference:    Ω = Ω1 \ Ω2

Signed-distance conventions (library convention)
------------------------------------------------
Assume each Geometry provides a signed distance / signed indicator-like function
d(X) such that:
- d(X) >= 0  indicates "inside" (or at least in the kept region)
- d(X) <  0  indicates "outside"
- d(X) =  0  on the boundary

With this convention, Boolean operations can be formed by simple min/max rules:
- Union:         d_union = max(d1, d2, ...)
- Intersection:  d_inter = min(d1, d2, ...)
- Difference:    d_diff  = min(d1, -d2)  (keep g1 but carve out g2)

Bounding boxes (bbox)
---------------------
Each Geometry returns an axis-aligned bounding box (lo, hi).

- Union: bbox spans all components:
      lo = min_i lo_i,  hi = max_i hi_i
- Intersection: bbox is the overlap of all components:
      lo = max_i lo_i,  hi = min_i hi_i
- Difference: bbox equals the minuend bbox (g1), since carving out does not enlarge it.

Notes for API docs
------------------
- These operators do not require meshes; they operate directly on signed_distance.
- They are suitable for building complex embedded domains for spectral solvers.

## `Union`

*Type*: class

CSG Union: Ω = Ω1 ∪ Ω2 ∪ ...

Parameters
----------
*geoms : Geometry
    At least two Geometry objects.

Attributes
----------
geoms : tuple[Geometry, ...]
    Children geometries.
dim : int
    Spatial dimension inferred from the first geometry.

Notes
-----
- Requires all children to share the same dimension and coordinate convention.
- Signed-distance combination (inside-positive convention):
      d_union = max(d_i)

## `Intersection`

*Type*: class

CSG Intersection: Ω = Ω1 ∩ Ω2 ∩ ...

Parameters
----------
*geoms : Geometry
    At least two Geometry objects.

Attributes
----------
geoms : tuple[Geometry, ...]
    Children geometries.
dim : int
    Spatial dimension inferred from the first geometry.

Notes
-----
Signed-distance combination (inside-positive convention):
    d_intersection = min(d_i)

## `Difference`

*Type*: class

CSG Difference: Ω = Ω1 \ Ω2

Keep geometry g1 and subtract geometry g2.

Parameters
----------
g1 : Geometry
    Minuend geometry (kept region).
g2 : Geometry
    Subtrahend geometry (carved-out region).

Attributes
----------
g1, g2 : Geometry
    Child geometries.
dim : int
    Spatial dimension inherited from g1.

Notes
-----
Signed-distance combination (inside-positive convention):
    d_diff = min(d1, -d2)

Interpretation:
- Points inside g1 but outside g2 remain inside.
- Points inside g2 are removed (become outside), hence the negation.
