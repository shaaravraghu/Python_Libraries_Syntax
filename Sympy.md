# SymPy Production Technical Reference

## 1. INTRODUCTION & HISTORY

### What is SymPy
SymPy is a Python library for symbolic mathematics. It aims to become a full-featured computer algebra system (CAS) while keeping the code as simple as possible in order to be comprehensible and easily extensible. SymPy is written entirely in Python and does not require any external libraries.

### Why SymPy (Symbolic Computation vs Numerical Computation)
Symbolic computation deals with the computation of mathematical objects exactly, not approximately. Traditional numerical libraries like NumPy calculate with floating-point numbers, inherently introducing precision limitations and rounding errors. SymPy evaluates algebraic expressions exactly, keeping symbols unevaluated until numerical approximations are explicitly requested.

| Feature | Symbolic Computation (SymPy) | Numerical Computation (NumPy) |
| :--- | :--- | :--- |
| **Precision** | Infinite precision (exact mathematical answers) | Floating-point (approximations, limited by IEEE 754) |
| **Variables** | Symbols representing mathematical variables | Variables referencing memory holding numeric values |
| **Speed** | Generally slower (requires algebraic manipulation overhead) | Extremely fast (vectorized operations in C/Fortran) |
| **Output** | Exact algebraic expressions (e.g., `sqrt(2)`) | Floating point numbers (e.g., `1.41421356`) |

### Comparison with NumPy/SciPy
- **NumPy/SciPy:** Best for large-scale data manipulation, machine learning, and numerical simulations. Computes roots of polynomials numerically, integrates numerical arrays, solves linear systems numerically.
- **SymPy:** Best for deriving formulas, solving equations exactly, performing exact calculus (derivatives/integrals), and finding analytical solutions.

### Use Cases
- **Algebra:** Expanding polynomials, factoring expressions, simplifying trigonometric identities.
- **Calculus:** Exact analytical differentiation and integration. Finding limits and series expansions.
- **Equation Solving:** Analytically solving algebraic systems, differential equations (ODEs/PDEs), and non-linear systems.
- **Code Generation:** Deriving formulas symbolically and generating highly optimized C, C++, or Fortran code from them.

### Version Evolution
- Created by Ondřej Čertík in 2006.
- 1.0 (2016): Major stabilization of core modules.
- 1.12+ (Present): Advanced solveset integration, superior ODE/PDE modules, refined tensor calculus, and performance optimizations using structural caching.

> [!TIP]
> **Pro Tip:** SymPy is brilliant for the *derivation* phase of your algorithm design. Derive complex equations symbolically, then use SymPy's code generation (`lambdify`) to turn them into ultra-fast NumPy functions for the execution phase.

> [!WARNING]
> **Common Pitfall:** Do not use SymPy for large-array heavy-lifting calculations inside a hot loop. Symbolic representations take up significantly more memory and compute time.

---

## 2. INSTALLATION & SETUP

### pip install sympy
To install SymPy, simply use pip:

```bash
# Standard installation
pip install sympy

# Installation with optional fast-cache backend
pip install sympy gmpy2 mpmath
```
*Note: `gmpy2` provides a fast C-backend for multiple-precision arithmetic, speeding up SymPy computations.*

### Import Conventions
By convention, SymPy is often imported entirely, or specifically. Best practice in production is explicit importing.

```python
# Standard explicit import (Recommended for Production)
import sympy as sp

# Importing specific commonly used functions/classes
from sympy import Symbol, symbols, sin, cos, pi, exp, integrate, diff
```

### Initializing Printing (Pretty, LaTeX)
SymPy has a fantastic printing system that renders expressions beautifully in terminals and notebooks.

```python
import sympy as sp

# Initializes the best available printer for your environment
# In a Jupyter notebook, this will use MathJax to render LaTeX.
# In a terminal, it will use Unicode characters.
sp.init_printing(use_unicode=True)

# Forcing LaTeX printing:
# sp.init_printing(use_latex=True)
```

### Basic Setup in Jupyter
When opening a Jupyter Notebook, standard practice is:

```python
import sympy as sp
sp.init_printing()

# Define common variables to start working immediately
x, y, z = sp.symbols('x y z')
```

> [!TIP]
> **Pro Tip:** If you are building a CLI application that outputs math, `init_printing(use_unicode=True)` creates readable ASCII/Unicode art equations.

---

## 3. SYMBOLS & EXPRESSIONS

### Defining Symbols
At the core of SymPy are Symbols. A Symbol object represents a mathematical variable.

```python
import sympy as sp

# Define a single symbol
x = sp.Symbol('x')

# Define multiple symbols at once (highly recommended)
y, z, t = sp.symbols('y z t')

# You can name the symbol differently from the Python variable
alpha = sp.Symbol('a') # Python variable alpha refers to math symbol 'a'
```

### Symbol Assumptions
By default, symbols are assumed to be complex numbers. Adding assumptions drastically helps the `simplify()` engine and solvers.

```python
# Symbol that is explicitly real and positive
# This is crucial for expressions like sqrt(r**2) to simplify to r
r = sp.Symbol('r', real=True, positive=True)

# Integer symbol
n = sp.Symbol('n', integer=True)

# Check assumptions
print(r.is_positive) # True
print(r.is_complex)  # True (Reals are a subset of Complex in SymPy)
```

### Expressions and Operations
Expressions are built using standard Python operators `+`, `-`, `*`, `/`, `**`.

```python
x, y = sp.symbols('x y')

# Building an expression
expr = x + 2*y**2 - sp.sin(x) / x

# Note: Python's standard `^` operator is XOR, not exponentiation!
# DO NOT use `x^2`, use `x**2`.
expr2 = x**2 + y**2
```

### Simplification Basics
Evaluating an expression usually leaves it in its raw structural form. SymPy can simplify it.

```python
# Creating an unsimplified expression
expr = sp.sin(x)**2 + sp.cos(x)**2

# Simplify it
simplified_expr = sp.simplify(expr)
print(simplified_expr) # Output: 1
```

> [!WARNING]
> **Common Pitfall:** Python's integer division.
> `1/2` in standard Python yields `0.5` (a float). If you write `x + 1/2`, SymPy sees `x + 0.5`. To keep exact rational numbers, use `sp.Rational`.
> ```python
> exact_expr = x + sp.Rational(1, 2)
> ```

> [!TIP]
> **Pro Tip:** Always declare assumptions if you know them. E.g., `sp.Symbol('x', real=True)` solves many headaches where SymPy refuses to simplify logarithms or square roots because it must account for complex plane branches.

---

## 4. ALGEBRAIC MANIPULATION

Algebraic manipulation is one of SymPy's most powerful features.

### expand()
Expands polynomial expressions and removes grouping.

```python
import sympy as sp
x, y = sp.symbols('x y')

expr = (x + y)**3
expanded = sp.expand(expr)
# Output: x**3 + 3*x**2*y + 3*x*y**2 + y**3

# expand() also works for trig functions if you pass specific hints
trig_expr = sp.sin(x + y)
expanded_trig = sp.expand(trig_expr, trig=True)
# Output: sin(x)*cos(y) + sin(y)*cos(x)
```

### factor()
The opposite of `expand()`. Factors a polynomial into irreducible factors.

```python
expr = x**3 - x**2 + x - 1
factored = sp.factor(expr)
# Output: (x - 1)*(x**2 + 1)
```

### simplify()
Attempts to intelligently find the simplest form of an expression. It applies heuristics and tries multiple algorithms.

```python
expr = (x**3 + x**2 - x - 1)/(x**2 + 2*x + 1)
simplified = sp.simplify(expr)
# Output: x - 1
```

### collect()
Collects common powers of a term. Excellent for polynomial reorganization.

```python
expr = x*y + x - 3 + 2*x**2 - z*x**2 + x**3
# Collect terms with respect to x
collected = sp.collect(expr, x)
# Output: x**3 + x**2*(2 - z) + x*(y + 1) - 3

# You can access the coefficients after collecting
coeffs = collected.coeff(x, 2) # Gets coefficient of x**2
# Output: 2 - z
```

### apart()
Computes the partial fraction decomposition of a rational function. Crucial for integration and inverse Laplace transforms.

```python
expr = (4*x**3 + 21*x**2 + 10*x + 12)/(x**4 + 5*x**3 + 5*x**2 + 4*x)
fractionated = sp.apart(expr)
# Output: (2*x - 1)/(x**2 + x + 1) - 1/(x + 4) + 3/x
```

> [!TIP]
> **Pro Tip:** `simplify()` is computationally expensive because it tries many strategies. If you know exactly what you want (e.g., factoring, expanding trig), use specific functions like `factor()`, `trigsimp()`, `radsimp()`, or `cancel()`. They are much faster and more predictable.

---

## 5. EQUATION SOLVING

SymPy distinguishes between equations (`Eq`) and simple expressions. If you pass an expression to a solver, SymPy assumes it equals zero.

### solve()
The classic algebraic solver. It returns a list of solutions.

```python
import sympy as sp
x, y = sp.symbols('x y')

# Solving x**2 - 4 = 0
sol = sp.solve(x**2 - 4, x)
# Output: [-2, 2]

# You can define exact equations using Eq
eq = sp.Eq(x**2, 4)
sol2 = sp.solve(eq, x)
```

### solveset()
The modern, mathematically rigorous replacement for `solve()`. It returns a Set object representing the mathematical set of solutions. `solveset` is preferred over `solve` for univariate equations.

```python
# Solving sin(x) = 0
sol_set = sp.solveset(sp.sin(x), x, domain=sp.S.Reals)
# Output: Intersection({2*n*pi | n in Integers}, Reals) U Intersection({2*n*pi + pi | n in Integers}, Reals)

sol_set_complex = sp.solveset(x**2 + 1, x, domain=sp.S.Complexes)
# Output: {-I, I}
```

### Linear and Nonlinear Equations
SymPy has dedicated solvers for systems of equations: `linsolve` and `nonlinsolve`.

#### Systems of Equations
```python
x, y, z = sp.symbols('x y z')

# Linear System
# x + y + z = 1
# x + y + 2*z = 3
system = [x + y + z - 1, x + y + 2*z - 3]
sol_lin = sp.linsolve(system, (x, y, z))
# Output: {(-y - 1, y, 2)}

# Nonlinear System
# x**2 + y**2 = 1
# x - y = 0
system_nonlin = [x**2 + y**2 - 1, x - y]
sol_nonlin = sp.nonlinsolve(system_nonlin, (x, y))
# Output: {(-sqrt(2)/2, -sqrt(2)/2), (sqrt(2)/2, sqrt(2)/2)}
```

> [!WARNING]
> **Common Pitfall:** Never use Python's `==` to create an equation. `x == y` evaluates to a Boolean `False` immediately because `x` and `y` are distinct object instances. Always use `sp.Eq(x, y)` to represent mathematical equality.

> [!TIP]
> **Pro Tip:** Transition your codebase to use `solveset()`, `linsolve()`, and `nonlinsolve()`. `solve()` is legacy and has an inconsistent output type (sometimes returning a list of values, sometimes a list of dictionaries).

---

## 6. CALCULUS

SymPy handles exact limits, differentiation, and integration.

### Differentiation (diff)
Calculates derivatives of expressions.

```python
import sympy as sp
x, y, z = sp.symbols('x y z')

expr = sp.sin(x) * sp.exp(x)

# First derivative
deriv_1 = sp.diff(expr, x)
# Output: exp(x)*sin(x) + exp(x)*cos(x)

# Second derivative
deriv_2 = sp.diff(expr, x, 2)
# Output: 2*exp(x)*cos(x)

# Partial derivatives
expr_multivar = sp.exp(x*y*z)
partial_xyz = sp.diff(expr_multivar, x, y, z)
# Output: x**2*y**2*exp(x*y*z) + 3*x*y*exp(x*y*z) + exp(x*y*z)
```

### Integration (integrate)
Handles both definite and indefinite integrals.

```python
expr = sp.exp(-x)

# Indefinite Integral
indef_int = sp.integrate(expr, x)
# Output: -exp(-x)
# Note: SymPy does not include the constant of integration (C).

# Definite Integral from 0 to infinity
# oo is SymPy's representation of infinity
def_int = sp.integrate(sp.exp(-x**2), (x, -sp.oo, sp.oo))
# Output: sqrt(pi)

# Multiple integrals
expr_multi = x*y
# Integrate x from 0 to 1, then y from 0 to 2
multi_int = sp.integrate(expr_multi, (x, 0, 1), (y, 0, 2))
# Output: 1
```

### Limits (limit)
Calculates limits, resolving singularities cleanly.

```python
expr = sp.sin(x) / x

# Limit as x approaches 0
lim = sp.limit(expr, x, 0)
# Output: 1

# One-sided limits
expr2 = 1 / x
lim_right = sp.limit(expr2, x, 0, dir='+') # Output: oo
lim_left = sp.limit(expr2, x, 0, dir='-')  # Output: -oo
```

### Series Expansion
Computes Taylor and Maclaurin series expansions.

```python
expr = sp.exp(x)

# Expand around x=0 up to O(x**4)
series_exp = expr.series(x, 0, 4)
# Output: 1 + x + x**2/2 + x**3/6 + O(x**4)

# Remove the Big-O term to use as a normal polynomial
poly_approx = series_exp.removeO()
```

> [!TIP]
> **Pro Tip:** When integrating complex expressions, it may hang or fail. If you only need a numerical value, use `scipy.integrate.quad`. If you need an exact formulation, try calling `.simplify()` or `.expand()` on your expression *before* passing it to `integrate()`.

---

## 7. MATRICES

SymPy has robust exact linear algebra capabilities.

### Matrix Creation
Matrices are instantiated using the `Matrix` class.

```python
import sympy as sp
x, y = sp.symbols('x y')

# Creating a matrix from a list of lists
M = sp.Matrix([[1, -1], [3, 4], [0, 2]])

# Creating a symbolic matrix
M_sym = sp.Matrix([[x, y], [x**2, y**2]])

# Special matrices
eye = sp.eye(3)        # 3x3 Identity matrix
zeros = sp.zeros(2, 3) # 2x3 Matrix of zeros
ones = sp.ones(3)      # 3x3 Matrix of ones
diag = sp.diag(1, x, y) # Diagonal matrix
```

### Matrix Operations
Standard arithmetic operations are overloaded.

```python
M1 = sp.Matrix([[1, 2], [3, 4]])
M2 = sp.Matrix([[5, 6], [7, 8]])

# Addition and Subtraction
M_add = M1 + M2

# Matrix Multiplication
M_mult = M1 * M2

# Element-wise multiplication is NOT M1 * M2.
# You need to iterate or use specific functions for element-wise.

# Transpose
M_trans = M1.T
```

### Determinant & Inverse
```python
M = sp.Matrix([[1, 2], [3, 4]])

# Determinant
det = M.det() # Output: -2

# Inverse
inv = M.inv() # Output: Matrix([[-2, 1], [3/2, -1/2]])

# Inverse is symbolically exact!
M_sym = sp.Matrix([[x, y], [1, x]])
inv_sym = M_sym.inv()
```

### Eigenvalues and Eigenvectors
```python
M = sp.Matrix([[3, -2,  4], [-2,  6,  2], [4,  2,  3]])

# Eigenvalues returns a dictionary: {eigenvalue: algebraic_multiplicity}
evals = M.eigenvals()

# Eigenvectors returns a list of tuples: (eigenvalue, multiplicity, [eigenvectors])
evects = M.eigenvects()

# Diagonalization (M = P * D * P**-1)
P, D = M.diagonalize()
```

> [!WARNING]
> **Common Pitfall:** Unlike NumPy arrays, SymPy Matrices are mutable by default. If you need immutable matrices (for instance, to use them as dictionary keys), use `ImmutableMatrix`.

> [!TIP]
> **Pro Tip:** SymPy Matrices are 1-D indexed inherently in Python list comprehension style but are usually addressed as `M[row, col]`. They are extremely slow for dimensions larger than 50x50. Always switch to NumPy/SciPy for numeric linear algebra.

---

## 8. FUNCTIONS

### Built-in Functions
SymPy provides all standard mathematical functions. They handle exact values and simplifications seamlessly.

```python
import sympy as sp
x = sp.Symbol('x')

expr = sp.sin(sp.pi/2) + sp.cos(0) + sp.exp(sp.log(x))
# Output: x + 2

# Piecewise functions
piece = sp.Piecewise((0, x < 0), (x**2, x >= 0))
```

### Custom/Undefined Functions
When analyzing differential equations or functional relationships, you often need an undefined function $f(x)$.

```python
# Create an undefined function symbol
f = sp.Function('f')
g = sp.Function('g')

# Applying the function to a variable
expr = f(x) + g(x, y)

# Derivative of an undefined function yields the notation: Derivative(f(x), x)
diff_f = sp.diff(f(x), x)
```

### Lambda Functions
SymPy's `Lambda` is the symbolic equivalent of Python's `lambda`. Useful for defining reusable exact mathematical mappings.

```python
# Equivalent to: lambda x: x**2 + 2*x - 1
l_func = sp.Lambda(x, x**2 + 2*x - 1)

# Applying it
val = l_func(3) # Output: 14
val_sym = l_func(sp.Symbol('z')) # Output: z**2 + 2*z - 1
```

> [!TIP]
> **Pro Tip:** Use `sp.Function('f')` over defining variables when dealing with time-series or ODEs. `x = sp.Function('x')` and then analyzing `x(t)` makes the derivatives `sp.diff(x(t), t)` resolve correctly structurally.

---

## 9. SUBSTITUTION & EVALUATION

### subs()
`subs` replaces symbols with values, other symbols, or entire expressions.

```python
import sympy as sp
x, y, z = sp.symbols('x y z')
expr = x**3 + 4*x*y - z

# Single substitution
sub1 = expr.subs(x, 2)
# Output: 8*y - z + 8

# Multiple substitutions using a list of tuples
sub2 = expr.subs([(x, 2), (y, 4), (z, 0)])
# Output: 40

# Multiple substitutions using a dictionary (Recommended)
sub3 = expr.subs({x: 2, y: 4, z: 0})
```

### evalf()
Evaluates an expression to a floating-point approximation.

```python
expr = sp.sqrt(2)

# Evaluates to 15 decimal places by default
val = expr.evalf() # Output: 1.41421356237310

# Evaluate to 50 decimal places
val_50 = expr.evalf(50)

# Substituting and evaluating in one step
expr2 = sp.cos(x)
val2 = expr2.evalf(subs={x: 3.14})
```

### Numerical Evaluation (lambdify vs evalf)
While `evalf()` is great for a single evaluation, it is inherently slow because it relies on SymPy's high-precision backend (`mpmath`). If you need to evaluate an expression thousands of times over an array, use `lambdify()` (covered in Section 17).

> [!WARNING]
> **Common Pitfall:** Doing `expr.subs(x, 0.5)` will introduce floating point contamination into your exact algebraic expression. If you plan to continue doing calculus/algebra, use `expr.subs(x, sp.Rational(1, 2))`. Use floats ONLY at the very end.

> [!TIP]
> **Pro Tip:** If `subs()` doesn't seem to replace terms exactly as you expect, try `xreplace()`. `subs` relies on algebraic matching, whereas `xreplace` strictly replaces exact structural nodes in the expression tree.

---

## 10. PRINTING & FORMATTING

SymPy's printing capabilities convert internal expression trees into human-readable math or source code.

### Pretty Printing
```python
import sympy as sp
x = sp.Symbol('x')
expr = sp.Integral(sp.sqrt(1/x), x)

# Standard Python string (str)
print(str(expr))
# Output: Integral(sqrt(1/x), x)

# Pretty print (ASCII/Unicode)
sp.pprint(expr)
```

### LaTeX Output
Easily convert your derived expressions into LaTeX code for papers and reports.

```python
latex_code = sp.latex(expr)
print(latex_code)
# Output: \int \sqrt{\frac{1}{x}}\, dx
```

### Structural Representation (srepr)
To understand how SymPy parses an expression internally (the abstract syntax tree), use `srepr`.

```python
expr = sp.Add(x, x)
print(sp.srepr(expr))
# Output: Mul(Integer(2), Symbol('x'))
# Notice how Add(x, x) is automatically evaluated to Mul(2, x)!
```

### Code Generation
Generate target language source code.

```python
from sympy.utilities.codegen import codegen

expr = sp.sin(x) * sp.cos(x)
# Generate C code
[(c_name, c_code), (h_name, c_header)] = codegen(("my_func", expr), "C", "file_prefix")
print(c_code)
# Output will contain standard C math library calls: return sin(x)*cos(x);
```

> [!TIP]
> **Pro Tip:** In a Jupyter Notebook, if an expression is too large, the MathJax rendering might crash the browser page. Use `print(expr)` or `sp.pprint(expr)` for enormous equations instead of relying on the notebook's default LaTeX renderer.

---

## 11. POLYNOMIALS

SymPy has a highly optimized `Poly` domain that is much faster for polynomial algebra than standard expressions.

### Polynomial Manipulation
When you define a `Poly` object, SymPy locks it into a strictly mathematical polynomial representation.

```python
import sympy as sp
x, y = sp.symbols('x y')

# Create a Poly object
p = sp.Poly(x**2 + 2*x - 3, x)

# Get coefficients (ordered highest degree to lowest)
coeffs = p.coeffs() # Output: [1, 2, -3]

# Get the degree
deg = p.degree() # Output: 2
```

### Roots
Finding the exact roots of polynomials.

```python
# Find roots (returns dictionary: {root: multiplicity})
r = sp.roots(x**3 - 1, x)
# Output: {1: 1, -1/2 - sqrt(3)*I/2: 1, -1/2 + sqrt(3)*I/2: 1}

# If exact analytical roots are impossible (degree >= 5 generally), use nroots for numerical
p_high = sp.Poly(x**5 - x + 1)
numeric_roots = p_high.nroots()
```

### Factorization and GCD
```python
p1 = x**2 - 1
p2 = x**2 + 2*x + 1

# Greatest Common Divisor
gcd_val = sp.gcd(p1, p2) # Output: x + 1

# Factorization is significantly faster within the Poly module
factored_p1 = sp.factor(p1) # (x - 1)*(x + 1)
```

> [!WARNING]
> **Common Pitfall:** Mixing `Poly` objects with standard expressions can sometimes lead to unexpected types. Always call `.as_expr()` on a `Poly` object when you want to return it to standard SymPy expression manipulation.

> [!TIP]
> **Pro Tip:** If you are performing heavy algebraic geometry (Groebner bases, resultants), ALWAYS use `sp.Poly`. The standard `simplify()` will be magnitudes slower and might hang.

---

## 12. LOGIC & SETS

SymPy natively supports Boolean logic and Set theory.

### Logical Expressions
```python
import sympy as sp
A, B = sp.symbols('A B')

# Boolean operators
expr = sp.And(A, sp.Or(sp.Not(A), B))

# Simplify boolean logic
simplified = sp.simplify_logic(expr)
# Output: A & B
```

### Sets and Intervals
```python
# Finite sets
s1 = sp.FiniteSet(1, 2, 3)
s2 = sp.FiniteSet(3, 4, 5)

# Union and Intersection
union_set = sp.Union(s1, s2)
inter_set = sp.Intersection(s1, s2) # Output: {3}

# Intervals (Continuous ranges)
# [0, 5) -> Left inclusive, Right exclusive
interval = sp.Interval.Ropen(0, 5)

# Checking containment
check = 2 in interval # Output: True
```

### ConditionSet
Used when a set cannot be evaluated explicitly.

```python
x = sp.Symbol('x')
# The set of all x in Reals such that sin(x) > 0
c_set = sp.ConditionSet(x, sp.sin(x) > 0, sp.S.Reals)
```

> [!TIP]
> **Pro Tip:** Sets are fundamental to how the new `solveset` module works. Get comfortable with `Intersection`, `Union`, `Interval`, and `ImageSet` if you are parsing solutions from trigonometric or polynomial systems.

---

## 13. DIFFERENTIAL EQUATIONS

SymPy's `dsolve` is extremely powerful, capable of finding exact solutions to linear ODEs, exact ODEs, separable ODEs, and systems of ODEs.

### Solving ODEs
```python
import sympy as sp

# Define symbols and the function
x = sp.Symbol('x')
f = sp.Function('f')

# Define the differential equation: f''(x) - 2f'(x) + f(x) = sin(x)
diffeq = sp.Eq(f(x).diff(x, x) - 2*f(x).diff(x) + f(x), sp.sin(x))

# Solve the ODE
sol = sp.dsolve(diffeq, f(x))
# Output will be an Eq() object representing the solution with constants C1, C2
print(sol)
```

### Initial Conditions (IVPs)
To solve an Initial Value Problem, pass an `ics` dictionary to `dsolve`.

```python
# f(x)' = -f(x), initial condition f(0) = 5
diffeq_ivp = sp.Eq(f(x).diff(x), -f(x))
sol_ivp = sp.dsolve(diffeq_ivp, f(x), ics={f(0): 5})
# Output: Eq(f(x), 5*exp(-x))
```

### Systems of ODEs
```python
g = sp.Function('g')
t = sp.Symbol('t')

# System: f'(t) = g(t), g'(t) = -f(t)
eq1 = sp.Eq(f(t).diff(t), g(t))
eq2 = sp.Eq(g(t).diff(t), -f(t))

sys_sol = sp.dsolve([eq1, eq2])
```

> [!WARNING]
> **Common Pitfall:** Forgetting to declare `f = sp.Function('f')`. If you just use `f = sp.Symbol('f')`, calling `f(x)` will throw a `TypeError`, and `diff(f, x)` will return `0` because SymPy thinks `f` is a constant with respect to `x`.

> [!TIP]
> **Pro Tip:** You can ask SymPy what classification algorithms it applied using `sp.classify_ode(diffeq)`. This is highly educational for math students to see if an ODE is Bernoulli, Separable, or Exact.

---

## 14. SERIES & APPROXIMATIONS

### Taylor / Maclaurin Series
```python
import sympy as sp
x = sp.Symbol('x')

# cos(x) around x=0 up to 6th order
cos_series = sp.series(sp.cos(x), x, 0, 6)
# Output: 1 - x**2/2 + x**4/24 + O(x**6)

# Extracting the polynomial part (removing Big-O)
approx = cos_series.removeO()
```

### Asymptotic Approximations
SymPy handles asymptotic series naturally via the `nseries` method and Limits.

```python
# Limit at infinity
expr = (x**2 + x + 1) / (3*x**2 - 2)
limit_inf = sp.limit(expr, x, sp.oo) # Output: 1/3
```

### Pade Approximants
Rational function approximations, often converging faster than Taylor series for functions with poles.

```python
from sympy.series.polyroots import roots
# Note: Currently SymPy exposes Pade via the specific function or series tools.
# Often calculated explicitly by deriving Taylor coefficients and solving linear systems.
```

> [!TIP]
> **Pro Tip:** When evaluating `series()`, the `O(x**n)` term is an active object (`sp.O`). If you try to substitute values into the series while the `O()` term is present, you might get an error or a generic `O(1)`. Always call `.removeO()` before numerical substitution.

---

## 15. VECTOR CALCULUS

SymPy provides a dedicated module for 3D vector calculus.

### Setup and Coordinate Systems
```python
from sympy.vector import CoordSys3D, gradient, divergence, curl

# Define a Cartesian coordinate system
N = CoordSys3D('N')

# N.i, N.j, N.k are the unit vectors
# N.x, N.y, N.z are the scalar coordinates
```

### Gradient, Divergence, Curl
```python
# Scalar field (e.g., potential)
scalar_field = N.x**2 * N.y * N.z

# Gradient (returns a Vector field)
grad = gradient(scalar_field)
# Output: 2*N.x*N.y*N.z N.i + N.x**2*N.z N.j + N.x**2*N.y N.k

# Vector field (e.g., force)
vector_field = N.x*N.i + N.y*N.j + N.z*N.k

# Divergence (returns a Scalar)
div = divergence(vector_field) # Output: 3

# Curl (returns a Vector)
crl = curl(vector_field) # Output: 0
```

### The Del Operator
You can also use the explicit `Del()` operator.

```python
from sympy.vector import Del
del_op = Del()

# del_op(scalar_field) == gradient
# del_op.dot(vector_field) == divergence
# del_op.cross(vector_field) == curl
```

> [!WARNING]
> **Common Pitfall:** Mixing standard SymPy symbols `x, y, z` with Vector module coordinates `N.x, N.y, N.z`. They are NOT compatible out of the box. Use the `CoordSys3D` scalar bases for all vector equations.

> [!TIP]
> **Pro Tip:** The vector module supports transformation between coordinate systems (e.g., Cartesian to Spherical/Cylindrical). It handles the complex Jacobian transformations automatically when computing gradients in curvilinear coordinates.

---

## 16. GEOMETRY

SymPy includes an exact Euclidean geometry module.

### Points, Lines, Planes
```python
import sympy as sp
from sympy.geometry import Point, Line, Circle, intersection

# Define Points
p1 = Point(0, 0)
p2 = Point(1, 1)
p3 = Point(2, 0)

# Define a Line
l1 = Line(p1, p2)

# Define a Circle
c1 = Circle(Point(0, 0), 5) # Center at origin, radius 5
```

### Distance and Intersections
```python
# Exact distance between points
dist = p1.distance(p2) # Output: sqrt(2)

# Intersections
# Intersection of line y=x and circle x^2 + y^2 = 25
inter = intersection(l1, c1)
# Output: [Point2D(-5*sqrt(2)/2, -5*sqrt(2)/2), Point2D(5*sqrt(2)/2, 5*sqrt(2)/2)]
```

### Geometric Transformations
```python
# Rotate point p2 around the origin by 45 degrees (pi/4)
rotated_p2 = p2.rotate(sp.pi/4, Point(0,0))
```

> [!TIP]
> **Pro Tip:** The geometry module is fantastic for computational geometry proofs because it relies on exact Rational numbers and exact Surds (square roots). There is zero floating point inaccuracy in intersection logic.

---

## 17. NUMERICAL BRIDGE (lambdify)

This is the most critical feature for bridging SymPy derivations with production execution.

### lambdify()
Converts a SymPy expression into a highly efficient Python function, automatically mapping SymPy functions (like `sp.sin`) to numerical backend functions (like `numpy.sin`).

```python
import sympy as sp
import numpy as np

x, y = sp.symbols('x y')
expr = sp.sin(x) + sp.cos(y)**2

# Create a NumPy-compatible function
# Signature: lambdify(variables, expression, modules)
f_num = sp.lambdify((x, y), expr, "numpy")

# Now execute it with NumPy arrays at C-speed
x_arr = np.linspace(0, 10, 1000)
y_arr = np.linspace(0, 10, 1000)

result = f_num(x_arr, y_arr) # Executes instantly
```

### Supported Backends
- `"numpy"`: Uses NumPy (standard for array calculations).
- `"math"`: Uses Python's standard math library (fastest for single scalar values).
- `"numexpr"`: Uses NumExpr, heavily parallelized and optimized array operations.
- `"scipy"`: Allows mapping to SciPy special functions.

### Hybrid Symbolic-Numeric Workflows
1. **Derivation Phase:** Use SymPy to solve PDEs, take complex Jacobians, or find analytical Hessian matrices.
2. **Translation Phase:** Use `lambdify` to convert the massive analytical result into a NumPy lambda.
3. **Execution Phase:** Pass the `lambdify` function to `scipy.optimize.minimize` or solve massive arrays of data.

> [!WARNING]
> **Common Pitfall:** When your expression has matrix outputs, ensure that `lambdify` is configured correctly. Sometimes it outputs a list of arrays instead of a clean 2D NumPy array. Wrapping the SymPy matrix in `sp.Array` before lambdifying often resolves this.

> [!TIP]
> **Pro Tip:** If performance is absolutely critical, use `"numexpr"` as the backend in `lambdify`. NumExpr evaluates the array operations in parallel and avoids creating massive intermediate arrays in memory.

---

## 18. PERFORMANCE CONSIDERATIONS

SymPy is written in pure Python. It prioritizes exactness over speed.

### Simplification Costs
`simplify()` is an umbrella function that calls dozens of heuristics. It is incredibly slow.
**Fix:** Profile your expressions. If you only need to factor, use `factor()`. If you only need to group, use `collect()`.

### Expression Complexity
Massive expanded polynomials take exponential time to compute derivatives or integrals.
**Fix:** Keep expressions factored if possible. Use `sp.cancel()` on rational functions before doing calculus.

### evaluate=False
By default, SymPy automatically evaluates simple arithmetic (`x + x -> 2*x`). For complex trees, this automatic evaluation can take time. You can build large trees without evaluation and evaluate later.

```python
import sympy as sp
x = sp.Symbol('x')

# Prevents automatic summation
expr = sp.Add(x, x, evaluate=False)
```

### Using gmpy2
SymPy automatically uses `gmpy2` if installed. This shifts large integer and rational arithmetic from pure Python to C. Always install `gmpy2` in a production environment.

> [!TIP]
> **Pro Tip:** If SymPy hangs indefinitely on an integral, it's likely searching for a closed-form solution via the Meijer G-function algorithm, which can be astronomically slow. Pass `manual=True` to `integrate()` to restrict it to simpler heuristic techniques.

---

## 19. COMMON PITFALLS

1. **Floating Point Contamination:**
   ```python
   # BAD: Uses float 0.5
   eq = x**2 + 0.5
   
   # GOOD: Uses exact Rational
   eq = x**2 + sp.Rational(1, 2)
   ```
2. **XOR vs Exponentiation:**
   Python uses `^` for bitwise XOR. `x^2` is valid Python but mathematically wrong in SymPy. ALWAYS use `x**2`.
3. **Implicit vs Explicit Multiplication:**
   SymPy cannot parse `2x`. You must explicitly write `2*x`. When parsing strings, you can use `sp.parse_expr("2*x", transformations="implicit_multiplication")`.
4. **is_ vs == :**
   To check if an expression mathematically equals another, use `.equals()` or `simplify(A - B) == 0`. The `==` operator only checks if the object representation is identical.
   ```python
   A = (x+1)**2
   B = x**2 + 2*x + 1
   
   print(A == B) # False! Structural difference.
   print(A.equals(B)) # True! Mathematical equality.
   ```

---

## 20. ADVANCED TOPICS

### Extending SymPy / Custom Symbolic Functions
You can subclass `sp.Function` to create mathematical objects with specific derivation rules.

```python
import sympy as sp

class MyCustomFunc(sp.Function):
    @classmethod
    def eval(cls, arg):
        if arg.is_Number:
            # Evaluate exactly if a number is passed
            return arg**2 + 1
            
    def fdiff(self, argindex=1):
        # Define how the derivative works
        x = self.args[0]
        return 2*x

x = sp.Symbol('x')
f = MyCustomFunc(x)

# diff knows how to handle MyCustomFunc because we defined fdiff!
df = sp.diff(f, x) # Output: 2*x
```

### Internals Overview (Core, Args, Func)
Every SymPy object is a tree. You can navigate this tree using `.func` and `.args`.

```python
expr = 2 * sp.sin(x) + y

# .func returns the top-level operator
print(expr.func) # Output: <class 'sympy.core.add.Add'>

# .args returns the children nodes
print(expr.args) # Output: (y, 2*sin(x))

# You can rebuild the expression exactly:
rebuilt = expr.func(*expr.args)
print(expr == rebuilt) # True
```

This structural understanding allows you to write custom recursive parsers or simplifiers tailored strictly to your domain, avoiding the overhead of `simplify()`.

### Final Production Advice
SymPy is an exceptional engine for abstracting algebraic complexity out of your production application.
1. **Never put SymPy in a low-level computation loop.**
2. **Compute derivatives, Jacobians, and Hessians analytically at initialization or compile-time.**
3. **Use `lambdify` to convert those massive symbolic trees into pure, vectorized NumPy/NumExpr functions.**
4. **Execute at C-speed while retaining the exact precision of an analytically derived system.**
