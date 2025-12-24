# `optixde.post.plotting`

optixde.post.plotting
Unified plotting utilities for OptiXDE post-processing.

Design goals
------------
- One-call cloud plots (imshow/contourf) with a colorbar that ALWAYS shows min/max.
- Mask/NaN safe for embedded/penalty domains.
- Robust color scaling (percentiles) and symmetric scaling (error/modes).
- Grid panels with a shared colorbar for method/time comparisons.
- Time-series and modal plotting helpers.

Conventions
-----------
- Input fields are assumed to be 2D arrays (Ny, Nx) for cloud plots.
- `mask` is a boolean array with the SAME shape as U:
    mask==True means "hide" (outside Ω or invalid region).
- NaN/Inf values are automatically masked.

Defaults
--------
- Colormap defaults to "jet" (project preference).
- axes_aspect defaults to "equal" for spatial fields.

## `PlotStyle`

*Type*: class

Global plotting style configuration.

Attributes
----------
cmap : str
    Matplotlib colormap name. Default "jet".
fontsize : int
    Base font size for labels/text.
title_size : int
    Title font size.
label_size : int
    Axis label font size.
tick_size : int
    Tick label font size.
cbar_tick_size : int
    Colorbar tick label font size.
figure_dpi : int
    DPI for figures (screen and saved files).
tight_layout : bool
    If True, call fig.tight_layout() in high-level functions.
axes_aspect : str or float
    Default aspect for spatial plots ("equal" recommended).

## `set_plot_style(style)`

*Type*: function

Apply a consistent plotting style to matplotlib rcParams.

Parameters
----------
style : PlotStyle
    Style object that controls global matplotlib defaults.

Notes
-----
- This function sets global rcParams; if you need local-only styling, adjust
  the returned fig/ax objects after calling plot functions.

## `plot_cloud(U, ax, extent, mask, title, xlabel, ylabel, unit, cmap, mode, interp, origin, aspect, nlevels, robust, robust_q, vmin, vmax, symmetric, center, cb, cb_label, cb_ticks, cb_nticks, cb_format, annotate_minmax, fontsize, save, dpi, tight)`

*Type*: function

Plot a single 2D field as a cloud map (imshow/contourf) with a colorbar.

This is the "one-call" plotting function used throughout OptiXDE examples.

Parameters
----------
U : np.ndarray, shape (Ny, Nx)
    Field values to plot. NaN/Inf values are automatically ignored.
ax : matplotlib.axes.Axes or None
    If provided, draw into this axis; otherwise create a new figure.
extent : (xmin, xmax, ymin, ymax) or None
    If provided, sets physical coordinate extent for imshow/contourf.
    If None, pixel coordinates are used.
mask : np.ndarray[bool] or None, shape (Ny, Nx)
    Boolean mask where True indicates "hide" (e.g., outside embedded domain).
title, xlabel, ylabel : str
    Plot annotations.
unit : str or None
    Unit string to show on the colorbar title (if cb_label not provided).
cmap : str or None
    Matplotlib colormap. Defaults to DEFAULT_STYLE.cmap ("jet").
mode : {"imshow","contourf"}
    Rendering mode. "imshow" is faster; "contourf" produces filled contours.
interp : str
    Interpolation for imshow (e.g., "nearest", "bilinear").
origin : str
    "lower" recommended for x-y fields.
aspect : str or float
    Axis aspect ratio ("equal" for spatial fields).
nlevels : int
    Number of contour levels when mode="contourf".
robust : bool
    If True, determine vmin/vmax using percentiles robust_q (good for outliers).
robust_q : (q0,q1)
    Percentiles for robust scaling (default 1–99%).
vmin, vmax : float or None
    Explicit color scale bounds. If None, computed automatically.
symmetric : bool
    If True, enforce symmetric colormap range around `center`. Useful for errors/modes.
center : float
    Center used when symmetric=True (default 0).
cb : bool
    If True, show a colorbar.
cb_label : str or None
    Colorbar title. If None, use `unit` (if provided).
cb_ticks : str or sequence
    Tick strategy; see _ensure_minmax_ticks for options.
cb_nticks : int or None
    Used when cb_ticks="linspace".
cb_format : "auto", printf-str, or None
    Tick label formatting.
annotate_minmax : bool
    If True, write explicit "min=..., max=..." next to colorbar.
fontsize : int or None
    Base font size. Defaults to DEFAULT_STYLE.fontsize.
save : str or None
    If provided, save figure to this path.
dpi : int or None
    Save DPI. Defaults to DEFAULT_STYLE.figure_dpi.
tight : bool or None
    If True, call fig.tight_layout().

Returns
-------
fig : matplotlib.figure.Figure
ax : matplotlib.axes.Axes
mappable : matplotlib.cm.ScalarMappable
    The object used to create the colorbar (imshow image or contour set).
cbar : matplotlib.colorbar.Colorbar or None
(vmin, vmax) : tuple[float,float]
    Final color scale bounds actually used.

Examples
--------
>>> fig, ax, _, _, _ = plot_cloud(u, extent=(0,Lx,0,Ly), title="u", unit="m")
>>> fig.savefig("u.png", dpi=300)
