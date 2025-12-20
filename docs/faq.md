# FAQ

## Is OptiXDE open-source?

Not yet. This site shares public-facing documentation, figures, and citations while the core repository remains private.

## Can I get access?

Yes—see **Cite & Contact** for the access request process.

## What problems does it solve?

List your current supported PDEs and planned roadmap.

## How does it compare to FEM/PINNs/Neural Operators?

Keep it simple and honest:
- OptiXDE is a **deterministic numerical solver** (no training data required).
- FFT-based operators are efficient on uniform grids.
- Embedded-domain handling is convenient for complex geometries.

Add a short limitations section:
- uniform grid (currently)
- boundary accuracy trade-offs with penalty
- sharp corners / discontinuous coefficients require care
