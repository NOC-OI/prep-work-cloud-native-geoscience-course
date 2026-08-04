---
title: Working with data in Xarray
teaching: 60
exercises: 30
---


:::::::::::::::::::::::::::::::::::::::::: objectives

- "Understand the concept of lazy loading and how it helps work with datasets bigger than memory."
- "Understand whole-array operations and the performance advantages they bring."
- "Apply Xarray operations to load, manipulate, and visualise data."

::::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::: questions

- "How do I load data with Xarray?"
- "How does Xarray index data?"
- "How do I apply operations to the whole or part of an array?"
- "How do I work with time series data in Xarray?"
- "How do I visualise data from Xarray?"

::::::::::::::::::::::::::::::::::::::::::::::::::

## Introducing Xarray

**For this lesson, you need to have the setup completed and the example data downloaded. See the [setup instructions](../learners/setup.md) for details.**

Xarray is a Python library for working with multi-dimensional array data.
Many concepts are inspired by Pandas, but Xarray is designed to work well with very large array-based scientific datasets.
It integrates with core scientific Python libraries such as NumPy and Matplotlib, and supports data larger than memory through lazy evaluation and chunked workflows.

Xarray can read and write NetCDF and also supports other formats such as GRIB and Zarr.

A useful way to think about Xarray is:

- NumPy gives fast arrays.
- Pandas gives labeled 1D/2D tables.
- Xarray gives labeled N-dimensional arrays.

Those labels (for example `valid_time`, `latitude`, `longitude`) are the key feature that makes code easier to read and less error-prone.

## Datasets and DataArrays

Xarray has two core data types:

- `DataArray`: a single n-dimensional array with named dimensions and coordinates.
- `Dataset`: a collection of multiple `DataArray` objects plus metadata.

Because Xarray follows duck-typing conventions with NumPy-like APIs, many NumPy-style operations can be applied directly to Xarray objects.

As you work through this lesson:

- Use a `Dataset` when you have multiple related variables in one file.
- Use a `DataArray` when you want to work on one variable at a time.

Most operations in this lesson start from `dataset["sst"]`, which returns a `DataArray`.

## Opening a NetCDF Dataset

For this lesson, use the example dataset provided during setup:

- `data/era5_sst/ocean_temperature.nc`
- Main variable: `sst` (sea surface temperature)


To open the dataset, use `xarray.open_dataset`:

```python
import xarray as xr

dataset = xr.open_dataset("data/era5_sst/ocean_temperature.nc")
```

`open_dataset` reads metadata first and delays loading full data values until needed. This is called lazy loading.
It is important for large datasets because you can inspect structure before using memory for computations.

You can also pass optional arguments such as:

- `engine=...` to choose a backend reader.
- `chunks=...` to prepare Dask-backed parallel operations.

To inspect the dataset, simply type its name in a Jupyter notebook cell:

```python
dataset
print(dataset.attrs)
print(dataset.dims)
print(dataset.variables)
```

The `attrs` attribute contains metadata about the dataset, such as title, source, and history.
The `dims` attribute lists dimension names and sizes.
The `variables` attribute lists all variables, including coordinates and data variables.

To understand the file structure more clearly, inspect these explicitly:

```python
print(dataset.data_vars)
print(dataset.coords)
```

- `data_vars` are the main scientific variables (such as `sst`).
- `coords` are coordinate variables used for indexing (such as time and latitude).

## Accessing data variables

Access a variable:

```python
print(dataset["sst"])
```

As you can see, the `sst` variable has dimensions of `valid_time`, `latitude`, and `longitude`.

`dataset["sst"]` is usually the safest style because it is explicit and still works when variable names are not valid Python identifiers.

Access dimensions and elements:

```python
print(dataset["sst"]["valid_time"]) # prints the valid_time coordinate
print(dataset["sst"]["valid_time"][0]) # prints the first valid_time value
```

You can also use dot-style access to retrieve variables:

```python
print(dataset.sst)
```

Dot-style access is convenient, but bracket style is safer and more general.

## Indexing and slicing

There are two ways to index data in Xarray: by label and by integer index.

To index by label, use the `sel` method:

```python
dataset["sst"]["valid_time"].sel(valid_time="2025-01-01T00:00:00")
```

This will return the sea surface temperature at the specified timestamp.

`sel` matches coordinate labels, not integer positions.
It is the best choice when you care about physical meaning (specific dates, latitudes, longitudes).

To index by integer position, use the `isel` method:

```python
dataset["sst"]["valid_time"].isel(valid_time=0)
```

This will return the sea surface temperature at the first timestamp in the dataset.

`isel` is useful when you want "first", "last", or "every nth" item, independent of coordinate values.

You can also combine indexing methods to select data by both label and integer position:

```python
dataset["sst"].sel(valid_time="2025-01-01T00:00:00").isel(latitude=0, longitude=0)
```

You can also use standard Python slicing syntax to select ranges of data:

```python
dataset["sst"][:12] # or dataset.sst[:12]
dataset["sst"][::2] # every other time step
```

For label-based slices with `sel`, remember:

- The slice endpoints are coordinate labels.
- The stop label is usually included when present.

Slice by labels:

```python
dataset["sst"].sel(valid_time=slice("2025-01-01", "2025-01-02"))
```

To retrieve raw values, use `.values` (otherwise, Xarray returns a DataArray object):

```python
dataset["sst"].sel(valid_time=slice("2025-01-01", "2025-01-02")).values
```

This will return a NumPy array of the sea surface temperature values for the specified time range.

Use `.values` only when you really need a NumPy array.
Keeping data as Xarray objects preserves coordinate labels and metadata, which is often useful for later steps.

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 1: Slicing

1. Write a slicing command to get every other time step from the sea surface temperature dataset.
2. Write a slicing command to get the first 12 time steps from the sea surface temperature dataset, using `isel`.

::::::::::::::: solution

```python
dataset["sst"]["valid_time"][::2]
dataset["sst"].isel(valid_time=slice(0, 12))
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

### Nearest-neighbour lookups

A direct lookup fails when the timestamp is not present in the coordinate:

```python
dataset["sst"].sel(valid_time="2025-01-01T01:10:00")
```

This may raise an error because that exact timestamp is not present.

Use nearest-neighbour matching instead:

```python
dataset["sst"].sel(valid_time="2025-01-01T01:10:00", method="nearest")
```

You can add tolerance limits:

```python
dataset["sst"].sel(valid_time="2025-01-01T01:10:00", method="nearest", tolerance="30min")
dataset["sst"].sel(valid_time="2025-01-01T01:10:00", method="nearest", tolerance="2h")
```

`tolerance` is important in scientific workflows because it prevents accidental matching to points that are too far from your requested value.

## Plotting Xarray data

Plot a time series at one location:

```python
dataset["sst"].sel(latitude=-30, longitude=320, method="nearest").plot()
```

Xarray automatically uses metadata to label axes and variables.

`DataArray.plot()` chooses a sensible default plot type based on dimensions:

- 1D data -> line plot
- 2D data -> image/pseudocolor plot
- 1D flattened values -> histogram with `plot.hist()`

### Plotting two-dimensional data

Plot a 2D field:

```python
dataset["sst"].sel(valid_time="2025-01-01T00:00:00").plot()
```

Use Matplotlib options, such as a grayscale colormap:

```python
import matplotlib.pyplot as plt

dataset["sst"].sel(valid_time="2025-01-01T00:00:00").plot(cmap=plt.cm.Grays)
```

You can pass most Matplotlib-like keyword arguments (`cmap`, `vmin`, `vmax`, `figsize`) directly through Xarray plotting helpers.

### Plotting histograms

You can also plot a histogram of values:

```python
dataset["sst"].sel(valid_time="2025-01-01T00:00:00").plot.hist()
```

Histograms are useful for quick quality checks (for example, unrealistic value ranges or skewed distributions).

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 2: Slicing and plotting

Using a slice of the array, plot a transect of sea surface temperature across the Atlantic at 23 degrees North between 70 and 17 degrees West on:

1. 2025-01-01 00:00
2. 2025-01-03 00:00

Remember that the longitude values are in degrees East, so you will need to convert the West values to East.

::::::::::::::: solution

```python
dataset["sst"].sel(valid_time="2025-01-01T00:00:00", method="nearest").sel(longitude=slice(290, 343), latitude=23).plot()
dataset["sst"].sel(valid_time="2025-01-03T00:00:00", method="nearest").sel(longitude=slice(290, 343), latitude=23).plot()
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Array operations

### Map operations

Xarray supports vectorised, whole-array operations, which are usually faster and clearer than manual loops.

These operations are applied element-wise and preserve array alignment by dimension names.
This is one of Xarray's main safety advantages compared with manual indexing.

Apply an offset:

```python
dataset_corrected = dataset["sst"] - 1.0
```

This subtracts `1.0` from every element in the array.

Apply a linear correction:

```python
dataset_corrected = dataset["sst"] * 1.1 - 1.0
```

Chaining operations like this is common and keeps code concise.

Apply a custom function with `apply_ufunc`:

```python
def apply_correction(x):
    return x * 1.01 + 0.1

corrected_sst = xr.apply_ufunc(apply_correction, dataset["sst"])
```

Use `apply_ufunc` when you want to apply a custom function while still working with labeled data.
For many pure NumPy-style functions, direct operations (`+`, `-`, `*`, `/`) are simpler.

Apply NumPy functions:

```python
import numpy as np

dataset_clipped = xr.apply_ufunc(np.clip, dataset["sst"], -2, 2)
```

`np.clip` limits all values to a fixed range. This can help control outliers before plotting or statistics.

### Reduce operations

Reduce operations aggregate data to fewer values.

Typical reducers include `mean`, `sum`, `min`, `max`, `std`, and `quantile`.

```python
sst_mean = dataset["sst"].mean()
print(sst_mean.values)
```

Without a `dim=...` argument, `mean()` reduces all dimensions.
To reduce along one axis only, specify dimensions explicitly, for example:

```python
dataset["sst"].mean(dim="valid_time")
```

On a slice:

```python
transect_mean = dataset["sst"].sel(
    valid_time="2025-01-01T00:00:00",
    longitude=slice(290, 343),
    latitude=23,
).mean()
print(transect_mean.values)
```

Note that this dataset uses longitudes in degrees East (0 to 360), so West longitudes must be converted.

### Conditionally selecting and replacing data

Mask negatives to NaN:

```python
dataset["sst"].where(dataset["sst"] >= 0.0)
```

`where` keeps values where the condition is `True` and sets other values to `NaN` (unless `other=` is provided).

Conditional replacement with `xr.where`:

```python
xr.where(dataset["sst"] < 0.0, 0, dataset["sst"] * 2.0)
```

`xr.where(cond, x, y)` is a full if/else expression over arrays:

- where `cond` is true -> use `x`
- where `cond` is false -> use `y`

Mask by coordinate condition (for example, keeping eastern hemisphere values):

```python
dataset["sst"].where(dataset["sst"].longitude > 0)
```

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 3: Map, reduce, and where

Using the example dataset:

1. Calculate the 95th percentile using `quantile`.
2. Remove data above the 95th percentile with `where`.
3. Multiply remaining values by a correction factor of `0.90`.
4. Plot both original and corrected data for 2025-01-01 00:00.

::::::::::::::: solution

```python
threshold = dataset["sst"].quantile(0.95)
lower_95th = dataset["sst"].where(dataset["sst"] < threshold)
lower_95th = lower_95th * 0.90

lower_95th.sel(valid_time="2025-01-01T00:00:00").plot()
# Run this in a separate cell to view both clearly
dataset["sst"].sel(valid_time="2025-01-01T00:00:00").plot()
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

## Xarray patterns

Xarray provides common computational patterns including resampling, grouping, rolling windows, and coarsening.

These are high-level APIs for common scientific time-series and gridded-data workflows.

### Resampling

```python
resampled = dataset["sst"].sel(latitude=53, longitude=330, method="nearest").resample(valid_time="1D")
```

`resample` creates a resampler object; it does not compute values until you call an aggregation like `.mean()`.

Apply a reducer and compare against original data:

```python
import matplotlib.pyplot as plt

dataset["sst"].sel(latitude=53, longitude=330, method="nearest").plot(label="Original Data")
resampled.mean().plot(label="Resampled Daily Data", marker="o")
plt.legend()
```

### Groupby

```python
grouped = dataset["sst"].sel(latitude=53, longitude=330, method="nearest").groupby("valid_time.day")
grouped_mean = grouped.mean()
plt.bar(grouped_mean.day, grouped_mean)
```

`groupby` splits data into groups based on coordinate-derived labels, then applies operations per group.

### Rolling windows

```python
rolling = dataset["sst"].rolling(valid_time=12, center=True)
ds_rolling = rolling.mean()

dataset.sst.sel(longitude=330, latitude=53, method="nearest").plot(label="SST")
ds_rolling.sel(longitude=330, latitude=53, method="nearest").plot(label="12-step rolling mean")
plt.legend()
```

Rolling operations smooth short-term variability and highlight trends.
`center=True` centers each window on the output coordinate instead of right-aligning.

### Coarsening

Spatial coarsening:

```python
coarse = dataset.coarsen(latitude=5, longitude=5, boundary="trim")
coarse.mean()["sst"].sel(valid_time="2025-01-01T00:00:00").plot()
```

`coarsen` aggregates fixed-size blocks and is often used to reduce spatial resolution.
`boundary="trim"` drops leftover cells that do not fill a complete window.

Temporal coarsening:

```python
coarse = dataset.coarsen(valid_time=12)
coarse.mean()["sst"].sel(latitude=53, longitude=330, method="nearest").plot()
dataset["sst"].sel(latitude=53, longitude=330, method="nearest").plot()
```

## Writing data

Write processed output to NetCDF:

```python
dataset_corrected = dataset["sst"] * 1.1 - 1.0
dataset_corrected.to_netcdf("corrected.nc")
```

`to_netcdf` writes a new NetCDF file to disk.
In real workflows, choose output filenames that describe processing steps clearly, for example `sst_bias_corrected_2025.nc`.

## Function reference: what each command does

This quick reference summarises the core functions used in this lesson.

- `xr.open_dataset(path)`: open a file as an Xarray `Dataset` (metadata first, data lazily).
- `dataset["var"]`: select one variable as a `DataArray`.
- `sel(...)`: select by coordinate labels (dates, lat/lon values, named categories).
- `isel(...)`: select by integer positions (first, last, nth items).
- `where(condition, other=...)`: keep values where condition is true, replace others.
- `mean`, `sum`, `min`, `max`, `quantile`: reduce over one or more dimensions.
- `resample(...)`: regroup data to a new time frequency (then aggregate, for example `.mean()`).
- `groupby(...)`: split data into groups by labels (then aggregate per group).
- `rolling(...)`: define moving windows for smoothing/trend analysis.
- `coarsen(...)`: aggregate fixed-size blocks across dimensions.
- `plot()`, `plot.hist()`: quick visualisations directly from labeled arrays.
- `to_netcdf(path)`: write arrays back to NetCDF.

A useful workflow to follow is:

1. Open and inspect metadata.
2. Select a subset (`sel`/`isel`).
3. Apply operations (`where`, map/reduce).
4. Visualise results (`plot`).
5. Save outputs (`to_netcdf`).

::::::::::::::::::::::::::::::::::::::: challenge

## Exercise 4: Xarray tutorial dataset workflow

Using Xarray's `ersstv5` tutorial dataset (available with `xr.tutorial.load_dataset("ersstv5")`), complete the following workflow:

1. Select data before 2000.
2. Resample to annual means.
3. Compute global annual mean.
4. Plot the annual global mean time series.
5. Save results to NetCDF.

::::::::::::::: solution

```python
import xarray as xr

sst = xr.tutorial.load_dataset("ersstv5")
sst_20c = sst.sel(time=slice("1970-01-01", "1999-12-31"))
sst_annual = sst_20c.resample(time="1YE").mean()
sst_global = sst_annual.mean(dim=["lat", "lon"])

sst_global["sst"].plot()
sst_global.to_netcdf("global-mean-sst.nc")
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::::: keypoints

- "Xarray can load NetCDF files (and other formats such as GRIB and Zarr)."
- "We can address dimensions by name with dot syntax, bracket syntax, `sel`, and `isel`."
- "With lazy loading, data are only loaded into memory when needed."
- "Whole-array math operations are usually more efficient than explicit Python loops."
- "Custom functions can be applied across arrays with tools such as `apply_ufunc`."
- "Xarray can plot directly through Matplotlib-backed plotting methods."
- "Hvplot enables interactive visualisations."
- "Xarray includes built-in patterns such as resampling, grouping, rolling, and coarsening."

::::::::::::::::::::::::::::::::::::::::::::::::::
