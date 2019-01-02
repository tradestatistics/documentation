# Data Processing

## Tidy Data

We followed Tidy Data principles exposed in [@tidydata2014] and  [@r4ds2016]. Those principles are closely tied to those of relational databases and Codd’s relational algebra.

<div class="figure">
<img src="fig/tidy-data.svg" alt="Data pipeline"  />
<p class="caption">(\#fig:unnamed-chunk-1)Data pipeline</p>
</div>

## Filling gaps in our data

We use mirrored flows to cover gaps in raw data. Some countries report zero exports for some products, but we can inspect what their trade partners reported. If country A reported zero exports (imports) of product B to (from) country C, then we searched what country C reported of imports (exports) of product B from (to) country A.

Exports are reported FOB (free on board) while imports are reported CIF (cost, insurance and freight). When country A sends products to country C that will be registered with a larger value when it arrives to destination because the importer is including cost, insurance and freight that was not registered before shipping. There are different approaches to solve this difficulty, and in particular [@tradecosts2004], [@geography2009] and [@baci2010] discuss this in detail and propose that an 8% CIF/FOB ratio is suitable to discount costs and compare imports and exports.

There are some noble and remarkable approaches such as gravitational models. As, to our knowledge at the moment, there is no literature reporting the estimation of a gravity equation for this purpose that returns a satisfactory fitting.

Let $x_{c,c',p}$ represent the exports of country $c$ to country $c'$ in product $p$ and $m_{c',c,p}$ the imports of country $c'$ from country $c$. Under this notation we defined corrected flows as:

$$\hat{x}_{c,c',p} = \max\left\{x_{c,c',p}, \frac{m_{c',c,p}}{1.08}\right\}$$
$$\hat{m}_{c,c',p} = \max\left\{x_{c',c,p}, \frac{m_{c,c',p}}{1.08}\right\}$$
After symmetrization all observations are rounded to zero decimals.

## GitHub repositories

* [Getting and cleaning data from UN COMTRADE (OTS Yearly Data)](https://github.com/tradestatistics/ts-yearly-datasets)
* [Scraping data from The Atlas of Economic Complexity (OTS Atlas Data)](https://github.com/tradestatistics/ts-atlas-data)
<!--
* [Product and country codes (OTS Observatory Codes)](https://github.com/tradestatistics/ts-observatory-codes)
-->
* [Product and country codes (OTS Comtrade Codes)](https://github.com/tradestatistics/ts-comtrade-codes)
* [R packages library for reproducibility (OTS Packrat Library)](https://github.com/tradestatistics/ts-packrat-library/)

## Software information

We used R 3.4.3 and RStudio Desktop 1.1 on Ubuntu Desktop 18.04.

We built R from binaries in order to obtain a setup linked with multi-threaded numeric libraries. Our build is linked to OpenBLAS which we used over alternatives such as Intel MKL, BLAS or ATLAS that can also be used.

If you use Windows the scripts will only use a single core because we used a parallelization that depends on fork system call that is only supported on Unix systems. You can always run the scripts on Windows and the only difference will be that it will use less RAM and it will be slower to compute.

Also, before running the scripts on Windows verify that you installed GNU Utilities beforehand. One easy option is to install [Chocolatey](https://chocolatey.org/) first and then install the GNU Utilities by running `choco install unxutils` on Cmd or Power Shell as administrator.

## Hardware information

All the data processing was done by using a Lenovo Thinkpad L380 that features an Intel i5-8250U 1.60 GHz processor and 32 GB (two DDR4 cards of sixteen gigabytes each).

The functions are executed using parallelization on four cores because empirically we detected and overhead due to data communication with the cores when using more cores.

Please notice that running our scripts with parallelization demands more RAM than the amount you can find on an average laptop. You can always disable parallelization in the scripts or reduce the default number of cores.

## Reproducibility notes

To guarantee reproducibility we provide [Packrat](https://rstudio.github.io/packrat/) snapshot and bundles. This prevents changes in syntax, functions or dependencies.

Our R installation is isolated from apt-get to avoid any accidental updates that can alter the data pipeline and/or the output.

The projects are related to each other. In order to avoid multiple copies of files some projects read files from other projects. For example, [OTS Yearly Datasets](https://github.com/tradestatistics/ts-yearly-datasets) input is the output of [OTS Atlas Data](https://github.com/tradestatistics/ts-atlas-data).

The only reproducibility flaw of this project lies in data downloading. Obtaining raw datasets from UN COMTRADE demands an API key that can only be obtained with institutional access that is limited to some universities and institutes.

## Coding style and performant code

We used the [Tidyverse Style Guide](http://style.tidyverse.org/). As cornerstone references for performant code we followed [@advancedr2014] and [@masteringsoftware2017].

Some matrix operations are written in Rcpp to take advantage of C++ speed. To take full advantage of hardware and numerical libraries we am using sparse matrices as it is explained in [@rcpparmadillo2018].

<!--
## Materials of interest

* [Product space layouts (Pacha's Product Space)](https://github.com/pachamaltese/oec-product-space)
-->

## Clean datasets

There is a special consideration you should have with our datasets, and is that you should always read the trade values as a numeric column and the commodity codes as a character column.

Different R packages, and statistical software in general, have includeded functions to autodetect column types. In our experience, that can read commodity codes as integers and that would ignore leading zeroes in commodity codes. The same applies to trade values that can be detected as integers after the program reads the first $n$ rows, and that would lead to read large values incorrectly due to integer class maximum value of 2,147,483,647.

As an example, let's read 1962 data:


```r
# packages

library(data.table)
library(dplyr)
library(stringr)
library(janitor)

# custom functions

messageline <- function() {
  message(rep("-", 60))
}

fread2 <- function(file, select = NULL, character = NULL, numeric = NULL) {
  messageline()
  message("function fread2")
  message("file: ", file)
  
  if(str_sub(file, start = -2) == "gz") {
    d <- fread(
      cmd = paste("zcat", file),
      select = select,
      colClasses = list(
        character = character,
        numeric = numeric
      )
    ) %>%
      as_tibble() %>%
      clean_names()
  } else {
    d <- fread(
      input = file,
      select = select,
      colClasses = list(
        character = character,
        numeric = numeric
      )
    ) %>%
      as_tibble() %>%
      clean_names()
  }
  
  return(d)
}

# download data

url_1962 <- "https://tradestatistics.io/data/04-unified-data/hs-rev2007/hs-rev2007-1962.csv.gz"
gz_1962 <- "hs-rev2007-1962.csv.gz"

if (!file.exists(gz_1962)) {
  try(download.file(url_1962, gz_1962))
}

# read data

data_1962 <- fread2(gz_1962, numeric = "trade_value_usd", character = "commodity_code")

data_1962
```

```
## # A tibble: 983,643 x 6
##     year reporter_iso partner_iso commodity_code commodity_code_…
##    <int> <chr>        <chr>       <chr>                     <int>
##  1  1962 afg          aus         5702                          4
##  2  1962 afg          aus         570210                        6
##  3  1962 afg          aut         0501                          4
##  4  1962 afg          aut         0504                          4
##  5  1962 afg          aut         050400                        6
##  6  1962 afg          aut         5702                          4
##  7  1962 afg          aut         570210                        6
##  8  1962 afg          bel         0501                          4
##  9  1962 afg          bel         0504                          4
## 10  1962 afg          bel         050400                        6
## # ... with 983,633 more rows, and 1 more variable: trade_value_usd <dbl>
```

