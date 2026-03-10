# Method

## Core idea

OptiXDE solves differential equations from an operator viewpoint on an FFT-compatible uniform grid.  
Instead of generating a mesh and assembling stiffness matrices, the method represents the unknown field directly on a Cartesian grid and advances the solution through spectral propagation.

The workflow is:

1. represent the field on a uniform grid;
2. apply the FFT to move the field into frequency space;
3. update each frequency mode with a propagation operator or Green’s operator derived from the PDE;
4. apply the inverse FFT to recover the updated field in physical space;
5. enforce geometry and boundary conditions through masks and penalty operators when the physical domain is embedded in a larger Cartesian box.

In this way, the dominant computation is reduced to FFTs plus pointwise multiplications, giving a matrix-free and reusable solver structure.

## Spectral propagation viewpoint

For a linear constant-coefficient PDE, spatial derivatives become algebraic multipliers in Fourier space.  
If \(u(\mathbf{x},t)\) is the field and \(\hat u(\mathbf{k},t)\) is its Fourier transform, then the PDE is converted into a modewise update of the form

\[
\hat u^{\,n+1}(\mathbf{k}) = G(\mathbf{k},\Delta t)\,\hat u^{\,n}(\mathbf{k}),
\]

where \(G(\mathbf{k},\Delta t)\) is the propagation operator associated with the target equation.  
The physical solution is then recovered by

\[
u^{n+1}(\mathbf{x})=\mathcal{F}^{-1}\!\left[G(\mathbf{k},\Delta t)\,\mathcal{F}[u^n(\mathbf{x})]\right].
\]

This gives a unified transform--propagate--inverse-transform backbone for different PDE types.

## Embedded geometry and boundary handling

To handle non-rectangular domains while keeping FFT efficiency, the physical domain \(\Omega\) is embedded into a larger rectangular computational domain \(\tilde{\Omega}\).  
Geometry is represented by a binary or smoothed mask field on the same uniform grid.

A correction or enforcement operator is then applied in physical space:

\[
u^{n+1}=\mathcal{E}\!\left(\mathcal{T}_G[u^n]\right),
\]

where \(\mathcal{T}_G\) is the spectral propagation step and \(\mathcal{E}\) enforces masks, penalties, and boundary constraints.

At a high level:

- the FFT-based propagation is geometry-agnostic;
- the mask identifies the valid physical region;
- penalty terms weakly impose boundary conditions near embedded interfaces.

This separation allows the same spectral core to be reused across regular and irregular domains.

## Stability and time stepping

For diffusion-type linear problems, the spectral propagator integrates each Fourier mode analytically, so the linear propagation step is unconditionally stable.  
For wave-type problems, the same framework preserves the correct modal evolution through exact or near-exact propagators.  
For nonlinear or mixed problems, OptiXDE can be interpreted as an operator-splitting method, where the stiff linear part is handled spectrally and the remaining terms are updated in physical space.

## Computational characteristics

Because each update consists mainly of FFTs and elementwise multiplications, the per-step cost is

\[
\mathcal{O}(N\log N),
\]

where \(N\) is the number of grid degrees of freedom.  
This makes the method naturally compatible with vectorized CPU execution and standard GPU FFT backends.

## Advantages

- no mesh generation or element integration;
- no stiffness-matrix assembly;
- unified operator form for multiple PDE classes;
- efficient FFT-based implementation;
- straightforward CPU/GPU acceleration;
- practical handling of embedded irregular geometries.

## Current limitations

The method is most natural for uniform Cartesian grids and linear constant-coefficient operators.  
For irregular domains, errors are often dominated by the embedded boundary treatment rather than by the spectral kernel itself.  
Strongly nonlinear problems, variable coefficients, and sharp interface physics require more careful splitting, stabilization, and enforcement design.

## Outlook

Future development can focus on:

- sharper boundary-penalty design;
- more accurate embedded-interface treatment;
- variable-coefficient and nonlinear extensions;
- adaptive resolution strategies;
- tighter integration with operator-learning pipelines.

## Diagram

![pipeline](assets/pipeline.svg){ width="900" }
