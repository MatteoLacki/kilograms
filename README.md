A simple package for making histograms and simple summaries of rather large data-frames.
For me, it is faster than numpy and fast-histogram.
But your mileage might vary.

# Installation

```{bash}
pip install kilograms
```

or 

```{bash}
pip install git+https://github.com/MatteoLacki/kilograms
```


# Scatterplot Matrix

So, we have some bigger data:
```{python3}
          mz_extent  inv_ion_mobility_extent  retention_time_extent  \
1          0.036465                 0.111729              17.646362
2          0.036499                 0.102713              15.293213
3          0.029226                 0.079561              12.940674
4          0.037246                 0.120728              16.469666
5          0.033551                 0.118480              16.469666
...             ...                      ...                    ...
45695322   0.020608                 0.011571              12.937378
45695325   0.015745                 0.009255               5.882202
45695326   0.015708                 0.013889               4.704956
45695333   0.020197                 0.009259               3.529663
45695336   0.019840                 0.013889               7.056519

          log10_intensity  log10_vol
1                4.270305  -1.143297
2                3.582283  -1.241600
3                2.874973  -1.521569
4                4.767440  -1.130434
5                4.284236  -1.183966
...                   ...        ...
45695322         2.671344  -2.510728
45695325         2.550970  -3.066955
45695326         2.549371  -2.988647
45695333         2.187869  -3.180395
45695336         2.092367  -2.711186

[34_739_252 rows x 5 columns]
```
like ~35M points.

Say we want to bin `mz_extent` between min and max value:
```
xx = df.mz_extent.to_numpy()
extent = min_max(xx)
counts1D = histogram1D(xx, extent=extent, bins=100)
len(counts1D)# 100
sum(counts1D)# 34_739_252
```

You cannot have smaller extents: that will not work (for speed, no border checks).

Say we want to bin both `mz_extent` and `inv_ion_mobility_extent`
```{python}
xx = df.mz_extent.to_numpy()
yy = df.inv_ion_mobility_extent.to_numpy()
counts2D = histogram2D(xx, yy, extent=(min_max(xx), min_max(yy)), bins=(100,100))
counts2D.shape
np.all(np.sum(counts2D, axis=1) == counts1D) # True
np.sum(counts2D) == len(df) # True
```

To make a plot:
```{python3}
import matplotlib.pyplot as plt
from kilograms import scatterplot_matrix

# df is your data-frame with uniquely named columns
# each dimension has a numerical value.

with plt.style.context('dark_background'):
    scatterplot_matrix(df, y_labels_offset=-.2)
```
![](https://github.com/MatteoLacki/kilograms/blob/main/scatterplot_matrix.png "Scatterplot Matrix")

Do you have a pd.Series with weights?
We support that to, check out:

```{python3}
with plt.style.context('dark_background'):
    scatterplot_matrix(df, weights=weights, y_labels_offset=-.2)
```

# Crossplots

Another examplary use case: let us take another dataset with roughly 12 million rows:

```{python3}
In [97]: hprs_cut[extent_cols]
Out[97]:
          mz_extent  inv_ion_mobility_extent  retention_time_extent
0          0.039250                 0.097748              16.467407
1          0.036475                 0.107114              16.467407
2          0.039312                 0.070504              14.114502
3          0.028103                 0.102347              15.291748
4          0.039372                 0.095450              14.114304
...             ...                      ...                    ...
14845145   0.011898                 0.006939              11.766785
14845146   0.012180                 0.006939               8.236397
14845151   0.018146                 0.006939               9.410645
14845153   0.015214                 0.006939               5.882019
14845155   0.015426                 0.006939              11.762817

[12114407 rows x 3 columns]
```

Also, we might have an additional `pd.Series`, say `log10_intensity`:
```{python3}
In [103]: log10_intensity
Out[103]:
0           5.101457
1           4.642086
2           4.046946
3           3.362281
4           5.066818
              ...
14845145    3.036944
14845146    3.021195
14845151    2.814021
14845153    2.684862
14845155    2.618048
Name: intensity, Length: 12114407, dtype: float64
```

You can plot the 2D marginal distributions with x-axis corresponding to colums of the data frame and y-axis correspoding to `log10_intensities` with:

```{python3}
with plt.style.context('dark_background'):
    fig, axs = crossplot(
        hprs_cut[extent_cols],
        yy=np.log10(MS1_cut.intensity),
        show=False)
    fig.suptitle("Cross Plot")
    fig.show()
```

which results in:
![](https://github.com/MatteoLacki/kilograms/blob/main/crossplot_unweighted.png "Crossplot")

You might also weight the whole distribution.
Say you want to do it by `peak_count`:

```{python3}
In [106]: peak_count
Out[106]:
0           863
1           720
2           277
3            99
4           440
           ...
14845145     18
14845146     15
14845151     13
14845153      7
14845155      8
Name: peak_count, Length: 12114407, dtype: int64
```

That is as simple as:

```{python3}
with plt.style.context('dark_background'):
    fig, axs = crossplot(
        hprs_cut[extent_cols],
        yy=np.log10(MS1_cut.intensity),
        weights=peak_count,
        show=False
    )
    fig.suptitle("Cross Plot, peak_count weighted.")
    fig.show()
```

which results in:
![](https://github.com/MatteoLacki/kilograms/blob/main/crossplot_peak_count_weighted.png "Crossplot")


# TODO list

* Handle missing values or infinities
* Better handling of data transformations:
    * one can pass in and compile (and cache the compilation) of a ![](https://stackoverflow.com/questions/45976662/speed-up-function-that-takes-a-function-as-argument-with-numba, "function").
    * the ticks and sub-ticks should be modified.
* Better handling of discrete int-based clustering
    * use of their natural binning 
* Overplotting multiple 2D histograms with majority value-based basis.
