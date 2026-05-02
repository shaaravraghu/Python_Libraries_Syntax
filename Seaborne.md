# SEABORN: EXHAUSTIVE PRODUCTION-GRADE REFERENCE GUIDE

This document serves as an exhaustive, production-grade technical reference for the Seaborn library. It covers every aspect from fundamental plotting concepts to advanced configuration, statistical transformations, and integration with the broader Python data science ecosystem.

---

## 1. INTRODUCTION & HISTORY

### What is Seaborn?
Seaborn is a Python data visualization library based on Matplotlib. It provides a high-level, declarative interface for drawing attractive and informative statistical graphics. While Matplotlib provides the "plumbing" (figures, axes, rendering), Seaborn provides the "design system" and statistical aggregation capabilities, allowing developers to focus on what the data means rather than how to draw it.

### Why Seaborn?
- **Statistical Visualization:** Seaborn inherently understands statistical concepts. It automatically computes and plots confidence intervals, aggregations, and distributions.
- **Simplicity over Matplotlib:** Creating a grouped bar chart with error bars takes 20+ lines in Matplotlib but only 1 line in Seaborn.
- **Data-Aware API:** Seaborn operates directly on Pandas DataFrames, natively understanding column names and index structures.
- **Aesthetic Defaults:** Seaborn replaces Matplotlib's default styling with modern, publication-ready themes.

### Relationship with Matplotlib
Seaborn is not a replacement for Matplotlib; it is an extension. Every Seaborn plot is fundamentally a Matplotlib plot. 
- Seaborn creates Matplotlib `Figure` and `Axes` objects.
- You can (and often must) use Matplotlib functions to fine-tune Seaborn plots (e.g., `plt.title()`, `ax.set_xticks()`).

### Comparison with Matplotlib and Plotly

| Feature | Matplotlib | Seaborn | Plotly |
| :--- | :--- | :--- | :--- |
| **Paradigm** | Imperative (How to draw) | Declarative (What to draw) | Declarative / Interactive |
| **Interactivity**| Static (mostly) | Static | Highly Interactive |
| **Verbosity** | High | Low | Medium |
| **Statistical** | Requires manual calculation | Built-in | Requires manual calculation |
| **3D Plotting** | Basic support | No native support | Excellent support |

### Design Philosophy
Seaborn's design philosophy centers on the **dataset**. It assumes that you have a "tidy" (long-form) dataset where each column is a variable and each row is an observation. The API is designed to map these variables to visual aesthetics (x, y, hue, size, style).

### Version Evolution
- **Pre-0.11:** Relied heavily on `distplot`, `factorplot`. 
- **0.11.0:** Overhauled distribution plots (`histplot`, `kdeplot`, `ecdfplot`, `displot`).
- **0.12.0:** Introduced the **Objects API** (`seaborn.objects`), a completely new interface inspired by the Grammar of Graphics (similar to ggplot2 in R).

💡 **Pro Tip:** Always check your Seaborn version (`sns.__version__`). The API has changed significantly, and older tutorials using `distplot` or `factorplot` are deprecated.

⚠️ **Common Pitfalls:** Trying to build interactive dashboards using only Seaborn. Seaborn generates static images. For interactive web dashboards, use Plotly, Bokeh, or Altair instead.

---

## 2. INSTALLATION & SETUP

### Installation
Seaborn can be installed via pip or conda. It automatically installs dependencies like NumPy, Pandas, and Matplotlib.

```bash
# Using pip
pip install seaborn

# Using conda
conda install seaborn -c conda-forge
```

### Import Conventions
The community standard is to import Seaborn as `sns` (a nod to the character Samuel Norman Seaborn from The West Wing).

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Verify version
print(f"Seaborn Version: {sns.__version__}")
```

### Setting Themes
Seaborn can completely override Matplotlib's default visual parameters using `set_theme()`.

```python
# Apply the default Seaborn theme, scaling, and color palette
sns.set_theme(
    context='notebook',   # Scale of plot elements (paper, notebook, talk, poster)
    style='darkgrid',     # Background style (darkgrid, whitegrid, dark, white, ticks)
    palette='deep',       # Color palette mapping
    font='sans-serif',    # Font family
    font_scale=1.0,       # Base font scaling factor
    color_codes=True,     # Map shorthand color codes (e.g., 'b', 'g') to the palette
    rc=None               # Override specific matplotlib rcParams
)

# Create a dummy plot to see the theme
plt.plot([1, 2, 3], [4, 5, 2])
plt.show()
```

### Default Styles and Contexts
- **Styles (`style`)**: Dictate the background and gridlines. `whitegrid` is best for plots with heavy data elements, while `darkgrid` works well for sparse scatter plots.
- **Contexts (`context`)**: Control the size of labels, lines, and markers. 
  - `paper`: Smallest, for reports.
  - `notebook`: Default, for Jupyter notebooks.
  - `talk`: For presentations.
  - `poster`: For large printed posters.

### Integration with Jupyter
To ensure plots render inline within a Jupyter Notebook, use the IPython magic command:

```python
# This must be run in a Jupyter cell to render output directly below the cell
%matplotlib inline

# For interactive Matplotlib widgets in Jupyter (optional, rarely used with pure Seaborn)
# %matplotlib notebook 
```

💡 **Pro Tip:** If you only want to change the style temporarily for a specific plot without affecting the global state, use the `axes_style()` context manager:
```python
with sns.axes_style("white"):
    sns.scatterplot(x=[1,2], y=[1,2])
```

⚠️ **Common Pitfalls:** Calling `sns.set_theme()` multiple times with conflicting parameters can lead to unexpected Matplotlib `rcParams` state. Call it once at the top of your script/notebook.

---

## 3. DATASETS & DATA HANDLING

### Built-in Datasets
Seaborn provides access to several sample datasets hosted on GitHub. These are excellent for testing and prototyping.

```python
# List all available dataset names
dataset_names = sns.get_dataset_names()
print(dataset_names[:5])  # ['anagrams', 'anscombe', 'attention', 'brain_networks', 'car_crashes']

# Load a dataset as a Pandas DataFrame
tips = sns.load_dataset('tips')
iris = sns.load_dataset('iris')
titanic = sns.load_dataset('titanic')

# Inspect the data
print(tips.head())
```

### Working with Pandas DataFrames
Seaborn thrives on Pandas DataFrames. You specify the DataFrame in the `data` parameter and pass the column names as strings to the `x`, `y`, `hue`, etc., parameters.

```python
# Example: Using explicit Pandas columns
sns.scatterplot(data=tips, x='total_bill', y='tip', hue='smoker')
plt.show()
```

### Long-form vs Wide-form Data

**Long-form Data (Tidy Data):**
- Each variable forms a column.
- Each observation forms a row.
- Seaborn prefers this format. `tips` is long-form.

**Wide-form Data:**
- Each row represents an entity, and columns represent repeated measurements or categories.
- Example: A correlation matrix or pivot table.

Seaborn functions can handle wide-form data, usually by treating the index as the x-axis and each column as a separate line/series.

```python
# Creating wide-form data
np.random.seed(42)
wide_data = pd.DataFrame(
    np.random.randn(100, 3).cumsum(axis=0), 
    columns=['Series 1', 'Series 2', 'Series 3']
)

# Seaborn automatically plots each column as a separate line
sns.lineplot(data=wide_data)
plt.title("Wide-form Data Plot")
plt.show()
```

### Data Preprocessing Basics
Often, you need to melt wide-form data into long-form using Pandas before feeding it into Seaborn, especially for complex categorical plots.

```python
# Melting wide data to long data
long_data = wide_data.melt(
    var_name='Series Name', 
    value_name='Value', 
    ignore_index=False
).reset_index()

# Now we can explicitly map aesthetics
sns.lineplot(data=long_data, x='index', y='Value', hue='Series Name')
plt.show()
```

💡 **Pro Tip:** If your plot looks like a tangled mess or Seaborn throws an error about dimensions, your data is likely wide-form when Seaborn expects long-form. Use `pd.melt()` to fix it.

⚠️ **Common Pitfalls:** Passing NumPy arrays or lists instead of Pandas DataFrames. While Seaborn *can* accept them (e.g., `sns.scatterplot(x=[1,2], y=[3,4])`), you lose the automatic axis labeling and legend generation that DataFrames provide.

---

## 4. BASIC PLOTS

### scatterplot()
Used to show the relationship between two continuous variables.

```python
plt.figure(figsize=(8, 6))
# scatterplot maps data columns to visual properties
sns.scatterplot(
    data=tips, 
    x='total_bill', 
    y='tip', 
    hue='time',      # Color points by time (Lunch/Dinner)
    style='smoker',  # Change marker shape by smoker status
    size='size',     # Change marker size by party size
    sizes=(20, 200), # Define min and max sizes for the 'size' mapping
    palette='Set1',  # Use a specific color palette
    alpha=0.8        # Transparency to handle mild overplotting
)
plt.title("Scatterplot: Bill vs Tip")
plt.show()
```

### lineplot()
Used to show the relationship between two continuous variables, typically where one is time or a sequence. By default, `lineplot` aggregates multiple y values at each x value and plots a confidence interval.

```python
# Load time series dataset
fmri = sns.load_dataset("fmri")

plt.figure(figsize=(8, 6))
sns.lineplot(
    data=fmri, 
    x="timepoint", 
    y="signal", 
    hue="region", 
    style="event",
    errorbar=("ci", 95), # Show 95% confidence interval (default)
    # errorbar="sd",     # Alternatively, show standard deviation
    # estimator=None     # Disable aggregation, plot all raw lines
)
plt.title("Lineplot with Confidence Intervals")
plt.show()
```

### relplot()
`relplot()` is the figure-level equivalent of `scatterplot` and `lineplot`. It is used for **relational** plots and automatically creates multiple subplots (facets) if you use the `col` or `row` arguments.

```python
# Figure-level function: Returns a FacetGrid, not an Axes object
g = sns.relplot(
    data=tips,
    x="total_bill", 
    y="tip",
    col="time",      # Create separate columns for Lunch and Dinner
    row="sex",       # Create separate rows for Male and Female
    hue="smoker",
    kind="scatter",  # Can be "scatter" or "line"
    height=4,        # Height of each individual facet
    aspect=1.2       # Aspect ratio of each facet (width = height * aspect)
)
g.fig.suptitle("Relplot Grid", y=1.05) # Add a title to the entire figure
plt.show()
```

### Customizing labels and titles
Since Seaborn returns Matplotlib objects, you use Matplotlib functions to customize the text.

```python
ax = sns.scatterplot(data=tips, x='total_bill', y='tip')
ax.set_title("Customized Title", fontsize=16, fontweight='bold')
ax.set_xlabel("Total Bill (USD)", fontsize=12)
ax.set_ylabel("Tip Amount (USD)", fontsize=12)
```

💡 **Pro Tip:** Use `scatterplot()` or `lineplot()` (Axes-level functions) when you want to place the plot inside an existing complex layout using `plt.subplots()`. Use `relplot()` (Figure-level function) when you want Seaborn to handle the layout grid for you.

⚠️ **Common Pitfalls:** Trying to pass an `ax` argument to `relplot()`. Figure-level functions create their own figures and axes and do not accept an `ax` parameter.

---

## 5. DISTRIBUTION PLOTS

Understanding the distribution of variables is a core step in Exploratory Data Analysis (EDA).

### histplot()
Replaced the deprecated `distplot`. It provides extensive control over histograms.

```python
penguins = sns.load_dataset("penguins")

plt.figure(figsize=(8, 5))
sns.histplot(
    data=penguins,
    x="flipper_length_mm",
    hue="species",
    multiple="stack",     # "layer" (default), "stack", "fill", or "dodge"
    bins=30,              # Number of bins
    kde=True,             # Add a Kernel Density Estimate line
    palette="viridis",
    log_scale=False       # Set to True for logarithmic bins
)
plt.title("Stacked Histogram of Flipper Lengths")
plt.show()
```

### kdeplot()
Plots the Kernel Density Estimate, which is a smoothed, continuous version of a histogram. 

```python
plt.figure(figsize=(8, 5))
# 2D KDE Plot (bivariate)
sns.kdeplot(
    data=penguins, 
    x="bill_length_mm", 
    y="bill_depth_mm", 
    hue="species",
    fill=True,            # Fill the area under the curve
    alpha=0.5,            # Transparency
    levels=5,             # Number of contour levels
    thresh=0.1            # Lowest contour level to draw
)
plt.title("Bivariate KDE Plot")
plt.show()
```

### rugplot()
Draws a small vertical tick at each observation. Often combined with a scatterplot or kdeplot to show the exact location of data points.

```python
plt.figure(figsize=(8, 5))
sns.kdeplot(data=tips, x="total_bill")
# Add a rugplot to the existing axes
sns.rugplot(data=tips, x="total_bill", height=0.1, color="red")
plt.title("KDE with Rugplot")
plt.show()
```

### displot()
The figure-level equivalent for distribution plots.

```python
sns.displot(
    data=penguins, 
    x="flipper_length_mm", 
    hue="species", 
    col="island",     # Facet by island
    kind="kde",       # "hist", "kde", or "ecdf"
    fill=True,
    height=4,
    aspect=1.5
)
plt.show()
```

💡 **Pro Tip:** When using `histplot` with multiple categories (`hue`), the default `multiple="layer"` can obscure data. Using `multiple="stack"` or `multiple="dodge"` (side-by-side) often yields a clearer comparison. 

⚠️ **Common Pitfalls:** Trusting KDE plots with very small sample sizes. KDE mathematically interpolates data and can invent "tails" or peaks that don't exist in reality if the data is sparse. Use `rugplot` alongside it to verify.

---

## 6. CATEGORICAL PLOTS

Categorical plots handle scenarios where one of your main variables is discrete/categorical.

### barplot()
Shows an aggregate observation (by default, the mean) and confidence intervals for categorical data.

```python
plt.figure(figsize=(8, 5))
sns.barplot(
    data=tips, 
    x="day", 
    y="total_bill", 
    hue="sex",
    estimator=np.median,  # Change aggregate from mean to median
    errorbar=("ci", 95),  # 95% Bootstrap confidence interval
    capsize=0.1,          # Add caps to the error bars
    palette="muted"
)
plt.title("Barplot: Median Bill by Day and Sex")
plt.show()
```

### countplot()
A special case of `barplot` where the estimator is explicitly `len` (i.e., counts the number of occurrences).

```python
plt.figure(figsize=(8, 5))
# Only x OR y should be specified, not both
sns.countplot(
    data=titanic, 
    x="class", 
    hue="survived", 
    palette="pastel"
)
plt.title("Passenger Count by Class and Survival")
plt.show()
```

### boxplot()
Shows the distribution of quantitative data in a way that facilitates comparisons between variables or across levels of a categorical variable. Shows median, quartiles, and outliers.

```python
plt.figure(figsize=(8, 5))
sns.boxplot(
    data=tips, 
    x="day", 
    y="total_bill", 
    hue="smoker",
    showfliers=True,      # Show outlier points
    notch=True            # Add notches representing CI of the median
)
plt.title("Boxplot of Total Bill")
plt.show()
```

### violinplot()
Combines a boxplot with a KDE plot. Shows the full distribution of the data.

```python
plt.figure(figsize=(8, 5))
sns.violinplot(
    data=tips, 
    x="day", 
    y="total_bill", 
    hue="sex",
    split=True,           # Draw half of a violin for each hue level
    inner="quartile",     # Show quartiles inside the violin
    palette="Set2"
)
plt.title("Split Violinplot")
plt.show()
```

### stripplot() and swarmplot()
Plot all individual data points. `stripplot` adds random jitter, while `swarmplot` prevents points from overlapping.

```python
plt.figure(figsize=(8, 5))
# Base violinplot
sns.violinplot(data=tips, x="day", y="total_bill", color=".8", inner=None)

# Overlay swarmplot
sns.swarmplot(
    data=tips, 
    x="day", 
    y="total_bill", 
    hue="smoker", 
    size=4,
    dodge=True           # Separate the swarms by hue
)
plt.title("Violinplot with Swarmplot Overlay")
plt.show()
```

### catplot()
The figure-level categorical function.

```python
sns.catplot(
    data=titanic, 
    x="sex", 
    y="survived", 
    col="class", 
    kind="bar",       # "strip", "swarm", "box", "violin", "boxen", "point", "bar", or "count"
    height=4, 
    aspect=0.8
)
plt.show()
```

💡 **Pro Tip:** Boxplots hide multi-modal distributions (bimodal data looks like a normal boxplot). Violin plots reveal multi-modality but are hard to read for small datasets. Overlaying a `swarmplot` on a `violinplot` provides the ultimate transparency.

⚠️ **Common Pitfalls:** Assuming error bars on `barplot` represent standard deviation. In Seaborn, they default to 95% bootstrapped confidence intervals. To show standard deviation, set `errorbar="sd"`.

---

## 7. MATRIX PLOTS

Matrix plots allow you to plot data as color-encoded matrices, which is ideal for correlation matrices or heatmaps of 2D data.

### heatmap()
Plots rectangular data as a color-encoded matrix.

```python
# Generate a correlation matrix using pandas
corr = mtcars.corr() if 'mtcars' in locals() else sns.load_dataset('mpg').drop(columns=['origin', 'name']).corr()

plt.figure(figsize=(10, 8))
sns.heatmap(
    data=corr,
    annot=True,           # Write data values in each cell
    fmt=".2f",            # String formatting code (2 decimal places)
    cmap="coolwarm",      # Colormap (diverging works best for correlations)
    center=0,             # Set the center of the colormap to 0
    vmin=-1, vmax=1,      # Set explicit min and max bounds for color scaling
    linewidths=0.5,       # Add lines between cells
    linecolor="white",
    cbar_kws={"shrink": .8} # Shrink colorbar slightly
)
plt.title("Correlation Heatmap")
plt.show()
```

### clustermap()
Plots a matrix dataset as a hierarchically-clustered heatmap. It automatically performs agglomerative clustering on the rows and columns.

```python
iris_features = iris.drop("species", axis=1)

# Note: clustermap is a figure-level function
g = sns.clustermap(
    data=iris_features,
    cmap="viridis",
    standard_scale=1,     # Standardize data across columns (1) or rows (0)
    # z_score=1,          # Alternatively, calculate Z-scores
    metric="euclidean",   # Distance metric for clustering
    method="ward",        # Linkage method
    figsize=(8, 8)
)
g.fig.suptitle("Clustermap of Iris Features", y=1.05)
plt.show()
```

💡 **Pro Tip:** When plotting correlations, always use a **diverging colormap** (like `coolwarm`, `vlag`, or `icefire`) and set `center=0`. This ensures that negative correlations are one color (e.g., blue), positive are another (e.g., red), and zero correlation is neutral (e.g., white).

⚠️ **Common Pitfalls:** Feeding unscaled data with vastly different magnitudes into `clustermap()`. Features with large numbers will dominate the distance metrics. Always use `standard_scale=1` or `z_score=1` if your columns represent different units.

---

## 8. REGRESSION & STATISTICAL PLOTS

Seaborn can quickly fit and plot linear regression models alongside your data.

### regplot()
Plots data and a linear regression model fit.

```python
plt.figure(figsize=(8, 5))
sns.regplot(
    data=tips, 
    x="total_bill", 
    y="tip",
    scatter_kws={"color": "blue", "alpha": 0.5}, # Kwargs for the scatter points
    line_kws={"color": "red", "linewidth": 2},   # Kwargs for the regression line
    ci=95,                # Confidence interval size (0 disables it)
    order=1               # Order of polynomial (1 = linear)
)
plt.title("Linear Regression: Total Bill vs Tip")
plt.show()
```

### lmplot()
The figure-level equivalent of `regplot()`. Crucially, `lmplot` allows for grouping via `hue`, `col`, and `row`, which `regplot` does not.

```python
sns.lmplot(
    data=anscombe := sns.load_dataset("anscombe"),
    x="x", 
    y="y", 
    col="dataset", 
    hue="dataset",
    col_wrap=2,           # Wrap to a new row after 2 columns
    height=4,
    ci=None               # Disable CI calculation for speed
)
plt.show()
```

### Advanced Regression Options
Seaborn supports several advanced regression parameters via statsmodels (which must be installed).

```python
plt.figure(figsize=(8, 5))
# Polynomial regression (e.g., quadratic)
sns.regplot(data=anscombe.query("dataset == 'II'"), x="x", y="y", order=2, color="green", label="Quadratic (order=2)")

# Robust regression (ignores outliers)
sns.regplot(data=anscombe.query("dataset == 'III'"), x="x", y="y", robust=True, color="purple", label="Robust")

plt.legend()
plt.title("Advanced Regression Techniques")
plt.show()
```

### residplot()
Plots the residuals of a linear regression. Ideally, residuals should be randomly scattered around the y=0 line. If you see a pattern (like a U-shape), a linear model is not appropriate.

```python
plt.figure(figsize=(8, 5))
sns.residplot(
    data=tips, 
    x="total_bill", 
    y="tip", 
    lowess=True,          # Add a locally weighted scatterplot smoothing line
    line_kws={"color": "red"}
)
plt.title("Residual Plot")
plt.show()
```

💡 **Pro Tip:** Seaborn's regression plots are for visual exploration, not rigorous statistical modeling. To extract the actual coefficients, p-values, or R-squared metrics, you must use `scikit-learn` or `statsmodels` separately.

⚠️ **Common Pitfalls:** Using linear regression (`order=1`) on clearly non-linear data. Always inspect your scatterplots first or use `residplot` to check model assumptions.

---

## 9. MULTI-PLOT GRIDS

Grids allow you to visualize multi-dimensional relationships by creating small multiples (facets).

### FacetGrid
The foundational class behind `relplot`, `catplot`, and `lmplot`. You can use it manually to map custom Matplotlib or Seaborn functions across a grid.

```python
# 1. Initialize the grid, defining the dataset and faceting variables
g = sns.FacetGrid(
    data=tips, 
    col="time", 
    row="sex", 
    hue="smoker",
    margin_titles=True # Put row titles on the right margin
)

# 2. Map a plotting function to each facet
# The first argument is the function (plt.scatter, sns.histplot, etc.)
# Subsequent arguments are column names from the dataset
g.map(sns.scatterplot, "total_bill", "tip", alpha=0.7)

# 3. Add a legend and adjust titles
g.add_legend()
g.set_axis_labels("Total Bill", "Tip")
g.fig.subplots_adjust(top=0.9)
g.fig.suptitle("Manual FacetGrid Mapping")
plt.show()
```

### PairGrid and pairplot()
Plots pairwise relationships across an entire dataset. `pairplot` is a high-level wrapper, while `PairGrid` offers manual control.

```python
# pairplot (easy way)
sns.pairplot(
    data=iris, 
    hue="species", 
    diag_kind="kde",      # Distribution plots on the diagonal (kde or hist)
    plot_kws={"alpha": 0.6, "s": 50}, # Pass args to bivariate plots
    height=2.5
)
plt.show()

# PairGrid (manual way, for fine-grained control)
g = sns.PairGrid(iris, hue="species")
g.map_diag(sns.histplot, multiple="stack")     # Diagonal gets histograms
g.map_upper(sns.scatterplot, s=20)             # Upper triangle gets scatter
g.map_lower(sns.kdeplot, fill=True, alpha=0.5) # Lower triangle gets 2D KDE
g.add_legend()
plt.show()
```

### JointGrid and jointplot()
Combines a bivariate scatter/hex/kde plot with univariate marginal distributions on the top and right axes.

```python
# jointplot (easy way)
sns.jointplot(
    data=penguins, 
    x="bill_length_mm", 
    y="bill_depth_mm", 
    hue="species",
    kind="scatter",       # "scatter", "kde", "hist", "hex", "reg", "resid"
    height=7,
    marginal_ticks=True   # Show ticks on the marginal axes
)
plt.show()
```

💡 **Pro Tip:** `pairplot` is incredibly powerful for initial EDA, but on datasets with >10 columns or >100,000 rows, it will take a massive amount of time and memory to render. Select a subset of columns first: `sns.pairplot(df[['col1', 'col2', 'target']], hue='target')`.

⚠️ **Common Pitfalls:** Forgetting that `map()` requires the function to accept `color` and `label` kwargs if `hue` is used in the grid initialization. Standard Matplotlib functions handle this, but custom functions might break.

---

## 10. STYLING & THEMES

Seaborn allows you to globally or locally control the aesthetics of Matplotlib.

### Core Styling Functions

```python
# 1. set_style: Controls background and grid
sns.set_style("whitegrid") # options: darkgrid, whitegrid, dark, white, ticks

# 2. set_context: Controls scale (font sizes, line widths)
sns.set_context("talk", font_scale=1.2) # options: paper, notebook, talk, poster

# 3. set_palette: Controls default colors
sns.set_palette("husl") 
```

### Custom Color Palettes
You can define custom palettes using lists of hex codes, or generate them using Seaborn's palette functions.

```python
# Custom hex palette
my_colors = ["#FF5733", "#33FF57", "#3357FF"]
sns.set_palette(my_colors)

# Generating a palette
# Return a list of colors defining a colorblind-friendly palette
cb_palette = sns.color_palette("colorblind", n_colors=5)

# View the palette
sns.palplot(cb_palette)
plt.show()
```

### Despine
Removing the top and right borders (spines) makes plots look cleaner and more modern.

```python
sns.set_style("ticks")
plt.figure(figsize=(6, 4))
sns.scatterplot(x=[1,2,3], y=[3,1,2])

# Remove top and right spines
sns.despine(
    offset=10,        # Move spines away from the data
    trim=True         # Limit spine range to data range
)
plt.title("Despined Plot")
plt.show()

# Reset to defaults for next sections
sns.set_theme()
```

💡 **Pro Tip:** Building a branded internal dashboard? Create a list of your company's official hex codes, apply it via `sns.set_palette(company_hex_list)`, and set your company font via `sns.set_theme(font="CompanyFont")`.

⚠️ **Common Pitfalls:** Using `despine()` before plotting. `sns.despine()` acts on the *current active axes*. You must call it *after* you have drawn your plot.

---

## 11. CUSTOMIZATION

Since Seaborn returns standard Matplotlib `Axes` (or `Figure` for grid functions), you can use Matplotlib to fully customize the plot.

### Manipulating Axes Objects

```python
fig, ax = plt.subplots(figsize=(10, 6))

# Pass the specific ax to seaborn
sns.barplot(data=tips, x="day", y="total_bill", ax=ax, color="skyblue")

# 1. Title and Labels
ax.set_title("Total Revenue by Day", fontsize=18, pad=20)
ax.set_xlabel("Day of the Week", fontsize=14, labelpad=10)
ax.set_ylabel("Revenue ($)", fontsize=14, labelpad=10)

# 2. Tick Formatting
import matplotlib.ticker as ticker
# Format y-axis as currency
formatter = ticker.FormatStrFormatter('$%1.0f')
ax.yaxis.set_major_formatter(formatter)

# 3. Rotating X-ticks (useful for long labels)
ax.tick_params(axis='x', rotation=45)

# 4. Adding Annotations
# Annotate the highest bar
max_val = tips.groupby("day")["total_bill"].mean().max()
ax.annotate(
    f"Peak: ${max_val:.2f}",
    xy=(3, max_val),           # Data coordinates to point to (Sunday = index 3)
    xytext=(2, max_val + 5),   # Text location
    arrowprops=dict(facecolor='black', arrowstyle="->")
)

plt.tight_layout()
plt.show()
```

### Legend Manipulation
Seaborn legends can sometimes overlap data or appear in sub-optimal locations.

```python
ax = sns.scatterplot(data=tips, x="total_bill", y="tip", hue="day")

# Move legend completely outside the plot
sns.move_legend(
    ax, 
    "center left",        # Position
    bbox_to_anchor=(1, 0.5), # Anchor coordinate (x=1 right edge, y=0.5 middle)
    title="Day of Week",
    frameon=False         # Remove border box
)
plt.show()
```

💡 **Pro Tip:** `sns.move_legend` (introduced in v0.11.2) is significantly easier than the old Matplotlib approach (`ax.legend(bbox_to_anchor=...)`), as it preserves the legend title automatically.

⚠️ **Common Pitfalls:** Trying to use `ax.set_title()` on a Figure-level plot like `relplot`. Because `relplot` returns a `FacetGrid`, you must use `g.fig.suptitle()` for the main title, or `g.set_titles()` for subplot titles.

---

## 12. COLOR THEORY IN SEABORN

Choosing the correct colormap is crucial for accurate data representation. Seaborn categorizes palettes into three types:

### 1. Sequential Palettes
Use for continuous data that progresses from low to high (e.g., density, count, temperature). Colors transition from light to dark or across a spectrum.
- **Names:** `Blues`, `Reds`, `viridis`, `plasma`, `magma`, `flare`, `crest`

```python
plt.figure(figsize=(6, 4))
sns.kdeplot(data=tips, x="total_bill", y="tip", fill=True, cmap="mako")
plt.title("Sequential Colormap (Mako)")
plt.show()
```

### 2. Diverging Palettes
Use for continuous data that has a meaningful central point (e.g., correlation, temperature deviations from 0, election swings). Colors transition from one extreme, through a neutral center, to another extreme.
- **Names:** `vlag`, `icefire`, `coolwarm`, `RdBu_r`, `Spectral`

```python
plt.figure(figsize=(6, 4))
np.random.seed(0)
data = np.random.randn(10, 10) # Centered around 0
sns.heatmap(data, cmap="vlag", center=0)
plt.title("Diverging Colormap (Vlag)")
plt.show()
```

### 3. Qualitative Palettes
Use for categorical data where no logical order exists (e.g., categories, species, regions). Colors are distinct and non-sequential.
- **Names:** `deep`, `muted`, `pastel`, `bright`, `dark`, `colorblind`, `Set1`, `Set2`

```python
plt.figure(figsize=(6, 4))
sns.countplot(data=tips, x="day", palette="colorblind")
plt.title("Qualitative Colormap (Colorblind)")
plt.show()
```

💡 **Pro Tip:** Never use a rainbow/jet colormap for sequential data. It has inconsistent luminance mapping, creating visual artifacts (false edges) that don't exist in the data. Stick to perceptually uniform palettes like `viridis` or `mako`.

⚠️ **Common Pitfalls:** Using a sequential palette for diverging data (e.g., plotting correlation matrices with `Blues`). It makes it impossible to visually distinguish between negative correlation and zero correlation.

---

## 13. WORKING WITH TIME SERIES

Seaborn inherently supports datetime objects via Pandas.

```python
# Generate dummy time series data
dates = pd.date_range("2023-01-01", periods=100, freq="D")
ts_data = pd.DataFrame({
    "Date": dates,
    "Stock_A": np.cumsum(np.random.randn(100)) + 100,
    "Stock_B": np.cumsum(np.random.randn(100)) + 100
})

# Melt for seaborn
ts_melted = ts_data.melt(id_vars="Date", var_name="Stock", value_name="Price")

plt.figure(figsize=(10, 5))
ax = sns.lineplot(data=ts_melted, x="Date", y="Price", hue="Stock")

# Auto-format datetime x-axis (Matplotlib feature)
import matplotlib.dates as mdates
ax.xaxis.set_major_locator(mdates.MonthLocator())
ax.xaxis.set_major_formatter(mdates.DateFormatter('%b %Y'))
plt.xticks(rotation=45)

plt.title("Time Series Plot with Datetime Formatting")
plt.tight_layout()
plt.show()
```

### Aggregation Over Time
If you have multiple observations per timestamp (e.g., multiple sales per day), `lineplot` will automatically plot the mean and the 95% confidence interval area. To show rolling averages, calculate them in Pandas first.

```python
# Calculate a 7-day rolling average in Pandas
ts_data['Stock_A_Rolling'] = ts_data['Stock_A'].rolling(window=7).mean()
```

💡 **Pro Tip:** When dealing with heavy, dense time series, `lineplot` can become slow. If you don't need confidence interval aggregation (i.e., you have exactly one observation per timestamp), pass `estimator=None` to massively speed up rendering.

---

## 14. PERFORMANCE & BEST PRACTICES

### Handling Large Datasets (Overplotting)
Plotting a scatterplot with 1,000,000 points will result in a solid blob of color and slow rendering.

**Solutions:**
1. **Alpha Blending:** Make points nearly transparent. `alpha=0.01`.
2. **Subsampling:** Plot a random sample of the data. `df.sample(10000)`.
3. **Hexbin Plots:** Bin the data into hexagons and color by count.

```python
# Generate large dataset
x = np.random.normal(size=50000)
y = x + np.random.normal(size=50000)

# Inefficient/Overplotted:
# sns.scatterplot(x=x, y=y) 

# Efficient: Hexbin plot
sns.jointplot(x=x, y=y, kind="hex", color="purple")
plt.show()

# Efficient: 2D Histogram
sns.displot(x=x, y=y, binwidth=(0.1, 0.1), cbar=True)
plt.show()
```

### Best Practices for Readability
- **Data-to-Ink Ratio:** Minimize gridlines, borders, and unnecessary colors. Maximize the ink used to represent data. Use `sns.despine()`.
- **Color Dependency:** Ensure your plot is understandable without color. Use `style` (markers/dashes) in addition to `hue` in `lineplot` or `scatterplot`.
- **Direct Labeling:** Instead of a legend, consider annotating lines directly at their endpoints. It reduces cognitive load for the reader.

---

## 15. INTEGRATION

### Seaborn and Pandas
Seaborn accepts Pandas GroupBy objects indirectly. Usually, you `reset_index()` after grouping to restore columns for Seaborn.

```python
# Aggregate data in pandas
summary_df = tips.groupby(["day", "time"])["total_bill"].mean().reset_index()

# Plot the aggregated data
sns.barplot(data=summary_df, x="day", y="total_bill", hue="time")
plt.show()
```

### Seaborn in Dashboards (Streamlit example)
While Seaborn is static, it can easily be embedded in web apps like Streamlit or Flask by passing the `fig` object.

```python
# Example Streamlit integration code
import streamlit as st
fig, ax = plt.subplots()
sns.histplot(data=tips, x="total_bill", ax=ax)
st.pyplot(fig)  # Renders the matplotlib figure in the web app
```

---

## 16. COMMON PITFALLS SUMMARY

1. **Statefulness:** Forgetting that Matplotlib/Seaborn uses a global state machine. If you don't call `plt.figure()` or `plt.clf()` between plots in a loop, your graphics will draw on top of each other.
2. **Missing `plt.show()`:** Scripts (unlike Jupyter notebooks) require `plt.show()` to render the window.
3. **Axes vs Figure level functions:** 
   - `sns.boxplot(ax=ax1)` -> Works.
   - `sns.catplot(ax=ax1)` -> Fails. Returns a `FacetGrid`.
4. **Data Type Coercion:** If you pass numbers as categories (e.g., years 2010, 2011), `lineplot` treats them continuously, but `barplot` treats them categorically. If you want a categorical X-axis, ensure the pandas column is of type string or `category`.

---

## 17. ADVANCED CUSTOMIZATION

### Custom Plotting Functions in FacetGrid
You can map completely custom functions to a `FacetGrid` as long as they adhere to the expected signature `func(x, y, **kwargs)`.

```python
def custom_scatter_line(x, y, **kwargs):
    """Draws a scatter point and drops a vertical line to the x-axis"""
    ax = plt.gca()
    # Handle color from kwargs passed by FacetGrid
    color = kwargs.get('color', 'blue') 
    ax.scatter(x, y, color=color, alpha=0.5)
    for px, py in zip(x, y):
        ax.plot([px, px], [0, py], color=color, alpha=0.2)

g = sns.FacetGrid(tips, col="time", hue="smoker", height=4)
g.map(custom_scatter_line, "total_bill", "tip")
g.add_legend()
plt.show()
```

### Accessing Internal Data
Sometimes you need to know exactly what Seaborn computed (e.g., the exact values of the KDE curve). You can extract this from the Matplotlib `Axes` object after plotting.

```python
ax = sns.kdeplot(data=tips, x="total_bill")
# Extract the Line2D object drawn by KDE
line = ax.lines[0]
x_data = line.get_xdata()
y_data = line.get_ydata()
print(f"Max density occurs at: x={x_data[np.argmax(y_data)]:.2f}")
```

---

## 18. INTERACTIVE VISUALIZATION (LIMITATIONS)

Seaborn produces static rasters (PNG) or vectors (SVG, PDF). It cannot natively produce HTML with hover tooltips, zooming, or panning.

**Workarounds:**
1. **Plotly:** If you need interactivity, rewrite the plot using `plotly.express`, which has an API heavily inspired by Seaborn.
2. **mplcursors:** A library that adds interactive data cursors (tooltips) to static Matplotlib/Seaborn plots.

```python
import mplcursors
ax = sns.scatterplot(data=tips, x="total_bill", y="tip")
# This allows you to click on points in the matplotlib window to see values
mplcursors.cursor(ax.collections)
plt.show()
```

---

## 19. PUBLICATION-QUALITY PLOTS

To export plots for academic papers or professional reports, you must control resolution and bounding boxes.

```python
plt.figure(figsize=(10, 6))
sns.boxplot(data=tips, x="day", y="total_bill")
plt.title("Publication Plot")

# Save as vector graphic (PDF/SVG) for infinite scaling
plt.savefig("output_plot.pdf", format="pdf", bbox_inches="tight")

# Save as high-res raster graphic (PNG)
plt.savefig(
    "output_plot.png", 
    dpi=300,               # Dots per inch (300 is standard print quality)
    bbox_inches="tight",   # Prevents labels from being cut off
    transparent=False      # True for transparent background
)
```

💡 **Pro Tip:** Always use `bbox_inches="tight"`. Otherwise, if you move legends outside the plot or have long rotated x-labels, they will be clipped off in the exported image.

---

## 20. ADVANCED TOPICS: THE OBJECTS API (SO)

Introduced in Seaborn v0.12, the `seaborn.objects` module provides a completely new, composable API inspired by the Grammar of Graphics. It allows you to build complex plots by layering geometries (`Mark`) and statistical transformations (`Stat`).

```python
import seaborn.objects as so

# Create a base plot definition mapped to data
p = so.Plot(tips, x="total_bill", y="tip", color="smoker")

# Compose the plot by adding layers
(
    p
    # Layer 1: Scatter points
    .add(so.Dot(alpha=0.5))
    # Layer 2: A linear regression line per group
    .add(so.Line(linewidth=2), so.PolyFit(order=1))
    # Facet the plot into columns by 'time'
    .facet("time")
    # Apply a theme scale
    .theme({"axes.facecolor": "white", "axes.edgecolor": "black"})
    .show()
)
```

### Why use the Objects API?
- **Composability:** You can add multiple different plot types (Marks) onto the same axes easily.
- **Explicit Stats:** Transformations like `so.Hist`, `so.Agg`, or `so.Dodge` are explicitly defined as objects, rather than hidden behind string parameters like `estimator="mean"`.
- **Reusability:** You can save a base `so.Plot` object and add different layers to it in different parts of your code.

```python
# Reusing visualization pipelines
base_hist = so.Plot(penguins, x="flipper_length_mm").facet("species")

# Pipeline branch A: Layered Area
p1 = base_hist.add(so.Area(alpha=0.5), so.Hist(), color="sex")

# Pipeline branch B: Dodged Bars
p2 = base_hist.add(so.Bar(), so.Hist(), so.Dodge(), color="sex")
```

---
*End of Seaborn Technical Reference. This document provides the foundational and advanced knowledge required to utilize Seaborn in a production data science environment.*
