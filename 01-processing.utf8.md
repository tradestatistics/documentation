# Data processing

## Tidy Data

We followed Tidy Data principles exposed in [@tidydata2014] and  [@r4ds2016]. Those principles are closely tied to those of relational databases and Codd’s relational algebra.

<div class="figure">
<img src="fig/data-science.svg" alt="Data pipeline"  />
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

## Data cleaning

You can check the GitHub repository, but here we provide a simplified and commented example that reproduces the exact steps we performed to clean the data.

Provided that Comtrade raw data cannot be redistributed, I'll limit the example to four countries in the year 1962 (the first year with available data).
    
Let's define two files for the example:


```r
raw_file <- "raw_data_1962_four_countries.rda"
clean_file <- "clean_data_1962_four_countries.rda"
```

### Required packages


```r
library(dplyr)
library(stringr)
library(purrr)
library(janitor)
```

### CIF-FOB rate, commodity codes length and ISO codes


```r
# CIF-FOB rate ------------------------------------------------------------

# See Anderson & van Wincoop, 2004, Hummels, 2006 and Gaulier & Zignago, 2010 about 8% rate consistency
cif_fob_rate <- 1.08

# Commodity codes length --------------------------------------------------

J <- 4

# ISO-3 codes -------------------------------------------------------------

load("../comtrade-codes/01-2-tidy-country-data/country-codes.RData")

country_codes <- country_codes %>%
  select(iso3_digit_alpha) %>%
  mutate(iso3_digit_alpha = str_to_lower(iso3_digit_alpha)) %>%
  filter(!iso3_digit_alpha %in% c("wld", "null")) %>%
  as_vector()
```

### Read raw data


```r
if (!file.exists(raw_file)) {
  raw_data <- readRDS("../yearly-datasets/01-raw-data/sitc-rev1/rds/type-C_r-ALL_ps-1962_freq-A_px-S1_pub-20050214_fmt-csv_ex-20151113.rds") %>%
    select("Year", "Aggregate Level", "Trade Flow", "Reporter ISO", "Partner ISO", "Commodity Code", "Trade Value (US$)") %>%
    filter(
      `Reporter ISO` == "CHL",
      `Partner ISO` %in% c("ARG", "BRA", "PER")
    )

  save(raw_data, file = raw_file, compress = "xz")

  raw_data
} else {
  load(raw_file)

  raw_data
}
```

```
##      Year Aggregate Level Trade Flow Reporter ISO Partner ISO Commodity Code
## 1    1962               3     Import          CHL         BRA            071
## 2    1962               3     Import          CHL         PER            071
## 3    1962               4     Export          CHL         PER           2218
## 4    1962               3     Export          CHL         ARG            265
## 5    1962               3     Import          CHL         BRA            265
## 6    1962               3     Import          CHL         PER            265
## 7    1962               5     Export          CHL         BRA          29193
## 8    1962               1     Export          CHL         ARG              3
## 9    1962               1     Import          CHL         BRA              3
## 10   1962               1     Import          CHL         PER              3
## 11   1962               3     Import          CHL         ARG            513
## 12   1962               3     Export          CHL         ARG            513
## 13   1962               3     Export          CHL         BRA            513
## 14   1962               3     Import          CHL         PER            513
## 15   1962               2     Import          CHL         ARG             53
## 16   1962               2     Import          CHL         BRA             53
## 17   1962               2     Import          CHL         PER             53
## 18   1962               3     Export          CHL         ARG            514
## 19   1962               3     Export          CHL         BRA            514
## 20   1962               4     Import          CHL         ARG           5417
## 21   1962               3     Import          CHL         ARG            641
## 22   1962               3     Export          CHL         ARG            641
## 23   1962               3     Export          CHL         BRA            641
## 24   1962               3     Import          CHL         PER            641
## 25   1962               3     Export          CHL         PER            641
## 26   1962               3     Import          CHL         BRA            676
## 27   1962               4     Export          CHL         PER           6822
## 28   1962               3     Import          CHL         ARG            693
## 29   1962               4     Import          CHL         ARG           7141
## 30   1962               4     Import          CHL         BRA           7141
## 31   1962               3     Import          CHL         ARG            717
## 32   1962               3     Import          CHL         BRA            717
## 33   1962               3     Import          CHL         PER            717
## 34   1962               3     Import          CHL         ARG            715
## 35   1962               3     Import          CHL         BRA            715
## 36   1962               4     Import          CHL         ARG           7173
## 37   1962               5     Import          CHL         BRA          71999
## 38   1962               5     Import          CHL         PER          71999
## 39   1962               3     Import          CHL         ARG            725
## 40   1962               3     Import          CHL         BRA            725
## 41   1962               3     Import          CHL         PER            725
## 42   1962               2     Import          CHL         PER             81
## 43   1962               2     Import          CHL         ARG             04
## 44   1962               2     Import          CHL         BRA             04
## 45   1962               2     Export          CHL         BRA             04
## 46   1962               2     Export          CHL         PER             04
## 47   1962               4     Import          CHL         ARG           0452
## 48   1962               3     Import          CHL         BRA            072
## 49   1962               3     Import          CHL         PER            072
## 50   1962               4     Import          CHL         ARG           0811
## 51   1962               4     Import          CHL         ARG           2119
## 52   1962               4     Import          CHL         PER           2119
## 53   1962               3     Import          CHL         ARG            221
## 54   1962               3     Import          CHL         BRA            221
## 55   1962               3     Export          CHL         PER            221
## 56   1962               3     Import          CHL         ARG            262
## 57   1962               3     Import          CHL         PER            262
## 58   1962               4     Import          CHL         ARG           2762
## 59   1962               4     Export          CHL         ARG           2762
## 60   1962               3     Export          CHL         ARG            521
## 61   1962               2     Import          CHL         ARG             58
## 62   1962               3     Import          CHL         ARG            599
## 63   1962               3     Import          CHL         BRA            599
## 64   1962               3     Import          CHL         PER            599
## 65   1962               3     Export          CHL         PER            599
## 66   1962               4     Import          CHL         ARG           6645
## 67   1962               4     Import          CHL         BRA           6645
## 68   1962               4     Import          CHL         PER           6842
## 69   1962               4     Import          CHL         ARG           7191
## 70   1962               4     Import          CHL         ARG           8619
## 71   1962               3     Import          CHL         ARG            862
## 72   1962               3     Import          CHL         PER            891
## 73   1962               3     Export          CHL         PER            891
## 74   1962               5     Import          CHL         PER          89441
## 75   1962               4     Import          CHL         ARG           0483
## 76   1962               2     Import          CHL         ARG             01
## 77   1962               2     Export          CHL         PER             01
## 78   1962               2     Import          CHL         ARG             02
## 79   1962               2     Import          CHL         PER             02
## 80   1962               4     Import          CHL         ARG           0460
## 81   1962               4     Import          CHL         ARG           0488
## 82   1962               4     Import          CHL         PER           0544
## 83   1962               2     Import          CHL         ARG             06
## 84   1962               2     Import          CHL         BRA             06
## 85   1962               2     Import          CHL         PER             06
## 86   1962               4     Import          CHL         BRA           0711
## 87   1962               4     Import          CHL         PER           0711
## 88   1962               4     Import          CHL         PER           0751
## 89   1962               3     Import          CHL         ARG            081
## 90   1962               3     Import          CHL         ARG            112
## 91   1962               3     Export          CHL         ARG            112
## 92   1962               3     Export          CHL         BRA            112
## 93   1962               3     Import          CHL         PER            112
## 94   1962               3     Export          CHL         PER            112
## 95   1962               2     Import          CHL         ARG             22
## 96   1962               2     Import          CHL         BRA             22
## 97   1962               2     Export          CHL         PER             22
## 98   1962               2     Export          CHL         ARG             24
## 99   1962               2     Import          CHL         PER             24
## 100  1962               2     Export          CHL         PER             24
## 101  1962               3     Export          CHL         ARG            243
## 102  1962               3     Import          CHL         PER            243
## 103  1962               3     Export          CHL         PER            243
## 104  1962               5     Export          CHL         ARG          24331
## 105  1962               5     Import          CHL         PER          24331
## 106  1962               5     Export          CHL         PER          24331
## 107  1962               2     Export          CHL         ARG             25
## 108  1962               2     Export          CHL         BRA             25
## 109  1962               2     Export          CHL         PER             25
## 110  1962               4     Import          CHL         ARG           2628
## 111  1962               4     Import          CHL         BRA           2654
## 112  1962               3     Import          CHL         ARG            274
## 113  1962               3     Export          CHL         PER            274
## 114  1962               4     Export          CHL         ARG           2813
## 115  1962               2     Import          CHL         ARG             29
## 116  1962               2     Export          CHL         ARG             29
## 117  1962               2     Export          CHL         BRA             29
## 118  1962               2     Import          CHL         BRA             33
## 119  1962               2     Import          CHL         PER             33
## 120  1962               3     Import          CHL         ARG            421
## 121  1962               3     Import          CHL         ARG            533
## 122  1962               3     Import          CHL         PER            533
## 123  1962               5     Import          CHL         PER          53335
## 124  1962               2     Import          CHL         ARG             54
## 125  1962               2     Export          CHL         ARG             54
## 126  1962               2     Import          CHL         BRA             54
## 127  1962               4     Import          CHL         ARG           5541
## 128  1962               5     Import          CHL         PER          61195
## 129  1962               2     Export          CHL         BRA             62
## 130  1962               4     Import          CHL         PER           6321
## 131  1962               2     Import          CHL         ARG             64
## 132  1962               2     Export          CHL         ARG             64
## 133  1962               2     Export          CHL         BRA             64
## 134  1962               3     Import          CHL         PER            655
## 135  1962               3     Export          CHL         PER            655
## 136  1962               3     Import          CHL         ARG            662
## 137  1962               3     Export          CHL         ARG            662
## 138  1962               3     Import          CHL         PER            662
## 139  1962               3     Export          CHL         PER            662
## 140  1962               4     Import          CHL         BRA           6631
## 141  1962               3     Import          CHL         ARG            663
## 142  1962               3     Import          CHL         BRA            663
## 143  1962               3     Import          CHL         PER            663
## 144  1962               2     Import          CHL         ARG             67
## 145  1962               2     Export          CHL         ARG             67
## 146  1962               2     Import          CHL         BRA             67
## 147  1962               2     Export          CHL         BRA             67
## 148  1962               2     Import          CHL         PER             67
## 149  1962               2     Export          CHL         PER             67
## 150  1962               2     Import          CHL         ARG             68
## 151  1962               2     Export          CHL         ARG             68
## 152  1962               2     Export          CHL         BRA             68
## 153  1962               4     Export          CHL         ARG           6821
## 154  1962               4     Export          CHL         BRA           6821
## 155  1962               3     Import          CHL         PER            684
## 156  1962               4     Import          CHL         PER           6851
## 157  1962               5     Import          CHL         PER          69221
## 158  1962               3     Import          CHL         ARG            698
## 159  1962               3     Import          CHL         PER            698
## 160  1962               3     Export          CHL         PER            698
## 161  1962               1     Import          CHL         ARG              7
## 162  1962               1     Export          CHL         ARG              7
## 163  1962               1     Import          CHL         BRA              7
## 164  1962               1     Export          CHL         BRA              7
## 165  1962               1     Import          CHL         PER              7
## 166  1962               1     Export          CHL         PER              7
## 167  1962               3     Import          CHL         ARG            711
## 168  1962               3     Export          CHL         ARG            711
## 169  1962               3     Export          CHL         BRA            711
## 170  1962               3     Import          CHL         ARG            712
## 171  1962               3     Export          CHL         ARG            712
## 172  1962               3     Import          CHL         PER            712
## 173  1962               3     Import          CHL         ARG            714
## 174  1962               3     Import          CHL         BRA            714
## 175  1962               3     Import          CHL         ARG            719
## 176  1962               3     Export          CHL         ARG            719
## 177  1962               3     Import          CHL         BRA            719
## 178  1962               3     Import          CHL         PER            719
## 179  1962               4     Import          CHL         ARG           7199
## 180  1962               4     Import          CHL         BRA           7199
## 181  1962               4     Import          CHL         PER           7199
## 182  1962               3     Import          CHL         PER            812
## 183  1962               2     Import          CHL         ARG             82
## 184  1962               2     Import          CHL         BRA             82
## 185  1962               2     Import          CHL         PER             82
## 186  1962               2     Import          CHL         ARG             84
## 187  1962               3     Import          CHL         PER            851
## 188  1962               2     Import          CHL         ARG             86
## 189  1962               2     Import          CHL         BRA             86
## 190  1962               2     Import          CHL         PER             86
## 191  1962               3     Import          CHL         ARG            861
## 192  1962               3     Import          CHL         BRA            861
## 193  1962               3     Import          CHL         PER            861
## 194  1962               4     Import          CHL         ARG           8641
## 195  1962               5     Import          CHL         ARG          89143
## 196  1962               3     Import          CHL         ARG            892
## 197  1962               3     Export          CHL         ARG            892
## 198  1962               3     Import          CHL         BRA            892
## 199  1962               3     Import          CHL         ARG            894
## 200  1962               3     Import          CHL         BRA            894
## 201  1962               3     Import          CHL         PER            894
## 202  1962               3     Import          CHL         ARG            895
## 203  1962               4     Export          CHL         PER           8960
## 204  1962               1     Import          CHL         BRA              9
## 205  1962               1     Export          CHL         BRA              9
## 206  1962               1     Import          CHL         PER              9
## 207  1962               1     Export          CHL         PER              9
## 208  1962               3     Import          CHL         ARG            931
## 209  1962               3     Export          CHL         ARG            931
## 210  1962               3     Import          CHL         BRA            931
## 211  1962               3     Export          CHL         BRA            931
## 212  1962               3     Import          CHL         PER            931
## 213  1962               3     Export          CHL         PER            931
## 214  1962               2     Import          CHL         ARG             94
## 215  1962               4     Import          CHL         ARG           9310
## 216  1962               4     Export          CHL         ARG           9310
## 217  1962               4     Import          CHL         BRA           9310
## 218  1962               4     Export          CHL         BRA           9310
## 219  1962               4     Import          CHL         PER           9310
## 220  1962               4     Export          CHL         PER           9310
## 221  1962               3     Import          CHL         ARG            941
## 222  1962               1     Import          CHL         ARG              0
## 223  1962               1     Export          CHL         ARG              0
## 224  1962               1     Import          CHL         BRA              0
## 225  1962               1     Export          CHL         BRA              0
## 226  1962               1     Import          CHL         PER              0
## 227  1962               1     Export          CHL         PER              0
## 228  1962               4     Import          CHL         PER           0014
## 229  1962               4     Export          CHL         ARG           0512
## 230  1962               4     Import          CHL         PER           0512
## 231  1962               4     Import          CHL         PER           0519
## 232  1962               4     Export          CHL         PER           0519
## 233  1962               3     Import          CHL         ARG            054
## 234  1962               3     Export          CHL         ARG            054
## 235  1962               3     Export          CHL         BRA            054
## 236  1962               3     Import          CHL         PER            054
## 237  1962               3     Export          CHL         PER            054
## 238  1962               3     Import          CHL         ARG            055
## 239  1962               3     Export          CHL         ARG            055
## 240  1962               3     Import          CHL         BRA            055
## 241  1962               3     Export          CHL         BRA            055
## 242  1962               3     Export          CHL         PER            055
## 243  1962               4     Export          CHL         PER           0555
## 244  1962               4     Export          CHL         ARG           2433
## 245  1962               4     Import          CHL         PER           2433
## 246  1962               4     Export          CHL         PER           2433
## 247  1962               3     Export          CHL         BRA            629
## 248  1962               3     Import          CHL         PER            629
## 249  1962               5     Import          CHL         PER          65321
## 250  1962               5     Export          CHL         ARG          66231
## 251  1962               5     Export          CHL         PER          66231
## 252  1962               4     Import          CHL         ARG           6635
## 253  1962               3     Import          CHL         ARG            682
## 254  1962               3     Export          CHL         ARG            682
## 255  1962               3     Export          CHL         BRA            682
## 256  1962               3     Export          CHL         PER            682
## 257  1962               5     Export          CHL         PER          68225
## 258  1962               2     Import          CHL         ARG             72
## 259  1962               2     Export          CHL         ARG             72
## 260  1962               2     Import          CHL         BRA             72
## 261  1962               2     Import          CHL         PER             72
## 262  1962               2     Export          CHL         PER             72
## 263  1962               5     Import          CHL         ARG          82103
## 264  1962               5     Import          CHL         BRA          82103
## 265  1962               5     Import          CHL         PER          82103
## 266  1962               4     Import          CHL         BRA           8617
## 267  1962               5     Import          CHL         ARG          86194
## 268  1962               3     Import          CHL         ARG            899
## 269  1962               3     Export          CHL         ARG            271
## 270  1962               3     Export          CHL         BRA            271
## 271  1962               3     Import          CHL         PER            271
## 272  1962               3     Export          CHL         PER            271
## 273  1962               4     Import          CHL         ARG           2741
## 274  1962               4     Export          CHL         PER           2741
## 275  1962               3     Import          CHL         ARG            012
## 276  1962               3     Export          CHL         ARG            032
## 277  1962               3     Import          CHL         ARG            044
## 278  1962               3     Import          CHL         BRA            044
## 279  1962               5     Import          CHL         ARG          04601
## 280  1962               4     Export          CHL         BRA           0520
## 281  1962               4     Export          CHL         PER           0520
## 282  1962               4     Import          CHL         ARG           0539
## 283  1962               4     Import          CHL         BRA           0539
## 284  1962               4     Export          CHL         PER           0539
## 285  1962               4     Import          CHL         ARG           0741
## 286  1962               4     Import          CHL         BRA           0741
## 287  1962               4     Import          CHL         PER           0741
## 288  1962               2     Import          CHL         PER             11
## 289  1962               2     Export          CHL         PER             11
## 290  1962               1     Import          CHL         PER              2
## 291  1962               1     Export          CHL         PER              2
## 292  1962               2     Import          CHL         ARG             26
## 293  1962               2     Export          CHL         ARG             26
## 294  1962               2     Import          CHL         BRA             26
## 295  1962               2     Import          CHL         PER             26
## 296  1962               4     Export          CHL         ARG           2929
## 297  1962               4     Import          CHL         BRA           3321
## 298  1962               4     Import          CHL         PER           3321
## 299  1962               4     Import          CHL         BRA           4314
## 300  1962               2     Import          CHL         ARG             51
## 301  1962               2     Export          CHL         ARG             51
## 302  1962               2     Import          CHL         BRA             51
## 303  1962               2     Export          CHL         BRA             51
## 304  1962               2     Import          CHL         PER             51
## 305  1962               2     Export          CHL         PER             51
## 306  1962               4     Import          CHL         PER           5333
## 307  1962               1     Import          CHL         PER              6
## 308  1962               1     Export          CHL         PER              6
## 309  1962               4     Import          CHL         PER           6210
## 310  1962               3     Import          CHL         PER            632
## 311  1962               3     Export          CHL         PER            632
## 312  1962               5     Import          CHL         PER          65403
## 313  1962               2     Import          CHL         BRA             66
## 314  1962               2     Import          CHL         PER             66
## 315  1962               2     Export          CHL         PER             66
## 316  1962               2     Import          CHL         PER             68
## 317  1962               2     Export          CHL         PER             68
## 318  1962               3     Import          CHL         PER            693
## 319  1962               4     Import          CHL         PER           7111
## 320  1962               4     Import          CHL         ARG           7122
## 321  1962               4     Export          CHL         ARG           7122
## 322  1962               4     Import          CHL         PER           7122
## 323  1962               4     Import          CHL         BRA           7231
## 324  1962               4     Export          CHL         PER           7231
## 325  1962               1     Import          CHL         ARG              8
## 326  1962               1     Export          CHL         ARG              8
## 327  1962               1     Import          CHL         BRA              8
## 328  1962               1     Import          CHL         PER              8
## 329  1962               1     Export          CHL         PER              8
## 330  1962               4     Export          CHL         ARG           8972
## 331  1962               4     Export          CHL         PER           8972
## 332  1962               2     Import          CHL         BRA             93
## 333  1962               2     Export          CHL         BRA             93
## 334  1962               2     Import          CHL         PER             93
## 335  1962               2     Export          CHL         PER             93
## 336  1962               0     Import          CHL         PER          TOTAL
## 337  1962               0     Export          CHL         PER          TOTAL
## 338  1962               4     Import          CHL         ARG           0819
## 339  1962               4     Import          CHL         ARG           0913
## 340  1962               3     Import          CHL         PER            686
## 341  1962               4     Import          CHL         PER           7121
## 342  1962               4     Import          CHL         ARG           0541
## 343  1962               4     Export          CHL         ARG           0541
## 344  1962               4     Import          CHL         PER           0541
## 345  1962               4     Export          CHL         PER           0541
## 346  1962               4     Export          CHL         ARG           0752
## 347  1962               4     Export          CHL         BRA           0752
## 348  1962               5     Import          CHL         ARG          09904
## 349  1962               2     Export          CHL         ARG             52
## 350  1962               5     Import          CHL         ARG          58192
## 351  1962               5     Import          CHL         PER          63272
## 352  1962               5     Export          CHL         PER          63272
## 353  1962               4     Import          CHL         PER           6731
## 354  1962               4     Export          CHL         PER           6731
## 355  1962               5     Import          CHL         PER          67351
## 356  1962               5     Export          CHL         ARG          68221
## 357  1962               5     Export          CHL         PER          68221
## 358  1962               5     Import          CHL         PER          86198
## 359  1962               4     Import          CHL         ARG           0111
## 360  1962               3     Import          CHL         ARG            042
## 361  1962               5     Import          CHL         PER          11102
## 362  1962               4     Import          CHL         BRA           2631
## 363  1962               4     Import          CHL         PER           2631
## 364  1962               5     Import          CHL         PER          33251
## 365  1962               2     Import          CHL         ARG             41
## 366  1962               2     Export          CHL         ARG             41
## 367  1962               2     Import          CHL         BRA             41
## 368  1962               5     Export          CHL         ARG          51284
## 369  1962               4     Import          CHL         PER           6514
## 370  1962               5     Import          CHL         BRA          66311
## 371  1962               5     Import          CHL         BRA          72323
## 372  1962               5     Import          CHL         PER          72323
## 373  1962               5     Import          CHL         ARG          72998
## 374  1962               5     Import          CHL         BRA          72998
## 375  1962               4     Import          CHL         PER           7353
## 376  1962               4     Export          CHL         PER           7353
## 377  1962               4     Import          CHL         ARG           0011
## 378  1962               4     Export          CHL         BRA           0011
## 379  1962               4     Import          CHL         ARG           0133
## 380  1962               4     Export          CHL         BRA           0482
## 381  1962               4     Import          CHL         PER           0513
## 382  1962               4     Export          CHL         PER           0515
## 383  1962               4     Import          CHL         ARG           0742
## 384  1962               4     Import          CHL         BRA           0742
## 385  1962               4     Import          CHL         ARG           2621
## 386  1962               4     Import          CHL         ARG           2911
## 387  1962               3     Export          CHL         ARG            321
## 388  1962               4     Export          CHL         ARG           3214
## 389  1962               4     Import          CHL         ARG           4215
## 390  1962               4     Import          CHL         ARG           5121
## 391  1962               4     Export          CHL         ARG           5121
## 392  1962               5     Import          CHL         PER          51313
## 393  1962               4     Import          CHL         ARG           5333
## 394  1962               3     Import          CHL         ARG            554
## 395  1962               5     Import          CHL         ARG          59955
## 396  1962               5     Import          CHL         PER          63273
## 397  1962               5     Import          CHL         ARG          64211
## 398  1962               5     Import          CHL         PER          65121
## 399  1962               4     Import          CHL         PER           6636
## 400  1962               4     Export          CHL         PER           6715
## 401  1962               5     Import          CHL         ARG          67701
## 402  1962               5     Import          CHL         PER          67701
## 403  1962               3     Import          CHL         PER            685
## 404  1962               5     Import          CHL         ARG          69603
## 405  1962               5     Import          CHL         ARG          69721
## 406  1962               4     Import          CHL         PER           6984
## 407  1962               5     Import          CHL         ARG          72502
## 408  1962               5     Import          CHL         BRA          72502
## 409  1962               5     Import          CHL         PER          72502
## 410  1962               5     Export          CHL         ARG          72993
## 411  1962               4     Export          CHL         ARG           7317
## 412  1962               4     Import          CHL         ARG           7321
## 413  1962               4     Export          CHL         ARG           7321
## 414  1962               4     Import          CHL         BRA           7321
## 415  1962               4     Export          CHL         BRA           7321
## 416  1962               5     Import          CHL         ARG          89111
## 417  1962               5     Import          CHL         BRA          89111
## 418  1962               4     Import          CHL         ARG           8914
## 419  1962               4     Export          CHL         ARG           8914
## 420  1962               5     Import          CHL         ARG          89523
## 421  1962               5     Import          CHL         BRA          89712
## 422  1962               5     Import          CHL         PER          89712
## 423  1962               4     Import          CHL         ARG           9510
## 424  1962               4     Import          CHL         ARG           0121
## 425  1962               3     Import          CHL         PER            025
## 426  1962               3     Import          CHL         ARG            041
## 427  1962               4     Export          CHL         BRA           0430
## 428  1962               4     Export          CHL         PER           0430
## 429  1962               5     Import          CHL         ARG          04882
## 430  1962               4     Import          CHL         ARG           0990
## 431  1962               3     Export          CHL         BRA            212
## 432  1962               4     Import          CHL         PER           2623
## 433  1962               4     Import          CHL         ARG           5324
## 434  1962               4     Import          CHL         BRA           5324
## 435  1962               4     Import          CHL         ARG           5997
## 436  1962               4     Import          CHL         BRA           5997
## 437  1962               5     Import          CHL         PER          62104
## 438  1962               4     Export          CHL         ARG           6515
## 439  1962               4     Export          CHL         BRA           6515
## 440  1962               4     Import          CHL         PER           6556
## 441  1962               4     Export          CHL         PER           6556
## 442  1962               5     Import          CHL         PER          66244
## 443  1962               4     Import          CHL         ARG           6665
## 444  1962               4     Import          CHL         BRA           6665
## 445  1962               4     Import          CHL         PER           6665
## 446  1962               4     Import          CHL         PER           6735
## 447  1962               4     Import          CHL         ARG           7194
## 448  1962               4     Import          CHL         BRA           7194
## 449  1962               4     Import          CHL         PER           7194
## 450  1962               4     Import          CHL         PER           8122
## 451  1962               4     Import          CHL         PER           2632
## 452  1962               3     Import          CHL         PER            514
## 453  1962               3     Export          CHL         PER            514
## 454  1962               5     Import          CHL         ARG          54163
## 455  1962               5     Export          CHL         ARG          54163
## 456  1962               5     Import          CHL         ARG          69311
## 457  1962               5     Import          CHL         PER          69311
## 458  1962               5     Import          CHL         ARG          69606
## 459  1962               4     Import          CHL         BRA           7173
## 460  1962               4     Import          CHL         PER           7173
## 461  1962               4     Import          CHL         PER           7193
## 462  1962               5     Import          CHL         BRA          72995
## 463  1962               4     Export          CHL         BRA           7317
## 464  1962               4     Import          CHL         PER           7321
## 465  1962               4     Export          CHL         PER           7321
## 466  1962               5     Import          CHL         PER          89111
## 467  1962               4     Import          CHL         PER           8930
## 468  1962               4     Export          CHL         PER           8930
## 469  1962               4     Import          CHL         ARG           9410
## 470  1962               2     Import          CHL         ARG             95
## 471  1962               3     Export          CHL         BRA            052
## 472  1962               3     Export          CHL         PER            052
## 473  1962               4     Import          CHL         PER           6618
## 474  1962               5     Import          CHL         PER          66392
## 475  1962               5     Import          CHL         PER          69894
## 476  1962               4     Export          CHL         BRA           2120
## 477  1962               4     Import          CHL         PER           0250
## 478  1962               4     Import          CHL         ARG           0459
## 479  1962               4     Export          CHL         PER           0459
## 480  1962               5     Export          CHL         BRA          05209
## 481  1962               5     Export          CHL         PER          05209
## 482  1962               3     Import          CHL         ARG            091
## 483  1962               5     Export          CHL         ARG          11213
## 484  1962               5     Export          CHL         BRA          11213
## 485  1962               5     Export          CHL         PER          24332
## 486  1962               5     Import          CHL         ARG          27662
## 487  1962               4     Import          CHL         ARG           4229
## 488  1962               5     Import          CHL         BRA          43143
## 489  1962               5     Import          CHL         ARG          51369
## 490  1962               4     Import          CHL         ARG           5819
## 491  1962               4     Import          CHL         PER           6324
## 492  1962               5     Import          CHL         PER          65561
## 493  1962               5     Export          CHL         PER          65561
## 494  1962               5     Import          CHL         PER          66245
## 495  1962               4     Import          CHL         BRA           6674
## 496  1962               3     Export          CHL         PER            671
## 497  1962               5     Import          CHL         PER          67311
## 498  1962               5     Export          CHL         PER          67311
## 499  1962               4     Export          CHL         ARG           6743
## 500  1962               4     Export          CHL         BRA           6743
## 501  1962               4     Import          CHL         PER           6743
## 502  1962               4     Export          CHL         PER           6743
## 503  1962               5     Export          CHL         ARG          67431
## 504  1962               5     Export          CHL         BRA          67431
## 505  1962               5     Import          CHL         PER          67431
## 506  1962               5     Export          CHL         PER          67431
## 507  1962               4     Export          CHL         ARG           6748
## 508  1962               4     Import          CHL         PER           6748
## 509  1962               4     Import          CHL         PER           6782
## 510  1962               3     Import          CHL         PER            681
## 511  1962               4     Import          CHL         ARG           6922
## 512  1962               4     Import          CHL         PER           6922
## 513  1962               4     Export          CHL         ARG           6923
## 514  1962               4     Import          CHL         BRA           6923
## 515  1962               4     Export          CHL         PER           6923
## 516  1962               5     Import          CHL         PER          69885
## 517  1962               5     Import          CHL         PER          69887
## 518  1962               3     Import          CHL         PER            723
## 519  1962               3     Export          CHL         PER            723
## 520  1962               4     Import          CHL         PER           7291
## 521  1962               5     Import          CHL         PER          72912
## 522  1962               4     Import          CHL         PER           7327
## 523  1962               5     Import          CHL         PER          85101
## 524  1962               4     Import          CHL         BRA           8618
## 525  1962               5     Import          CHL         BRA          86182
## 526  1962               5     Import          CHL         PER          89294
## 527  1962               5     Import          CHL         PER          89299
## 528  1962               5     Export          CHL         PER          89299
## 529  1962               5     Import          CHL         ARG          89601
## 530  1962               5     Import          CHL         BRA          89601
## 531  1962               5     Export          CHL         PER          89601
## 532  1962               3     Import          CHL         ARG            951
## 533  1962               3     Import          CHL         ARG            045
## 534  1962               3     Export          CHL         PER            045
## 535  1962               4     Import          CHL         ARG           0612
## 536  1962               4     Import          CHL         ARG           2765
## 537  1962               3     Export          CHL         ARG            281
## 538  1962               5     Import          CHL         ARG          53101
## 539  1962               5     Import          CHL         PER          66381
## 540  1962               4     Import          CHL         ARG           7333
## 541  1962               4     Import          CHL         PER           7333
## 542  1962               4     Import          CHL         ARG           8923
## 543  1962               4     Import          CHL         PER           8923
## 544  1962               5     Import          CHL         ARG          09906
## 545  1962               5     Import          CHL         ARG          01189
## 546  1962               3     Export          CHL         BRA            043
## 547  1962               3     Export          CHL         PER            043
## 548  1962               4     Import          CHL         ARG           0511
## 549  1962               4     Import          CHL         PER           0511
## 550  1962               4     Export          CHL         ARG           0551
## 551  1962               4     Export          CHL         PER           0551
## 552  1962               5     Import          CHL         ARG          05551
## 553  1962               5     Import          CHL         BRA          05551
## 554  1962               5     Export          CHL         PER          05551
## 555  1962               4     Import          CHL         ARG           0611
## 556  1962               4     Import          CHL         BRA           0611
## 557  1962               4     Import          CHL         PER           0611
## 558  1962               4     Import          CHL         PER           1110
## 559  1962               5     Export          CHL         PER          11213
## 560  1962               4     Import          CHL         ARG           2114
## 561  1962               4     Import          CHL         ARG           2211
## 562  1962               4     Import          CHL         BRA           2211
## 563  1962               4     Import          CHL         BRA           2658
## 564  1962               4     Import          CHL         PER           2658
## 565  1962               5     Import          CHL         ARG          29111
## 566  1962               4     Export          CHL         ARG           4111
## 567  1962               4     Import          CHL         ARG           4212
## 568  1962               4     Import          CHL         ARG           4216
## 569  1962               5     Export          CHL         ARG          51212
## 570  1962               4     Import          CHL         PER           5131
## 571  1962               4     Import          CHL         PER           5133
## 572  1962               5     Export          CHL         PER          51429
## 573  1962               5     Import          CHL         ARG          59951
## 574  1962               5     Import          CHL         BRA          59971
## 575  1962               4     Import          CHL         PER           6512
## 576  1962               5     Import          CHL         PER          65141
## 577  1962               4     Import          CHL         PER           6624
## 578  1962               4     Import          CHL         ARG           6641
## 579  1962               4     Import          CHL         BRA           6641
## 580  1962               5     Export          CHL         ARG          67321
## 581  1962               5     Import          CHL         PER          67321
## 582  1962               4     Import          CHL         PER           6747
## 583  1962               5     Export          CHL         ARG          67481
## 584  1962               5     Import          CHL         PER          67481
## 585  1962               5     Import          CHL         PER          68111
## 586  1962               5     Export          CHL         ARG          68212
## 587  1962               5     Export          CHL         BRA          68212
## 588  1962               5     Export          CHL         ARG          68222
## 589  1962               5     Export          CHL         PER          68222
## 590  1962               5     Import          CHL         ARG          68226
## 591  1962               5     Export          CHL         PER          68226
## 592  1962               5     Import          CHL         PER          68421
## 593  1962               5     Export          CHL         PER          69892
## 594  1962               5     Import          CHL         ARG          71942
## 595  1962               5     Import          CHL         BRA          71942
## 596  1962               5     Import          CHL         PER          71942
## 597  1962               5     Import          CHL         ARG          72321
## 598  1962               4     Import          CHL         ARG           7241
## 599  1962               4     Import          CHL         PER           7241
## 600  1962               4     Export          CHL         PER           7341
## 601  1962               5     Import          CHL         ARG          86243
## 602  1962               5     Import          CHL         ARG          86411
## 603  1962               5     Export          CHL         ARG          05199
## 604  1962               5     Import          CHL         PER          05199
## 605  1962               5     Export          CHL         ARG          65151
## 606  1962               5     Export          CHL         BRA          65151
## 607  1962               5     Import          CHL         ARG          66233
## 608  1962               4     Import          CHL         ARG           0440
## 609  1962               4     Import          CHL         BRA           0440
## 610  1962               5     Import          CHL         ARG          08119
## 611  1962               5     Import          CHL         PER          27693
## 612  1962               5     Import          CHL         ARG          41132
## 613  1962               5     Export          CHL         ARG          51225
## 614  1962               5     Import          CHL         ARG          51325
## 615  1962               5     Export          CHL         BRA          51325
## 616  1962               5     Import          CHL         PER          51332
## 617  1962               5     Import          CHL         PER          66362
## 618  1962               4     Import          CHL         PER           6861
## 619  1962               4     Import          CHL         PER           7325
## 620  1962               5     Import          CHL         ARG          59953
## 621  1962               5     Export          CHL         PER          59953
## 622  1962               5     Export          CHL         ARG          89141
## 623  1962               4     Export          CHL         ARG           2652
## 624  1962               5     Import          CHL         BRA          07232
## 625  1962               4     Import          CHL         ARG           2116
## 626  1962               5     Export          CHL         BRA          68211
## 627  1962               4     Import          CHL         ARG           8945
## 628  1962               5     Import          CHL         ARG          27695
## 629  1962               5     Export          CHL         BRA          68213
## 630  1962               5     Import          CHL         ARG          66413
## 631  1962               5     Import          CHL         BRA          66413
## 632  1962               5     Import          CHL         PER          05195
## 633  1962               5     Import          CHL         ARG          27652
## 634  1962               5     Import          CHL         ARG          89913
## 635  1962               5     Import          CHL         PER          66182
## 636  1962               3     Import          CHL         PER            687
## 637  1962               5     Import          CHL         PER          59957
## 638  1962               5     Import          CHL         ARG          95104
## 639  1962               4     Export          CHL         PER           2712
## 640  1962               4     Export          CHL         ARG           6411
## 641  1962               4     Export          CHL         BRA           6411
## 642  1962               4     Export          CHL         PER           6411
## 643  1962               4     Import          CHL         ARG           0015
## 644  1962               4     Export          CHL         ARG           0015
## 645  1962               4     Import          CHL         BRA           0015
## 646  1962               4     Import          CHL         PER           0015
## 647  1962               4     Export          CHL         PER           0015
## 648  1962               2     Export          CHL         ARG             03
## 649  1962               2     Export          CHL         BRA             03
## 650  1962               2     Import          CHL         PER             03
## 651  1962               2     Export          CHL         PER             03
## 652  1962               5     Import          CHL         PER          68521
## 653  1962               5     Import          CHL         PER          51226
## 654  1962               4     Import          CHL         ARG           0012
## 655  1962               4     Export          CHL         ARG           0012
## 656  1962               4     Import          CHL         ARG           0013
## 657  1962               4     Import          CHL         ARG           0112
## 658  1962               4     Export          CHL         PER           0112
## 659  1962               5     Export          CHL         PER          05171
## 660  1962               5     Export          CHL         PER          05192
## 661  1962               5     Export          CHL         BRA          05203
## 662  1962               5     Export          CHL         PER          05203
## 663  1962               4     Import          CHL         BRA           0723
## 664  1962               4     Import          CHL         PER           2711
## 665  1962               4     Import          CHL         ARG           2761
## 666  1962               4     Import          CHL         ARG           0422
## 667  1962               3     Import          CHL         ARG            046
## 668  1962               3     Import          CHL         ARG            061
## 669  1962               3     Import          CHL         BRA            061
## 670  1962               3     Import          CHL         PER            061
## 671  1962               3     Import          CHL         ARG            099
## 672  1962               5     Import          CHL         PER          11101
## 673  1962               3     Import          CHL         ARG            023
## 674  1962               3     Import          CHL         ARG            024
## 675  1962               4     Export          CHL         ARG           0311
## 676  1962               4     Export          CHL         ARG           0313
## 677  1962               4     Export          CHL         ARG           0517
## 678  1962               4     Import          CHL         BRA           0517
## 679  1962               4     Export          CHL         BRA           0517
## 680  1962               4     Export          CHL         PER           0517
## 681  1962               4     Import          CHL         ARG           0535
## 682  1962               4     Export          CHL         ARG           0545
## 683  1962               4     Export          CHL         BRA           0545
## 684  1962               4     Import          CHL         PER           0545
## 685  1962               4     Import          CHL         ARG           0555
## 686  1962               4     Import          CHL         BRA           0555
## 687  1962               4     Export          CHL         BRA           0555
## 688  1962               4     Import          CHL         ARG           0813
## 689  1962               4     Export          CHL         ARG           1121
## 690  1962               4     Import          CHL         ARG           2111
## 691  1962               4     Export          CHL         ARG           2432
## 692  1962               4     Import          CHL         PER           2432
## 693  1962               4     Export          CHL         PER           2432
## 694  1962               3     Export          CHL         ARG            251
## 695  1962               3     Export          CHL         BRA            251
## 696  1962               3     Export          CHL         PER            251
## 697  1962               4     Import          CHL         ARG           2769
## 698  1962               4     Import          CHL         PER           2769
## 699  1962               3     Import          CHL         ARG            411
## 700  1962               3     Export          CHL         ARG            411
## 701  1962               3     Import          CHL         BRA            411
## 702  1962               2     Import          CHL         ARG             42
## 703  1962               4     Import          CHL         ARG           5132
## 704  1962               4     Export          CHL         ARG           5132
## 705  1962               4     Export          CHL         BRA           5132
## 706  1962               4     Export          CHL         PER           5132
## 707  1962               4     Import          CHL         ARG           5310
## 708  1962               4     Import          CHL         ARG           5530
## 709  1962               4     Export          CHL         PER           5530
## 710  1962               3     Import          CHL         PER            611
## 711  1962               4     Import          CHL         PER           6119
## 712  1962               3     Import          CHL         PER            621
## 713  1962               4     Import          CHL         PER           6419
## 714  1962               3     Import          CHL         PER            654
## 715  1962               4     Import          CHL         PER           6540
## 716  1962               4     Import          CHL         ARG           6623
## 717  1962               4     Export          CHL         ARG           6623
## 718  1962               4     Import          CHL         PER           6639
## 719  1962               4     Import          CHL         PER           6651
## 720  1962               4     Import          CHL         ARG           6652
## 721  1962               4     Import          CHL         BRA           6652
## 722  1962               3     Import          CHL         BRA            667
## 723  1962               3     Export          CHL         ARG            674
## 724  1962               3     Export          CHL         BRA            674
## 725  1962               3     Import          CHL         PER            674
## 726  1962               3     Export          CHL         PER            674
## 727  1962               4     Import          CHL         PER           6811
## 728  1962               4     Import          CHL         PER           6852
## 729  1962               3     Import          CHL         PER            691
## 730  1962               4     Import          CHL         ARG           6972
## 731  1962               4     Import          CHL         ARG           8912
## 732  1962               4     Import          CHL         BRA           8912
## 733  1962               4     Export          CHL         PER           8912
## 734  1962               5     Export          CHL         ARG          24321
## 735  1962               5     Import          CHL         PER          24321
## 736  1962               5     Export          CHL         PER          24321
## 737  1962               4     Export          CHL         ARG           2512
## 738  1962               4     Export          CHL         BRA           2512
## 739  1962               4     Export          CHL         PER           2512
## 740  1962               4     Import          CHL         ARG           2622
## 741  1962               4     Import          CHL         PER           2622
## 742  1962               3     Import          CHL         ARG            022
## 743  1962               5     Import          CHL         PER          03201
## 744  1962               5     Export          CHL         ARG          03202
## 745  1962               5     Export          CHL         BRA          03202
## 746  1962               5     Export          CHL         PER          03202
## 747  1962               5     Export          CHL         ARG          05172
## 748  1962               5     Import          CHL         BRA          05172
## 749  1962               5     Export          CHL         BRA          05172
## 750  1962               5     Export          CHL         PER          05172
## 751  1962               3     Import          CHL         PER            242
## 752  1962               5     Import          CHL         ARG          41139
## 753  1962               5     Import          CHL         BRA          41139
## 754  1962               4     Import          CHL         ARG           4213
## 755  1962               5     Import          CHL         PER          51339
## 756  1962               4     Import          CHL         ARG           5136
## 757  1962               4     Export          CHL         ARG           5142
## 758  1962               4     Export          CHL         BRA           5142
## 759  1962               4     Import          CHL         PER           5142
## 760  1962               4     Export          CHL         PER           5142
## 761  1962               5     Import          CHL         PER          51424
## 762  1962               4     Import          CHL         ARG           6415
## 763  1962               5     Import          CHL         PER          65661
## 764  1962               5     Import          CHL         ARG          65691
## 765  1962               5     Import          CHL         PER          65691
## 766  1962               5     Import          CHL         PER          66183
## 767  1962               4     Export          CHL         ARG           6732
## 768  1962               4     Import          CHL         PER           6732
## 769  1962               5     Import          CHL         ARG          71931
## 770  1962               5     Import          CHL         BRA          71931
## 771  1962               5     Import          CHL         PER          71931
## 772  1962               5     Import          CHL         ARG          72491
## 773  1962               5     Import          CHL         BRA          72491
## 774  1962               5     Import          CHL         ARG          72492
## 775  1962               5     Import          CHL         BRA          72503
## 776  1962               4     Import          CHL         PER           7323
## 777  1962               5     Import          CHL         ARG          89211
## 778  1962               5     Import          CHL         BRA          89211
## 779  1962               5     Import          CHL         PER          89211
## 780  1962               4     Import          CHL         ARG           8952
## 781  1962               5     Import          CHL         ARG          05552
## 782  1962               5     Export          CHL         BRA          05552
## 783  1962               5     Export          CHL         PER          05552
## 784  1962               5     Import          CHL         ARG          06201
## 785  1962               5     Import          CHL         ARG          27623
## 786  1962               5     Export          CHL         ARG          29291
## 787  1962               5     Import          CHL         ARG          51213
## 788  1962               5     Import          CHL         BRA          51223
## 789  1962               4     Export          CHL         ARG           5128
## 790  1962               5     Export          CHL         ARG          51322
## 791  1962               5     Export          CHL         BRA          51322
## 792  1962               5     Export          CHL         PER          51322
## 793  1962               4     Import          CHL         ARG           5415
## 794  1962               4     Import          CHL         BRA           5415
## 795  1962               5     Import          CHL         ARG          59978
## 796  1962               5     Import          CHL         ARG          59999
## 797  1962               5     Import          CHL         PER          62105
## 798  1962               5     Import          CHL         PER          66511
## 799  1962               5     Import          CHL         PER          68422
## 800  1962               5     Import          CHL         ARG          69221
## 801  1962               5     Export          CHL         ARG          69231
## 802  1962               5     Import          CHL         BRA          69231
## 803  1962               5     Export          CHL         PER          69231
## 804  1962               5     Import          CHL         ARG          69523
## 805  1962               5     Import          CHL         BRA          69523
## 806  1962               5     Import          CHL         PER          69523
## 807  1962               5     Import          CHL         ARG          69524
## 808  1962               5     Export          CHL         ARG          69524
## 809  1962               5     Import          CHL         BRA          69524
## 810  1962               5     Import          CHL         PER          69524
## 811  1962               5     Export          CHL         PER          69524
## 812  1962               5     Import          CHL         BRA          69711
## 813  1962               4     Import          CHL         PER           6983
## 814  1962               4     Import          CHL         PER           6988
## 815  1962               4     Import          CHL         ARG           7142
## 816  1962               5     Import          CHL         ARG          71992
## 817  1962               5     Import          CHL         BRA          71992
## 818  1962               5     Import          CHL         PER          71994
## 819  1962               4     Import          CHL         ARG           7232
## 820  1962               4     Import          CHL         BRA           7232
## 821  1962               4     Import          CHL         PER           7232
## 822  1962               4     Import          CHL         ARG           7293
## 823  1962               4     Export          CHL         ARG           7293
## 824  1962               4     Import          CHL         PER           7293
## 825  1962               4     Export          CHL         PER           7293
## 826  1962               5     Import          CHL         ARG          72951
## 827  1962               5     Import          CHL         ARG          73289
## 828  1962               5     Import          CHL         BRA          73289
## 829  1962               5     Import          CHL         ARG          82109
## 830  1962               5     Import          CHL         BRA          82109
## 831  1962               5     Import          CHL         PER          82109
## 832  1962               5     Import          CHL         ARG          84112
## 833  1962               5     Import          CHL         BRA          86171
## 834  1962               5     Import          CHL         ARG          86197
## 835  1962               5     Import          CHL         ARG          86309
## 836  1962               5     Import          CHL         ARG          89299
## 837  1962               5     Import          CHL         ARG          89442
## 838  1962               5     Import          CHL         BRA          89442
## 839  1962               2     Import          CHL         ARG             00
## 840  1962               2     Export          CHL         ARG             00
## 841  1962               2     Import          CHL         BRA             00
## 842  1962               2     Export          CHL         BRA             00
## 843  1962               2     Import          CHL         PER             00
## 844  1962               2     Export          CHL         PER             00
## 845  1962               4     Import          CHL         ARG           0138
## 846  1962               4     Import          CHL         ARG           0240
## 847  1962               4     Export          CHL         ARG           0320
## 848  1962               4     Export          CHL         BRA           0320
## 849  1962               4     Import          CHL         PER           0320
## 850  1962               4     Export          CHL         PER           0320
## 851  1962               3     Import          CHL         ARG            051
## 852  1962               3     Export          CHL         ARG            051
## 853  1962               3     Import          CHL         BRA            051
## 854  1962               3     Export          CHL         BRA            051
## 855  1962               3     Import          CHL         PER            051
## 856  1962               3     Export          CHL         PER            051
## 857  1962               3     Import          CHL         ARG            053
## 858  1962               3     Import          CHL         BRA            053
## 859  1962               3     Export          CHL         PER            053
## 860  1962               3     Import          CHL         ARG            062
## 861  1962               2     Import          CHL         ARG             07
## 862  1962               2     Export          CHL         ARG             07
## 863  1962               2     Import          CHL         BRA             07
## 864  1962               2     Export          CHL         BRA             07
## 865  1962               2     Import          CHL         PER             07
## 866  1962               3     Export          CHL         ARG            075
## 867  1962               3     Import          CHL         BRA            075
## 868  1962               3     Export          CHL         BRA            075
## 869  1962               3     Import          CHL         PER            075
## 870  1962               2     Import          CHL         ARG             08
## 871  1962               1     Import          CHL         ARG              1
## 872  1962               1     Export          CHL         ARG              1
## 873  1962               1     Export          CHL         BRA              1
## 874  1962               1     Import          CHL         PER              1
## 875  1962               1     Export          CHL         PER              1
## 876  1962               4     Import          CHL         ARG           1124
## 877  1962               1     Import          CHL         ARG              2
## 878  1962               1     Export          CHL         ARG              2
## 879  1962               1     Import          CHL         BRA              2
## 880  1962               1     Export          CHL         BRA              2
## 881  1962               2     Import          CHL         ARG             21
## 882  1962               2     Export          CHL         BRA             21
## 883  1962               2     Import          CHL         PER             21
## 884  1962               3     Import          CHL         ARG            211
## 885  1962               3     Import          CHL         PER            211
## 886  1962               2     Import          CHL         ARG             27
## 887  1962               2     Export          CHL         ARG             27
## 888  1962               2     Export          CHL         BRA             27
## 889  1962               2     Import          CHL         PER             27
## 890  1962               2     Export          CHL         PER             27
## 891  1962               3     Import          CHL         ARG            291
## 892  1962               3     Export          CHL         BRA            291
## 893  1962               4     Export          CHL         BRA           2919
## 894  1962               3     Export          CHL         ARG            292
## 895  1962               3     Export          CHL         BRA            292
## 896  1962               3     Import          CHL         PER            292
## 897  1962               3     Export          CHL         PER            292
## 898  1962               4     Export          CHL         ARG           2925
## 899  1962               4     Import          CHL         PER           2925
## 900  1962               3     Import          CHL         ARG            512
## 901  1962               3     Export          CHL         ARG            512
## 902  1962               3     Import          CHL         BRA            512
## 903  1962               3     Import          CHL         PER            512
## 904  1962               3     Import          CHL         ARG            531
## 905  1962               3     Import          CHL         ARG            581
## 906  1962               2     Import          CHL         ARG             59
## 907  1962               2     Import          CHL         BRA             59
## 908  1962               2     Import          CHL         PER             59
## 909  1962               2     Export          CHL         PER             59
## 910  1962               2     Import          CHL         PER             61
## 911  1962               4     Export          CHL         BRA           6291
## 912  1962               4     Import          CHL         PER           6291
## 913  1962               2     Import          CHL         PER             63
## 914  1962               2     Export          CHL         PER             63
## 915  1962               2     Import          CHL         ARG             65
## 916  1962               2     Export          CHL         ARG             65
## 917  1962               2     Import          CHL         BRA             65
## 918  1962               2     Export          CHL         BRA             65
## 919  1962               2     Import          CHL         PER             65
## 920  1962               2     Export          CHL         PER             65
## 921  1962               4     Import          CHL         ARG           6576
## 922  1962               4     Import          CHL         BRA           6576
## 923  1962               4     Import          CHL         PER           6576
## 924  1962               3     Import          CHL         PER            661
## 925  1962               3     Export          CHL         PER            661
## 926  1962               4     Import          CHL         ARG           6612
## 927  1962               4     Import          CHL         PER           6612
## 928  1962               4     Export          CHL         PER           6612
## 929  1962               4     Import          CHL         PER           6638
## 930  1962               4     Import          CHL         ARG           6664
## 931  1962               4     Import          CHL         PER           6664
## 932  1962               4     Import          CHL         BRA           6761
## 933  1962               3     Import          CHL         BRA            678
## 934  1962               3     Import          CHL         PER            678
## 935  1962               2     Import          CHL         ARG             69
## 936  1962               2     Export          CHL         ARG             69
## 937  1962               2     Import          CHL         BRA             69
## 938  1962               4     Import          CHL         PER           6911
## 939  1962               3     Import          CHL         ARG            695
## 940  1962               3     Export          CHL         ARG            695
## 941  1962               3     Import          CHL         BRA            695
## 942  1962               3     Import          CHL         PER            695
## 943  1962               3     Export          CHL         PER            695
## 944  1962               4     Import          CHL         ARG           6952
## 945  1962               4     Export          CHL         ARG           6952
## 946  1962               4     Import          CHL         BRA           6952
## 947  1962               4     Import          CHL         PER           6952
## 948  1962               4     Export          CHL         PER           6952
## 949  1962               4     Import          CHL         ARG           6960
## 950  1962               3     Import          CHL         ARG            697
## 951  1962               3     Import          CHL         BRA            697
## 952  1962               4     Import          CHL         BRA           6971
## 953  1962               2     Import          CHL         ARG             71
## 954  1962               2     Export          CHL         ARG             71
## 955  1962               2     Import          CHL         BRA             71
## 956  1962               2     Export          CHL         BRA             71
## 957  1962               2     Import          CHL         PER             71
## 958  1962               4     Import          CHL         ARG           7115
## 959  1962               4     Export          CHL         ARG           7115
## 960  1962               4     Export          CHL         BRA           7115
## 961  1962               4     Import          CHL         PER           7115
## 962  1962               4     Import          CHL         ARG           7151
## 963  1962               4     Import          CHL         BRA           7151
## 964  1962               4     Import          CHL         ARG           7221
## 965  1962               4     Import          CHL         PER           7221
## 966  1962               4     Import          CHL         ARG           7249
## 967  1962               4     Import          CHL         BRA           7249
## 968  1962               4     Import          CHL         PER           7249
## 969  1962               3     Import          CHL         ARG            729
## 970  1962               3     Export          CHL         ARG            729
## 971  1962               3     Import          CHL         BRA            729
## 972  1962               3     Import          CHL         PER            729
## 973  1962               3     Export          CHL         PER            729
## 974  1962               2     Import          CHL         ARG             73
## 975  1962               2     Export          CHL         ARG             73
## 976  1962               2     Import          CHL         BRA             73
## 977  1962               2     Export          CHL         BRA             73
## 978  1962               2     Import          CHL         PER             73
## 979  1962               2     Export          CHL         PER             73
## 980  1962               4     Import          CHL         PER           8123
## 981  1962               4     Import          CHL         ARG           8210
## 982  1962               4     Import          CHL         BRA           8210
## 983  1962               4     Import          CHL         PER           8210
## 984  1962               4     Import          CHL         ARG           8411
## 985  1962               2     Import          CHL         PER             85
## 986  1962               4     Import          CHL         PER           8510
## 987  1962               3     Import          CHL         ARG            864
## 988  1962               2     Import          CHL         ARG             89
## 989  1962               2     Export          CHL         ARG             89
## 990  1962               2     Import          CHL         BRA             89
## 991  1962               2     Import          CHL         PER             89
## 992  1962               2     Export          CHL         PER             89
## 993  1962               4     Import          CHL         ARG           8911
## 994  1962               4     Import          CHL         BRA           8911
## 995  1962               4     Import          CHL         PER           8911
## 996  1962               4     Import          CHL         ARG           8921
## 997  1962               4     Import          CHL         BRA           8921
## 998  1962               4     Import          CHL         PER           8921
## 999  1962               4     Import          CHL         ARG           8929
## 1000 1962               4     Import          CHL         PER           8929
## 1001 1962               4     Export          CHL         PER           8929
## 1002 1962               4     Import          CHL         ARG           8944
## 1003 1962               4     Import          CHL         BRA           8944
## 1004 1962               4     Import          CHL         PER           8944
## 1005 1962               3     Import          CHL         ARG            896
## 1006 1962               3     Import          CHL         BRA            896
## 1007 1962               3     Export          CHL         PER            896
## 1008 1962               3     Export          CHL         ARG            897
## 1009 1962               3     Import          CHL         BRA            897
## 1010 1962               3     Import          CHL         PER            897
## 1011 1962               3     Export          CHL         PER            897
## 1012 1962               4     Import          CHL         BRA           8971
## 1013 1962               4     Import          CHL         PER           8971
## 1014 1962               0     Import          CHL         ARG          TOTAL
## 1015 1962               0     Export          CHL         ARG          TOTAL
## 1016 1962               0     Import          CHL         BRA          TOTAL
## 1017 1962               0     Export          CHL         BRA          TOTAL
## 1018 1962               4     Import          CHL         ARG           0134
## 1019 1962               3     Export          CHL         ARG            031
## 1020 1962               2     Import          CHL         ARG             05
## 1021 1962               2     Export          CHL         ARG             05
## 1022 1962               2     Import          CHL         BRA             05
## 1023 1962               2     Export          CHL         BRA             05
## 1024 1962               2     Import          CHL         PER             05
## 1025 1962               2     Export          CHL         PER             05
## 1026 1962               5     Export          CHL         ARG          05193
## 1027 1962               5     Export          CHL         PER          05193
## 1028 1962               3     Import          CHL         ARG            074
## 1029 1962               3     Import          CHL         BRA            074
## 1030 1962               3     Import          CHL         PER            074
## 1031 1962               3     Import          CHL         ARG            276
## 1032 1962               3     Export          CHL         ARG            276
## 1033 1962               3     Import          CHL         PER            276
## 1034 1962               5     Import          CHL         ARG          27651
## 1035 1962               1     Import          CHL         ARG              4
## 1036 1962               1     Export          CHL         ARG              4
## 1037 1962               1     Import          CHL         BRA              4
## 1038 1962               1     Import          CHL         ARG              5
## 1039 1962               1     Export          CHL         ARG              5
## 1040 1962               1     Import          CHL         BRA              5
## 1041 1962               1     Export          CHL         BRA              5
## 1042 1962               1     Import          CHL         PER              5
## 1043 1962               1     Export          CHL         PER              5
## 1044 1962               4     Import          CHL         ARG           5416
## 1045 1962               4     Export          CHL         ARG           5416
## 1046 1962               2     Import          CHL         ARG             55
## 1047 1962               2     Export          CHL         ARG             55
## 1048 1962               2     Export          CHL         PER             55
## 1049 1962               1     Import          CHL         ARG              6
## 1050 1962               1     Export          CHL         ARG              6
## 1051 1962               1     Import          CHL         BRA              6
## 1052 1962               1     Export          CHL         BRA              6
## 1053 1962               4     Import          CHL         PER           6327
## 1054 1962               4     Export          CHL         PER           6327
## 1055 1962               3     Import          CHL         ARG            665
## 1056 1962               3     Import          CHL         BRA            665
## 1057 1962               3     Import          CHL         PER            665
## 1058 1962               4     Import          CHL         PER           6942
## 1059 1962               4     Import          CHL         ARG           6989
## 1060 1962               4     Import          CHL         PER           6989
## 1061 1962               4     Export          CHL         PER           6989
## 1062 1962               4     Import          CHL         PER           7172
## 1063 1962               5     Import          CHL         ARG          71915
## 1064 1962               4     Import          CHL         ARG           7192
## 1065 1962               4     Import          CHL         PER           7192
## 1066 1962               4     Import          CHL         BRA           7197
## 1067 1962               4     Import          CHL         PER           7197
## 1068 1962               4     Import          CHL         ARG           7198
## 1069 1962               4     Export          CHL         ARG           7198
## 1070 1962               4     Import          CHL         BRA           7198
## 1071 1962               4     Import          CHL         PER           7198
## 1072 1962               4     Import          CHL         ARG           7222
## 1073 1962               4     Import          CHL         BRA           7222
## 1074 1962               4     Import          CHL         ARG           7250
## 1075 1962               4     Import          CHL         BRA           7250
## 1076 1962               4     Import          CHL         PER           7250
## 1077 1962               4     Import          CHL         ARG           7299
## 1078 1962               4     Export          CHL         ARG           7299
## 1079 1962               4     Import          CHL         BRA           7299
## 1080 1962               5     Import          CHL         ARG          27621
## 1081 1962               5     Export          CHL         ARG          27621
## 1082 1962               4     Import          CHL         ARG           2766
## 1083 1962               5     Import          CHL         ARG          27699
## 1084 1962               4     Export          CHL         ARG           2924
## 1085 1962               4     Export          CHL         BRA           2924
## 1086 1962               4     Import          CHL         PER           2924
## 1087 1962               4     Export          CHL         PER           2924
## 1088 1962               4     Import          CHL         ARG           4113
## 1089 1962               4     Import          CHL         BRA           4113
## 1090 1962               3     Import          CHL         ARG            422
## 1091 1962               5     Export          CHL         ARG          51425
## 1092 1962               5     Export          CHL         BRA          51425
## 1093 1962               4     Export          CHL         ARG           5214
## 1094 1962               4     Import          CHL         ARG           5323
## 1095 1962               3     Export          CHL         ARG            551
## 1096 1962               4     Export          CHL         ARG           5511
## 1097 1962               3     Import          CHL         ARG            553
## 1098 1962               3     Export          CHL         PER            553
## 1099 1962               3     Export          CHL         ARG            651
## 1100 1962               3     Export          CHL         BRA            651
## 1101 1962               3     Import          CHL         PER            651
## 1102 1962               3     Import          CHL         ARG            657
## 1103 1962               3     Import          CHL         BRA            657
## 1104 1962               3     Import          CHL         PER            657
## 1105 1962               2     Import          CHL         ARG             66
## 1106 1962               2     Export          CHL         ARG             66
## 1107 1962               3     Import          CHL         ARG            664
## 1108 1962               3     Import          CHL         BRA            664
## 1109 1962               3     Export          CHL         ARG            673
## 1110 1962               3     Import          CHL         PER            673
## 1111 1962               3     Export          CHL         PER            673
## 1112 1962               4     Import          CHL         PER           6871
## 1113 1962               2     Import          CHL         PER             69
## 1114 1962               2     Export          CHL         PER             69
## 1115 1962               3     Import          CHL         ARG            692
## 1116 1962               3     Export          CHL         ARG            692
## 1117 1962               3     Import          CHL         BRA            692
## 1118 1962               3     Import          CHL         PER            692
## 1119 1962               3     Export          CHL         PER            692
## 1120 1962               5     Import          CHL         ARG          69893
## 1121 1962               5     Import          CHL         ARG          69896
## 1122 1962               5     Import          CHL         ARG          71999
## 1123 1962               3     Import          CHL         ARG            722
## 1124 1962               3     Import          CHL         BRA            722
## 1125 1962               3     Import          CHL         PER            722
## 1126 1962               3     Import          CHL         ARG            724
## 1127 1962               3     Import          CHL         BRA            724
## 1128 1962               3     Import          CHL         PER            724
## 1129 1962               3     Import          CHL         ARG            732
## 1130 1962               3     Export          CHL         ARG            732
## 1131 1962               3     Import          CHL         BRA            732
## 1132 1962               3     Export          CHL         BRA            732
## 1133 1962               3     Import          CHL         PER            732
## 1134 1962               3     Export          CHL         PER            732
## 1135 1962               3     Import          CHL         ARG            733
## 1136 1962               3     Import          CHL         PER            733
## 1137 1962               3     Import          CHL         ARG            821
## 1138 1962               3     Import          CHL         BRA            821
## 1139 1962               3     Import          CHL         PER            821
## 1140 1962               3     Import          CHL         ARG            891
## 1141 1962               3     Export          CHL         ARG            891
## 1142 1962               3     Import          CHL         BRA            891
## 1143 1962               3     Import          CHL         ARG            893
## 1144 1962               3     Import          CHL         PER            893
## 1145 1962               3     Export          CHL         PER            893
## 1146 1962               3     Import          CHL         ARG            001
## 1147 1962               3     Export          CHL         ARG            001
## 1148 1962               3     Import          CHL         BRA            001
## 1149 1962               3     Export          CHL         BRA            001
## 1150 1962               3     Import          CHL         PER            001
## 1151 1962               3     Export          CHL         PER            001
## 1152 1962               3     Import          CHL         ARG            011
## 1153 1962               3     Export          CHL         PER            011
## 1154 1962               4     Import          CHL         ARG           0113
## 1155 1962               4     Import          CHL         ARG           0221
## 1156 1962               4     Import          CHL         ARG           0222
## 1157 1962               3     Export          CHL         BRA            032
## 1158 1962               3     Import          CHL         PER            032
## 1159 1962               3     Export          CHL         PER            032
## 1160 1962               4     Import          CHL         ARG           0410
## 1161 1962               4     Export          CHL         ARG           0542
## 1162 1962               4     Export          CHL         BRA           0542
## 1163 1962               4     Export          CHL         PER           0542
## 1164 1962               4     Export          CHL         BRA           1121
## 1165 1962               4     Export          CHL         PER           1121
## 1166 1962               4     Import          CHL         PER           1123
## 1167 1962               4     Import          CHL         PER           2429
## 1168 1962               3     Import          CHL         BRA            263
## 1169 1962               3     Import          CHL         PER            263
## 1170 1962               4     Export          CHL         ARG           2712
## 1171 1962               4     Export          CHL         BRA           2712
## 1172 1962               2     Export          CHL         ARG             28
## 1173 1962               3     Import          CHL         PER            332
## 1174 1962               4     Import          CHL         PER           3325
## 1175 1962               2     Import          CHL         BRA             43
## 1176 1962               3     Import          CHL         BRA            431
## 1177 1962               5     Import          CHL         PER          51311
## 1178 1962               5     Export          CHL         PER          51425
## 1179 1962               3     Import          CHL         ARG            532
## 1180 1962               3     Import          CHL         BRA            532
## 1181 1962               5     Import          CHL         ARG          53331
## 1182 1962               4     Import          CHL         ARG           5995
## 1183 1962               4     Import          CHL         PER           5995
## 1184 1962               4     Export          CHL         PER           5995
## 1185 1962               2     Import          CHL         PER             62
## 1186 1962               2     Import          CHL         PER             64
## 1187 1962               2     Export          CHL         PER             64
## 1188 1962               3     Import          CHL         ARG            656
## 1189 1962               3     Import          CHL         PER            656
## 1190 1962               4     Import          CHL         PER           6566
## 1191 1962               3     Import          CHL         ARG            666
## 1192 1962               3     Import          CHL         BRA            666
## 1193 1962               3     Import          CHL         PER            666
## 1194 1962               4     Import          CHL         ARG           6770
## 1195 1962               4     Import          CHL         PER           6770
## 1196 1962               4     Import          CHL         ARG           6822
## 1197 1962               4     Export          CHL         ARG           6822
## 1198 1962               3     Import          CHL         PER            694
## 1199 1962               4     Import          CHL         ARG           7193
## 1200 1962               4     Import          CHL         BRA           7193
## 1201 1962               4     Import          CHL         ARG           7231
## 1202 1962               4     Import          CHL         ARG           7242
## 1203 1962               4     Import          CHL         BRA           7242
## 1204 1962               4     Import          CHL         ARG           7328
## 1205 1962               4     Import          CHL         BRA           7328
## 1206 1962               4     Import          CHL         PER           7328
## 1207 1962               3     Import          CHL         PER            735
## 1208 1962               3     Export          CHL         PER            735
## 1209 1962               3     Import          CHL         ARG            841
## 1210 1962               5     Import          CHL         ARG          84111
## 1211 1962               4     Import          CHL         ARG           8624
## 1212 1962               3     Import          CHL         ARG            863
## 1213 1962               4     Import          CHL         ARG           8630
## 1214 1962               4     Import          CHL         ARG           8930
## 1215 1962               1     Import          CHL         ARG              9
## 1216 1962               1     Export          CHL         ARG              9
## 1217 1962               4     Export          CHL         PER           0111
## 1218 1962               4     Import          CHL         ARG           0118
## 1219 1962               3     Import          CHL         ARG            013
## 1220 1962               4     Import          CHL         ARG           0230
## 1221 1962               3     Import          CHL         ARG            048
## 1222 1962               3     Export          CHL         BRA            048
## 1223 1962               4     Export          CHL         PER           0514
## 1224 1962               4     Export          CHL         ARG           0519
## 1225 1962               4     Import          CHL         ARG           0620
## 1226 1962               4     Import          CHL         BRA           0721
## 1227 1962               4     Import          CHL         PER           0721
## 1228 1962               4     Import          CHL         BRA           0751
## 1229 1962               5     Export          CHL         ARG          07529
## 1230 1962               5     Export          CHL         BRA          07529
## 1231 1962               5     Import          CHL         ARG          08199
## 1232 1962               2     Import          CHL         ARG             09
## 1233 1962               2     Import          CHL         ARG             11
## 1234 1962               2     Export          CHL         ARG             11
## 1235 1962               2     Export          CHL         BRA             11
## 1236 1962               3     Import          CHL         PER            111
## 1237 1962               2     Import          CHL         PER             29
## 1238 1962               2     Export          CHL         PER             29
## 1239 1962               2     Export          CHL         ARG             32
## 1240 1962               3     Import          CHL         BRA            332
## 1241 1962               4     Export          CHL         ARG           5122
## 1242 1962               4     Import          CHL         BRA           5122
## 1243 1962               4     Import          CHL         PER           5122
## 1244 1962               3     Export          CHL         PER            513
## 1245 1962               5     Import          CHL         PER          53332
## 1246 1962               3     Import          CHL         ARG            541
## 1247 1962               3     Export          CHL         ARG            541
## 1248 1962               3     Import          CHL         BRA            541
## 1249 1962               4     Import          CHL         ARG           5999
## 1250 1962               3     Import          CHL         ARG            642
## 1251 1962               4     Import          CHL         ARG           6421
## 1252 1962               3     Import          CHL         PER            653
## 1253 1962               4     Import          CHL         PER           6532
## 1254 1962               4     Import          CHL         ARG           6569
## 1255 1962               4     Import          CHL         PER           6569
## 1256 1962               3     Import          CHL         ARG            661
## 1257 1962               4     Export          CHL         PER           6623
## 1258 1962               3     Import          CHL         ARG            677
## 1259 1962               3     Import          CHL         PER            677
## 1260 1962               4     Import          CHL         PER           6781
## 1261 1962               4     Import          CHL         BRA           6785
## 1262 1962               4     Import          CHL         ARG           6931
## 1263 1962               4     Import          CHL         PER           6931
## 1264 1962               5     Import          CHL         PER          69421
## 1265 1962               3     Import          CHL         ARG            696
## 1266 1962               3     Import          CHL         PER            711
## 1267 1962               5     Import          CHL         ARG          71921
## 1268 1962               5     Import          CHL         PER          71921
## 1269 1962               3     Import          CHL         ARG            723
## 1270 1962               3     Import          CHL         BRA            723
## 1271 1962               5     Import          CHL         PER          72491
## 1272 1962               4     Import          CHL         ARG           7295
## 1273 1962               3     Export          CHL         ARG            731
## 1274 1962               3     Export          CHL         BRA            731
## 1275 1962               5     Import          CHL         PER          73289
## 1276 1962               3     Export          CHL         PER            734
## 1277 1962               4     Import          CHL         PER           8619
## 1278 1962               3     Import          CHL         PER            892
## 1279 1962               3     Export          CHL         PER            892
## 1280 1962               4     Import          CHL         ARG           8922
## 1281 1962               4     Export          CHL         ARG           8922
## 1282 1962               4     Import          CHL         BRA           8922
## 1283 1962               4     Export          CHL         PER           8922
## 1284 1962               4     Import          CHL         ARG           8960
## 1285 1962               4     Import          CHL         BRA           8960
## 1286 1962               4     Import          CHL         ARG           8991
## 1287 1962               2     Import          CHL         ARG             93
## 1288 1962               2     Export          CHL         ARG             93
##      Trade Value (US$)
## 1              2652104
## 2                19205
## 3                 1575
## 4                 7676
## 5                35408
## 6                 2804
## 7                21675
## 8                 4275
## 9                96410
## 10             3364412
## 11                3108
## 12               42776
## 13               66651
## 14               49416
## 15              761818
## 16                5604
## 17                9509
## 18               48175
## 19              629575
## 20               10604
## 21                1104
## 22             1039575
## 23              603676
## 24                 504
## 25              573575
## 26                7604
## 27              154203
## 28               31904
## 29                 704
## 30                8104
## 31                2304
## 32              207304
## 33                5608
## 34                9004
## 35                6104
## 36                2304
## 37               35205
## 38               15504
## 39                 804
## 40                1508
## 41                 504
## 42                3408
## 43             3605635
## 44                6004
## 45              587452
## 46               39051
## 47              328405
## 48              234009
## 49               12004
## 50                 604
## 51               83304
## 52                 804
## 53                8905
## 54                3004
## 55                1575
## 56              986313
## 57               51809
## 58               62410
## 59                5575
## 60               83375
## 61                2004
## 62               52622
## 63                 704
## 64                1004
## 65               10676
## 66                1004
## 67                4405
## 68               11008
## 69                3604
## 70                1408
## 71                7705
## 72                 904
## 73                1676
## 74               12804
## 75                3604
## 76             4139035
## 77               24450
## 78              641817
## 79               12205
## 80               12804
## 81                1705
## 82               20205
## 83             1572012
## 84             1046804
## 85              956104
## 86             2652104
## 87               19205
## 88                2705
## 89               30012
## 90                4304
## 91                4075
## 92                9375
## 93               17505
## 94               45576
## 95                8905
## 96                3004
## 97                1575
## 98             2032150
## 99               13813
## 100              50427
## 101            2032150
## 102              11908
## 103              50427
## 104            1629775
## 105               1304
## 106              10176
## 107            2035476
## 108             766775
## 109             118175
## 110             327205
## 111              34904
## 112               4804
## 113              24276
## 114            1955476
## 115             116404
## 116              42427
## 117              30450
## 118              96410
## 119            3364412
## 120             137617
## 121               3205
## 122               9509
## 123               1304
## 124              37913
## 125               6575
## 126               1604
## 127              15804
## 128                704
## 129                675
## 130               3405
## 131               1708
## 132            1039575
## 133             603676
## 134              12804
## 135                675
## 136              16005
## 137              10476
## 138               2008
## 139               8575
## 140               2804
## 141              33104
## 142               2804
## 143               7112
## 144                604
## 145            1502026
## 146              12309
## 147              15575
## 148              54541
## 149              26328
## 150                704
## 151            2959426
## 152           14049626
## 153            2614875
## 154           14049626
## 155              11008
## 156              68505
## 157              11205
## 158               1208
## 159              21920
## 160               6375
## 161             633821
## 162             960828
## 163             357188
## 164              15326
## 165            1313416
## 166             115379
## 167               3004
## 168               3075
## 169               2476
## 170              11104
## 171                675
## 172               2308
## 173              13909
## 174               8104
## 175              92129
## 176               8075
## 177              96427
## 178             259830
## 179              15308
## 180              37009
## 181              18808
## 182               3408
## 183               4708
## 184               4708
## 185              78009
## 186               5208
## 187               1205
## 188              49322
## 189              12509
## 190               4104
## 191               1408
## 192              12509
## 193               4104
## 194                504
## 195               1905
## 196            2449319
## 197               9275
## 198              31609
## 199               3708
## 200                504
## 201              12804
## 202               1205
## 203                675
## 204               7405
## 205              35175
## 206              11705
## 207              27276
## 208              19205
## 209              74875
## 210               7405
## 211              35175
## 212              11705
## 213              27276
## 214               1205
## 215              19205
## 216              74875
## 217               7405
## 218              35175
## 219              11705
## 220              27276
## 221               1205
## 222           35929876
## 223            1245354
## 224            6371750
## 225            1240231
## 226            1328677
## 227            1506306
## 228               1604
## 229              34376
## 230               9504
## 231              74710
## 232              96951
## 233              31005
## 234             656226
## 235             177352
## 236              55913
## 237             148450
## 238              10909
## 239               1375
## 240               1104
## 241               7275
## 242              88426
## 243              73051
## 244            1629775
## 245               1304
## 246              10851
## 247                675
## 248              27205
## 249               5604
## 250              10476
## 251               8575
## 252              33104
## 253                704
## 254            2959426
## 255           14049626
## 256             154203
## 257              66175
## 258             477955
## 259              10251
## 260              25139
## 261              52730
## 262              38252
## 263                604
## 264                604
## 265              12705
## 266              10804
## 267                804
## 268                504
## 269             297075
## 270             916676
## 271             230205
## 272             277476
## 273               4804
## 274              24276
## 275               2604
## 276               1976
## 277              15504
## 278               6004
## 279              12804
## 280               2051
## 281              96150
## 282                804
## 283               2705
## 284             534676
## 285               3104
## 286              12304
## 287              12304
## 288              39415
## 289              45576
## 290           17170856
## 291             472804
## 292             986313
## 293               7676
## 294             117812
## 295           16838622
## 296               2976
## 297              96410
## 298            3274108
## 299              52404
## 300               7112
## 301             543177
## 302               6304
## 303             696226
## 304              91225
## 305             102426
## 306               9509
## 307             526737
## 308             793632
## 309               8108
## 310              11017
## 311               1476
## 312                504
## 313              17126
## 314              37146
## 315              11450
## 316             316631
## 317             154203
## 318                504
## 319               3504
## 320              11104
## 321                675
## 322                504
## 323               1905
## 324              31776
## 325            2532597
## 326              17725
## 327              53248
## 328             116859
## 329              42554
## 330               7775
## 331               4176
## 332               7405
## 333              35175
## 334              11705
## 335              27276
## 336           23973816
## 337            3117604
## 338              25904
## 339            1813405
## 340             110505
## 341               1804
## 342              31005
## 343             629575
## 344               6604
## 345               4275
## 346              18175
## 347              19175
## 348               1604
## 349              83375
## 350               2004
## 351                604
## 352               1476
## 353               1405
## 354               5176
## 355               3405
## 356             330575
## 357              26776
## 358               4104
## 359            3794205
## 360               5804
## 361              17705
## 362              82404
## 363           16747504
## 364              90304
## 365             308908
## 366              17475
## 367               2504
## 368               1275
## 369               1205
## 370               2804
## 371               1405
## 372                804
## 373               1004
## 374                704
## 375             603804
## 376              54776
## 377           23836804
## 378               2775
## 379              21205
## 380             305476
## 381             112005
## 382               6575
## 383              50205
## 384            2404604
## 385               1004
## 386             116404
## 387               4275
## 388               4275
## 389               6304
## 390               4004
## 391             449676
## 392                604
## 393               3205
## 394              15804
## 395              10804
## 396               6104
## 397                604
## 398               2905
## 399               5804
## 400               8676
## 401                604
## 402               1304
## 403              69910
## 404               2004
## 405               3604
## 406               2504
## 407                804
## 408                804
## 409                504
## 410               6775
## 411             903676
## 412               9804
## 413              35076
## 414              12705
## 415               3775
## 416               2405
## 417               1405
## 418               1905
## 419                675
## 420               1205
## 421                604
## 422                704
## 423               1705
## 424               2604
## 425              12205
## 426            3232104
## 427             281976
## 428              38376
## 429               1705
## 430               2208
## 431              55475
## 432              48904
## 433             757205
## 434               5604
## 435              13205
## 436                704
## 437               3004
## 438                975
## 439              77076
## 440              12804
## 441                675
## 442               1504
## 443               1604
## 444               1705
## 445                804
## 446               3405
## 447               2205
## 448               2104
## 449               1405
## 450                804
## 451              36505
## 452              13104
## 453             101551
## 454               6905
## 455               6575
## 456              31904
## 457                504
## 458               5504
## 459             207304
## 460               5004
## 461              49404
## 462               5405
## 463               9075
## 464               2004
## 465               2275
## 466                904
## 467               4104
## 468               1176
## 469               1205
## 470               1705
## 471               2051
## 472              96150
## 473               9909
## 474                604
## 475               7104
## 476              55475
## 477              12205
## 478               5705
## 479                675
## 480               1476
## 481              95475
## 482            1813405
## 483               4075
## 484               9375
## 485                675
## 486              63404
## 487              34505
## 488              52404
## 489               1004
## 490               2004
## 491                904
## 492              12804
## 493                675
## 494                504
## 495               1004
## 496               8676
## 497               1405
## 498               5176
## 499            1483176
## 500              15575
## 501               5405
## 502              12476
## 503            1483176
## 504              15575
## 505               5405
## 506              12476
## 507                875
## 508              22604
## 509               4905
## 510              58404
## 511               2205
## 512              11205
## 513               6476
## 514               1604
## 515              17675
## 516               9604
## 517                904
## 518                804
## 519              31776
## 520                604
## 521                604
## 522              70005
## 523               1205
## 524               1705
## 525               1705
## 526                604
## 527               2604
## 528               3075
## 529                604
## 530                504
## 531                675
## 532               1705
## 533             334110
## 534                675
## 535             188604
## 536               4610
## 537            1955476
## 538                604
## 539                704
## 540              12104
## 541              39005
## 542               2705
## 543                504
## 544                604
## 545              31705
## 546             281976
## 547              38376
## 548               8804
## 549              31705
## 550               1375
## 551              15375
## 552               2504
## 553               1104
## 554               1176
## 555            1382304
## 556            1046804
## 557             956104
## 558              21910
## 559              45576
## 560              10004
## 561               8905
## 562               3004
## 563                504
## 564               2804
## 565             116404
## 566              17475
## 567                704
## 568             125505
## 569             449676
## 570              30208
## 571              19208
## 572              10275
## 573                804
## 574                704
## 575               2905
## 576               1205
## 577               2008
## 578               7705
## 579               5604
## 580              17975
## 581                804
## 582               6804
## 583                875
## 584              22604
## 585              58404
## 586            2614875
## 587           13982975
## 588              13976
## 589              57276
## 590                704
## 591               3976
## 592               9504
## 593               6375
## 594               2205
## 595               2104
## 596               1405
## 597                804
## 598               1905
## 599                604
## 600              20076
## 601               7705
## 602                504
## 603               1575
## 604              20705
## 605                975
## 606              77076
## 607              16005
## 608              15504
## 609               6004
## 610                604
## 611              57104
## 612               5804
## 613               1275
## 614               2104
## 615              60376
## 616                804
## 617               5804
## 618             110505
## 619              21005
## 620              23404
## 621              10676
## 622                675
## 623               7676
## 624              45705
## 625               9104
## 626              61776
## 627               1104
## 628                604
## 629               4875
## 630               7705
## 631               5604
## 632              54005
## 633               1905
## 634                504
## 635                504
## 636              66804
## 637               1004
## 638               1705
## 639             277476
## 640            1039575
## 641             603676
## 642             573575
## 643              29804
## 644               1775
## 645               5304
## 646              16205
## 647              57375
## 648              12827
## 649                975
## 650              12504
## 651               2676
## 652               1405
## 653              28705
## 654             120505
## 655              11375
## 656              31205
## 657              81904
## 658              11375
## 659               3476
## 660              66375
## 661                575
## 662                675
## 663              45705
## 664             230205
## 665               1504
## 666               5804
## 667              12804
## 668            1570908
## 669            1046804
## 670             956104
## 671               2208
## 672               4205
## 673             615804
## 674               1205
## 675                875
## 676               9976
## 677             506275
## 678               1804
## 679             443176
## 680              35151
## 681               2604
## 682               1775
## 683              19076
## 684              29104
## 685              10909
## 686               1104
## 687               7275
## 688               3504
## 689               4075
## 690             705804
## 691             402375
## 692              10604
## 693              39576
## 694            2035476
## 695             766775
## 696             118175
## 697              86309
## 698              57104
## 699             308908
## 700              17475
## 701               2504
## 702             172122
## 703               2104
## 704              42776
## 705              66651
## 706                875
## 707                604
## 708               4205
## 709                975
## 710                704
## 711                704
## 712               8108
## 713                504
## 714                504
## 715                504
## 716              16005
## 717              10476
## 718                604
## 719              15604
## 720               2705
## 721               1604
## 722               1004
## 723            1484051
## 724              15575
## 725              34813
## 726              12476
## 727              58404
## 728               1405
## 729               3304
## 730               3604
## 731               3405
## 732               1405
## 733               1676
## 734             402375
## 735              10604
## 736              39576
## 737            2035476
## 738             766775
## 739             118175
## 740             658104
## 741               2905
## 742              24808
## 743              12504
## 744               1976
## 745                975
## 746               2676
## 747             506275
## 748               1804
## 749             443176
## 750              31675
## 751               1905
## 752             303104
## 753               2504
## 754               5104
## 755              18404
## 756               1004
## 757              48175
## 758             629575
## 759              13104
## 760             101551
## 761              13104
## 762               1104
## 763               3905
## 764               1405
## 765               1604
## 766               9405
## 767              17975
## 768                804
## 769                804
## 770              38404
## 771              49404
## 772             397304
## 773               6304
## 774               3205
## 775                704
## 776             208804
## 777            1342804
## 778               4705
## 779               7905
## 780               1205
## 781               8405
## 782               7275
## 783              71875
## 784               1104
## 785              14905
## 786               2976
## 787               4004
## 788               6304
## 789               1275
## 790              42776
## 791               6275
## 792                875
## 793              20404
## 794               1604
## 795              13205
## 796               4405
## 797               5104
## 798              15604
## 799               1504
## 800               2205
## 801               6476
## 802               1604
## 803              17675
## 804               3705
## 805                804
## 806                804
## 807              73705
## 808                875
## 809               5804
## 810               2705
## 811               1875
## 812                604
## 813               1804
## 814              10508
## 815              13205
## 816               1304
## 817               1804
## 818               3304
## 819                804
## 820               1405
## 821                804
## 822                704
## 823               3476
## 824               1004
## 825               6476
## 826               2804
## 827               2504
## 828               1405
## 829               4104
## 830               4104
## 831              65304
## 832               2604
## 833              10804
## 834                604
## 835              39705
## 836               1905
## 837               2604
## 838                504
## 839           24018316
## 840              13150
## 841               5304
## 842               2775
## 843              17809
## 844              57375
## 845               3104
## 846               1205
## 847               1976
## 848                975
## 849              12504
## 850               2676
## 851               8804
## 852             543601
## 853               1804
## 854             443176
## 855             227924
## 856             515052
## 857               3408
## 858               2705
## 859             534676
## 860               1104
## 861              53309
## 862              18175
## 863            5308025
## 864              19175
## 865              46218
## 866              18175
## 867               5004
## 868              19175
## 869               2705
## 870              30012
## 871               4304
## 872               4075
## 873               9375
## 874              39415
## 875              45576
## 876               4304
## 877            2142879
## 878            6375855
## 879             120816
## 880            1769376
## 881             808216
## 882              55475
## 883                804
## 884             808216
## 885                804
## 886             223041
## 887             302650
## 888             916676
## 889             287309
## 890             301752
## 891             116404
## 892              21675
## 893              21675
## 894              42427
## 895               8775
## 896              30308
## 897                875
## 898                675
## 899              27804
## 900               4004
## 901             452226
## 902               6304
## 903              28705
## 904                604
## 905               2004
## 906              52622
## 907                704
## 908               1004
## 909              10676
## 910                704
## 911                675
## 912              27205
## 913              11017
## 914               1476
## 915               3509
## 916                975
## 917               1104
## 918              77076
## 919              29435
## 920                675
## 921               2104
## 922               1104
## 923                904
## 924              11114
## 925               2875
## 926               1205
## 927               1205
## 928               2875
## 929                704
## 930               1304
## 931                504
## 932               7604
## 933               4705
## 934              12810
## 935             123839
## 936               7351
## 937               8816
## 938               3304
## 939              77410
## 940                875
## 941               6608
## 942               3509
## 943               1875
## 944              77410
## 945                875
## 946               6608
## 947               3509
## 948               1875
## 949               7508
## 950               3604
## 951                604
## 952                604
## 953             131454
## 954              11825
## 955             317939
## 956               2476
## 957             295255
## 958               3004
## 959               3075
## 960               2476
## 961              24005
## 962               9004
## 963               6104
## 964              13705
## 965              45505
## 966             400509
## 967               6304
## 968               3705
## 969               5116
## 970              10251
## 971               6109
## 972               1608
## 973               6476
## 974              24412
## 975             938752
## 976              14110
## 977              12850
## 978             965431
## 979              77127
## 980               2604
## 981               4708
## 982               4708
## 983              78009
## 984               5208
## 985               1205
## 986               1205
## 987                504
## 988            2473359
## 989              17725
## 990              36031
## 991              30133
## 992              42554
## 993               2405
## 994               1405
## 995                904
## 996            1342804
## 997               4705
## 998               7905
## 999               1905
## 1000              3208
## 1001              3075
## 1002              2604
## 1003               504
## 1004             12804
## 1005               604
## 1006               504
## 1007               675
## 1008              7775
## 1009               604
## 1010               704
## 1011              4176
## 1012               604
## 1013               704
## 1014          42823100
## 1015          14854293
## 1016           7115296
## 1017          18512336
## 1018               704
## 1019             10851
## 1020             54126
## 1021           1201202
## 1022              5613
## 1023            629854
## 1024            283837
## 1025           1382754
## 1026              1375
## 1027             30576
## 1028             53309
## 1029           2416908
## 1030             12304
## 1031            218237
## 1032              5575
## 1033             57104
## 1034              2705
## 1035            481030
## 1036             17475
## 1037             54908
## 1038            881478
## 1039            634002
## 1040             14216
## 1041            696226
## 1042            101738
## 1043            114077
## 1044              6905
## 1045              6575
## 1046             20009
## 1047               875
## 1048               975
## 1049            195000
## 1050           5519829
## 1051             39355
## 1052          14746628
## 1053              6708
## 1054              1476
## 1055              2705
## 1056              1604
## 1057             15604
## 1058              1004
## 1059              1208
## 1060              7104
## 1061              6375
## 1062               604
## 1063              3604
## 1064              1304
## 1065             32005
## 1066              2705
## 1067              1104
## 1068             68904
## 1069              8075
## 1070             16205
## 1071            157104
## 1072             19804
## 1073              7104
## 1074               804
## 1075              1508
## 1076               504
## 1077              1608
## 1078              6775
## 1079              6109
## 1080             47505
## 1081              5575
## 1082             63404
## 1083             85705
## 1084             38776
## 1085              8775
## 1086              2504
## 1087               875
## 1088            308908
## 1089              2504
## 1090             34505
## 1091             48175
## 1092            629575
## 1093             83375
## 1094               804
## 1095               875
## 1096               875
## 1097              4205
## 1098               975
## 1099               975
## 1100             77076
## 1101              4110
## 1102              2104
## 1103              1104
## 1104               904
## 1105             64636
## 1106             10476
## 1107              8709
## 1108             10009
## 1109             17975
## 1110              5614
## 1111              5176
## 1112             66804
## 1113             41446
## 1114             25925
## 1115              2205
## 1116              6476
## 1117              1604
## 1118             11205
## 1119             17675
## 1120               704
## 1121               504
## 1122             14004
## 1123             33509
## 1124              7104
## 1125             45505
## 1126            403918
## 1127              7108
## 1128              4309
## 1129             12308
## 1130             35076
## 1131             14110
## 1132              3775
## 1133            322622
## 1134              2275
## 1135             12104
## 1136             39005
## 1137              4708
## 1138              4708
## 1139             78009
## 1140              7715
## 1141               675
## 1142              2810
## 1143             10304
## 1144              4104
## 1145              1176
## 1146          24018316
## 1147             13150
## 1148              5304
## 1149              2775
## 1150             17809
## 1151             57375
## 1152           4111418
## 1153             24450
## 1154            203604
## 1155             19804
## 1156              5004
## 1157               975
## 1158             12504
## 1159              2676
## 1160           3232104
## 1161             24876
## 1162            158276
## 1163            144175
## 1164              9375
## 1165             45576
## 1166             17505
## 1167              1905
## 1168             82404
## 1169          16784008
## 1170            297075
## 1171            916676
## 1172           1955476
## 1173           3364412
## 1174             90304
## 1175             52404
## 1176             52404
## 1177             29604
## 1178             91276
## 1179            758009
## 1180              5604
## 1181              3205
## 1182             35012
## 1183              1004
## 1184             10676
## 1185             35313
## 1186               504
## 1187            573575
## 1188              1405
## 1189              5509
## 1190              3905
## 1191              2908
## 1192              1705
## 1193              1308
## 1194               604
## 1195              1304
## 1196               704
## 1197            344551
## 1198              1004
## 1199               804
## 1200             38404
## 1201             33804
## 1202              1504
## 1203               804
## 1204              2504
## 1205              1405
## 1206             20804
## 1207            603804
## 1208             54776
## 1209              5208
## 1210              2604
## 1211              7705
## 1212             39705
## 1213             39705
## 1214             10304
## 1215             22115
## 1216             74875
## 1217             13075
## 1218             31705
## 1219             25013
## 1220            615804
## 1221              5309
## 1222            305476
## 1223            376375
## 1224              2950
## 1225              1104
## 1226            188304
## 1227             12004
## 1228              5004
## 1229             18175
## 1230             19175
## 1231             25904
## 1232           1815613
## 1233              4304
## 1234              4075
## 1235              9375
## 1236             21910
## 1237             30308
## 1238               875
## 1239              4275
## 1240             96410
## 1241              1275
## 1242              6304
## 1243             28705
## 1244               875
## 1245              8205
## 1246             37913
## 1247              6575
## 1248              1604
## 1249              4405
## 1250               604
## 1251               604
## 1252              5604
## 1253              5604
## 1254              1405
## 1255              1604
## 1256              1205
## 1257              8575
## 1258               604
## 1259              1304
## 1260              7905
## 1261              4705
## 1262             31904
## 1263               504
## 1264              1004
## 1265              7508
## 1266             27509
## 1267              1304
## 1268             32005
## 1269             34608
## 1270              3310
## 1271              3705
## 1272              2804
## 1273            903676
## 1274              9075
## 1275             20804
## 1276             20076
## 1277              4104
## 1278             11617
## 1279             34851
## 1280           1101905
## 1281              9275
## 1282             26904
## 1283             31776
## 1284               604
## 1285               504
## 1286               504
## 1287             19205
## 1288             74875
```

### Clean data


```r
if (!file.exists(clean_file)) {
  clean_data <- raw_data %>%
    clean_names() %>%
    rename(
      product_code = commodity_code,
      trade_value_usd = trade_value_us
    ) %>%
    filter(aggregate_level %in% J) %>%
    filter(trade_flow %in% c("Export", "Import")) %>%
    filter(
      !is.na(product_code),
      product_code != "",
      product_code != " "
    ) %>%
    mutate(
      reporter_iso = str_to_lower(reporter_iso),
      partner_iso = str_to_lower(partner_iso),

      reporter_iso = ifelse(reporter_iso == "rou", "rom", reporter_iso),
      partner_iso = ifelse(partner_iso == "rou", "rom", partner_iso)
    ) %>%
    filter(
      reporter_iso %in% country_codes,
      partner_iso %in% country_codes
    ) %>%
    as_tibble()

  save(clean_data, file = clean_file, compress = "xz")

  clean_data
} else {
  load(clean_file)

  clean_data
}
```

```
## # A tibble: 468 x 7
##     year aggregate_level trade_flow reporter_iso partner_iso product_code
##    <int>           <int> <chr>      <chr>        <chr>       <chr>       
##  1  1962               4 Export     chl          per         2218        
##  2  1962               4 Import     chl          arg         5417        
##  3  1962               4 Export     chl          per         6822        
##  4  1962               4 Import     chl          arg         7141        
##  5  1962               4 Import     chl          bra         7141        
##  6  1962               4 Import     chl          arg         7173        
##  7  1962               4 Import     chl          arg         0452        
##  8  1962               4 Import     chl          arg         0811        
##  9  1962               4 Import     chl          arg         2119        
## 10  1962               4 Import     chl          per         2119        
## # … with 458 more rows, and 1 more variable: trade_value_usd <dbl>
```

### Symmetric (clean) data


```r
# Exports data ------------------------------------------------------------
exports <- clean_data %>%
  filter(trade_flow == "Export") %>%
  select(reporter_iso, partner_iso, product_code, trade_value_usd) %>%
  mutate(trade_value_usd = ceiling(trade_value_usd))

exports_mirrored <- clean_data %>%
  filter(trade_flow == "Import") %>%
  select(reporter_iso, partner_iso, product_code, trade_value_usd) %>%
  mutate(trade_value_usd = ceiling(trade_value_usd / cif_fob_rate))

# Reporter and Partner must be inverted
colnames(exports_mirrored) <- c("partner_iso", "reporter_iso", "product_code", "trade_value_usd")

exports_conciliated <- exports %>%
  full_join(exports_mirrored, by = c("reporter_iso", "partner_iso", "product_code")) %>%
  rowwise() %>%
  mutate(trade_value_usd = max(trade_value_usd.x, trade_value_usd.y, na.rm = T)) %>%
  ungroup() %>%
  mutate(product_code_parent = str_sub(product_code, 1, 4)) %>%
  group_by(reporter_iso, partner_iso, product_code_parent) %>%
  mutate(parent_count = n()) %>%
  ungroup() %>%
  select(reporter_iso, partner_iso, product_code, product_code_parent, parent_count, trade_value_usd)

exports_conciliated
```

```
## # A tibble: 468 x 6
##    reporter_iso partner_iso product_code product_code_pa… parent_count
##    <chr>        <chr>       <chr>        <chr>                   <int>
##  1 chl          per         2218         2218                        1
##  2 chl          per         6822         6822                        1
##  3 chl          arg         2762         2762                        1
##  4 chl          arg         2813         2813                        1
##  5 chl          arg         6821         6821                        1
##  6 chl          bra         6821         6821                        1
##  7 chl          per         8960         8960                        1
##  8 chl          arg         9310         9310                        1
##  9 chl          bra         9310         9310                        1
## 10 chl          per         9310         9310                        1
## # … with 458 more rows, and 1 more variable: trade_value_usd <dbl>
```

## GitHub repositories

* [Getting and cleaning data from UN COMTRADE (OTS Yearly Data)](https://github.com/tradestatistics/yearly-datasets)
* [Scraping data from The Atlas of Economic Complexity (OTS Atlas Data)](https://github.com/tradestatistics/atlas-data)
* [Product and country codes (OTS Comtrade Codes)](https://github.com/tradestatistics/comtrade-codes)
* [R packages library for reproducibility (OTS Packrat Library)](https://github.com/tradestatistics/packrat-library/)

## Software information

We used R 3.4.3 and RStudio Desktop 1.1 on Ubuntu Desktop 18.04.

We built R from binaries in order to obtain a setup linked with multi-threaded numeric libraries. Our build is linked to OpenBLAS which we used over alternatives such as Intel MKL, BLAS or ATLAS that can also be used.

## Hardware information

All the data processing was done by using a DigitalOcean droplet with 6 vCPUs and 32GB in RAM to accelerate data processing.

The functions were executed using parallelization on four cores. Please notice that running our scripts with parallelization demands more RAM than the amount you can find on an average laptop. You can always disable parallelization in the scripts or reduce the default number of cores.

## Reproducibility notes

To guarantee reproducibility we provide [Packrat](https://rstudio.github.io/packrat/) snapshot and bundles. This prevents changes in syntax, functions or dependencies.

Our R installation is isolated from apt-get to avoid any accidental updates that can alter the data pipeline and/or the output.

The projects are related to each other. In order to avoid multiple copies of files some projects read files from other projects. For example, [OTS Yearly Datasets](https://github.com/tradestatistics/yearly-datasets) uses [OTS Atlas Data](https://github.com/tradestatistics/atlas-data) as inpu.

The only reproducibility flaw of this project lies in data downloading. Obtaining raw datasets from UN COMTRADE demands an API key that can only be obtained with institutional access that is limited to some universities and institutes.

## Coding style and performant code

We used the [Tidyverse Style Guide](http://style.tidyverse.org/). As cornerstone references for performant code we followed [@advancedr2014] and [@masteringsoftware2017].

## Note for Windows users

If you use Windows the scripts will only use a single core because we used a parallelization that depends on fork system call that is only supported on Unix systems. You can always run the scripts on Windows and the only difference will be that it will use less RAM and processor, and it will be slower to compute.

Also, before running the scripts on Windows verify that you installed GNU Utilities beforehand. One easy option is to install [Chocolatey](https://chocolatey.org/) first and then install the GNU Utilities by running `choco install unxutils` on Cmd or Power Shell as administrator.
