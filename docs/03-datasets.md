# Datasets

## Code of Conduct

Before you proceed to download the data, please read this carefully.

No matter about your gender, gender identity and expression, age, sexual orientation, disability, physical appearance, body size, race, ethnicity, religion (or lack thereof), or technology choices you are able to use this data for any non-commercial purpose, including academic.

Commercial purposes are strictly out of the boundaries of what you can do with this data according to UN Comtrade dissemination clauses.

Our datasets are distributed under Creative Commons Attribution-NonCommercial 4.0 International License.

Before downloading you agree to the usage conditions explained both to [UN Comtrade Online Usage Agreement](https://comtrade.un.org/db/help/licenseagreement.aspx) and [Creative Commons BY-NC 4.0 License](https://creativecommons.org/licenses/by-nc/4.0/).

## Compressed data

### How to use

There is a special consideration you should have with our datasets, and is that you should always read the trade values as a numeric column and the commodity codes as a character column.

Different R packages, and statistical software in general, have includeded functions to autodetect column types. In our experience, that can read commodity codes as integers and that would ignore leading zeroes in commodity codes. The same applies to trade values that can be detected as integers after the program reads the first $n$ rows, and that would lead to read large values incorrectly due to integer class maximum value of 2,147,483,647.

As an example, let's read 1962 data and explore what the United Stated exported:


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

url_1962 <- "https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1962.csv.gz"
gz_1962 <- "yrpc-1962.csv.gz"

if (!file.exists(gz_1962)) {
  try(download.file(url_1962, gz_1962))
}

# read data

rda_1962 <- "api_data_1962.rda"

if (!file.exists(rda_1962)) {
  data_1962 <- fread2(
        gz_1962,
        character = "commodity_code",
        numeric = c(
          "export_value_usd",
          "import_value_usd",
          "export_value_usd_change_1_year",
          "export_value_usd_change_5_years",
          "export_value_usd_percentage_change_1_year",
          "export_value_usd_percentage_change_5_years",
          "import_value_usd_change_1_year",
          "import_value_usd_change_5_years",
          "import_value_usd_percentage_change_1_year",
          "import_value_usd_percentage_change_5_years"
        )
      ) %>% 
    filter(reporter_iso == "usa")
} else {
  load(rda_1962)
}

data_1962
```

```
## # A tibble: 49,383 x 7
##     year reporter_iso partner_iso commodity_code commodity_code_…
##    <int> <chr>        <chr>       <chr>                     <int>
##  1  1962 usa          afg         0201                          4
##  2  1962 usa          afg         0210                          4
##  3  1962 usa          afg         0401                          4
##  4  1962 usa          afg         0402                          4
##  5  1962 usa          afg         0405                          4
##  6  1962 usa          afg         0406                          4
##  7  1962 usa          afg         0409                          4
##  8  1962 usa          afg         0710                          4
##  9  1962 usa          afg         0801                          4
## 10  1962 usa          afg         0802                          4
## # … with 49,373 more rows, and 2 more variables: export_value_usd <int>,
## #   import_value_usd <int>
```

### Available datasets

Please check the [md5sums](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/md5sums.txt) to verify data integrity after downloading.

|Year                                                                                |Last updated|File size (MB)|md5sum                          |
|------------------------------------------------------------------------------------|------------|--------------|--------------------------------|
|[1962](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1962.csv.gz)|2018-12-26  |9.19          |0ee7b2a390d2f377e0bae1b630e28542|
|[1963](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1963.csv.gz)|2018-12-26  |23.87         |08b8af0a652b6e0c470c11ca7275f12b|
|[1964](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1964.csv.gz)|2018-12-26  |26.84         |57496ec83e7c1b9893c1250162eff7e3|
|[1965](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1965.csv.gz)|2018-12-26  |29.69         |bda7ba503f93c1d5faa4989b943ba737|
|[1966](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1966.csv.gz)|2018-12-26  |31.81         |21b8a1fb12431e1bd395035a8c1cf10a|
|[1967](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1967.csv.gz)|2018-12-26  |44.65         |ab62aeaaa32f054fca88b146aa65f960|
|[1968](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1968.csv.gz)|2018-12-26  |47.85         |db43871ebb70c1d7065f9effa6f69890|
|[1969](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1969.csv.gz)|2018-12-26  |50.47         |4dffda0775f3baf17ad5d8b202d77fab|
|[1970](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1970.csv.gz)|2018-12-26  |54.27         |a1dffb9b9e8f72f541c31734f45a3aa9|
|[1971](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1971.csv.gz)|2018-12-26  |57.00         |d9987b341f5ee211371dc71083a91e3b|
|[1972](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1972.csv.gz)|2018-12-26  |59.30         |e20d9967738d7bd5eabb7e71ecc8d260|
|[1973](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1973.csv.gz)|2018-12-26  |62.38         |0eae7662b263ba9f2a34b1ffa9e0c8b6|
|[1974](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1974.csv.gz)|2018-12-26  |66.50         |e702d1481b9bba1b22221159d0ece98e|
|[1975](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1975.csv.gz)|2018-12-26  |68.99         |78f71d0d52dc034e507e5fdbac2017a8|
|[1976](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1976.csv.gz)|2018-12-26  |68.18         |9b45cefd538278095e7fc1b0fcfdbbd0|
|[1977](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1977.csv.gz)|2018-12-26  |73.94         |cc31ade14b25f695263b025ee220207e|
|[1978](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1978.csv.gz)|2018-12-26  |80.57         |dd4641c9b2930c952fff623ab0e38566|
|[1979](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1979.csv.gz)|2018-12-26  |89.33         |9461d28a6a78c1f758042626bfe61b96|
|[1980](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1980.csv.gz)|2018-12-26  |93.72         |72a71cff48fecca1e163ea2df469b7d3|
|[1981](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1981.csv.gz)|2018-12-26  |97.12         |6d79c42219b91bc00dfa404b209216e3|
|[1982](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1982.csv.gz)|2018-12-26  |98.41         |8bf61f594f5c2784640fb1376a192b98|
|[1983](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1983.csv.gz)|2018-12-26  |103.35        |be6d2e91a2f57f3726b86f8023fdac55|
|[1984](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1984.csv.gz)|2018-12-26  |104.76        |f4e8b0ff8fab6b7c3a04f0a01e2e13fa|
|[1985](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1985.csv.gz)|2018-12-26  |108.06        |bbd7e0ae505d92a9c2349b958111728f|
|[1986](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1986.csv.gz)|2018-12-26  |111.57        |e7b483da859ac3b6e5c6960be2135b00|
|[1987](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1987.csv.gz)|2018-12-26  |114.11        |8910c7a185beed6de76453cc5ae05518|
|[1988](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1988.csv.gz)|2018-12-26  |116.44        |0b1e03cef2e34ad2863b281fe0712e2d|
|[1989](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1989.csv.gz)|2018-12-26  |121.48        |3860a7c7f258d45572cb4fe05a7b3084|
|[1990](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1990.csv.gz)|2018-12-26  |126.79        |173fdd675c79d56309e1aa5bd468ea49|
|[1991](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1991.csv.gz)|2018-12-26  |128.31        |f0ad8f0c27c74da2a12b14c6bcfc2d1a|
|[1992](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1992.csv.gz)|2018-12-26  |142.85        |a2e5b03eb3bec5f17e4a0b1642194764|
|[1993](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1993.csv.gz)|2018-12-26  |183.53        |c7d6d0710c92359515037f5082a9f8ee|
|[1994](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1994.csv.gz)|2018-12-26  |209.98        |55b0176c4758e6fa22ee5e8f32cc1104|
|[1995](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1995.csv.gz)|2018-12-26  |231.29        |3d76b808d6591e151408428c2a17baa8|
|[1996](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1996.csv.gz)|2018-12-26  |225.08        |bb96e401745a5d938920a5feac6ad021|
|[1997](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1997.csv.gz)|2018-12-26  |271.01        |92e29059c5b2ac445f72808f515b58d2|
|[1998](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1998.csv.gz)|2018-12-26  |293.61        |1b363cf978437d76fb3ea08a2b87664b|
|[1999](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-1999.csv.gz)|2018-12-26  |314.98        |e9a07d7ea42b8de69247cf83fde1d251|
|[2000](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2000.csv.gz)|2018-12-26  |357.59        |3ff097a95a7b4296c5d008f1c42738c3|
|[2001](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2001.csv.gz)|2018-12-26  |380.78        |dadaeca7f9978cc86ba56ea6f36f5378|
|[2002](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2002.csv.gz)|2018-12-26  |374.03        |67c77761b776c862b954a9c82e273dec|
|[2003](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2003.csv.gz)|2018-12-26  |390.76        |ae450ddcbaacc0107098d7a5b76bd4e4|
|[2004](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2004.csv.gz)|2018-12-26  |419.84        |749c6522a55cdaa5ebce49c5a89e7ef9|
|[2005](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2005.csv.gz)|2018-12-26  |454.80        |5094d98e973a3eeb238d2af21206a2a5|
|[2006](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2006.csv.gz)|2018-12-26  |477.17        |09a4821eed7d29e70ced090e5b35e59d|
|[2007](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2007.csv.gz)|2018-12-26  |452.87        |f5a91e1d237089a6a22ffad6c8bf02c5|
|[2008](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2008.csv.gz)|2018-12-26  |482.61        |fcb85ae525ec351f2d7db4c2460989b1|
|[2009](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2009.csv.gz)|2018-12-26  |507.08        |7db4e1e78830caf52271815090e93027|
|[2010](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2010.csv.gz)|2018-12-26  |531.34        |e3f281de51f49177e815a24eb5c4e9b3|
|[2011](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2011.csv.gz)|2018-12-26  |553.89        |1320d282a6b8d46e70f9508be129b866|
|[2012](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2012.csv.gz)|2018-12-26  |563.45        |7ceb21d7db6606578ec4204bebae0dab|
|[2013](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2013.csv.gz)|2018-12-26  |586.78        |9b25e8221b607dc5d3b706db118e78a0|
|[2014](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2014.csv.gz)|2018-12-26  |596.02        |bec1182c084629bb472aec24c0faea58|
|[2015](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2015.csv.gz)|2018-12-26  |606.73        |9226c31eaf7f353dd8b3400b1cba605a|
|[2016](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2016.csv.gz)|2018-12-26  |613.93        |83b4da48894c675563e0b2888a350bab|
|[2017](https://data.tradestatistics.io/06-tables/hs-rev2007/1-yrpc/yrpc-2017.csv.gz)|1888-01-31  |610.42        |1b68922b01cde532ae0ea6ffb5aad8f7|

## API

The advantage of the API over https download is that you can filter what to obtain and also access some additional tables.

To obtain exactly the same data as with compressed files, please refer to \@ref(yrpc).

If you use R you'll need `jsonlite` package.


```r
library(jsonlite)
```

### Available tables


```r
as_tibble(fromJSON("https://api.tradestatistics.io/tables"))
```

```
## # A tibble: 14 x 3
##    table      description                           source                 
##    <chr>      <chr>                                 <chr>                  
##  1 countries  Countries metadata                    UN Comtrade            
##  2 products   Product metadata                      UN Comtrade            
##  3 communiti… Product communities                   Harvard's Center for I…
##  4 reporters  Reporting countries                   UN Comtrade            
##  5 country_r… Ranking of countries                  Open Trade Statistics  
##  6 product_r… Ranking of products                   Open Trade Statistics  
##  7 yrpc       Bilateral trade at commodity level (… Open Trade Statistics  
##  8 yrp        Bilateral trade at aggregated level … Open Trade Statistics  
##  9 yrc        Bilateral trade at aggregated level … Open Trade Statistics  
## 10 yrc_expor… Bilateral trade at aggregated level … Open Trade Statistics  
## 11 yrc_impor… Reporter trade at commodity level (Y… Open Trade Statistics  
## 12 yr         Reporter trade at aggregated level (… Open Trade Statistics  
## 13 yr_short   Reporter trade at aggregated level (… Open Trade Statistics  
## 14 yc         Commodity trade at aggregated level … Open Trade Statistics
```

### Metadata


```r
## Countries (no filter)
rda_countries <- "countries.rda"

if (!file.exists(rda_countries)) {
  countries <- as_tibble(fromJSON(
    "https://api.tradestatistics.io/countries"
  ))
  
  save(countries, file = rda_countries, compress = "xz")
  
  countries
} else {
  load(rda_countries)
  
  countries
}
```

```
## # A tibble: 249 x 6
##    country_iso country_name_en… country_fullnam… continent_id continent
##    <chr>       <chr>            <chr>                   <int> <chr>    
##  1 afg         Afghanistan      Afghanistan                 1 Asia     
##  2 alb         Albania          Albania                     2 Europe   
##  3 dza         Algeria          Algeria                     3 Africa   
##  4 asm         American Samoa   American Samoa              4 Oceania  
##  5 and         Andorra          Andorra                     2 Europe   
##  6 ago         Angola           Angola                      3 Africa   
##  7 aia         Anguilla         Anguilla                    5 Americas 
##  8 atg         Antigua and Bar… Antigua and Bar…            5 Americas 
##  9 arg         Argentina        Argentina                   5 Americas 
## 10 arm         Armenia          Armenia                     1 Asia     
## # … with 239 more rows, and 1 more variable: eu28_member <int>
```

```r
## Products (no filter)
rda_products <- "products.rda"

if (!file.exists(rda_products)) {
  products <- as_tibble(fromJSON(
    "https://api.tradestatistics.io/products"
  ))
  
  save(products, file = rda_products, compress = "xz")
  
  products
} else {
  load(rda_products)
  
  products
}
```

```
## # A tibble: 6,373 x 4
##    commodity_code commodity_fullname_english          group_code group_name
##    <chr>          <chr>                               <chr>      <chr>     
##  1 0101           Horses, asses, mules and hinnies; … 01         Animals; …
##  2 010110         Horses, asses, mules and hinnies; … 01         Animals; …
##  3 010190         Horses, asses, mules and hinnies; … 01         Animals; …
##  4 0102           Bovine animals; live                01         Animals; …
##  5 010210         Bovine animals; live, pure-bred br… 01         Animals; …
##  6 010290         Bovine animals; live, other than p… 01         Animals; …
##  7 0103           Swine; live                         01         Animals; …
##  8 010310         Swine; live, pure-bred breeding an… 01         Animals; …
##  9 010391         Swine; live, (other than pure-bred… 01         Animals; …
## 10 010392         Swine; live, (other than pure-bred… 01         Animals; …
## # … with 6,363 more rows
```

Please notice that these tables include some aliases. 

`countries` includes some meta-codes, `c-xx` where `xx` must the first two letters of a continent and `all`, this is:


alias   meaning                                       
------  ----------------------------------------------
c-af    Alias for all valid ISO codes in Africa       
c-am    Alias for all valid ISO codes in the Americas 
c-as    Alias for all valid ISO codes in Asia         
c-eu    Alias for all valid ISO codes in Europe       
c-oc    Alias for all valid ISO codes in Oceania      
all     Alias for all valid ISO codes in the World    

`products` also includes some meta-codes, `xx` for the first two digits of a code and those digits are the product group and `all`, this is:


alias   meaning                                                                                                                                                                                                                                                                    
------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
01      Alias for all codes in the group Animals; live                                                                                                                                                                                                                             
02      Alias for all codes in the group Meat and edible meat offal                                                                                                                                                                                                                
03      Alias for all codes in the group Fish and crustaceans, molluscs and other aquatic invertebrates                                                                                                                                                                            
04      Alias for all codes in the group Dairy produce; birds' eggs; natural honey; edible products of animal origin, not elsewhere specified or included                                                                                                                          
06      Alias for all codes in the group Trees and other plants, live; bulbs, roots and the like; cut flowers and ornamental foliage                                                                                                                                               
05      Alias for all codes in the group Animal originated products; not elsewhere specified or included                                                                                                                                                                           
07      Alias for all codes in the group Vegetables and certain roots and tubers; edible                                                                                                                                                                                           
29      Alias for all codes in the group Organic chemicals                                                                                                                                                                                                                         
08      Alias for all codes in the group Fruit and nuts, edible; peel of citrus fruit or melons                                                                                                                                                                                    
09      Alias for all codes in the group Coffee, tea, mate and spices                                                                                                                                                                                                              
10      Alias for all codes in the group Cereals                                                                                                                                                                                                                                   
11      Alias for all codes in the group Products of the milling industry; malt, starches, inulin, wheat gluten                                                                                                                                                                    
12      Alias for all codes in the group Oil seeds and oleaginous fruits; miscellaneous grains, seeds and fruit, industrial or medicinal plants; straw and fodder                                                                                                                  
26      Alias for all codes in the group Ores, slag and ash                                                                                                                                                                                                                        
13      Alias for all codes in the group Lac; gums, resins and other vegetable saps and extracts                                                                                                                                                                                   
14      Alias for all codes in the group Vegetable plaiting materials; vegetable products not elsewhere specified or included                                                                                                                                                      
15      Alias for all codes in the group Animal or vegetable fats and oils and their cleavage products; prepared animal fats; animal or vegetable waxes                                                                                                                            
22      Alias for all codes in the group Beverages, spirits and vinegar                                                                                                                                                                                                            
16      Alias for all codes in the group Meat, fish or crustaceans, molluscs or other aquatic invertebrates; preparations thereof                                                                                                                                                  
17      Alias for all codes in the group Sugars and sugar confectionery                                                                                                                                                                                                            
18      Alias for all codes in the group Cocoa and cocoa preparations                                                                                                                                                                                                              
19      Alias for all codes in the group Preparations of cereals, flour, starch or milk; pastrycooks' products                                                                                                                                                                     
20      Alias for all codes in the group Preparations of vegetables, fruit, nuts or other parts of plants                                                                                                                                                                          
21      Alias for all codes in the group Miscellaneous edible preparations                                                                                                                                                                                                         
23      Alias for all codes in the group Food industries, residues and wastes thereof; prepared animal fodder                                                                                                                                                                      
24      Alias for all codes in the group Tobacco and manufactured tobacco substitutes                                                                                                                                                                                              
25      Alias for all codes in the group Salt; sulphur; earths, stone; plastering materials, lime and cement                                                                                                                                                                       
27      Alias for all codes in the group Mineral fuels, mineral oils and products of their distillation; bituminous substances; mineral waxes                                                                                                                                      
28      Alias for all codes in the group Inorganic chemicals; organic and inorganic compounds of precious metals; of rare earth metals, of radio-active elements and of isotopes                                                                                                   
30      Alias for all codes in the group Pharmaceutical products                                                                                                                                                                                                                   
35      Alias for all codes in the group Albuminoidal substances; modified starches; glues; enzymes                                                                                                                                                                                
31      Alias for all codes in the group Fertilizers                                                                                                                                                                                                                               
32      Alias for all codes in the group Tanning or dyeing extracts; tannins and their derivatives; dyes, pigments and other colouring matter; paints, varnishes; putty, other mastics; inks                                                                                       
39      Alias for all codes in the group Plastics and articles thereof                                                                                                                                                                                                             
33      Alias for all codes in the group Essential oils and resinoids; perfumery, cosmetic or toilet preparations                                                                                                                                                                  
34      Alias for all codes in the group Soap, organic surface-active agents; washing, lubricating, polishing or scouring preparations; artificial or prepared waxes, candles and similar articles, modelling pastes, dental waxes and dental preparations with a basis of plaster 
36      Alias for all codes in the group Explosives; pyrotechnic products; matches; pyrophoric alloys; certain combustible preparations                                                                                                                                            
37      Alias for all codes in the group Photographic or cinematographic goods                                                                                                                                                                                                     
38      Alias for all codes in the group Chemical products n.e.c.                                                                                                                                                                                                                  
40      Alias for all codes in the group Rubber and articles thereof                                                                                                                                                                                                               
44      Alias for all codes in the group Wood and articles of wood; wood charcoal                                                                                                                                                                                                  
41      Alias for all codes in the group Raw hides and skins (other than furskins) and leather                                                                                                                                                                                     
70      Alias for all codes in the group Glass and glassware                                                                                                                                                                                                                       
42      Alias for all codes in the group Articles of leather; saddlery and harness; travel goods, handbags and similar containers; articles of animal gut (other than silk-worm gut)                                                                                               
43      Alias for all codes in the group Furskins and artificial fur; manufactures thereof                                                                                                                                                                                         
45      Alias for all codes in the group Cork and articles of cork                                                                                                                                                                                                                 
46      Alias for all codes in the group Manufactures of straw, esparto or other plaiting materials; basketware and wickerwork                                                                                                                                                     
47      Alias for all codes in the group Pulp of wood or other fibrous cellulosic material; recovered (waste and scrap) paper or paperboard                                                                                                                                        
52      Alias for all codes in the group Cotton                                                                                                                                                                                                                                    
48      Alias for all codes in the group Paper and paperboard; articles of paper pulp, of paper or paperboard                                                                                                                                                                      
49      Alias for all codes in the group Printed books, newspapers, pictures and other products of the printing industry; manuscripts, typescripts and plans                                                                                                                       
50      Alias for all codes in the group Silk                                                                                                                                                                                                                                      
51      Alias for all codes in the group Wool, fine or coarse animal hair; horsehair yarn and woven fabric                                                                                                                                                                         
53      Alias for all codes in the group Vegetable textile fibres; paper yarn and woven fabrics of paper yarn                                                                                                                                                                      
54      Alias for all codes in the group Man-made filaments; strip and the like of man-made textile materials                                                                                                                                                                      
55      Alias for all codes in the group Man-made staple fibres                                                                                                                                                                                                                    
62      Alias for all codes in the group Apparel and clothing accessories; not knitted or crocheted                                                                                                                                                                                
56      Alias for all codes in the group Wadding, felt and nonwovens, special yarns; twine, cordage, ropes and cables and articles thereof                                                                                                                                         
57      Alias for all codes in the group Carpets and other textile floor coverings                                                                                                                                                                                                 
58      Alias for all codes in the group Fabrics; special woven fabrics, tufted textile fabrics, lace, tapestries, trimmings, embroidery                                                                                                                                           
59      Alias for all codes in the group Textile fabrics; impregnated, coated, covered or laminated; textile articles of a kind suitable for industrial use                                                                                                                        
60      Alias for all codes in the group Fabrics; knitted or crocheted                                                                                                                                                                                                             
61      Alias for all codes in the group Apparel and clothing accessories; knitted or crocheted                                                                                                                                                                                    
95      Alias for all codes in the group Toys, games and sports requisites; parts and accessories thereof                                                                                                                                                                          
63      Alias for all codes in the group Textiles, made up articles; sets; worn clothing and worn textile articles; rags                                                                                                                                                           
64      Alias for all codes in the group Footwear; gaiters and the like; parts of such articles                                                                                                                                                                                    
65      Alias for all codes in the group Headgear and parts thereof                                                                                                                                                                                                                
66      Alias for all codes in the group Umbrellas, sun umbrellas, walking-sticks, seat sticks, whips, riding crops; and parts thereof                                                                                                                                             
67      Alias for all codes in the group Feathers and down, prepared; and articles made of feather or of down; artificial flowers; articles of human hair                                                                                                                          
68      Alias for all codes in the group Stone, plaster, cement, asbestos, mica or similar materials; articles thereof                                                                                                                                                             
69      Alias for all codes in the group Ceramic products                                                                                                                                                                                                                          
71      Alias for all codes in the group Natural, cultured pearls; precious, semi-precious stones; precious metals, metals clad with precious metal, and articles thereof; imitation jewellery; coin                                                                               
72      Alias for all codes in the group Iron and steel                                                                                                                                                                                                                            
73      Alias for all codes in the group Iron or steel articles                                                                                                                                                                                                                    
74      Alias for all codes in the group Copper and articles thereof                                                                                                                                                                                                               
75      Alias for all codes in the group Nickel and articles thereof                                                                                                                                                                                                               
76      Alias for all codes in the group Aluminium and articles thereof                                                                                                                                                                                                            
81      Alias for all codes in the group Metals; n.e.c., cermets and articles thereof                                                                                                                                                                                              
78      Alias for all codes in the group Lead and articles thereof                                                                                                                                                                                                                 
79      Alias for all codes in the group Zinc and articles thereof                                                                                                                                                                                                                 
80      Alias for all codes in the group Tin; articles thereof                                                                                                                                                                                                                     
82      Alias for all codes in the group Tools, implements, cutlery, spoons and forks, of base metal; parts thereof, of base metal                                                                                                                                                 
83      Alias for all codes in the group Metal; miscellaneous products of base metal                                                                                                                                                                                               
84      Alias for all codes in the group Nuclear reactors, boilers, machinery and mechanical appliances; parts thereof                                                                                                                                                             
85      Alias for all codes in the group Electrical machinery and equipment and parts thereof; sound recorders and reproducers; television image and sound recorders and reproducers, parts and accessories of such articles                                                       
88      Alias for all codes in the group Aircraft, spacecraft and parts thereof                                                                                                                                                                                                    
91      Alias for all codes in the group Clocks and watches and parts thereof                                                                                                                                                                                                      
86      Alias for all codes in the group Railway, tramway locomotives, rolling-stock and parts thereof; railway or tramway track fixtures and fittings and parts thereof; mechanical (including electro-mechanical) traffic signalling equipment of all kinds                      
87      Alias for all codes in the group Vehicles; other than railway or tramway rolling stock, and parts and accessories thereof                                                                                                                                                  
89      Alias for all codes in the group Ships, boats and floating structures                                                                                                                                                                                                      
90      Alias for all codes in the group Optical, photographic, cinematographic, measuring, checking, medical or surgical instruments and apparatus; parts and accessories                                                                                                         
92      Alias for all codes in the group Musical instruments; parts and accessories of such articles                                                                                                                                                                               
93      Alias for all codes in the group Arms and ammunition; parts and accessories thereof                                                                                                                                                                                        
94      Alias for all codes in the group Furniture; bedding, mattresses, mattress supports, cushions and similar stuffed furnishings; lamps and lighting fittings, n.e.c.; illuminated signs, illuminated name-plates and the like; prefabricated buildings                        
96      Alias for all codes in the group Miscellaneous manufactured articles                                                                                                                                                                                                       
97      Alias for all codes in the group Works of art; collectors' pieces and antiques                                                                                                                                                                                             
99      Alias for all codes in the group Commodities not specified according to kind                                                                                                                                                                                               
all     Alias for all codes                                                                                                                                                                                                                                                        

### API parameters

The tables provided withing our API contain at least one of these fields:

* Year (`y`) 
* Reporter ISO (`r`)
* Partner ISO (`p`)
* Commodity code (`c`)

The most detailed table is `yrpc` that contains all bilateral flows at product level.

With respect to `y` you can pass any integer contained in $[1962,2017]$.

Both `r` and `p` accept any valid ISO code or alias contained in the [countries](https://api.tradestatistics.io/countries) table. For example, both `chl` (valid ISO code) and `c-am` (continent Americas, an alias) are valid API filtering parameters.

`c` takes any valid commodity code or alias from the [products](https://api.tradestatistics.io/products). For example, both `0101` (valid HS product code) and `01` (valid HS group code) are valid API filtering parameters.

In addition to `y, r, p, c` parameters, the length (`l`) parameter allows efficient queries provided our data contains both 4 and 6 digits long commodity codes. Because 4 digits code contain 6 digits codes, our approach is to allow the user to use `l=4`, `l=6` or `l=all` to provide just the requested data.

By default the API takes `c = "all"` and `l = 4` as defaults.

You can always skip `c` or `l`, but `y`, `r` and `p` are requiered to return data.

### Available reporters

The only applicable filter is by year.


```r
# Available reporters (filter by year)
as_tibble(fromJSON(
  "https://api.tradestatistics.io/reporters?y=2015"
))
```

```
## # A tibble: 224 x 1
##    reporter_iso
##    <chr>       
##  1 afg         
##  2 ago         
##  3 aia         
##  4 alb         
##  5 and         
##  6 are         
##  7 arg         
##  8 arm         
##  9 asm         
## 10 atf         
## # … with 214 more rows
```

### YRPC

**Year - Reporter - Partner - Commodity**

By using the API you can obtain the same result as in section \@ref(compressed-data), but in a simpler way:


```r
# Year - Reporter - Partner - Commodity (filter by year, reporter and partner)

## filter by commodity length (parameter `l`)
if (!file.exists(rda_1962)) {
  yrpc_1 <- as_tibble(fromJSON(
    "https://api.tradestatistics.io/yrpc?y=1962&r=usa&p=all&l=4"
  ))
  
  yrpc_1
} else {
  load(rda_1962)
  
  yrpc_1 <- data_1962
  
  yrpc_1
}
```

```
## # A tibble: 49,383 x 7
##     year reporter_iso partner_iso commodity_code commodity_code_…
##    <int> <chr>        <chr>       <chr>                     <int>
##  1  1962 usa          afg         0201                          4
##  2  1962 usa          afg         0210                          4
##  3  1962 usa          afg         0401                          4
##  4  1962 usa          afg         0402                          4
##  5  1962 usa          afg         0405                          4
##  6  1962 usa          afg         0406                          4
##  7  1962 usa          afg         0409                          4
##  8  1962 usa          afg         0710                          4
##  9  1962 usa          afg         0801                          4
## 10  1962 usa          afg         0802                          4
## # … with 49,373 more rows, and 2 more variables: export_value_usd <int>,
## #   import_value_usd <int>
```

```r
## filter by commodity group (parameter `c`)
yrpc_2 <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrpc?y=2015&r=chl&p=arg&c=01"
))

yrpc_2
```

```
## # A tibble: 5 x 15
##    year reporter_iso partner_iso commodity_code commodity_code_…
##   <int> <chr>        <chr>       <chr>                     <int>
## 1  2015 chl          arg         0101                          4
## 2  2015 chl          arg         010110                        6
## 3  2015 chl          arg         010190                        6
## 4  2015 chl          arg         0106                          4
## 5  2015 chl          arg         010619                        6
## # … with 10 more variables: export_value_usd <int>,
## #   import_value_usd <int>, export_value_usd_change_1_year <int>,
## #   export_value_usd_change_5_years <int>,
## #   export_value_usd_percentage_change_1_year <dbl>,
## #   export_value_usd_percentage_change_5_years <dbl>,
## #   import_value_usd_change_1_year <int>,
## #   import_value_usd_change_5_years <int>,
## #   import_value_usd_percentage_change_1_year <dbl>,
## #   import_value_usd_percentage_change_5_years <dbl>
```

Some columns requiere an explanation:

* `commodity_code`: HS07 product codes (e.g. according to the \code{products} table within this package, 0101 stands for "Horses, etc.")
* `commodity_code_length`: How many digits does `commodity_code` contain, this can be useful to filter by depth when using HS codes (HS 6 digits is a more detailed version of HS 4 digits, and therefore you don't have to sum both or you'll be counting exports/imports twice)
* `group_code`: International categorization of group products defined after product ID
* `group_name`: English name corresponding to `group_id`
* `export_value_usd`: Exports measured in nominal United States Dollars (USD)
* `import_value_usd`: Imports measured in nominal United States Dollars (USD)
* `export_value_usd_percentage_change_1_year`: Nominal increase/decrease in exports measured as percentage with respect to last year
* `export_value_usd_percentage_change_5_years`: Nominal increase/decrease in exports measured as percentage with respect to five years ago
* `export_value_usd_change_1_year`: Nominal increase/decrease in exports measured in USD with respect to last year
* `export_value_usd_change_5_years`: Nominal increase/decrease in exports measured in USD with respect to five years ago

### YRC

**Year - Reporter - Commodity**

The only applicable filter is by year and reporter.


```r
# Year - Reporter - Commodity (filter by year and reporter)

## filter by reporter ISO (parameter `r`)
yrc_1 <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrc?y=2015&r=chl&l=4"
))

yrc_1
```

```
## # A tibble: 1,214 x 16
##     year reporter_iso commodity_code commodity_code_… export_value_usd
##    <int> <chr>        <chr>                     <int>            <dbl>
##  1  2015 chl          0101                          4          9958013
##  2  2015 chl          0102                          4         88938806
##  3  2015 chl          0103                          4           956016
##  4  2015 chl          0104                          4               NA
##  5  2015 chl          0105                          4               NA
##  6  2015 chl          0106                          4          5290017
##  7  2015 chl          0201                          4          7812764
##  8  2015 chl          0202                          4         25146882
##  9  2015 chl          0203                          4        402815596
## 10  2015 chl          0204                          4         32743522
## # … with 1,204 more rows, and 11 more variables: import_value_usd <dbl>,
## #   export_rca_4_digits_commodity_code <dbl>,
## #   import_rca_4_digits_commodity_code <dbl>,
## #   export_value_usd_change_1_year <dbl>,
## #   export_value_usd_change_5_years <dbl>,
## #   export_value_usd_percentage_change_1_year <dbl>,
## #   export_value_usd_percentage_change_5_years <dbl>,
## #   import_value_usd_change_1_year <dbl>,
## #   import_value_usd_change_5_years <dbl>,
## #   import_value_usd_percentage_change_1_year <dbl>,
## #   import_value_usd_percentage_change_5_years <dbl>
```

```r
## filter by reporter alias (also parameter `r`)
yrc_2 <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrc?y=2015&r=c-am&l=4"
))

yrc_2
```

```
## # A tibble: 47,807 x 16
##     year reporter_iso commodity_code commodity_code_… import_value_usd
##    <int> <chr>        <chr>                     <int>            <dbl>
##  1  2015 aia          0101                          4            12000
##  2  2015 aia          0104                          4             1481
##  3  2015 aia          0105                          4            44859
##  4  2015 aia          0106                          4              111
##  5  2015 aia          0201                          4           216873
##  6  2015 aia          0202                          4           514211
##  7  2015 aia          0203                          4           166264
##  8  2015 aia          0204                          4            57070
##  9  2015 aia          0206                          4            33427
## 10  2015 aia          0207                          4          1117503
## # … with 47,797 more rows, and 11 more variables:
## #   import_rca_4_digits_commodity_code <dbl>,
## #   import_value_usd_change_1_year <dbl>,
## #   import_value_usd_percentage_change_1_year <dbl>,
## #   import_value_usd_change_5_years <dbl>,
## #   import_value_usd_percentage_change_5_years <dbl>,
## #   export_value_usd <dbl>, export_rca_4_digits_commodity_code <dbl>,
## #   export_value_usd_change_1_year <dbl>,
## #   export_value_usd_change_5_years <dbl>,
## #   export_value_usd_percentage_change_1_year <dbl>,
## #   export_value_usd_percentage_change_5_years <dbl>
```

Here the `export_rca*` and `import_rca*` fields contain the Revealed Comparative Advantage (RCA) of an exported product with respect to all the products with the same number of digits. The definition of RCA is detailed on [Open Trade Statistics Documentation](https://tradestatistics.github.io/documentation/).

### YRP

**Year - Reporter - Partner**

The only applicable filter is by year, reporter and partner.


```r
# Year - Reporter - Partner (filter by year, reporter and partner)
yrp <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrp?y=2015&r=chl&p=arg"
))
```

### YC

**Year - Commodity**

The only applicable filter is by year and commodity.


```r
# Year - Commodity (filter by year)
yc <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yc?y=2015&c=0101"
))
```

Let's explore the first rows of `yr`:


```r
yc
```

```
## # A tibble: 1 x 21
##    year commodity_code commodity_code_… export_value_usd import_value_usd
##   <int> <chr>                     <int>            <dbl>            <dbl>
## 1  2015 0101                          4       3464828975       3464828975
## # … with 16 more variables: pci_4_digits_commodity_code <dbl>,
## #   pci_rank_4_digits_commodity_code <int>,
## #   pci_rank_4_digits_commodity_code_delta_1_year <int>,
## #   pci_rank_4_digits_commodity_code_delta_5_years <int>,
## #   top_exporter_iso <chr>, top_exporter_trade_value_usd <int>,
## #   top_importer_iso <chr>, top_importer_trade_value_usd <int>,
## #   export_value_usd_change_1_year <int>,
## #   export_value_usd_change_5_years <int>,
## #   export_value_usd_percentage_change_1_year <dbl>,
## #   export_value_usd_percentage_change_5_years <dbl>,
## #   import_value_usd_change_1_year <int>,
## #   import_value_usd_change_5_years <int>,
## #   import_value_usd_percentage_change_1_year <dbl>,
## #   import_value_usd_percentage_change_5_years <dbl>
```

Here some fields deserve an explanation:

* `pci_4_digits_commodity_code`: Product Complexity Index (PCI) which is detailed on [Open Trade Statistics Documentation](https://tradestatistics.github.io/documentation/). This index is built by using just four digits commodity codes.
* `pci_6_digits_commodity_code`: Similar to the previous field but built by using just six digits commodity codes.
* `pci_rank_4_digits_commodity_code`: The rank of a product given its PCI (e.g. the highest PCI obtains the #1)
* `pci_rank_4_digits_commodity_code_delta_1_year`: How many places a country increased or decreased with respect to last year

#### YR

**Year - Reporter**

The only applicable filter is by year and reporter.


```r
## Year - Reporter (filter by year and reporter)
yr <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yr?y=2015&r=chl"
))
```

Let's explore the first rows of `yr`:


```r
yr
```

```
## # A tibble: 1 x 20
##    year reporter_iso export_value_usd import_value_usd eci_4_digits_co…
##   <int> <chr>                   <dbl>            <dbl>            <dbl>
## 1  2015 chl               69696214027      73736894538           -0.221
## # … with 15 more variables: eci_rank_4_digits_commodity_code <int>,
## #   eci_rank_4_digits_commodity_code_delta_1_year <int>,
## #   eci_rank_4_digits_commodity_code_delta_5_years <int>,
## #   top_export_commodity_code <chr>, top_export_trade_value_usd <dbl>,
## #   top_import_commodity_code <chr>, top_import_trade_value_usd <dbl>,
## #   export_value_usd_change_1_year <dbl>,
## #   export_value_usd_change_5_years <dbl>,
## #   export_value_usd_percentage_change_1_year <dbl>,
## #   export_value_usd_percentage_change_5_years <dbl>,
## #   import_value_usd_change_1_year <dbl>,
## #   import_value_usd_change_5_years <dbl>,
## #   import_value_usd_percentage_change_1_year <dbl>,
## #   import_value_usd_percentage_change_5_years <dbl>
```

Some fields here require more detail:

* `eci_4_digits_commodity_code`: Economic Complexity Index (ECI) which is detailed on [Open Trade Statistics Documentation](https://tradestatistics.github.io/documentation/). This index is built by using just four digits commodity codes.
* `eci_rank_4_digits_commodity_code`: The rank of a country given its ECI (e.g. the highest ECI obtains the #1)
* `eci_rank_4_digits_commodity_code_delta_1_year`: How many places a country increased or decreased with respect to last year

### Country rankings

The only applicable filter is by year.


```r
# Country rankings (filter by year)
country_rankings <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/country_rankings?y=2015"
))
```

### Product rankings

The only applicable filter is by year.


```r
# Product rankings (filter by year)
product_rankings <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/product_rankings?y=2015"
))
```

