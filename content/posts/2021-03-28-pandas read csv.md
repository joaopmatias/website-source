---
layout: post
date: 2021-03-28
title:  "A Pandas read_csv crash course"
---

Pandas is the most popular Python Library to analyze tabular data and, when it comes to loading data, the function `read_csv` is very convenient. Upon opening [its documentation](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html) one may be startled by the long list of arguments and options, but these allow the user to load tabular data in many different formats.

For that reason, reading a simple CSV file in Pandas can become a bit complex. In order to become better acquainted with the parameters in `read_csv` I'll share a few examples using it. Despite only focusing on a subset of all parameters, I hope it provides some value.

## Signature

We will only focus a subset of the arguments of `read_csv`. They are shown below with their default values. Only the `filepath` is a positional argument.

```python
pandas.read_csv(filepath_or_buffer: FilePathOrBuffer,
                sep=lib.no_default,
                header="infer",
                names=None,
                index_col=None,
                dtype=None,
                converters=None,
                parse_dates=False,
                infer_datetime_format=False,
                date_parser=None
    )
```

* `filepath_or_buffer` -- A path-like or string object describing the path to the file with data.

* `sep` -- The separator character used in the file. For CSV (comma separated values) files use `,`, but other characters like tabs, `\t`, may also be used. It is `,` by default.

* `header` -- Integer or sequence describing the indices of the rows to use as column names. By default, it is `"infer"`. If it is a sequence it creates `MultiIndex` column names.

* `names` -- Sequence containing the name for the dataframe columns. It is ignored if `header=0`.

* `index_col` -- Integer, string or sequence describing which columns of the file are used as index. If it is a sequence, it creates a `MultiIndex` index. If `False` it assigns consecutive integers starting at `0` to the index.

* `dtype` -- Type name or dictionary describing the type of each dataframe column. For example, `{‘a’: np.float64, ‘b’: np.int32, ‘c’: ‘Int64’}`.

* `converters` -- Dictionary of functions for converting values in certain columns.

* `parse_dates` -- If `True` it parses the index. If it is a list of integers it parses each corresponding column as a date column. If it is a list of lists it parses the contents corresponding to each inner list into a single date column.

* `infer_datetime_format` -- If `True` and `parse_dates` is enabled, pandas will attempt to infer the format of the datetime strings in the columns.

* `date_parser` -- Function to use for converting a sequence of string columns to an array of datetime instances. It is `dateutil.parser.parser` by default.



# Examples

Below, we show some examples reading a data file together with the output of the resulting dataframe in a jupyter notebook.

## 1)
Consider a file with 3 columns. 
```txt
"2021-01-01 00:15:00+00"	0	"0"
"2021-01-01 00:30:00+00"	1	"1"
"2021-01-01 00:45:00+00"	2	"2"
"2021-01-01 01:00:00+00"	3	"3"
"2021-01-01 01:15:00+00"	4	"4"
```
### 1.1)
In the first snippet, we use the left column as index and transform it into a date object. We also assign `col1` and `col2` as column names.
```python
pandas.read_csv("example.csv",
                sep="\t",
                names=["col1", "col2"],
                index_col=0,
                parse_dates=True
    )
```
The result is
![1.1](/images/blog/dataframe/df_1.1.png)

### 1.2)
Below, we use the row numbers as indexes and assign `col0`, `col1` and `col2` as column names. The entries in the first column are transformed into date objects.
```python
pandas.read_csv("example.csv",
                sep="\t",
                names=["col0", "col1", "col2"],
                index_col=False,
                parse_dates=[0]
    )
```
The result is
![1.2](/images/blog/dataframe/df_1.2.png)

### 1.3)
Now, we use the entries in the middle column as indexes and assign `col0` and `col2` as column names. The entries in the first column are transformed into date objects.
```python
pandas.read_csv("example.csv",
                sep="\t",
                names=["col0", "col2"],
                index_col=1,
                parse_dates=[0]
    )
```
The result is
![1.3](/images/blog/dataframe/df_1.3.png)

## 2)
Consider a different file with 3 columns. 
```txt
"col0","col1","col2"
"2021-01-01 00:15:00+00",0,"0"
"2021-01-01 00:30:00+00",1,"1"
"2021-01-01 00:45:00+00",2,"2"
"2021-01-01 01:00:00+00",3,"3"
"2021-01-01 01:15:00+00",4,"4"
```
### 2.1)
In this case, the separator argument can be omitted. The column names are infered to be `col0`, `col1` and `col2` and we use the left column entries as indexes. The indices are transformed into date objects. 
```python
pandas.read_csv("example.csv",
                index_col="col0",
                parse_dates=True
    )
```
The result is
![2.1](/images/blog/dataframe/df_2.1.png)

### 2.2)
Finally, the column names are infered to be `col0`, `col1` and `col2` and we use the middle column entries as indexes.
```python
pandas.read_csv("example.csv",
                index_col=1,
                parse_dates=[0]
    )
```
The result is
![2.2](/images/blog/dataframe/df_2.2.png)

# Conclusion 

That is it. I hope you enjoyed the post and feel free to reach out if you have any remarks!

# Reference
* https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html
