creating boolean Series/vectors
================

- <a href="#extract-information-from-a-string-variable"
  id="toc-extract-information-from-a-string-variable">Extract information
  from a string variable</a>

Attach libraries to R and choose a version of python

``` r
knitr::opts_chunk$set(echo = TRUE)
library(tidyverse)
library(reticulate)
use_python("c:/users/nabee/anaconda3/python.exe") 
```

Import python modules

``` python
import numpy as np
import pandas as pd
from palmerpenguins import load_penguins
import time
```

Create several dataframes from csv files and make the column names in
`iris` compatible with python.

``` r
iris <- read_csv(file = "data/iris.csv")
iris <- iris %>%
  rename_with(function(x) str_replace(x, "\\.", "_")) %>% 
  rename_with(tolower)

mpg <- read_csv(file = "data/mpg.csv")
economics <- read_csv(file = "data/economics.csv")
cps_samples <- readr::read_csv(file = "data/cps_samples.csv")
```

Share those dataframes with python.

``` python
iris = r.iris
mpg = r.mpg
economics = r.economics
cps_samples = r.cps_samples
```

# Extract information from a string variable

``` python
start_time = time.time()
iris.species.str.contains("^vir", regex = True).value_counts()
```

    False    100
    True      50
    Name: species, dtype: int64

``` python
print("Execution time in seconds: ", str(time.time() - start_time))
```

    Execution time in seconds:  0.013956785202026367

``` r
start_time <- Sys.time()
system.time(
mutate(iris, is_vir = str_detect(species, "^vir")) %>% count(is_vir)
)
```

       user  system elapsed 
       0.02    0.00    0.01 

``` r
print(paste0("Execution time in seconds: ", Sys.time() - start_time))
```

    [1] "Execution time in seconds: 0.187495946884155"
