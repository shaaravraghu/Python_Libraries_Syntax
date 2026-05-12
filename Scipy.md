# SciPy Production Technical Reference

## Contents
- [1. Introduction & History](#1-introduction--history)
- [2. Installation & Setup](#2-installation--setup)
- [3. SciPy Module Overview](#3-scipy-module-overview)
- [4. Linear Algebra (`scipy.linalg`)](#4-linear-algebra-scipylinalg)
- [5. Optimization (`scipy.optimize`)](#5-optimization-scipyoptimize)
- [6. Integration (`scipy.integrate`)](#6-integration-scipyintegrate)
- [7. Interpolation (`scipy.interpolate`)](#7-interpolation-scipyinterpolate)
- [8. Fourier Transforms (`scipy.fft`)](#8-fourier-transforms-scipyfft)
- [9. Signal Processing (`scipy.signal`)](#9-signal-processing-scipysignal)
- [10. Statistics (`scipy.stats`)](#10-statistics-scipystats)
- [11. Sparse Matrices (`scipy.sparse`)](#11-sparse-matrices-scipysparse)
- [12. Image Processing (`scipy.ndimage`)](#12-image-processing-scipyndimage)
- [13. Distance & Spatial Algorithms](#13-distance--spatial-algorithms)
- [14. Performance Optimization](#14-performance-optimization)
- [15. Interoperability](#15-interoperability)
- [16. Real-World Applications](#16-real-world-applications)
- [17. Common Pitfalls](#17-common-pitfalls)
- [18. Advanced Usage](#18-advanced-usage)
- [19. Internals Overview](#19-internals-overview)
- [20. Advanced Topics](#20-advanced-topics)

## 1. INTRODUCTION & HISTORY

### What is SciPy
SciPy (pronounced "Sigh Pie") is an open-source Python library used for scientific and technical computing. It is a core package in the Python scientific ecosystem, providing a vast collection of mathematical algorithms and convenience functions built on top of NumPy. 

### Why SciPy (scientific computing, advanced math functions)
While Python is a general-purpose language, it lacks built-in support for advanced mathematical operations. SciPy fills this gap by offering robust, efficient, and well-tested implementations of standard scientific algorithms.
- **Scientific Computing**: It provides tools for numerical integration, optimization, interpolation, signal processing, linear algebra, and more.
- **Advanced Math Functions**: SciPy includes special mathematical functions (e.g., Bessel, Gamma, Error functions) essential for physics and engineering.
- **Performance**: Many underlying algorithms are implemented in compiled languages like C, C++, and Fortran (e.g., BLAS, LAPACK, MINPACK), ensuring high performance.

### Relationship with NumPy
SciPy is intimately connected to NumPy. 
- NumPy provides the foundational multi-dimensional array data structure (`ndarray`) and basic mathematical operations.
- SciPy extends NumPy by adding a collection of algorithms and high-level commands for manipulating and visualizing data.
- **Rule of Thumb**: NumPy is for basic array operations and basic linear algebra/Fourier transforms. SciPy is for fully-featured, specialized mathematical and scientific computing.

### Ecosystem role (data science, engineering, research)
SciPy is a pillar of the Python scientific ecosystem (often called the SciPy Stack).
- **Data Science**: Provides the mathematical foundation for libraries like Scikit-Learn (machine learning) and Statsmodels (statistical modeling).
- **Engineering**: Used extensively in control theory, signal processing, and computer-aided design.
- **Research**: Widely adopted in physics, biology, chemistry, and other empirical sciences for data analysis and simulation.

### Module-based architecture overview
SciPy is organized into sub-packages covering different scientific computing domains:
- `scipy.cluster`: Hierarchical clustering, vector quantization, K-means
- `scipy.constants`: Physical constants and conversion factors
- `scipy.fft`: Discrete Fourier Transform algorithms
- `scipy.integrate`: Numerical integration routines
- `scipy.interpolate`: Interpolation tools
- `scipy.io`: Data input and output
- `scipy.linalg`: Linear algebra routines
- `scipy.ndimage`: Various functions for multi-dimensional image processing
- `scipy.odr`: Orthogonal distance regression
- `scipy.optimize`: Optimization and root-finding routines
- `scipy.signal`: Signal processing tools
- `scipy.sparse`: Sparse matrices and associated routines
- `scipy.spatial`: Spatial data structures and algorithms
- `scipy.special`: Any special mathematical functions
- `scipy.stats`: Statistical distributions and functions

### Version evolution
SciPy has evolved significantly since its inception in 2001. 
- **1.0.0 (2017)**: A major milestone marking stability and maturity.
- **1.4.0+**: Introduction of the new `scipy.fft` module replacing `scipy.fftpack`.
- **1.8.0+**: Dropped support for older Python versions, embracing modern C++ standards and optimizing many legacy Fortran codebases.

---

## 2. INSTALLATION & SETUP

### pip install scipy
The easiest way to install SciPy is via `pip` or `conda`.

```bash
# Standard installation
pip install scipy

# For the entire SciPy stack (NumPy, SciPy, Matplotlib, Pandas, IPython)
pip install numpy scipy matplotlib ipython jupyter pandas sympy nose

# Using conda (often preferred for scientific packages due to complex C/Fortran dependencies)
conda install scipy
```

### Dependencies (NumPy)
SciPy strongly depends on NumPy. Installing SciPy will automatically install a compatible version of NumPy.
- **Compiler Requirements**: Compiling SciPy from source requires C, C++, and Fortran compilers (e.g., GCC, gfortran), as well as BLAS and LAPACK development headers. It is highly recommended to use pre-built binary wheels via `pip` or `conda`.

### Import conventions
By convention, SciPy sub-modules are imported individually. The base `scipy` namespace does not automatically import all sub-modules to keep the namespace clean and loading times fast.

```python
# GOOD: Import specific sub-modules
import numpy as np
from scipy import linalg, optimize, integrate

# AVOID: Importing everything (pollutes namespace and is slow)
# from scipy import *
```

### Checking installation
You can verify the installation and check the configuration of underlying linear algebra libraries (BLAS/LAPACK).

```python
import scipy

# Check version
print(f"SciPy Version: {scipy.__version__}")

# Check underlying BLAS/LAPACK configuration
scipy.show_config()
```

> 💡 **Pro Tip**: Always verify `scipy.show_config()` in a production environment to ensure SciPy is linked against optimized BLAS/LAPACK libraries (like OpenBLAS or Intel MKL). Failing to do so can result in 10x-100x slower linear algebra operations.

> ⚠️ **Common Pitfall**: Mixing `pip` and `conda` installations for NumPy and SciPy can lead to binary incompatibilities and segmentation faults. Stick to one package manager per environment.

---

## 3. SCIPY MODULE OVERVIEW

This section provides a high-level overview of the most commonly used sub-packages in SciPy.

### scipy.linalg
Provides advanced linear algebra routines. While `numpy.linalg` exists, `scipy.linalg` contains all the functions in `numpy.linalg` plus many more advanced tools (e.g., Schur decomposition, generalized eigenvalues) and is always compiled with BLAS/LAPACK support, making it faster.

### scipy.optimize
Provides algorithms for function minimization (scalar or multi-dimensional), curve fitting, and root finding. It supports constrained and unconstrained optimization.

### scipy.integrate
Contains tools for evaluating definite integrals (numerical integration or quadrature) and for solving ordinary differential equations (ODEs).

### scipy.interpolate
Provides objects and functions to interpolate data, meaning estimating values between known data points. Supports 1-D, 2-D, and multi-dimensional interpolation using splines or polynomials.

### scipy.fft
The modern module for computing Discrete Fourier Transforms (DFTs). It provides fast algorithms for 1-D, 2-D, and N-D transforms.

### scipy.signal
Contains tools for signal processing, including convolution, 1-D and 2-D filtering, filter design (e.g., Butterworth, Chebyshev), and spectral analysis.

### scipy.stats
A massive module containing continuous and discrete probability distributions, statistical tests (e.g., T-test, ANOVA), and descriptive statistics.

### scipy.sparse
Provides 2-D sparse matrix classes (e.g., CSR, CSC, COO) and functions to perform linear algebra on them. Essential for problems involving huge matrices where most elements are zero (e.g., graph algorithms, NLP).

### scipy.ndimage
Provides functions for N-dimensional image processing, such as filtering (Gaussian, median), morphology (erosion, dilation), and interpolation (rotation, affine transforms).

---

## 4. LINEAR ALGEBRA (scipy.linalg)

`scipy.linalg` is preferred over `numpy.linalg` for its comprehensiveness and guaranteed compilation against accelerated BLAS/LAPACK.

### Matrix operations
Basic operations include finding the determinant, inverse, and norm of matrices.

```python
import numpy as np
from scipy import linalg

# Create a sample 2D square matrix
A = np.array([[1, 2], [3, 4]])

# Compute Determinant
det_A = linalg.det(A)
print(f"Determinant: {det_A}")

# Compute Inverse
# Note: For solving systems, linalg.solve is vastly preferred over multiplying by inverse
inv_A = linalg.inv(A)
print(f"Inverse:\n{inv_A}")

# Compute Matrix Norm
norm_A = linalg.norm(A, ord='fro') # Frobenius norm
print(f"Frobenius Norm: {norm_A}")

# Matrix Exponential (e^A) - Used heavily in differential equations and control theory
exp_A = linalg.expm(A)
print(f"Matrix Exponential:\n{exp_A}")
```

### Decompositions (LU, QR, SVD)
Matrix decompositions break down a matrix into simpler, constituent matrices, making complex operations easier.

```python
# LU Decomposition: A = P * L * U
# P is permutation, L is lower triangular, U is upper triangular
P, L, U = linalg.lu(A)
print("LU Decomposition:")
print(f"P:\n{P}\nL:\n{L}\nU:\n{U}")

# QR Decomposition: A = Q * R
# Q is orthogonal, R is upper triangular. Often used in least squares.
Q, R = linalg.qr(A)
print("QR Decomposition:")
print(f"Q:\n{Q}\nR:\n{R}")

# Singular Value Decomposition (SVD): A = U * Sigma * V^H
# Essential for PCA, data compression, and pseudo-inverse
U, s, Vh = linalg.svd(A)
print("SVD:")
print(f"U:\n{U}\nSingular Values:\n{s}\nVh:\n{Vh}")

# Reconstruct A from SVD (demonstration)
Sigma = linalg.diagsvd(s, A.shape[0], A.shape[1])
A_reconstructed = U @ Sigma @ Vh
print(f"Reconstructed A:\n{A_reconstructed}")
```

### Solving linear systems
To solve equations of the form `Ax = b`.

```python
# System:
# 3x + 2y = 2
#  x -  y = 4
A = np.array([[3, 2], [1, -1]])
b = np.array([2, 4])

# Solve for x
# This is faster and more numerically stable than x = inv(A) @ b
x = linalg.solve(A, b)
print(f"Solution x: {x}")

# Verify solution
print(f"A @ x (should equal b): {A @ x}")
```

### Eigenvalues and eigenvectors
Solving the equation `Ax = lambda * x`.

```python
A = np.array([[1, 2], [2, 1]])

# Compute eigenvalues and right eigenvectors
eigenvalues, eigenvectors = linalg.eig(A)

print(f"Eigenvalues: {eigenvalues}")
print(f"Eigenvectors:\n{eigenvectors}")

# If you know the matrix is symmetric/Hermitian, use eigh for better performance and guaranteed real eigenvalues
eigenvalues_sym, eigenvectors_sym = linalg.eigh(A)
print(f"Eigenvalues (Symmetric): {eigenvalues_sym}")
```

> 💡 **Pro Tip**: Always use `linalg.solve(A, b)` instead of `linalg.inv(A) @ b`. Matrix inversion is computationally expensive (O(n^3)) and prone to catastrophic numerical instability due to floating-point errors. `solve` uses specialized, highly optimized LAPACK routines.

> ⚠️ **Common Pitfall**: Passing ill-conditioned matrices to solvers. If the condition number (`np.linalg.cond(A)`) is very high, the solution will be highly sensitive to small errors. Consider using regularization or least-squares approximations in such cases.

---

## 5. OPTIMIZATION (scipy.optimize)

The `optimize` module is a powerhouse for finding minima, roots, or fitting curves to data.

### Minimization (minimize)
Finding the lowest point of a scalar function of one or more variables.

```python
from scipy.optimize import minimize

# 1. Define the objective function
# Let's minimize the Rosenbrock function: f(x, y) = (1-x)^2 + 100*(y-x^2)^2
# It has a global minimum at (1, 1) where f(x,y) = 0.
def rosenbrock(vars):
    x, y = vars
    return (1 - x)**2 + 100 * (y - x**2)**2

# 2. Initial guess
initial_guess = np.array([0.0, 0.0])

# 3. Call minimize
# 'Nelder-Mead' is a robust gradient-free method.
# For smooth functions, 'BFGS' or 'L-BFGS-B' (which uses gradients) are faster.
result = minimize(rosenbrock, initial_guess, method='BFGS')

print(f"Optimization Status: {result.message}")
print(f"Minimum found at: {result.x}")
print(f"Function value at minimum: {result.fun}")
print(f"Number of iterations: {result.nit}")
```

### Root finding (root, fsolve)
Finding where a function evaluates to zero (i.e., solving non-linear equations).

```python
from scipy.optimize import root, fsolve
import math

# Define a system of non-linear equations
# Eq 1: x + y^2 = 4
# Eq 2: e^x + xy = 3
def equations(vars):
    x, y = vars
    eq1 = x + y**2 - 4
    eq2 = np.exp(x) + x*y - 3
    return [eq1, eq2]

initial_guess = [1.0, 1.0]

# Using fsolve (legacy but very common wrapper around MINPACK's hybrd)
sol_fsolve = fsolve(equations, initial_guess)
print(f"Root found by fsolve: {sol_fsolve}")

# Using root (newer, unified interface)
sol_root = root(equations, initial_guess, method='hybr')
print(f"Root found by root: {sol_root.x}")
```

### Curve fitting (curve_fit)
Fitting a parameterized mathematical function to experimental data using non-linear least squares.

```python
from scipy.optimize import curve_fit
import matplotlib.pyplot as plt

# 1. Define the model function to fit
# The first argument must be the independent variable, followed by parameters to fit
def model_func(x, a, b, c):
    return a * np.exp(-b * x) + c

# 2. Generate dummy data with noise
x_data = np.linspace(0, 4, 50)
y_true = model_func(x_data, 2.5, 1.3, 0.5)
# Add normally distributed noise
np.random.seed(42)
y_noise = 0.2 * np.random.normal(size=x_data.size)
y_data = y_true + y_noise

# 3. Perform the curve fit
# popt: optimal values for parameters [a, b, c]
# pcov: estimated covariance of popt
popt, pcov = curve_fit(model_func, x_data, y_data)

print(f"Fitted parameters: a={popt[0]:.3f}, b={popt[1]:.3f}, c={popt[2]:.3f}")

# (Optional) Calculate standard deviations of parameters
perr = np.sqrt(np.diag(pcov))
print(f"Parameter standard deviations: {perr}")

# You would normally plot this using matplotlib to visually verify the fit
```

### Constraints handling
Optimizing functions where variables must obey certain boundaries or equations.

```python
# Objective: Minimize f(x,y) = (x-1)^2 + (y-2.5)^2
def objective(x):
    return (x[0] - 1)**2 + (x[1] - 2.5)**2

# Constraints:
# 1. Inequality constraint: x - 2y + 2 >= 0
# 2. Inequality constraint: -x - 2y + 6 >= 0
# 3. Inequality constraint: -x + 2y + 2 >= 0
# 4. Bounds: x >= 0, y >= 0
cons = ({'type': 'ineq', 'fun': lambda x:  x[0] - 2 * x[1] + 2},
        {'type': 'ineq', 'fun': lambda x: -x[0] - 2 * x[1] + 6},
        {'type': 'ineq', 'fun': lambda x: -x[0] + 2 * x[1] + 2})

bnds = ((0, None), (0, None))

# SLSQP (Sequential Least SQuares Programming) handles constraints and bounds well
res = minimize(objective, [2, 0], method='SLSQP', bounds=bnds, constraints=cons)

print(f"Constrained minimum at: {res.x}")
```

> 💡 **Pro Tip**: If your optimization is slow or failing to converge, provide the Jacobian (gradient) of your objective function explicitly using the `jac` argument in `minimize`. Finite-difference approximations of the Jacobian (the default) are slow and sensitive to scaling.

> ⚠️ **Common Pitfall**: Using local optimization methods (like BFGS) on non-convex functions (functions with multiple valleys). The optimizer will get stuck in the nearest local minimum, which may not be the global minimum. Use global optimizers (e.g., `differential_evolution`, `basinhopping`, `shgo`) for multi-modal landscapes.

---

## 6. INTEGRATION (scipy.integrate)

Tools to compute definite integrals and solve differential equations.

### Numerical integration (quad, dblquad)
Computing the area under a curve.

```python
from scipy.integrate import quad, dblquad, tplquad

# 1. Define the integrand function
def integrand(x):
    return np.exp(-x**2)

# 2. Compute the 1D integral using Gaussian quadrature
# Limits from 0 to infinity
# quad returns the integral value and an estimate of the absolute error
result, error = quad(integrand, 0, np.inf)

print(f"1D Integral Result: {result}")
print(f"Estimated Error: {error}")

# Double integration
# Integrate f(x, y) = x * y over x in [0, 1] and y in [0, 1-x]
def integrand_2d(y, x): # Note the order of arguments: y first, then x
    return x * y

# The boundaries for y can be functions of x
def bounds_y_low(x):
    return 0
def bounds_y_high(x):
    return 1 - x

res_2d, err_2d = dblquad(integrand_2d, 0, 1, bounds_y_low, bounds_y_high)
print(f"2D Integral Result: {res_2d}")
```

### ODE solvers (odeint, solve_ivp)
Solving Initial Value Problems (IVPs) for Ordinary Differential Equations (ODEs). `solve_ivp` is the modern, preferred method, replacing the older `odeint`.

```python
from scipy.integrate import solve_ivp

# Define the ODE system: dy/dt = f(t, y)
# Let's model a simple exponential decay: dy/dt = -k * y
def decay_model(t, y, k):
    return -k * y

# Parameters
k = 0.5
y0 = [10.0] # Initial condition (must be an array-like)
t_span = (0, 10) # Time interval to solve over
t_eval = np.linspace(0, 10, 100) # Specific time points to evaluate the solution at

# Solve the IVP
# 'RK45' is the default Explicit Runge-Kutta method, good for non-stiff problems
solution = solve_ivp(decay_model, t_span, y0, args=(k,), t_eval=t_eval, method='RK45')

print(f"Solver Status: {solution.message}")
print(f"Time points evaluated: {solution.t[:5]}...")
print(f"Solution y values: {solution.y[0][:5]}...")
```

### Practical examples
Modeling a pendulum (a classic non-linear ODE system).

```python
# Pendulum physics:
# theta'' + (g/L)*sin(theta) = 0
# To solve as first-order ODEs, define y = [theta, omega]
# dy/dt = [omega, -(g/L)*sin(theta)]

def pendulum(t, y, g, L):
    theta, omega = y
    dydt = [omega, -(g/L) * np.sin(theta)]
    return dydt

g = 9.81
L = 1.0
y0 = [np.pi/4, 0.0] # Initial angle 45 deg, initial angular velocity 0
t_span = (0, 10)

# Solve
sol_pendulum = solve_ivp(pendulum, t_span, y0, args=(g, L), dense_output=True)

# evaluate at arbitrary point using dense_output
t_arbitrary = 4.2
print(f"State at t={t_arbitrary}: {sol_pendulum.sol(t_arbitrary)}")
```

> 💡 **Pro Tip**: Use `solve_ivp` instead of the legacy `odeint`. `solve_ivp` provides a cleaner object-oriented interface, handles complex state types better, and offers multiple underlying solver algorithms (RK45, Radau, BDF).

> ⚠️ **Common Pitfall**: Using Explicit solvers (like `RK45` or `RK23`) on "Stiff" equations. Stiff equations (where some variables change vastly faster than others, e.g., chemical kinetics) will cause explicit solvers to take microscopic time steps, taking forever to compute. If your solver is stalling, switch to `method='Radau'` or `method='BDF'`.

---

## 7. INTERPOLATION (scipy.interpolate)

Used to construct new data points within the range of a discrete set of known data points.

### interp1d
The classic 1-D interpolation class.

```python
from scipy.interpolate import interp1d

# Generate some sparse experimental data
x_known = np.linspace(0, 10, num=11, endpoint=True)
y_known = np.cos(-x_known**2/9.0)

# Create interpolation functions
# 'linear' connects points with straight lines
f_linear = interp1d(x_known, y_known, kind='linear')
# 'cubic' fits cubic splines, resulting in a smooth curve
f_cubic = interp1d(x_known, y_known, kind='cubic')

# Generate a denser set of x values to evaluate the interpolants
x_new = np.linspace(0, 10, num=50, endpoint=True)
y_linear = f_linear(x_new)
y_cubic = f_cubic(x_new)

print("Linear Interpolation at x=1.5:", f_linear(1.5))
print("Cubic Interpolation at x=1.5:", f_cubic(1.5))
```

### splines
For more control over spline interpolation, use the object-oriented spline classes.

```python
from scipy.interpolate import UnivariateSpline, make_interp_spline

# UnivariateSpline can perform smoothing (doesn't have to pass exactly through data points)
# s is the smoothing factor. s=0 forces passing through all points.
spl = UnivariateSpline(x_known, y_known, s=0)
y_spline = spl(x_new)

# B-spline representation (modern approach)
# Requires data to be strictly increasing
bspline = make_interp_spline(x_known, y_known, k=3) # k=3 is cubic
y_bspline = bspline(x_new)

print("B-Spline evaluation:", y_bspline[:5])
```

### Multivariate interpolation
Interpolating data on a 2D grid or unstructured data points.

```python
from scipy.interpolate import griddata, RegularGridInterpolator

# --- Regular Grid (Data is on a structured mesh) ---
# Define a grid
x_grid = np.linspace(0, 5, 10)
y_grid = np.linspace(0, 5, 10)
X, Y = np.meshgrid(x_grid, y_grid)
Z = np.sin(X) * np.cos(Y)

# Create the interpolator object
# Note: RegularGridInterpolator expects points as a tuple of 1D arrays
interpolator = RegularGridInterpolator((x_grid, y_grid), Z)

# Evaluate at a new point (x=2.5, y=3.5)
val = interpolator((2.5, 3.5))
print(f"Value at (2.5, 3.5) on regular grid: {val}")

# --- Unstructured Data (Scattered points) ---
points = np.random.rand(100, 2) * 5 # 100 random 2D coordinates
values = np.sin(points[:, 0]) * np.cos(points[:, 1])

# We want to interpolate this unstructured data onto a regular grid for visualization
grid_x, grid_y = np.mgrid[0:5:100j, 0:5:100j]

# 'cubic' uses Clough-Tocher 2D interpolator
grid_z = griddata(points, values, (grid_x, grid_y), method='cubic')
```

> 💡 **Pro Tip**: `interp1d` is considered legacy. For new code, the SciPy documentation recommends using `scipy.interpolate.make_interp_spline` for 1D data or `RegularGridInterpolator` for gridded N-D data.

> ⚠️ **Common Pitfall**: Extrapolation. Interpolation is only mathematically sound *within* the bounds of your known data (the convex hull). Passing values outside this range to most SciPy interpolators will raise a `ValueError` or return `NaN` unless you explicitly enable extrapolation (e.g., `fill_value="extrapolate"` in `interp1d`), but extrapolated values are highly unreliable and can diverge wildly.

---

## 8. FOURIER TRANSFORMS (scipy.fft)

Fourier transforms decompose a function or signal into its constituent frequencies.

### FFT basics
The Fast Fourier Transform (FFT) algorithm efficiently computes the Discrete Fourier Transform (DFT).

```python
from scipy.fft import fft, ifft

# Create a sample signal: A sum of two sine waves
# Signal = 3*sin(2*pi*10*t) + sin(2*pi*20*t)
N = 600 # Number of sample points
T = 1.0 / 800.0 # Sample spacing (800 Hz sampling rate)
t = np.linspace(0.0, N*T, N, endpoint=False)
y = 3 * np.sin(50.0 * 2.0*np.pi*t) + 0.5*np.sin(80.0 * 2.0*np.pi*t)

# Compute the 1D FFT
yf = fft(y)

# yf is an array of complex numbers representing magnitude and phase.
print("First 5 complex coefficients:", yf[:5])

# Compute the Inverse FFT to recover the original signal
y_recovered = ifft(yf)
# Use np.allclose to handle minor floating point inaccuracies
print("Signal recovered successfully:", np.allclose(y, y_recovered.real))
```

### Signal transformation & Frequency analysis
To make sense of the FFT output, we need to map the coefficients to their corresponding physical frequencies.

```python
from scipy.fft import fftfreq, rfft, rfftfreq

# Calculate the frequencies corresponding to the FFT output array
# xf contains the frequency in Hz for each bin in yf
xf = fftfreq(N, T)[:N//2] # Only take positive frequencies

# Since our input signal 'y' is entirely real-valued, the FFT output is symmetric.
# We only need the positive frequencies.
# Calculate magnitude (absolute value) of the complex coefficients
magnitude = 2.0/N * np.abs(yf[0:N//2])

# Find the peak frequencies
peak_indices = np.argsort(magnitude)[-2:] # Get indices of top 2 magnitudes
print("Detected Frequencies (Hz):", xf[peak_indices])

# --- Better Approach for Real Signals ---
# If your input data is purely real, use rfft (Real FFT). It is faster and automatically
# returns only the positive frequency half of the spectrum.
yf_real = rfft(y)
xf_real = rfftfreq(N, T)
magnitude_real = 2.0/N * np.abs(yf_real)
```

> 💡 **Pro Tip**: Always prefer `scipy.fft.rfft` over `scipy.fft.fft` when dealing with physical sensor data, audio, or any purely real-valued array. It is computationally cheaper and avoids the clutter of mirror-image negative frequencies.

> ⚠️ **Common Pitfall**: Spectral Leakage. If your sampled data doesn't contain an integer number of wave periods, the FFT energy will "leak" into adjacent frequency bins, blurring the results. Apply a window function (e.g., Hann or Hamming window from `scipy.signal.windows`) before computing the FFT to mitigate this.

---

## 9. SIGNAL PROCESSING (scipy.signal)

Tools designed for digital signal processing, analyzing and modifying time-series or spatial data.

### Filtering
Designing and applying digital filters to remove noise or isolate frequency bands.

```python
from scipy import signal

# 1. Design a filter
# Let's design a Butterworth Low-Pass Filter
# order = 4, cutoff frequency = 100 Hz, sampling rate (fs) = 1000 Hz
order = 4
cutoff = 100
fs = 1000
# b, a are the numerator and denominator polynomials of the IIR filter
b, a = signal.butter(order, cutoff, fs=fs, btype='low')

# 2. Generate a noisy signal
t = np.linspace(0, 1, fs, endpoint=False)
clean_signal = np.sin(2 * np.pi * 5 * t) # 5 Hz low-frequency signal
noise = 0.5 * np.random.randn(len(t))    # Wideband noise
noisy_signal = clean_signal + noise

# 3. Apply the filter using lfilter (causal filter, introduces phase shift)
filtered_lfilter = signal.lfilter(b, a, noisy_signal)

# 4. Apply filter using filtfilt (zero-phase filter, applied forward and backward)
# Highly recommended for offline analysis to prevent shifting your peaks in time.
filtered_filtfilt = signal.filtfilt(b, a, noisy_signal)

print(f"Variance of noisy: {np.var(noisy_signal):.3f}")
print(f"Variance of filtered: {np.var(filtered_filtfilt):.3f}")
```

### Convolution
Applying a filter kernel to a signal (used heavily in moving averages, blurring, and CNNs).

```python
# Signal
sig = np.array([1, 2, 3, 4, 5, 6])
# Filter kernel (e.g., a simple moving average)
kernel = np.array([1/3, 1/3, 1/3])

# 'valid' mode means only compute where the kernel completely overlaps the signal
convolved = signal.convolve(sig, kernel, mode='valid')
print(f"Convolved signal: {convolved}")
```

### Spectral analysis
Estimating the Power Spectral Density (PSD) of a signal. Often better than a raw FFT for noisy data.

```python
# Welch's method computes the PSD by averaging periodograms of overlapping windowed segments.
# This reduces the variance of the periodogram, resulting in a cleaner frequency plot.
frequencies, psd = signal.welch(noisy_signal, fs=fs, nperseg=256)

# frequencies: Array of sample frequencies.
# psd: Power spectral density or power spectrum.
print(f"Max PSD at frequency: {frequencies[np.argmax(psd)]} Hz")
```

### Signal generation
Generating standard waveforms for testing.

```python
# Generate a sweep (chirp) signal: frequency increases from 1Hz to 10Hz over 10 seconds
t = np.linspace(0, 10, 1000)
w = signal.chirp(t, f0=1, f1=10, t1=10, method='linear')

# Generate a square wave
sq = signal.square(2 * np.pi * 5 * t) # 5 Hz square wave
```

> 💡 **Pro Tip**: For offline data processing, almost always use `signal.filtfilt()` instead of `signal.lfilter()`. `filtfilt` runs the filter forward, then backward, perfectly canceling out the phase delay introduced by IIR filters, keeping your signal aligned with real-world events.

> ⚠️ **Common Pitfall**: Filter instability. High-order IIR filters (e.g., `butter(order=10)`) can become numerically unstable due to floating-point precision issues when represented as `b, a` coefficients. It is much safer to design filters using Second-Order Sections (`output='sos'`) and apply them using `signal.sosfilt()`.

---

## 10. STATISTICS (scipy.stats)

Provides a comprehensive suite of probability distributions, summary statistics, and hypothesis tests.

### Probability distributions
SciPy contains continuous (`scipy.stats.norm`, `gamma`, `expon`) and discrete (`scipy.stats.binom`, `poisson`) distributions.

```python
from scipy.stats import norm, binom

# --- Continuous Distribution (Normal/Gaussian) ---
# Define a normal distribution with mean (loc) = 5, standard deviation (scale) = 2
my_norm = norm(loc=5, scale=2)

# Probability Density Function (PDF)
print("PDF at x=5:", my_norm.pdf(5))

# Cumulative Distribution Function (CDF) - Prob(X <= x)
print("CDF at x=5:", my_norm.cdf(5)) # Should be 0.5 for the mean

# Percent Point Function (PPF) - Inverse of CDF. Useful for finding confidence intervals.
# Find the value x such that 95% of the data falls below x.
print("95th Percentile:", my_norm.ppf(0.95))

# Generate random samples
samples = my_norm.rvs(size=1000)

# --- Discrete Distribution (Binomial) ---
# 10 trials, probability of success = 0.3
my_binom = binom(n=10, p=0.3)

# Probability Mass Function (PMF) - Exact probability of 3 successes
print("Prob of exactly 3 successes:", my_binom.pmf(3))
```

### Hypothesis testing
Comparing samples to determine statistical significance.

```python
from scipy import stats

# Generate two distinct random samples
np.random.seed(0)
sample1 = stats.norm.rvs(loc=5.0, scale=1.0, size=100)
sample2 = stats.norm.rvs(loc=5.5, scale=1.0, size=100)

# Independent T-Test: Are the means of these two independent samples significantly different?
t_stat, p_value = stats.ttest_ind(sample1, sample2)

print(f"T-statistic: {t_stat:.3f}")
print(f"P-value: {p_value:.5f}")
if p_value < 0.05:
    print("Conclusion: Reject the null hypothesis. The means are significantly different.")
else:
    print("Conclusion: Fail to reject the null hypothesis.")

# Shapiro-Wilk test for normality
stat, p = stats.shapiro(sample1)
print(f"Shapiro-Wilk p-value: {p:.3f} (If > 0.05, likely normal)")
```

### Statistical functions
Summary and descriptive statistics.

```python
data = np.array([1, 2, 2, 3, 4, 5, 5, 5, 6, 7])

# Mode
mode_res = stats.mode(data, keepdims=True)
print(f"Mode: {mode_res.mode[0]}, Count: {mode_res.count[0]}")

# Skewness and Kurtosis
print(f"Skewness: {stats.skew(data):.3f}")
print(f"Kurtosis: {stats.kurtosis(data):.3f}")

# Kernel Density Estimation (KDE) - smooth curve estimating the PDF from data
kde = stats.gaussian_kde(samples)
x_eval = np.linspace(0, 10, 100)
y_kde = kde(x_eval) # Evaluate the smoothed density
```

> 💡 **Pro Tip**: Use `scipy.stats` distributions directly to fit parameters to experimental data using Maximum Likelihood Estimation. E.g., `loc_fit, scale_fit = norm.fit(my_data)` will find the best-fitting mean and std dev for your data.

> ⚠️ **Common Pitfall**: P-value misinterpretation. A p-value is the probability of observing the data given that the null hypothesis is true. It is NOT the probability that the null hypothesis is true. Do not blindly use a `0.05` threshold without understanding the statistical power and effect size of your specific domain.

---

## 11. SPARSE MATRICES (scipy.sparse)

Essential when working with large matrices (e.g., 100k x 100k) where the vast majority of elements are zero. Using a dense NumPy array would consume terabytes of RAM.

### Sparse matrix types
SciPy offers different storage formats, each optimized for specific operations.

- **COO (Coordinate)**: Good for constructing matrices fast.
- **CSR (Compressed Sparse Row)**: Fast row slicing, fast matrix-vector products. The standard for machine learning (e.g., Scikit-Learn).
- **CSC (Compressed Sparse Column)**: Fast column slicing.
- **LIL (List of Lists)**: Good for altering the structure (adding new non-zeros).

```python
from scipy import sparse

# 1. Create a dense matrix with lots of zeros
dense_matrix = np.array([
    [0, 0, 3, 0],
    [4, 0, 0, 0],
    [0, 0, 0, 5],
    [0, 2, 0, 0]
])

# 2. Convert to CSR format
csr_matrix = sparse.csr_matrix(dense_matrix)

print(f"Dense memory: {dense_matrix.nbytes} bytes")
# sparse matrices use 3 arrays internally: data, indices, indptr
csr_memory = csr_matrix.data.nbytes + csr_matrix.indices.nbytes + csr_matrix.indptr.nbytes
print(f"CSR memory: {csr_memory} bytes")
print("Matrix string representation:\n", csr_matrix)
```

### Efficient storage and creation
It's inefficient to create a huge dense matrix first. Create sparse matrices directly using COO or LIL.

```python
# Create a 1000x1000 empty LIL matrix
sparse_lil = sparse.lil_matrix((1000, 1000))

# Populate values (LIL is fast for this)
sparse_lil[0, 500] = 3.14
sparse_lil[999, 10] = 2.71

# Convert to CSR for fast math operations
sparse_csr = sparse_lil.tocsr()

# Alternative: Construct directly using COO (Data, (Row_indices, Col_indices))
row = np.array([0, 3, 1, 0])
col = np.array([0, 3, 1, 2])
data = np.array([4, 5, 7, 9])
coo = sparse.coo_matrix((data, (row, col)), shape=(4, 4))
```

### Operations on sparse matrices
Most arithmetic operations work seamlessly, but are specialized.

```python
# Matrix-Vector multiplication
v = np.array([1, 0, -1, 0])
result = coo.tocsr().dot(v)
print(f"Sparse dot product: {result}")

# Sparse linear algebra algorithms are in scipy.sparse.linalg
# E.g., solving sparse linear systems or finding eigenvectors of huge matrices
from scipy.sparse.linalg import eigsh
# eigsh (Eigenvalues of Symmetric/Hermitian matrix) finds a few eigenvalues, not all of them,
# which is perfect for huge sparse matrices (like finding graph principal components).
```

> 💡 **Pro Tip**: The golden rule of `scipy.sparse`: Construct matrices in `COO` or `LIL` format. Convert them to `CSR` or `CSC` format for arithmetic operations (multiplication, solving).

> ⚠️ **Common Pitfall**: Accidentally densifying a sparse matrix. Operating on a sparse matrix with a dense array or calling a function that doesn't understand sparsity can silently convert the 100k x 100k matrix into a dense NumPy array, instantly triggering an Out-Of-Memory (OOM) kill by the OS.

---

## 12. IMAGE PROCESSING (scipy.ndimage)

Functions for N-dimensional image processing. Unlike OpenCV (which focuses on 2D/3D images), `ndimage` works on generic multi-dimensional arrays (like medical MRI scans or hyperspectral data).

### Image filtering
Applying spatial filters to N-D arrays.

```python
from scipy import ndimage
import numpy as np

# Create a noisy 2D image (100x100)
image = np.zeros((100, 100))
image[25:75, 25:75] = 1 # Draw a square
noisy_image = image + 0.5 * np.random.randn(*image.shape)

# Apply a Gaussian filter to blur the image and reduce noise
# sigma is the standard deviation of the Gaussian kernel
blurred_image = ndimage.gaussian_filter(noisy_image, sigma=2)

# Median filter is great for removing "salt and pepper" noise without blurring edges
median_filtered = ndimage.median_filter(noisy_image, size=3)
```

### Transformations
Geometric transformations like rotation and shifting.

```python
# Rotate the image by 45 degrees
# reshape=False means keep the original bounding box size, cropping the corners
rotated = ndimage.rotate(image, angle=45, reshape=False)

# Shift the image by (10, 20) pixels
shifted = ndimage.shift(image, shift=[10, 20])
```

### Feature extraction & Morphology
Morphological operations analyze the shape of features in binary images.

```python
# Create a binary image with holes
binary_image = np.zeros((20, 20), dtype=bool)
binary_image[5:15, 5:15] = True
binary_image[8:12, 8:12] = False # A hole

# Binary closing: Dilation followed by erosion. Used to fill small holes.
closed_image = ndimage.binary_closing(binary_image)

# Labeling: Find connected components (isolated blobs of pixels)
labeled_array, num_features = ndimage.label(closed_image)
print(f"Number of distinct objects found: {num_features}")

# Find center of mass for each labeled object
centers = ndimage.center_of_mass(closed_image, labeled_array, index=np.arange(1, num_features+1))
print(f"Center of mass of objects: {centers}")
```

> 💡 **Pro Tip**: `scipy.ndimage` operations generalize natively to 3D and higher dimensions. A 3D Gaussian filter `gaussian_filter(vol, sigma=2)` is executed efficiently in C, making it excellent for volumetric medical image (CT/MRI) processing.

> ⚠️ **Common Pitfall**: Boundary artifacts. When filtering an image, the kernel extends past the edge of the image array. The `mode` parameter defines how the edges are handled (e.g., `reflect`, `constant`, `nearest`). The default `mode='reflect'` is usually fine, but `mode='constant', cval=0` can introduce dark borders on your filtered images.

---

## 13. DISTANCE & SPATIAL ALGORITHMS

The `scipy.spatial` module computes spatial data structures like Voronoi diagrams, Delaunay triangulations, and nearest neighbors.

### Distance metrics
Computing distances between vectors or collections of vectors.

```python
from scipy.spatial import distance

pt1 = [1, 2, 3]
pt2 = [4, 5, 6]

# Euclidean distance
euclidean = distance.euclidean(pt1, pt2)
print(f"Euclidean distance: {euclidean:.3f}")

# Cosine distance (1 - cosine_similarity). Useful for high-dimensional data like text embeddings.
cos_dist = distance.cosine(pt1, pt2)
print(f"Cosine distance: {cos_dist:.3f}")

# Compute a distance matrix for a set of points (pairwise distances)
points = np.array([[0,0], [0,1], [1,0], [1,1]])
# pdist computes the condensed distance matrix (upper triangle)
dist_matrix_condensed = distance.pdist(points, metric='euclidean')
# squareform converts it to a redundant NxN matrix
dist_matrix = distance.squareform(dist_matrix_condensed)
print("Distance Matrix:\n", dist_matrix)
```

### KDTree for Nearest Neighbors
A KDTree is a data structure for extremely fast spatial queries, like "find the 5 closest points to X" or "find all points within radius R".

```python
from scipy.spatial import KDTree

# 10,000 random 3D points
data_points = np.random.rand(10000, 3)

# Build the tree (takes O(N log N) time)
tree = KDTree(data_points)

# Query the tree: Find the 3 nearest neighbors to the origin
query_point = [0, 0, 0]
distances, indices = tree.query(query_point, k=3)

print("Distances to nearest neighbors:", distances)
print("Indices of nearest neighbors:", indices)
print("Coordinates of nearest neighbors:\n", data_points[indices])

# Query points within a certain radius
indices_in_radius = tree.query_ball_point(query_point, r=0.1)
print(f"Number of points within radius 0.1: {len(indices_in_radius)}")
```

> 💡 **Pro Tip**: If you are repeatedly finding closest points in a large dataset, never use a brute-force loop. Build a `scipy.spatial.KDTree` (or `cKDTree`) once, and query it repeatedly. It drops search time from O(N) to O(log N).

---

## 14. PERFORMANCE OPTIMIZATION

### Efficient numerical computation
SciPy is fast because it delegates heavy lifting to compiled C/C++/Fortran code. To maximize performance, you must write code that minimizes the time spent in the Python interpreter loop.

### Vectorization with NumPy
Always pass NumPy arrays to SciPy functions rather than iterating with Python `for` loops. SciPy functions are almost entirely "vectorized", meaning they apply operations over whole arrays in C.

```python
import time
from scipy.special import gamma

# BAD: Python loop calling a SciPy function
data = np.random.rand(1000000)
start = time.time()
res_loop = np.zeros_like(data)
for i in range(len(data)):
    res_loop[i] = gamma(data[i])
print(f"Loop time: {time.time() - start:.3f} s")

# GOOD: Vectorized call. The C backend loops over the array.
start = time.time()
res_vec = gamma(data) # Passing the whole array directly
print(f"Vectorized time: {time.time() - start:.3f} s") # ~100x faster
```

### Memory considerations
- **In-place operations**: To save memory when modifying large arrays, use NumPy's `out` parameter if supported, or perform in-place arithmetic (e.g., `A += B` instead of `A = A + B`).
- **Data Types**: Be mindful of your `dtype`. Use `np.float32` instead of `np.float64` if your application doesn't require high precision. It halves memory usage and often speeds up computation, especially if your BLAS is optimized for single-precision.

---

## 15. INTEROPERABILITY

SciPy is designed to be the hub of the Python data ecosystem.

### SciPy with NumPy
SciPy consumes and produces NumPy `ndarray` objects exclusively. Any NumPy array can be passed to a SciPy function.

### SciPy with Pandas
Pandas DataFrames and Series are built on NumPy arrays.
To use SciPy on Pandas data:
```python
import pandas as pd
df = pd.DataFrame({'A': [1,2,3], 'B': [4,5,6]})

# Extract the underlying NumPy array using .values or .to_numpy()
matrix = df.to_numpy()
U, s, Vh = linalg.svd(matrix)
```

### SciPy with Matplotlib
SciPy performs the computation; Matplotlib handles the visualization.

```python
import matplotlib.pyplot as plt
from scipy.stats import norm

x = np.linspace(-5, 5, 100)
y = norm.pdf(x)

plt.plot(x, y, label='Standard Normal')
plt.title("SciPy Data plotted with Matplotlib")
plt.legend()
# plt.show()
```

---

## 16. REAL-WORLD APPLICATIONS

### Engineering simulations
- **Finite Element Analysis (FEA)**: Assembling stiffness matrices creates massive sparse matrices. `scipy.sparse.linalg.spsolve` solves these to find stress and strain in physical structures.
- **Control Theory**: The `scipy.signal` module is used to design PID controllers, analyze system transfer functions (Bode plots), and model state-space systems (`scipy.signal.StateSpace`).

### Signal processing pipelines
- **Audio Processing**: Noise reduction pipelines use `scipy.fft` to transform audio to the frequency domain, apply dynamic filtering to suppress non-vocal frequencies, and invert back.
- **Radar/Sonar**: Cross-correlation (`scipy.signal.correlate`) is used to detect the time delay of reflected signals to calculate target distance.

### Optimization problems
- **Logistics**: Route planning and resource allocation often involve mixed-integer linear programming. While SciPy handles continuous optimization well (`scipy.optimize.linprog`), it can serve as the engine for continuous relaxations in larger combinatorial solvers.
- **Machine Learning**: Custom loss functions in deep learning frameworks are often prototyped or cross-verified against `scipy.optimize` solvers like L-BFGS, which is sometimes preferred over Adam for full-batch optimization.

---

## 17. COMMON PITFALLS

1. **Numerical Instability in Linear Algebra**:
   - *Issue*: Directly computing inverses (`linalg.inv`) or high powers of ill-conditioned matrices results in massive floating point round-off errors.
   - *Solution*: Always use orthogonal transformations (QR, SVD) or solvers (`linalg.solve`) which are numerically stable by design.

2. **Misuse of Functions - Optimization Guesses**:
   - *Issue*: `scipy.optimize.minimize` fails to find the global minimum or returns `Success: False`.
   - *Solution*: Gradient-based methods heavily rely on the `initial_guess`. Provide a better guess, explicitly provide the gradient function (Jacobian), or switch to a global optimizer like `differential_evolution`.

3. **Performance Bottlenecks - Python Callbacks**:
   - *Issue*: ODE solvers (`solve_ivp`) or integrators (`quad`) are extremely slow.
   - *Solution*: The solver is in C, but it calls your Python target function millions of times. If possible, use Numba or Cython to compile your target function, or ensure your Python function uses vectorized NumPy operations effectively.

4. **Sparse Matrix Densification**:
   - *Issue*: Your program crashes with a MemoryError when multiplying sparse matrices.
   - *Solution*: Ensure both operands are sparse. Be careful with operations like `sparse_matrix + scalar`, which implies adding a scalar to every zero element, instantly making the matrix dense.

---

## 18. ADVANCED USAGE

### Custom optimization functions
When standard constraints aren't enough, you can write custom Jacobians and Hessians.

```python
def rosen_der(x):
    # Analytical Gradient (Jacobian) of Rosenbrock function
    der = np.zeros_like(x)
    der[0] = -400 * x[0] * (x[1] - x[0]**2) - 2 * (1 - x[0])
    der[1] = 200 * (x[1] - x[0]**2)
    return der

def rosen_hess(x):
    # Analytical Hessian (2nd derivative matrix)
    H = np.zeros((2, 2))
    H[0, 0] = -400 * x[1] + 1200 * x[0]**2 + 2
    H[0, 1] = -400 * x[0]
    H[1, 0] = -400 * x[0]
    H[1, 1] = 200
    return H

# Providing jac and hess allows Newton-CG to converge in mere iterations
res = minimize(rosenbrock, [0, 0], method='Newton-CG', jac=rosen_der, hess=rosen_hess)
```

### Multi-dimensional interpolation on unstructured grids
Using Radial Basis Functions (RBF) for highly irregular N-D scattered data.

```python
from scipy.interpolate import RBFInterpolator

# 10 random 3D coordinates
x_obs = np.random.rand(10, 3)
# Values at those coordinates
y_obs = np.random.rand(10)

# RBF is powerful for "meshless" interpolation
interp = RBFInterpolator(x_obs, y_obs, kernel='thin_plate_spline')

# Evaluate at a new 3D point
x_eval = np.array([[0.5, 0.5, 0.5]])
val = interp(x_eval)
```

---

## 19. INTERNALS OVERVIEW

### SciPy architecture
SciPy is fundamentally a Python wrapper around battle-tested, highly optimized scientific codebases written in compiled languages.
- Python serves as the API and orchestrator.
- Cython is heavily used to bridge Python and C/C++.
- The "heavy lifting" is offloaded.

### Underlying libraries (BLAS, LAPACK)
- **BLAS (Basic Linear Algebra Subprograms)**: Provides standard building blocks for performing basic vector and matrix operations (Level 1: Vector-Vector, Level 2: Matrix-Vector, Level 3: Matrix-Matrix).
- **LAPACK (Linear Algebra PACKage)**: Provides routines for solving systems of simultaneous linear equations, least-squares solutions, eigenvalue problems, and singular value problems. It is built *on top of* BLAS.
- **MINPACK**: Used for optimization and non-linear equations.
- **ODEPACK**: Used for legacy ODE solvers.
- **PROPACK/ARPACK**: Used for sparse eigenvalue problems.

SciPy does not write its own SVD algorithm; it formats the NumPy array memory buffers to be compatible with C/Fortran, calls the compiled LAPACK `dgesdd` routine, and wraps the result back into a Python object.

---

## 20. ADVANCED TOPICS

### Parallel computing basics
SciPy itself does not heavily utilize multithreading at the Python level due to the Global Interpreter Lock (GIL). However:
1. **BLAS/LAPACK Threads**: If SciPy is linked against OpenBLAS or Intel MKL, matrix operations will automatically execute in parallel across multiple CPU cores. You can control this via environment variables like `OMP_NUM_THREADS`.
2. **Python Multiprocessing**: For parallelizing independent optimization or integration tasks (e.g., parameter sweeps), use Python's `multiprocessing` module to run multiple SciPy solvers concurrently in separate processes, bypassing the GIL.

### Integration with machine learning pipelines
SciPy is the hidden workhorse of Machine Learning.
- Scikit-Learn uses `scipy.sparse` extensively for text feature extraction (TF-IDF matrices are almost always sparse).
- PyTorch and TensorFlow both implement APIs that mirror `scipy.linalg` and `scipy.signal`, allowing easy translation of CPU-bound SciPy prototypes into GPU-accelerated deep learning layers.
- Custom PyTorch autograd functions often use `scipy.optimize` solvers in the forward pass to solve convex optimization layers implicitly within neural networks.

### Extending SciPy
Developers can interface their own C/C++ libraries with Python using Cython or pybind11, and ensure compatibility with the SciPy ecosystem by strictly accepting and returning NumPy arrays conforming to standard memory layouts (C-contiguous or F-contiguous).

---
*End of Technical Reference.*
