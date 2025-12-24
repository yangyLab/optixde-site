# `optixde.solvers.base.etd`

## `etd1_step(y: Union[numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')], t: float, dt: float, lam: Union[float, numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')], forcing: Optional[Callable[[float], Union[numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')]]] = None, eps: float = 1e-14)`

*Type*: function

_No docstring provided._

## `phi1(z: Union[numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')], eps: float = 1e-14)`

*Type*: function

_No docstring provided._

## `solve_ode(y0: Union[numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')], t0: float, t1: float, dt: float, lam: Union[float, numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')], forcing: Optional[Callable[[float], Union[numpy.ndarray, ForwardRef('torch.Tensor'), ForwardRef('cupy.ndarray')]]] = None, eps: float = 1e-14, return_all: bool = True, stack: bool = False, save_stride: int = 1)`

*Type*: function

_No docstring provided._
