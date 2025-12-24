# `optixde.post.plotting`

optixde.post.plotting
Unified plotting utilities for OptiXDE post-processing.

Design goals:
- One-call cloud plots (imshow/contourf) with a colorbar that ALWAYS shows min/max.
- Mask/NaN safe for embedded/penalty domains.
- Robust color scaling (percentiles) and symmetric scaling (error/modes).
- Grid panels with a shared colorbar for method/time comparisons.
- Time-series and modal plotting helpers.

## `plot_cloud(U: 'np.ndarray', *, ax=None, extent: 'Optional[Tuple[float, float, float, float]]' = None, mask: 'Optional[np.ndarray]' = None, title: 'Optional[str]' = None, xlabel: 'str' = 'x', ylabel: 'str' = 'y', unit: 'Optional[str]' = None, cmap: 'Optional[str]' = None, mode: 'str' = 'imshow', interp: 'str' = 'nearest', origin: 'str' = 'lower', aspect: 'Union[str, float]' = 'equal', nlevels: 'int' = 64, robust: 'bool' = False, robust_q: 'Tuple[float, float]' = (1.0, 99.0), vmin: 'Optional[Number]' = None, vmax: 'Optional[Number]' = None, symmetric: 'bool' = False, center: 'float' = 0.0, cb: 'bool' = True, cb_label: 'Optional[str]' = None, cb_ticks: 'Union[str, Sequence[Number]]' = 'minmax', cb_nticks: 'Optional[int]' = None, cb_format: 'Union[str, None]' = 'auto', annotate_minmax: 'bool' = True, fontsize: 'Optional[int]' = None, save: 'Optional[str]' = None, dpi: 'Optional[int]' = None, tight: 'Optional[bool]' = None)`

*Type*: function

Plot a 2D field as a cloud map and attach a colorbar that always shows min/max.
Returns (fig, ax, mappable, cbar_or_None, (vmin, vmax)).

## `plot_cloud_grid(U_list: 'Sequence[np.ndarray]', *, titles: 'Optional[Sequence[str]]' = None, extent: 'Optional[Tuple[float, float, float, float]]' = None, mask: 'Optional[np.ndarray]' = None, layout: 'Optional[Tuple[int, int]]' = None, figsize: 'Optional[Tuple[float, float]]' = None, cmap: 'Optional[str]' = None, mode: 'str' = 'imshow', interp: 'str' = 'nearest', origin: 'str' = 'lower', aspect: 'Union[str, float]' = 'equal', nlevels: 'int' = 64, robust: 'bool' = False, robust_q: 'Tuple[float, float]' = (1.0, 99.0), vmin: 'Optional[Number]' = None, vmax: 'Optional[Number]' = None, symmetric: 'bool' = False, center: 'float' = 0.0, cb_shared: 'bool' = True, cb_label: 'Optional[str]' = None, unit: 'Optional[str]' = None, cb_ticks: 'Union[str, Sequence[Number]]' = 'minmax', cb_nticks: 'Optional[int]' = None, cb_format: 'Union[str, None]' = 'auto', annotate_minmax: 'bool' = True, fontsize: 'Optional[int]' = None, save: 'Optional[str]' = None, dpi: 'Optional[int]' = None, tight: 'Optional[bool]' = None)`

*Type*: function

Plot multiple fields in a grid. Optionally share one colorbar (recommended for comparisons).
Returns (fig, axes, mappables, cbar_or_None, (vmin, vmax)).

## `plot_mode_shapes(modes: 'Sequence[np.ndarray]', *, extent: 'Optional[Tuple[float, float, float, float]]' = None, mask: 'Optional[np.ndarray]' = None, titles: 'Optional[Sequence[str]]' = None, normalize: 'bool' = True, layout: 'Optional[Tuple[int, int]]' = None, cmap: 'Optional[str]' = None, symmetric: 'bool' = True, center: 'float' = 0.0, robust: 'bool' = False, robust_q: 'Tuple[float, float]' = (1.0, 99.0), cb_shared: 'bool' = True, cb_label: 'str' = '', save_prefix: 'Optional[str]' = None, dpi: 'Optional[int]' = None)`

*Type*: function

Convenience wrapper: normalize modes and call plot_cloud_grid.
- normalize=True scales each mode by its max abs (so shapes are comparable)
- symmetric=True makes diverging around 0 (even with jet it helps interpret sign)

## `plot_time_series(t: 'Sequence[Number]', y_list: 'Sequence[Sequence[Number]]', *, labels: 'Optional[Sequence[str]]' = None, ax=None, title: 'Optional[str]' = None, xlabel: 'str' = 't', ylabel: 'str' = 'value', legend: 'bool' = True, grid: 'bool' = True, linewidth: 'float' = 1.8, fontsize: 'Optional[int]' = None, save: 'Optional[str]' = None, dpi: 'Optional[int]' = None, tight: 'Optional[bool]' = None)`

*Type*: function

Plot multiple time-series on a single axis.
Returns (fig, ax).

## `PlotStyle(cmap: 'str' = 'jet', fontsize: 'int' = 11, title_size: 'int' = 12, label_size: 'int' = 11, tick_size: 'int' = 10, cbar_tick_size: 'int' = 10, figure_dpi: 'int' = 300, tight_layout: 'bool' = True, axes_aspect: 'Union[str, float]' = 'equal') -&gt; None`

*Type*: class

PlotStyle(cmap: 'str' = 'jet', fontsize: 'int' = 11, title_size: 'int' = 12, label_size: 'int' = 11, tick_size: 'int' = 10, cbar_tick_size: 'int' = 10, figure_dpi: 'int' = 300, tight_layout: 'bool' = True, axes_aspect: 'Union[str, float]' = 'equal')

**`__init__`**(self, cmap: 'str' = 'jet', fontsize: 'int' = 11, title_size: 'int' = 12, label_size: 'int' = 11, tick_size: 'int' = 10, cbar_tick_size: 'int' = 10, figure_dpi: 'int' = 300, tight_layout: 'bool' = True, axes_aspect: 'Union[str, float]' = 'equal') -&gt; None

## `set_plot_style(style: 'PlotStyle' = PlotStyle(cmap='jet', fontsize=11, title_size=12, label_size=11, tick_size=10, cbar_tick_size=10, figure_dpi=300, tight_layout=True, axes_aspect='equal')) -&gt; 'None'`

*Type*: function

Apply a consistent plotting style.
