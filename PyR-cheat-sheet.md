pandas - dplyr equivalents
================

- <a href="#selecting-columns" id="toc-selecting-columns">Selecting
  columns</a>
- <a href="#renaming-a-single-column"
  id="toc-renaming-a-single-column">Renaming a single column</a>
- <a href="#renaming-multiple-columns-based-on-condition"
  id="toc-renaming-multiple-columns-based-on-condition">Renaming multiple
  columns based on condition</a>
- <a href="#distinct-values-in-a-column"
  id="toc-distinct-values-in-a-column">distinct values in a column</a>
- <a href="#count-records-by-a-categorical-variable"
  id="toc-count-records-by-a-categorical-variable">count records by a
  categorical variable</a>
- <a href="#summarize-aggregate-over-entire-columns"
  id="toc-summarize-aggregate-over-entire-columns">summarize / aggregate
  over entire column(s)</a>
- <a href="#summarize-aggregate-by-group"
  id="toc-summarize-aggregate-by-group">summarize / aggregate by group</a>
- <a href="#column-math-adding-a-new-column"
  id="toc-column-math-adding-a-new-column">column math / adding a new
  column</a>
- <a href="#deleting-columns" id="toc-deleting-columns">deleting
  columns</a>
- <a href="#changing-order-of-columns"
  id="toc-changing-order-of-columns">changing order of columns</a>
- <a href="#slicing" id="toc-slicing">slicing</a>
- <a href="#slicing-first-and-last-records-head-tail"
  id="toc-slicing-first-and-last-records-head-tail">Slicing first and last
  records (head / tail)</a>
- <a href="#creating-boolean-vectorseries"
  id="toc-creating-boolean-vectorseries">creating boolean
  vector/series</a>

This “cheat sheet” is for people who know pandas and want to learn the
dplyr equivalent or vice versa.

Thanks to [Martin
Siklar](https://towardsdatascience.com/python-pandas-vs-r-dplyr-5b5081945ccb)
for getting this started. I have and will continue to add to this.

It takes too long to puzzle out how to do something in your non-dominant
language, so I’m hoping we can crowd source the equivalents and save
time learning.

I am in the know-dplyr learning pandas group. If there is a common
pandas task you are interested in knowing how to do with dplyr, please
share the pandas code with me. If there is a better or another way to do
a certain task in pandas, please share that code with me.

Use the table of contents to jump to a task.

This “cheat sheet” produced using the Quarto approach to mixing code and
text. Quarto supports both python and R in the same document. I rendered
it in Rstudo.

``` r
knitr::opts_chunk$set(echo = TRUE)
library(tidyverse)
library(reticulate)
# https://rstudio.github.io/reticulate/articles/versions.html
# https://cran.r-project.org/web/packages/reticulate/vignettes/python_packages.html
use_python("c:/users/nabee/anaconda3/python.exe") # has pandas
# use_condaenv(condaenv = "py4ds",
#             conda = "C:/Users/nabee/anaconda3/condabin/conda")
```

``` python
import numpy as np
import pandas as pd
from palmerpenguins import load_penguins
```

``` r
iris <- readr::read_csv(file = "data/iris.csv")
glimpse(iris)
```

    Rows: 150
    Columns: 5
    $ Sepal.Length <dbl> 5.1, 4.9, 4.7, 4.6, 5.0, 5.4, 4.6, 5.0, 4.4, 4.9, 5.4, 4.~
    $ Sepal.Width  <dbl> 3.5, 3.0, 3.2, 3.1, 3.6, 3.9, 3.4, 3.4, 2.9, 3.1, 3.7, 3.~
    $ Petal.Length <dbl> 1.4, 1.4, 1.3, 1.5, 1.4, 1.7, 1.4, 1.5, 1.4, 1.5, 1.5, 1.~
    $ Petal.Width  <dbl> 0.2, 0.2, 0.2, 0.2, 0.2, 0.4, 0.3, 0.2, 0.2, 0.1, 0.2, 0.~
    $ Species      <chr> "setosa", "setosa", "setosa", "setosa", "setosa", "setosa~

``` r
mpg <- readr::read_csv(file = "data/mpg.csv")

economics <- readr::read_csv(file = "data/economics.csv")

cps_samples <- read_csv(file = "data/cps_samples.csv")
```

Bring `iris` from R and edit variable names.

A big difference between Python and R is that Python methods often
“modify in place” whereas R functions always “copy on modify”. For that
reason and for the purpose of this tutorial, at the end of Python code
chunks that have modified `iris` dataframe the original `iris` dataframe
is restored.

``` r
# edit variable names to match what how they are in Python
iris <- iris %>%
  rename_with(.fn = ~ str_replace(.x, "\\.", "_")) %>% 
  rename_with(.fn = ~ str_replace(.x, "W", "w")) %>% 
  rename_with(.fn = ~ str_replace(.x, "L", "l"))

glimpse(iris)  
```

    Rows: 150
    Columns: 5
    $ Sepal_length <dbl> 5.1, 4.9, 4.7, 4.6, 5.0, 5.4, 4.6, 5.0, 4.4, 4.9, 5.4, 4.~
    $ Sepal_width  <dbl> 3.5, 3.0, 3.2, 3.1, 3.6, 3.9, 3.4, 3.4, 2.9, 3.1, 3.7, 3.~
    $ Petal_length <dbl> 1.4, 1.4, 1.3, 1.5, 1.4, 1.7, 1.4, 1.5, 1.4, 1.5, 1.5, 1.~
    $ Petal_width  <dbl> 0.2, 0.2, 0.2, 0.2, 0.2, 0.4, 0.3, 0.2, 0.2, 0.1, 0.2, 0.~
    $ Species      <chr> "setosa", "setosa", "setosa", "setosa", "setosa", "setosa~

``` python
iris = r.iris
mpg = r.mpg
```

# Selecting columns

And here it already gets confusing. First of all, there are multiple
ways on how to select columns from a dataframe in each framework. In
pandas you can either simply pass a list with the column names or use
the filter() method. This is confusing because the filter() function in
dplyr is used to subset rows based on conditions and not columns! In
dplyr we use the select() function instead:

``` python
#Use Filter Function
iris.filter(items=['Sepal_width', 'Petal_width'])

#Use Filter Function
```

         Sepal_width  Petal_width
    0            3.5          0.2
    1            3.0          0.2
    2            3.2          0.2
    3            3.1          0.2
    4            3.6          0.2
    ..           ...          ...
    145          3.0          2.3
    146          2.5          1.9
    147          3.0          2.0
    148          3.4          2.3
    149          3.0          1.8

    [150 rows x 2 columns]

``` python
iris.filter(items=['Sepal_width', 'Petal_width'])
```

         Sepal_width  Petal_width
    0            3.5          0.2
    1            3.0          0.2
    2            3.2          0.2
    3            3.1          0.2
    4            3.6          0.2
    ..           ...          ...
    145          3.0          2.3
    146          2.5          1.9
    147          3.0          2.0
    148          3.4          2.3
    149          3.0          1.8

    [150 rows x 2 columns]

``` r
#Pass columns as list
iris %>% select(Sepal_width, Petal_width) %>% head()
```

    # A tibble: 6 x 2
      Sepal_width Petal_width
            <dbl>       <dbl>
    1         3.5         0.2
    2         3           0.2
    3         3.2         0.2
    4         3.1         0.2
    5         3.6         0.2
    6         3.9         0.4

Filter based on conditions Again, there are multiple ways on how to
filter records in a dataframe based on conditions across one or multiple
columns.

pandas

In pandas you can either use the indexing approach or try out the handy
query API, which I personally prefer.

``` python
#indexing
iris[(iris["Sepal_width"] > 3.5) & (iris["Petal_width"] < 0.3)]
#query API
```

        Sepal_length  Sepal_width  Petal_length  Petal_width Species
    4            5.0          3.6           1.4          0.2  setosa
    10           5.4          3.7           1.5          0.2  setosa
    14           5.8          4.0           1.2          0.2  setosa
    22           4.6          3.6           1.0          0.2  setosa
    32           5.2          4.1           1.5          0.1  setosa
    33           5.5          4.2           1.4          0.2  setosa
    37           4.9          3.6           1.4          0.1  setosa
    46           5.1          3.8           1.6          0.2  setosa
    48           5.3          3.7           1.5          0.2  setosa

``` python
iris.query("Sepal_width > 3.5 & Petal_width < 0.3")
```

        Sepal_length  Sepal_width  Petal_length  Petal_width Species
    4            5.0          3.6           1.4          0.2  setosa
    10           5.4          3.7           1.5          0.2  setosa
    14           5.8          4.0           1.2          0.2  setosa
    22           4.6          3.6           1.0          0.2  setosa
    32           5.2          4.1           1.5          0.1  setosa
    33           5.5          4.2           1.4          0.2  setosa
    37           4.9          3.6           1.4          0.1  setosa
    46           5.1          3.8           1.6          0.2  setosa
    48           5.3          3.7           1.5          0.2  setosa

dplyr

The standard way of filtering records in dplyr is via the filter
function().

``` r
iris %>% filter(Sepal_width > 3.5 & Petal_width < 0.3) %>% head()
```

    # A tibble: 6 x 5
      Sepal_length Sepal_width Petal_length Petal_width Species
             <dbl>       <dbl>        <dbl>       <dbl> <chr>  
    1          5           3.6          1.4         0.2 setosa 
    2          5.4         3.7          1.5         0.2 setosa 
    3          5.8         4            1.2         0.2 setosa 
    4          4.6         3.6          1           0.2 setosa 
    5          5.2         4.1          1.5         0.1 setosa 
    6          5.5         4.2          1.4         0.2 setosa 

# Renaming a single column

Renaming sounds like an easy task, but be cautious and note the subtle
difference here. If we want to rename our column from Species to Class
in pandas we supply a dictionary that says {‘Species’: ‘Class’} and in
dplyr it is the exact opposite way Class=Species:

pandas

``` python
# {oldname : newname}
iris.rename(columns = {'Species': 'Class'}, inplace = True)

iris = r.iris # restore original
```

dplyr

``` r
# newname = oldname
iris %>% rename(Class=Species) %>% glimpse()
```

    Rows: 150
    Columns: 5
    $ Sepal_length <dbl> 5.1, 4.9, 4.7, 4.6, 5.0, 5.4, 4.6, 5.0, 4.4, 4.9, 5.4, 4.~
    $ Sepal_width  <dbl> 3.5, 3.0, 3.2, 3.1, 3.6, 3.9, 3.4, 3.4, 2.9, 3.1, 3.7, 3.~
    $ Petal_length <dbl> 1.4, 1.4, 1.3, 1.5, 1.4, 1.7, 1.4, 1.5, 1.4, 1.5, 1.5, 1.~
    $ Petal_width  <dbl> 0.2, 0.2, 0.2, 0.2, 0.2, 0.4, 0.3, 0.2, 0.2, 0.1, 0.2, 0.~
    $ Class        <chr> "setosa", "setosa", "setosa", "setosa", "setosa", "setosa~

# Renaming multiple columns based on condition

Let us say we want to rename multiple columns at once based on a
condition. For example, convert all our feature columns (Sepal_length,
Sepal_width, Petal_length, Petal_width) to upper case. In Python that’s
actually quite tricky and you need to first import another library and
iterate manually over each column. In dplyr there is a much cleaner
interface if you want to access/change multiple columns based on
conditions.

pandas

``` python
import re # regular expressions
#prepare pattern that columns have to match to be converted to upper case
pattern = re.compile(r".*(length|width)")
#iterate over columns and covert to upper case if pattern matches.
for col in iris.columns:
  if bool((pattern.match(col))):
    iris.rename(columns = {col: col.upper()}, inplace = True)
    
iris.describe()
```

           SEPAL_LENGTH  SEPAL_WIDTH  PETAL_LENGTH  PETAL_WIDTH
    count    150.000000   150.000000    150.000000   150.000000
    mean       5.843333     3.057333      3.758000     1.199333
    std        0.828066     0.435866      1.765298     0.762238
    min        4.300000     2.000000      1.000000     0.100000
    25%        5.100000     2.800000      1.600000     0.300000
    50%        5.800000     3.000000      4.350000     1.300000
    75%        6.400000     3.300000      5.100000     1.800000
    max        7.900000     4.400000      6.900000     2.500000

``` python
iris = r.iris # restore original
```

dplyr

``` r
iris %>% rename_with(toupper, matches("length|width")) %>% glimpse()
```

    Rows: 150
    Columns: 5
    $ SEPAL_LENGTH <dbl> 5.1, 4.9, 4.7, 4.6, 5.0, 5.4, 4.6, 5.0, 4.4, 4.9, 5.4, 4.~
    $ SEPAL_WIDTH  <dbl> 3.5, 3.0, 3.2, 3.1, 3.6, 3.9, 3.4, 3.4, 2.9, 3.1, 3.7, 3.~
    $ PETAL_LENGTH <dbl> 1.4, 1.4, 1.3, 1.5, 1.4, 1.7, 1.4, 1.5, 1.4, 1.5, 1.5, 1.~
    $ PETAL_WIDTH  <dbl> 0.2, 0.2, 0.2, 0.2, 0.2, 0.4, 0.3, 0.2, 0.2, 0.1, 0.2, 0.~
    $ Species      <chr> "setosa", "setosa", "setosa", "setosa", "setosa", "setosa~

Note the upper case feature column names.

Altering cell values based on conditions

Let us say we want to recode/alter cell values based on conditions: In
our example, we will try to recode the Species strings “setosa,
versicolor and virginica” to integers from 0 to 2:

pandas

``` python
# Error
iris.loc[iris['Species'] == 'setosa', "Species"] = 0
iris.loc[iris['Species'] == 'versicolor', "Species"] = 1
iris.loc[iris['Species'] == 'virginica', "Species"] = 2
```

dplyr

``` r
iris %>%
  mutate(Species = case_when(Species == 'setosa' ~ 0,
                             Species == 'versicolor' ~ 1,
                             Species == 'virginica' ~ 2)) %>%
  head()
```

    # A tibble: 6 x 5
      Sepal_length Sepal_width Petal_length Petal_width Species
             <dbl>       <dbl>        <dbl>       <dbl>   <dbl>
    1          5.1         3.5          1.4         0.2       0
    2          4.9         3            1.4         0.2       0
    3          4.7         3.2          1.3         0.2       0
    4          4.6         3.1          1.5         0.2       0
    5          5           3.6          1.4         0.2       0
    6          5.4         3.9          1.7         0.4       0

# distinct values in a column

Sometimes we want to see which values distinct/unique values we have in
a column. Note how different the function call is in both frameworks:
pandas uses the unique() method and dplyr() uses the distinct() function
to get to the same result:

pandas

``` python
iris.Species.unique()
#array(['setosa', 'versicolor', 'virginica'], dtype=object)
```

    array(['setosa', 'versicolor', 'virginica'], dtype=object)

dplyr

``` r
iris %>% select(Species) %>% distinct() %>% count(Species)
```

    # A tibble: 3 x 2
      Species        n
      <chr>      <int>
    1 setosa         1
    2 versicolor     1
    3 virginica      1

# count records by a categorical variable

If you want to count how many entries the iris dataframe has in total or
get a count for a certain group, you can do the following:

pandas

``` python
len(iris)
```

    150

``` python
iris.value_counts('Species')
```

    Species
    setosa        50
    versicolor    50
    virginica     50
    dtype: int64

``` python
iris.groupby(['Species']).size()
```

    Species
    setosa        50
    versicolor    50
    virginica     50
    dtype: int64

dplyr

``` r
iris %>% nrow()
```

    [1] 150

``` r
iris %>% group_by(Species) %>% tally()
```

    # A tibble: 3 x 2
      Species        n
      <chr>      <int>
    1 setosa        50
    2 versicolor    50
    3 virginica     50

``` r
iris %>% count(Species)
```

    # A tibble: 3 x 2
      Species        n
      <chr>      <int>
    1 setosa        50
    2 versicolor    50
    3 virginica     50

# summarize / aggregate over entire column(s)

If you want to create descriptive statistics for one or more columns in
your data frame, you can do the following:

pandas

``` python
iris.agg(['mean', 'min'])
```

          Sepal_length  Sepal_width  Petal_length  Petal_width Species
    mean      5.843333     3.057333         3.758     1.199333     NaN
    min       4.300000     2.000000         1.000     0.100000  setosa

    <string>:1: FutureWarning: ['Species'] did not aggregate successfully. If any error is raised this will raise in a future version of pandas. Drop these columns/ops to avoid this warning.

dplyr

`mean()` and `min()` would produce an error if a non-numeric vector is
passed to them, so the select_helper `where` is used to prevent that.

``` r
iris %>% summarise(across(where(is.numeric), list(mean = mean, min = min)))
```

    # A tibble: 1 x 8
      Sepal_length_mean Sepal_leng~1 Sepal~2 Sepal~3 Petal~4 Petal~5 Petal~6 Petal~7
                  <dbl>        <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
    1              5.84          4.3    3.06       2    3.76       1    1.20     0.1
    # ... with abbreviated variable names 1: Sepal_length_min, 2: Sepal_width_mean,
    #   3: Sepal_width_min, 4: Petal_length_mean, 5: Petal_length_min,
    #   6: Petal_width_mean, 7: Petal_width_min

# summarize / aggregate by group

If you want to have aggregate statistics for by group in your dataset,
you have to use the groupby() method in pandas and the group_by()
function in dplyr. You can either do this for all columns or for a
specific column:

pandas

Note how pandas uses multilevel indexing for a clean display of the
results:

``` python
# aggregation by group for all columns
iris.groupby(['Species']).agg(['mean', 'min'])
```

               Sepal_length      Sepal_width  ... Petal_length Petal_width     
                       mean  min        mean  ...          min        mean  min
    Species                                   ...                              
    setosa            5.006  4.3       3.428  ...          1.0       0.246  0.1
    versicolor        5.936  4.9       2.770  ...          3.0       1.326  1.0
    virginica         6.588  4.9       2.974  ...          4.5       2.026  1.4

    [3 rows x 8 columns]

``` python
# aggregation by group for a specific column
iris.groupby(['Species']).agg({'Sepal_length':['mean']})
```

               Sepal_length
                       mean
    Species                
    setosa            5.006
    versicolor        5.936
    virginica         6.588

Since dplyr doesn’t support multilevel indexing, the output of the first
call looks a little bit messy compared to pandas. The output column
names have the form inputColName_aggFunction.

``` r
# aggregation by group for all (numeric) columns
iris %>% group_by(Species) %>% summarise(across(where(is.numeric), list(mean = mean, min = min)))
```

    # A tibble: 3 x 9
      Species    Sepal_len~1 Sepal~2 Sepal~3 Sepal~4 Petal~5 Petal~6 Petal~7 Petal~8
      <chr>            <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>   <dbl>
    1 setosa            5.01     4.3    3.43     2.3    1.46     1     0.246     0.1
    2 versicolor        5.94     4.9    2.77     2      4.26     3     1.33      1  
    3 virginica         6.59     4.9    2.97     2.2    5.55     4.5   2.03      1.4
    # ... with abbreviated variable names 1: Sepal_length_mean,
    #   2: Sepal_length_min, 3: Sepal_width_mean, 4: Sepal_width_min,
    #   5: Petal_length_mean, 6: Petal_length_min, 7: Petal_width_mean,
    #   8: Petal_width_min

``` r
# aggregation by group for a specific column
iris %>% group_by(Species) %>% summarise(mean=mean(Sepal_length))
```

    # A tibble: 3 x 2
      Species     mean
      <chr>      <dbl>
    1 setosa      5.01
    2 versicolor  5.94
    3 virginica   6.59

# column math / adding a new column

Sometimes you want to create a new column and combine the values of two
or more existing columns with some mathematical operation. Here is how
to do it in both pandas and dplyr:

pandas

``` r
iris["New_feature"] = iris["Petal_width"] * iris["Petal_length"] / 2
```

dplyr

``` r
iris <- iris %>% mutate(New_feature= Petal_width * Petal_length/2)
iris %>% head()
```

    # A tibble: 6 x 6
      Sepal_length Sepal_width Petal_length Petal_width Species New_feature
             <dbl>       <dbl>        <dbl>       <dbl> <chr>         <dbl>
    1          5.1         3.5          1.4         0.2 setosa         0.14
    2          4.9         3            1.4         0.2 setosa         0.14
    3          4.7         3.2          1.3         0.2 setosa         0.13
    4          4.6         3.1          1.5         0.2 setosa         0.15
    5          5           3.6          1.4         0.2 setosa         0.14
    6          5.4         3.9          1.7         0.4 setosa         0.34

# deleting columns

To clean up a dataframe, deleting columns can sometimes be quite handy:

pandas

In pandas you can delete a column with drop(). You can also use
inplace=True to overwrite the current dataframe.

``` python
# ['New_feature'] not found in axis
iris.drop("New_feature", axis=1, inplace=True)
```

dplyr

In dplyr you specify the column name you want to remove inside the
select() function with a leading minus.

``` r
iris <- iris %>% select(-New_feature) 
iris %>% head()
```

    # A tibble: 6 x 5
      Sepal_length Sepal_width Petal_length Petal_width Species
             <dbl>       <dbl>        <dbl>       <dbl> <chr>  
    1          5.1         3.5          1.4         0.2 setosa 
    2          4.9         3            1.4         0.2 setosa 
    3          4.7         3.2          1.3         0.2 setosa 
    4          4.6         3.1          1.5         0.2 setosa 
    5          5           3.6          1.4         0.2 setosa 
    6          5.4         3.9          1.7         0.4 setosa 

Sorting records by values

To sort values you can use sort_values() in pandas and arrange() in
dplyr. The default sorting for both is ascending. Note the difference in
each function call on how to sort descending:

pandas

``` python
iris.sort_values('Petal_width', ascending=0)
```

         Sepal_length  Sepal_width  Petal_length  Petal_width    Species
    100           6.3          3.3           6.0          2.5  virginica
    109           7.2          3.6           6.1          2.5  virginica
    144           6.7          3.3           5.7          2.5  virginica
    114           5.8          2.8           5.1          2.4  virginica
    140           6.7          3.1           5.6          2.4  virginica
    ..            ...          ...           ...          ...        ...
    12            4.8          3.0           1.4          0.1     setosa
    13            4.3          3.0           1.1          0.1     setosa
    37            4.9          3.6           1.4          0.1     setosa
    32            5.2          4.1           1.5          0.1     setosa
    9             4.9          3.1           1.5          0.1     setosa

    [150 rows x 5 columns]

dplyr

``` r
iris %>% arrange(desc(Petal_width))
```

    # A tibble: 150 x 5
       Sepal_length Sepal_width Petal_length Petal_width Species  
              <dbl>       <dbl>        <dbl>       <dbl> <chr>    
     1          6.3         3.3          6           2.5 virginica
     2          7.2         3.6          6.1         2.5 virginica
     3          6.7         3.3          5.7         2.5 virginica
     4          5.8         2.8          5.1         2.4 virginica
     5          6.3         3.4          5.6         2.4 virginica
     6          6.7         3.1          5.6         2.4 virginica
     7          6.4         3.2          5.3         2.3 virginica
     8          7.7         2.6          6.9         2.3 virginica
     9          6.9         3.2          5.7         2.3 virginica
    10          7.7         3            6.1         2.3 virginica
    # ... with 140 more rows

# changing order of columns

I don’t use this functionality often, but sometimes it comes in handy if
I want to create a table for a presentation and the ordering of the
columns doesn’t make logical sense. Here is how to move around columns:

pandas

In Python pandas you need to reindex your columns by making use of a
list. Let’s say we want to move the column Species to the front.

``` python
iris.reindex(['Species','Petal_length','Sepal_length','Sepal_width','Petal_Width'], axis=1)
```

           Species  Petal_length  Sepal_length  Sepal_width  Petal_Width
    0       setosa           1.4           5.1          3.5          NaN
    1       setosa           1.4           4.9          3.0          NaN
    2       setosa           1.3           4.7          3.2          NaN
    3       setosa           1.5           4.6          3.1          NaN
    4       setosa           1.4           5.0          3.6          NaN
    ..         ...           ...           ...          ...          ...
    145  virginica           5.2           6.7          3.0          NaN
    146  virginica           5.0           6.3          2.5          NaN
    147  virginica           5.2           6.5          3.0          NaN
    148  virginica           5.4           6.2          3.4          NaN
    149  virginica           5.1           5.9          3.0          NaN

    [150 rows x 5 columns]

dplyr

In dplyr you can use the handy relocate() function. Again let’s say we
want to move the column Species to the front.

``` r
iris %>% relocate(Species)
```

    # A tibble: 150 x 5
       Species Sepal_length Sepal_width Petal_length Petal_width
       <chr>          <dbl>       <dbl>        <dbl>       <dbl>
     1 setosa           5.1         3.5          1.4         0.2
     2 setosa           4.9         3            1.4         0.2
     3 setosa           4.7         3.2          1.3         0.2
     4 setosa           4.6         3.1          1.5         0.2
     5 setosa           5           3.6          1.4         0.2
     6 setosa           5.4         3.9          1.7         0.4
     7 setosa           4.6         3.4          1.4         0.3
     8 setosa           5           3.4          1.5         0.2
     9 setosa           4.4         2.9          1.4         0.2
    10 setosa           4.9         3.1          1.5         0.1
    # ... with 140 more rows

Note that you can use .before or .after to place a columne before or
after another.

``` r
iris %>% relocate(Species, .before=Sepal_width)
```

    # A tibble: 150 x 5
       Sepal_length Species Sepal_width Petal_length Petal_width
              <dbl> <chr>         <dbl>        <dbl>       <dbl>
     1          5.1 setosa          3.5          1.4         0.2
     2          4.9 setosa          3            1.4         0.2
     3          4.7 setosa          3.2          1.3         0.2
     4          4.6 setosa          3.1          1.5         0.2
     5          5   setosa          3.6          1.4         0.2
     6          5.4 setosa          3.9          1.7         0.4
     7          4.6 setosa          3.4          1.4         0.3
     8          5   setosa          3.4          1.5         0.2
     9          4.4 setosa          2.9          1.4         0.2
    10          4.9 setosa          3.1          1.5         0.1
    # ... with 140 more rows

# slicing

Slicing is a whole topic on its own and there is a lot of ways on how to
do it. Let us step through the most frequently used slicing operation
below:

Slicing by row Sometimes you know the exact row number you want to
extract. Although the procedure in dplyr and pandas is quite similar
please note that Indexing in Python starts at 0 and in R at 1.

pandas

``` python
iris.iloc[[49,50]]
```

        Sepal_length  Sepal_width  Petal_length  Petal_width     Species
    49           5.0          3.3           1.4          0.2      setosa
    50           7.0          3.2           4.7          1.4  versicolor

dplyr

``` r
iris %>% slice(50, 51)
```

    # A tibble: 2 x 5
      Sepal_length Sepal_width Petal_length Petal_width Species   
             <dbl>       <dbl>        <dbl>       <dbl> <chr>     
    1            5         3.3          1.4         0.2 setosa    
    2            7         3.2          4.7         1.4 versicolor

# Slicing first and last records (head / tail)

Sometimes we want to see either the first or last records in a
dataframe. This can be either done by providing a fixed number n or a
proportion prop of values.

pandas

In pandas you can use the head() or tail() method to get a fixed amount
of records. If you want to extraction a proportion, you have to do some
math on your own:

``` python
# returns 5 records
iris.head(n=5)
```

       Sepal_length  Sepal_width  Petal_length  Petal_width Species
    0           5.1          3.5           1.4          0.2  setosa
    1           4.9          3.0           1.4          0.2  setosa
    2           4.7          3.2           1.3          0.2  setosa
    3           4.6          3.1           1.5          0.2  setosa
    4           5.0          3.6           1.4          0.2  setosa

``` python
# returns the last 10% of total records
iris.tail(n = len(iris) // 10)
```

         Sepal_length  Sepal_width  Petal_length  Petal_width    Species
    135           7.7          3.0           6.1          2.3  virginica
    136           6.3          3.4           5.6          2.4  virginica
    137           6.4          3.1           5.5          1.8  virginica
    138           6.0          3.0           4.8          1.8  virginica
    139           6.9          3.1           5.4          2.1  virginica
    140           6.7          3.1           5.6          2.4  virginica
    141           6.9          3.1           5.1          2.3  virginica
    142           5.8          2.7           5.1          1.9  virginica
    143           6.8          3.2           5.9          2.3  virginica
    144           6.7          3.3           5.7          2.5  virginica
    145           6.7          3.0           5.2          2.3  virginica
    146           6.3          2.5           5.0          1.9  virginica
    147           6.5          3.0           5.2          2.0  virginica
    148           6.2          3.4           5.4          2.3  virginica
    149           5.9          3.0           5.1          1.8  virginica

dplyr

In dplyr there are two designated functions for this use case:
slice_head() and slice_tail(). Please note how you can either specify a
fixed number or a proportion:

``` r
# returns first 5 records
iris %>% slice_head(n=5)
```

    # A tibble: 5 x 5
      Sepal_length Sepal_width Petal_length Petal_width Species
             <dbl>       <dbl>        <dbl>       <dbl> <chr>  
    1          5.1         3.5          1.4         0.2 setosa 
    2          4.9         3            1.4         0.2 setosa 
    3          4.7         3.2          1.3         0.2 setosa 
    4          4.6         3.1          1.5         0.2 setosa 
    5          5           3.6          1.4         0.2 setosa 

``` r
# returns the last 10% of total records
iris %>% slice_tail(prop=0.1)
```

    # A tibble: 15 x 5
       Sepal_length Sepal_width Petal_length Petal_width Species  
              <dbl>       <dbl>        <dbl>       <dbl> <chr>    
     1          7.7         3            6.1         2.3 virginica
     2          6.3         3.4          5.6         2.4 virginica
     3          6.4         3.1          5.5         1.8 virginica
     4          6           3            4.8         1.8 virginica
     5          6.9         3.1          5.4         2.1 virginica
     6          6.7         3.1          5.6         2.4 virginica
     7          6.9         3.1          5.1         2.3 virginica
     8          5.8         2.7          5.1         1.9 virginica
     9          6.8         3.2          5.9         2.3 virginica
    10          6.7         3.3          5.7         2.5 virginica
    11          6.7         3            5.2         2.3 virginica
    12          6.3         2.5          5           1.9 virginica
    13          6.5         3            5.2         2   virginica
    14          6.2         3.4          5.4         2.3 virginica
    15          5.9         3            5.1         1.8 virginica

# creating boolean vector/series

Such vector/series are useful for subsetting a dataframe and creating
“dummy” variables analysis

Using inequalities

``` python
# (iris.Petal_width > iris.Petal_width.median()).value_counts()

iris["wide_petal"] = iris["Petal_width"] > iris["Petal_width"].median()

iris["wide_petal"].value_counts()
```

    False    78
    True     72
    Name: wide_petal, dtype: int64

``` python
iris = r.iris
```

``` python
mpg["great_mpg"] = (mpg.cty) > 20 & (mpg.hwy > 25)
mpg.head()
```

      manufacturer model  displ    year  cyl  ...   cty   hwy  fl    class great_mpg
    0         audi    a4    1.8  1999.0  4.0  ...  18.0  29.0   p  compact      True
    1         audi    a4    1.8  1999.0  4.0  ...  21.0  29.0   p  compact      True
    2         audi    a4    2.0  2008.0  4.0  ...  20.0  31.0   p  compact      True
    3         audi    a4    2.0  2008.0  4.0  ...  21.0  30.0   p  compact      True
    4         audi    a4    2.8  1999.0  6.0  ...  16.0  26.0   p  compact      True

    [5 rows x 12 columns]

``` python
mpg = r.mpg
```

``` r
mutate(iris, wide_petal = Petal_width > quantile(Petal_width, probs = 0.50)) %>% count(wide_petal)
```

    # A tibble: 2 x 2
      wide_petal     n
      <lgl>      <int>
    1 FALSE         78
    2 TRUE          72

Using regex on a string variable

``` python
iris.Species.str.contains("^vir", regex = True).value_counts()
```

    False    100
    True      50
    Name: Species, dtype: int64

``` r
mutate(iris, is_vir = str_detect(Species, "^vir")) %>% count(is_vir)
```

    # A tibble: 2 x 2
      is_vir     n
      <lgl>  <int>
    1 FALSE    100
    2 TRUE      50
