# Post-processing

`optixde.post` provides Matplotlib-based helpers for consistent, mask-aware
scientific figures. Matplotlib is imported lazily: the numerical package can be
used without it until a plotting symbol is requested.

## Plot style

```python
PlotStyle(
    cmap="jet",
    fontsize=11,
    title_size=12,
    label_size=11,
    tick_size=10,
    cbar_tick_size=10,
    figure_dpi=300,
    tight_layout=True,
    axes_aspect="equal",
)
```

`DEFAULT_STYLE` is the module default.

```python
set_plot_style(style=DEFAULT_STYLE)
```

This updates global Matplotlib `rcParams`. Use a local Matplotlib context if
other figures in the same process require different global settings.

## Single field

```python
plot_cloud(
    U,
    *,
    ax=None,
    extent=None,
    mask=None,
    title=None,
    xlabel="x",
    ylabel="y",
    unit=None,
    cmap=None,
    mode="imshow",
    interp="nearest",
    origin="lower",
    aspect="equal",
    nlevels=64,
    robust=False,
    robust_q=(1.0, 99.0),
    vmin=None,
    vmax=None,
    symmetric=False,
    center=0.0,
    cb=True,
    cb_label=None,
    cb_ticks="minmax",
    cb_nticks=None,
    cb_format="auto",
    annotate_minmax=True,
    fontsize=None,
    save=None,
    dpi=None,
    tight=None,
)
```

`U` must be a two-dimensional array. `NaN` and infinite values are masked
automatically. If `mask` is supplied, `True` means hidden and its shape must
equal `U.shape`.

The return value is:

```text
fig, ax, mappable, colorbar_or_none, (vmin, vmax)
```

Color-scale controls:

- `robust=True` uses `robust_q` percentiles;
- `symmetric=True` expands the range symmetrically around `center`;
- explicit `vmin` and `vmax` take precedence;
- `cb_ticks` accepts `minmax`, `minmidmax`, `min0max`, `linspace`, or an
  explicit sequence.

```python
fig, ax, _, _, limits = plot_cloud(
    error,
    extent=(0.0, Lx, 0.0, Ly),
    title="Pointwise error",
    cmap="coolwarm",
    symmetric=True,
    center=0.0,
    save="error.pdf",
)
```

Matplotlib selects the output format from the extension, so `save="figure.pdf"`
creates vector PDF output.

## Field panels

```python
plot_cloud_grid(
    U_list,
    *,
    titles=None,
    extent=None,
    mask=None,
    layout=None,
    figsize=None,
    cmap=None,
    mode="imshow",
    interp="nearest",
    origin="lower",
    aspect="equal",
    nlevels=64,
    robust=False,
    robust_q=(1.0, 99.0),
    vmin=None,
    vmax=None,
    symmetric=False,
    center=0.0,
    cb_shared=True,
    cb_label=None,
    unit=None,
    cb_ticks="minmax",
    cb_nticks=None,
    cb_format="auto",
    annotate_minmax=True,
    fontsize=None,
    save=None,
    dpi=None,
    tight=None,
)
```

Returns:

```text
fig, axes, mappables, colorbar_or_none, (vmin, vmax)
```

By default, the scale is computed across all fields and a shared colorbar is
used. This is the correct choice for comparing snapshots or methods on a common
physical scale. If each panel requires its own range, call `plot_cloud`
separately on prepared axes.

## Time series

```python
plot_time_series(
    t,
    y_list,
    *,
    labels=None,
    ax=None,
    title=None,
    xlabel="t",
    ylabel="value",
    legend=True,
    grid=True,
    linewidth=1.8,
    fontsize=None,
    save=None,
    dpi=None,
    tight=None,
)
```

Returns `(fig, ax)`. `y_list` is a sequence of one-dimensional series sharing
the same time coordinate.

## Mode shapes

```python
plot_mode_shapes(
    modes,
    *,
    extent=None,
    mask=None,
    titles=None,
    normalize=True,
    layout=None,
    cmap=None,
    symmetric=True,
    center=0.0,
    robust=False,
    robust_q=(1.0, 99.0),
    cb_shared=True,
    cb_label="",
    save_prefix=None,
    dpi=None,
)
```

Each mode must be two-dimensional. With `normalize=True`, every mode is divided
by its own maximum absolute finite value before the common panel scale is
computed.

The return value is `(fig, axes, colorbar, (vmin, vmax))`. If `save_prefix` is
provided, the current implementation saves
`"{save_prefix}_modes.png"`.

## Geometry plots

```python
plot_geometry(
    geo,
    ax=None,
    *,
    title=None,
    bounds=None,
    pad=0.05,
    sample_nx=300,
    sample_ny=300,
    cmap=None,
    facecolor="tab:blue",
    edgecolor="k",
    linewidth=1.5,
    alpha=0.35,
    show_axes=True,
)

show_geometry(geo, **kwargs)
```

`plot_geometry` uses native Matplotlib patches for recognized rectangles,
disks, and polygons. Other geometries are sampled through their
`inside`/`contains` method. It returns `(fig, ax, info)`, where `info` records
the plotting mode, bounds, primitive type, and sample shape.

`show_geometry` calls `plt.show()` and returns the same tuple. Prefer
`plot_geometry` in scripts and test suites where blocking display behavior is
undesirable.

## Publication workflow

```python
from optixde.post import PlotStyle, plot_cloud, set_plot_style

set_plot_style(PlotStyle(
    cmap="viridis",
    fontsize=10,
    figure_dpi=300,
))

fig, _, _, _, _ = plot_cloud(
    u,
    extent=(xmin, xmax, ymin, ymax),
    mask=outside_mask,
    title=r"$u(x,y)$",
    cb_label="amplitude",
    annotate_minmax=False,
)
fig.savefig("solution.pdf", bbox_inches="tight")
```

For a paper, set physical extents explicitly, retain equal axis scaling for
spatial domains, use a common range for comparison panels, and close figures
after saving them in long batch jobs.
