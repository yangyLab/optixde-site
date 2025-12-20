# Method

## Core idea (public-safe)

Explain in **conceptual** terms:

- Represent the field on a uniform grid.
- Use FFT to move to frequency space.
- Apply a propagation / Green's operator per frequency.
- Inverse FFT to update the field.
- Use masks + penalties to enforce embedded boundary conditions.

## Diagram

![pipeline](assets/pipeline.svg){ width="900" }

## Notes you can add

- stability constraints (if any)
- time stepping / operator splitting view
- how boundary penalties are designed (high level)
- limitations and future work
