# Benchmarks

Put your strongest **public** evidence here: accuracy + speed + robustness.

## Suggested benchmark list

1. Periodic diffusion (analytical solution)
2. Poisson on a square (DST/DCT reference)
3. Poisson on an L-shaped embedded domain (re-entrant corner)
4. Wave propagation (phase accuracy + wave packet)

## Figure placeholders

Create a folder `docs/assets/figures/` and add your exported plots:

- `error_vs_N.png`
- `runtime_vs_N.png`
- `lshape_solution.png`
- `wave_packet.png`

Example embedding:

```md
![error](assets/figures/error_vs_N.png)
```

## Recommended table format

| Task | Grid | Runtime | Error | Notes |
|------|------|---------|-------|------|
| Diffusion | 256² | 12 ms | 1.2e-6 | periodic |
| Poisson (square) | 512² | 8 ms | 3.1e-8 | Dirichlet |
