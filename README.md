# NordslettenSummerSchool
For Graz Biomechanical Summer School

### Before Getting Started

Before you get started, make sure to install Python. To avoid contaminating your default Python setup, please use a virtual environment.

To create a virtual environment, e.g. in .venv of the current working directory, use the command
```console
python -m venv ./.venv
```
Activate your virtual environment. If you are using Windows, use
```console
.\.venv\Scripts\activate
```
If you are using Linux, macOS, or other Unix systems, use
```console
source ./venv/bin/activate
```
You can exit the virtual environment at any time by executing the command `deactivate`

## Getting Started

The NordslettenSummerSchool python module and all dependencies can be installed with `pip`.

For example, after you git clone or download the `nordsletten_summer_school` directory, you can install it with

```console
pip install ./nordsletten_summer_school
```
In case your `pip` executable is not bound to the same `python` executable, you call pip from python. I.e., you can alternatively call
```console
python -m pip install ./nordsletten_summer_school
```

If your current directory is inside `nordsletten_summer_school`, you can install the module via
```console
python -m pip install .
```

Once this is done, you are ready to go. All functions can be imported from the `biomechanics` module, i.e.,
```python
from biomechanics import *
```

See `example.py` for example.

## Content

### Kinematics

This module contains the following for generating deformation gradient data:
```python
def construct_tensor_uniaxial(stretch: NDArray[f64]) -> NDArray[f64]: pass

def construct_tensor_biaxial(
    F11: NDArray[f64] | float = 1.0,
    F12: NDArray[f64] | float = 0.0,
    F21: NDArray[f64] | float = 0.0,
    F22: NDArray[f64] | float = 1.0,
) -> NDArray[f64]: pass
```

This module also contains the following for converting the deformation gradient to other strain tensor types:
```python
def compute_right_cauchy_green(F: NDArray[f64]) -> NDArray[f64]: pass

def compute_left_cauchy_green(F: NDArray[f64]) -> NDArray[f64]: pass

def compute_green_lagrange_strain(F: NDArray[f64]) -> NDArray[f64]: pass
```

### Constitutive Models

All constitutive models are Python classes that instantiate with their material parameters and provide a method `pk2` that returns the second Piola Kirchhoff stress tensor.

This module also contains the following functions for converting to other stress tensor types:

```python
def compute_pk1_from_pk2(S: NDArray[f64], F: NDArray[f64]) -> NDArray[f64]: pass

def compute_cauchy_from_pk2(S: NDArray[f64], F: NDArray[f64]) -> NDArray[f64]: pass
```

#### Hyperelastic Constitutive Models
All hyperelastic models inherit from
```python
class HyperelasticModel(abc.ABC):

    @abc.abstractmethod
    def pk2(self, F: NDArray[f64]) -> NDArray[f64]:
        pass
```
i.e., they all have the method `pk2` for calculating the second Piola Kirchhoff stress tensor.



The list of models includes:
```python
class NeoHookeanModel(HyperelasticModel):

    def __init__(self,
        mu:float # Bulk Modulus
    ) -> None: pass
```


```python
class GuccioneModel(HyperelasticModel):

    def __init__(self,
        mu: float, # Bulk Modulus
        b_1: float, # Isotopic Exponent
        b_ff: float, # Fiber Exponent
        b_fs: float, # Fiber Shear Exponent
        b_sn: float, # Off-fiber interaction exponent
        v_f: NDArray[f64], # Unit vector for fiber direction
        v_s: NDArray[f64], # Unit vector for fiber sheet direction
    ) -> None: pass
```


```python
class CostaModel(HyperelasticModel):

    def __init__(self,
        mu: float, # Bulk modulus
        b_ff: float, # Fiber direction exponent
        b_ss: float, # Sheet direction exponent
        b_nn: float, # Normal direction exponent
        b_fs: float, # Fiber-sheet interation exponent
        b_fn: float, # Fiber-normal interation exponent
        b_sn: float, # Sheet-normal interation exponent
        v_f: NDArray[f64], # Unit vector for fiber direction
        v_s: NDArray[f64], # Unit vector for fiber sheet direction
    ) -> None: pass
```

```python
class HolzapfelOgdenModel(HyperelasticModel):

    def __init__(self,
        k_iso: float, # Isotropic part modulus
        b_iso: float, # Isotropic part exponent
        k_ff: float, # Fiber modulus
        b_ff: float, # Fiber exponent
        k_fs: float, # Fiber-sheet interaction modulus
        b_fs: float, # Fiber-sheet interaction exponent
        k_ss: float, # Sheet modulus
        b_ss: float, # Sheet exponent
        v_f: NDArray[f64], # Unit vector for fiber direction
        v_s: NDArray[f64], # Unit vector for fiber sheet direction
    ) -> None: pass
```



#### Viscoelastic Constitutive Models
All viscoelastic models inherit from
```python
class ViscoelasticModel(abc.ABC):

    @abc.abstractmethod
    def pk2(self, F: NDArray[f64], time: NDArray[f64]) -> NDArray[f64]:
        pass
```
Two fractional viscoelastic models are provided, they operate on hyperelastic laws.

```python
class FractionalVEModel(ViscoelasticModel):

    def __init__(self,
        alpha: float, # Fractional order
        Tf: float, # Periodicity
        Np: int = 9, # Number of Prony terms
        models: list[HyperelasticModel] | None = None, # Models being differentiated
    ) -> None: pass
```

```python
class FractionalDiffEqModel(ViscoelasticModel):

    def __init__(self,
        alpha: float, # Fractional order
        delta: float, # Fractional term weight
        Tf: float, # Periodicity
        Np: int = 9, # Number of Prony terms
        hyperelastic_models: list[HyperelasticModel] | None = None, # Hyperelastic Models on RHS
        viscoelastic_models: list[ViscoelasticModel] | None = None, # Viscoelastic Models on RHS
    ) -> None: pass
```


#### Hydrostatic Pressure
You can add a hydrostatic pressure with

```python
def add_hydrostatic_pressure(S: NDArray[f64], F: NDArray[f64]) -> NDArray[f64]: pass
```

### Composing Models
The following two classes are provided for composing models:
```python
class CompositeHyperelasticModel(HyperelasticModel):

    def __init__(self,
        models: list[HyperelasticModel]
    ) -> None: pass
```
```python
class CompositeViscoelasticModel(ViscoelasticModel):

    def __init__(self,
        hyperelastic_models: list[HyperelasticModel] | None = None,
        viscoelastic_models: list[ViscoelasticModel] | None = None,
    ) -> None: pass
```

For example,
```python
    fractional_holzapfel_model = FractionalVEModel(0.15, 10.0, 9, [
        HolzapfelOgdenModel(1.0, .5, 1.0, 1.0, 1.0, 0.25, 1.0, 0.5, np.array([1,0,0]), np.array([0,1,0]))
    ])
    composite_model = CompositeViscoelasticModel(
        hyperelastic_models = [
            NeoHookean(1.0),
            GuccioneModel(1.0, 0.0., 1.0, 0.5, 0.5, np.array([1,0,0]), np.array([0,1,0])),
        ],
        viscoelastic_models = [
            fractional_holzapfel_model,
        ],
    )

    stress = composite_model.pk2(F_tensor, time)
```
### Plotting

The following plot functions are provided:

```python
def plot_stress_vs_strain_1D(
    *data: tuple[NDArray[f64], NDArray[f64]] | list[NDArray[f64]], ...
) -> None: pass

def plot_stress_vs_strain_2D(
    *data: tuple[NDArray[f64], NDArray[f64]] | list[NDArray[f64]], ...
) -> None: pass

def plot_strain_vs_time_1D(
    time: NDArray[f64], *data: NDArray[f64], ...
) -> None: pass

def plot_strain_vs_time_2D(
    time: NDArray[f64], *data: NDArray[f64], ...
) -> None: pass

def plot_stress_vs_time_1D(
    time: NDArray[f64], *data: NDArray[f64], ...
) -> None: pass

def plot_stress_vs_time_2D(
    time: NDArray[f64], *data: NDArray[f64], ...
) -> None: pass
```

All plot functions have the following options:
```python
x_lim: list[float] | None = None # X plot range
y_lim: list[float] | None = None # Y plot range
figsize: tuple[float, float] = (4, 3) # figure size by inches
dpi: int = 150 # DPI
x_label: str|list[str] = r"$E$"
y_label: str|list[str] = r"$S$ (kPa)"
curve_labels: list[str] | None = None # in order create a legend with labels for each data passed in
color: list[str] | None = ["k", "r", "b", "g", "c", "m"],
alpha: list[float] | None = None,
linestyle: list[str] | None = ["-", "-", "-", "-", "-", "-"], # "-" for lines, "none" for no lines
linewidth: list[float] | None = None,
marker: list[str] | None = ["none", "none", "none", "none", "none", "none"], # "o" for markers, "none" for no markers
markersize: int | float = 4,
markerskip: int | list[int] | float | list[float] | None = None,
markeredgewidth: float = 0.3,
legendlabelcols: int = 4,
fillstyle: str = "full", # "full", "none", "top", "bottom", "left", "right"
fout: str | None = None,# "if None then figure will be displayed, if given a string, then attempt to save to str
**kwargs,# other kwargs will be pass to ax.plot
```

