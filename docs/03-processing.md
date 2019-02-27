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

### Required packages and objects


```r
# Packages ----------------------------------------------------------------

library(data.table)
library(dplyr)
library(stringr)
library(purrr)
library(janitor)

# Custom functions to read data -------------------------------------------

fread2 <- function(file, select = NULL, character = NULL, numeric = NULL) {
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

# CIF-FOB rate ------------------------------------------------------------
    
# See Anderson & van Wincoop, 2004, Hummels, 2006 and Gaulier & Zignago, 2010 about 8% rate consistency
cif_fob_rate <- 1.08

# Commodity codes length --------------------------------------------------

J <- c(4,6)

# ISO-3 codes -------------------------------------------------------------
    
load("../comtrade-codes/01-2-tidy-country-data/country-codes.RData")
    
country_codes <- country_codes %>% 
  select(iso3_digit_alpha) %>% 
  mutate(iso3_digit_alpha = str_to_lower(iso3_digit_alpha)) %>% 
  filter(!iso3_digit_alpha %in% c("wld","null")) %>% 
  as_vector()

# Five reporters rda file  ------------------------------------------------
rda_file <- "five_reporters_data.rda"
```

### Read raw data


```r
# Read data ---------------------------------------------------------------

if (!file.exists(rda_file)) {
  raw_data <- fread2(
    "../yearly-datasets/01-raw-data/hs-rev2007/gz/type-C_r-ALL_ps-2016_freq-A_px-H3_pub-20180612_fmt-csv_ex-20180705.csv.gz",
    select = c("Year", "Aggregate Level", "Trade Flow", "Reporter ISO", "Partner ISO", "Commodity Code", "Trade Value (US$)"),
    character = "Commodity Code",
    numeric = "Trade Value (US$)"
  )
  
  raw_data
}
```

```
## # A tibble: 25,790,743 x 7
##     year aggregate_level trade_flow reporter_iso partner_iso commodity_code
##    <int>           <int> <chr>      <chr>        <chr>       <chr>         
##  1  2016               2 Import     ALB          WLD         01            
##  2  2016               2 Export     ALB          WLD         01            
##  3  2016               2 Import     ALB          AUT         01            
##  4  2016               2 Import     ALB          BGR         01            
##  5  2016               2 Import     ALB          GRC         01            
##  6  2016               2 Import     ALB          HUN         01            
##  7  2016               2 Import     ALB          ITA         01            
##  8  2016               2 Export     ALB          ITA         01            
##  9  2016               2 Import     ALB          ROU         01            
## 10  2016               2 Export     ALB          SRB         01            
## # … with 25,790,733 more rows, and 1 more variable: trade_value_us <dbl>
```

### Clean data


```r
# Clean data --------------------------------------------------------------

if (!file.exists(rda_file)) {
  clean_data <- raw_data %>%
    
    rename(trade_value_usd = trade_value_us) %>%
    
    filter(aggregate_level %in% J) %>%
    filter(trade_flow %in% c("Export","Import")) %>%
    
    filter(
      !is.na(commodity_code),
      commodity_code != "",
      commodity_code != " "
    ) %>%
    
    mutate(
      reporter_iso = str_to_lower(reporter_iso),
      partner_iso = str_to_lower(partner_iso)
    ) %>%
    
    filter(
      reporter_iso %in% country_codes,
      partner_iso %in% country_codes
    ) %>% 
    
    # Provided Comtrade data cannot be redistributed, I'll limit the
    # example to five reporters
    
    filter(
      reporter_iso == "chl",
      partner_iso %in% c("arg", "bra", "per", "usa")
    )
  
  save(clean_data, file = rda_file, compress = "xz")
  
  clean_data
} else {
  load(rda_file)
  
  clean_data
}
```

```
## # A tibble: 22,517 x 7
##     year aggregate_level trade_flow reporter_iso partner_iso commodity_code
##    <int>           <int> <chr>      <chr>        <chr>       <chr>         
##  1  2016               4 Import     chl          arg         0101          
##  2  2016               4 Export     chl          arg         0101          
##  3  2016               4 Import     chl          bra         0101          
##  4  2016               4 Export     chl          per         0101          
##  5  2016               4 Import     chl          usa         0101          
##  6  2016               4 Export     chl          usa         0101          
##  7  2016               6 Import     chl          arg         010110        
##  8  2016               6 Export     chl          arg         010110        
##  9  2016               6 Import     chl          bra         010110        
## 10  2016               6 Export     chl          per         010110        
## # … with 22,507 more rows, and 1 more variable: trade_value_usd <dbl>
```

### Symmetric (clean) data

```r
# Exports data ------------------------------------------------------------

exports <- clean_data %>%
  filter(trade_flow == "Export") %>%
  select(reporter_iso, partner_iso, commodity_code, trade_value_usd) %>% 
  mutate(trade_value_usd = ceiling(trade_value_usd))

exports_mirrored <- clean_data %>%
  filter(trade_flow == "Import") %>%
  select(reporter_iso, partner_iso, commodity_code, trade_value_usd) %>% 
  mutate(trade_value_usd = ceiling(trade_value_usd / cif_fob_rate))

# Reporter and Partner must be inverted
colnames(exports_mirrored) <- c("partner_iso", "reporter_iso", "commodity_code", "trade_value_usd")

exports_model <- exports %>% 
  full_join(exports_mirrored, by = c("reporter_iso", "partner_iso", "commodity_code")) %>% 
  rowwise() %>% 
  mutate(trade_value_usd = max(trade_value_usd.x, trade_value_usd.y, na.rm = T)) %>% 
  ungroup() %>% 
  mutate(commodity_code_parent = str_sub(commodity_code, 1, 4)) %>% 
  group_by(reporter_iso, partner_iso, commodity_code_parent) %>% 
  mutate(parent_count = n()) %>% 
  ungroup() %>% 
  select(reporter_iso, partner_iso, commodity_code, commodity_code_parent, parent_count, trade_value_usd)

exports_model_unrepeated_parent <- exports_model %>% 
  filter(parent_count == 1)

exports_model_repeated_parent <- exports_model %>% 
  filter(
    parent_count > 1,
    str_length(commodity_code) %in% c(5,6)
  )

exports_model_repeated_parent_summary <- exports_model_repeated_parent %>% 
  group_by(reporter_iso, partner_iso, commodity_code_parent) %>% 
  summarise(trade_value_usd = sum(trade_value_usd, na.rm = T)) %>% 
  ungroup() %>% 
  rename(commodity_code = commodity_code_parent)

exports_model <- exports_model_unrepeated_parent %>% 
  bind_rows(exports_model_repeated_parent) %>% 
  bind_rows(exports_model_repeated_parent_summary) %>% 
  arrange(reporter_iso, partner_iso, commodity_code) %>% 
  mutate(
    year = 2016,
    commodity_code_length = str_length(commodity_code)
  ) %>% 
  select(year, reporter_iso, partner_iso, commodity_code, commodity_code_length, trade_value_usd) %>% 
  filter(trade_value_usd > 0)

exports_model
```

```
## # A tibble: 22,484 x 6
##     year reporter_iso partner_iso commodity_code commodity_code_…
##    <dbl> <chr>        <chr>       <chr>                     <int>
##  1  2016 arg          chl         0101                          4
##  2  2016 arg          chl         010110                        6
##  3  2016 arg          chl         010190                        6
##  4  2016 arg          chl         0106                          4
##  5  2016 arg          chl         010619                        6
##  6  2016 arg          chl         0201                          4
##  7  2016 arg          chl         020130                        6
##  8  2016 arg          chl         0202                          4
##  9  2016 arg          chl         020230                        6
## 10  2016 arg          chl         0206                          4
## # … with 22,474 more rows, and 1 more variable: trade_value_usd <dbl>
```

## GitHub repositories

* [Getting and cleaning data from UN COMTRADE (OTS Yearly Data)](https://github.com/tradestatistics/yearly-datasets)
* [Scraping data from The Atlas of Economic Complexity (OTS Atlas Data)](https://github.com/tradestatistics/atlas-data)
* [Product and country codes (OTS Comtrade Codes)](https://github.com/tradestatistics/comtrade-codes)
* [R packages library for reproducibility (OTS Packrat Library)](https://github.com/tradestatistics/packrat-library/)

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

The projects are related to each other. In order to avoid multiple copies of files some projects read files from other projects. For example, [OTS Yearly Datasets](https://github.com/tradestatistics/yearly-datasets) input is the output of [OTS Atlas Data](https://github.com/tradestatistics/atlas-data).

The only reproducibility flaw of this project lies in data downloading. Obtaining raw datasets from UN COMTRADE demands an API key that can only be obtained with institutional access that is limited to some universities and institutes.

## Coding style and performant code

We used the [Tidyverse Style Guide](http://style.tidyverse.org/). As cornerstone references for performant code we followed [@advancedr2014] and [@masteringsoftware2017].

Some matrix operations are written in Rcpp to take advantage of C++ speed. To take full advantage of hardware and numerical libraries we am using sparse matrices as it is explained in [@rcpparmadillo2018].
