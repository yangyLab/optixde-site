# `optixde.solvers.base.poisson_inhom`

## `build_tb_coons(bc, Lx, Ly)`

*Type*: function

Coons patch lifting: 用四条边界拼一个 Tb，保证 Tb|∂Ω = bc|∂Ω
bc: full grid (Ny,Nx) with boundary values filled.

## `detect_dirichlet_sign(Lx, Ly, Nx, Ny, workers=None)`

*Type*: function

_No docstring provided._

## `laplacian_fd(u, Lx, Ly)`

*Type*: function

2nd-order FD Laplacian on uniform grid, interior only; boundary returned as 0.

## `poisson2d_dirichlet_inhom(f, Lx, Ly, bc, workers=None)`

*Type*: function

通用非齐次 Dirichlet：
  solver 的符号约定自动适配（Δu=f 或 -Δu=f）
  返回 u 满足边界 bc（四边）
