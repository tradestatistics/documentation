# Accesing the data

## Before downloading datasets

If you are going to download data, you have to read the [Code of Conduct](https://docs.tradestatistics.io/index.html#code-of-conduct) first.

## API

The advantage of the API over https download is that you can filter what to obtain and also access some additional tables.

To obtain exactly the same data as with compressed files, please refer to \@ref(yrpc-year-reporter-partner-and-product-code).

If you use R you'll need `jsonlite` and `dplyr` packages.


```r
library(jsonlite)
```

These packages are also useful:


```r
library(dplyr)
library(stringr)
```

### Available tables


```r
as_tibble(fromJSON("https://api.tradestatistics.io/tables"))
```

```
## # A tibble: 21 x 3
##    table      description                        source                         
##    <chr>      <chr>                              <chr>                          
##  1 countries  Countries metadata                 UN Comtrade (with modification…
##  2 products   Product metadata                   UN Comtrade (with modification…
##  3 reporters  Reporting countries                UN Comtrade (with modification…
##  4 communiti… Product communities                Center for International Devel…
##  5 product_s… Product short names                The Observatory of Economic Co…
##  6 country_r… Ranking of countries               Open Trade Statistics          
##  7 product_r… Ranking of products                Open Trade Statistics          
##  8 yrpc       Reporter-Partner trade at product… Open Trade Statistics          
##  9 yrpc-ga    Reporter-Partner trade at group l… Open Trade Statistics          
## 10 yrpc-ca    Reporter-Partner trade at communi… Open Trade Statistics          
## # … with 11 more rows
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
## # A tibble: 1,320 x 4
##    product_code product_fullname_english              group_code group_name     
##    <chr>        <chr>                                 <chr>      <chr>          
##  1 0101         Horses, asses, mules and hinnies; li… 01         Animals; live  
##  2 0102         Bovine animals; live                  01         Animals; live  
##  3 0103         Swine; live                           01         Animals; live  
##  4 0104         Sheep and goats; live                 01         Animals; live  
##  5 0105         Poultry; live, fowls of the species … 01         Animals; live  
##  6 0106         Animals, n.e.c. in chapter 01; live   01         Animals; live  
##  7 0201         Meat of bovine animals; fresh or chi… 02         Meat and edibl…
##  8 0202         Meat of bovine animals; frozen        02         Meat and edibl…
##  9 0203         Meat of swine; fresh, chilled or fro… 02         Meat and edibl…
## 10 0204         Meat of sheep or goats; fresh, chill… 02         Meat and edibl…
## # … with 1,310 more rows
```

Please notice that these tables include some aliases. 

`countries` includes some meta-codes, `c-xx` where `xx` must the first two letters of a continent and `all`, this is:


Alias   Meaning                                       
------  ----------------------------------------------
c-af    Alias for all valid ISO codes in Africa       
c-am    Alias for all valid ISO codes in the Americas 
c-as    Alias for all valid ISO codes in Asia         
c-eu    Alias for all valid ISO codes in Europe       
c-oc    Alias for all valid ISO codes in Oceania      
all     Alias for all valid ISO codes in the World    

`products` also includes some meta-codes, `xx` for the first two digits of a code and those digits are the product group and `all`, this is:


Alias   Meaning                                                                                                                                                                                                                                                                    
------  ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
01      Alias for all codes in the group Animals; live                                                                                                                                                                                                                             
02      Alias for all codes in the group Meat and edible meat offal                                                                                                                                                                                                                
03      Alias for all codes in the group Fish and crustaceans, molluscs and other aquatic invertebrates                                                                                                                                                                            
04      Alias for all codes in the group Dairy produce; birds' eggs; natural honey; edible products of animal origin, not elsewhere specified or included                                                                                                                          
05      Alias for all codes in the group Animal originated products; not elsewhere specified or included                                                                                                                                                                           
06      Alias for all codes in the group Trees and other plants, live; bulbs, roots and the like; cut flowers and ornamental foliage                                                                                                                                               
07      Alias for all codes in the group Vegetables and certain roots and tubers; edible                                                                                                                                                                                           
08      Alias for all codes in the group Fruit and nuts, edible; peel of citrus fruit or melons                                                                                                                                                                                    
09      Alias for all codes in the group Coffee, tea, mate and spices                                                                                                                                                                                                              
10      Alias for all codes in the group Cereals                                                                                                                                                                                                                                   
11      Alias for all codes in the group Products of the milling industry; malt, starches, inulin, wheat gluten                                                                                                                                                                    
12      Alias for all codes in the group Oil seeds and oleaginous fruits; miscellaneous grains, seeds and fruit, industrial or medicinal plants; straw and fodder                                                                                                                  
13      Alias for all codes in the group Lac; gums, resins and other vegetable saps and extracts                                                                                                                                                                                   
14      Alias for all codes in the group Vegetable plaiting materials; vegetable products not elsewhere specified or included                                                                                                                                                      
15      Alias for all codes in the group Animal or vegetable fats and oils and their cleavage products; prepared animal fats; animal or vegetable waxes                                                                                                                            
16      Alias for all codes in the group Meat, fish or crustaceans, molluscs or other aquatic invertebrates; preparations thereof                                                                                                                                                  
17      Alias for all codes in the group Sugars and sugar confectionery                                                                                                                                                                                                            
18      Alias for all codes in the group Cocoa and cocoa preparations                                                                                                                                                                                                              
19      Alias for all codes in the group Preparations of cereals, flour, starch or milk; pastrycooks' products                                                                                                                                                                     
20      Alias for all codes in the group Preparations of vegetables, fruit, nuts or other parts of plants                                                                                                                                                                          
21      Alias for all codes in the group Miscellaneous edible preparations                                                                                                                                                                                                         
22      Alias for all codes in the group Beverages, spirits and vinegar                                                                                                                                                                                                            
23      Alias for all codes in the group Food industries, residues and wastes thereof; prepared animal fodder                                                                                                                                                                      
24      Alias for all codes in the group Tobacco and manufactured tobacco substitutes                                                                                                                                                                                              
25      Alias for all codes in the group Salt; sulphur; earths, stone; plastering materials, lime and cement                                                                                                                                                                       
26      Alias for all codes in the group Ores, slag and ash                                                                                                                                                                                                                        
27      Alias for all codes in the group Mineral fuels, mineral oils and products of their distillation; bituminous substances; mineral waxes                                                                                                                                      
28      Alias for all codes in the group Inorganic chemicals; organic and inorganic compounds of precious metals; of rare earth metals, of radio-active elements and of isotopes                                                                                                   
29      Alias for all codes in the group Organic chemicals                                                                                                                                                                                                                         
30      Alias for all codes in the group Pharmaceutical products                                                                                                                                                                                                                   
31      Alias for all codes in the group Fertilizers                                                                                                                                                                                                                               
32      Alias for all codes in the group Tanning or dyeing extracts; tannins and their derivatives; dyes, pigments and other colouring matter; paints, varnishes; putty, other mastics; inks                                                                                       
33      Alias for all codes in the group Essential oils and resinoids; perfumery, cosmetic or toilet preparations                                                                                                                                                                  
34      Alias for all codes in the group Soap, organic surface-active agents; washing, lubricating, polishing or scouring preparations; artificial or prepared waxes, candles and similar articles, modelling pastes, dental waxes and dental preparations with a basis of plaster 
35      Alias for all codes in the group Albuminoidal substances; modified starches; glues; enzymes                                                                                                                                                                                
36      Alias for all codes in the group Explosives; pyrotechnic products; matches; pyrophoric alloys; certain combustible preparations                                                                                                                                            
37      Alias for all codes in the group Photographic or cinematographic goods                                                                                                                                                                                                     
38      Alias for all codes in the group Chemical products n.e.c.                                                                                                                                                                                                                  
39      Alias for all codes in the group Plastics and articles thereof                                                                                                                                                                                                             
40      Alias for all codes in the group Rubber and articles thereof                                                                                                                                                                                                               
41      Alias for all codes in the group Raw hides and skins (other than furskins) and leather                                                                                                                                                                                     
42      Alias for all codes in the group Articles of leather; saddlery and harness; travel goods, handbags and similar containers; articles of animal gut (other than silk-worm gut)                                                                                               
43      Alias for all codes in the group Furskins and artificial fur; manufactures thereof                                                                                                                                                                                         
44      Alias for all codes in the group Wood and articles of wood; wood charcoal                                                                                                                                                                                                  
45      Alias for all codes in the group Cork and articles of cork                                                                                                                                                                                                                 
46      Alias for all codes in the group Manufactures of straw, esparto or other plaiting materials; basketware and wickerwork                                                                                                                                                     
47      Alias for all codes in the group Pulp of wood or other fibrous cellulosic material; recovered (waste and scrap) paper or paperboard                                                                                                                                        
48      Alias for all codes in the group Paper and paperboard; articles of paper pulp, of paper or paperboard                                                                                                                                                                      
49      Alias for all codes in the group Printed books, newspapers, pictures and other products of the printing industry; manuscripts, typescripts and plans                                                                                                                       
50      Alias for all codes in the group Silk                                                                                                                                                                                                                                      
51      Alias for all codes in the group Wool, fine or coarse animal hair; horsehair yarn and woven fabric                                                                                                                                                                         
52      Alias for all codes in the group Cotton                                                                                                                                                                                                                                    
53      Alias for all codes in the group Vegetable textile fibres; paper yarn and woven fabrics of paper yarn                                                                                                                                                                      
54      Alias for all codes in the group Man-made filaments; strip and the like of man-made textile materials                                                                                                                                                                      
55      Alias for all codes in the group Man-made staple fibres                                                                                                                                                                                                                    
56      Alias for all codes in the group Wadding, felt and nonwovens, special yarns; twine, cordage, ropes and cables and articles thereof                                                                                                                                         
57      Alias for all codes in the group Carpets and other textile floor coverings                                                                                                                                                                                                 
58      Alias for all codes in the group Fabrics; special woven fabrics, tufted textile fabrics, lace, tapestries, trimmings, embroidery                                                                                                                                           
59      Alias for all codes in the group Textile fabrics; impregnated, coated, covered or laminated; textile articles of a kind suitable for industrial use                                                                                                                        
60      Alias for all codes in the group Fabrics; knitted or crocheted                                                                                                                                                                                                             
61      Alias for all codes in the group Apparel and clothing accessories; knitted or crocheted                                                                                                                                                                                    
62      Alias for all codes in the group Apparel and clothing accessories; not knitted or crocheted                                                                                                                                                                                
63      Alias for all codes in the group Textiles, made up articles; sets; worn clothing and worn textile articles; rags                                                                                                                                                           
64      Alias for all codes in the group Footwear; gaiters and the like; parts of such articles                                                                                                                                                                                    
65      Alias for all codes in the group Headgear and parts thereof                                                                                                                                                                                                                
66      Alias for all codes in the group Umbrellas, sun umbrellas, walking-sticks, seat sticks, whips, riding crops; and parts thereof                                                                                                                                             
67      Alias for all codes in the group Feathers and down, prepared; and articles made of feather or of down; artificial flowers; articles of human hair                                                                                                                          
68      Alias for all codes in the group Stone, plaster, cement, asbestos, mica or similar materials; articles thereof                                                                                                                                                             
69      Alias for all codes in the group Ceramic products                                                                                                                                                                                                                          
70      Alias for all codes in the group Glass and glassware                                                                                                                                                                                                                       
71      Alias for all codes in the group Natural, cultured pearls; precious, semi-precious stones; precious metals, metals clad with precious metal, and articles thereof; imitation jewellery; coin                                                                               
72      Alias for all codes in the group Iron and steel                                                                                                                                                                                                                            
73      Alias for all codes in the group Iron or steel articles                                                                                                                                                                                                                    
74      Alias for all codes in the group Copper and articles thereof                                                                                                                                                                                                               
75      Alias for all codes in the group Nickel and articles thereof                                                                                                                                                                                                               
76      Alias for all codes in the group Aluminium and articles thereof                                                                                                                                                                                                            
78      Alias for all codes in the group Lead and articles thereof                                                                                                                                                                                                                 
79      Alias for all codes in the group Zinc and articles thereof                                                                                                                                                                                                                 
80      Alias for all codes in the group Tin; articles thereof                                                                                                                                                                                                                     
81      Alias for all codes in the group Metals; n.e.c., cermets and articles thereof                                                                                                                                                                                              
82      Alias for all codes in the group Tools, implements, cutlery, spoons and forks, of base metal; parts thereof, of base metal                                                                                                                                                 
83      Alias for all codes in the group Metal; miscellaneous products of base metal                                                                                                                                                                                               
84      Alias for all codes in the group Nuclear reactors, boilers, machinery and mechanical appliances; parts thereof                                                                                                                                                             
85      Alias for all codes in the group Electrical machinery and equipment and parts thereof; sound recorders and reproducers; television image and sound recorders and reproducers, parts and accessories of such articles                                                       
86      Alias for all codes in the group Railway, tramway locomotives, rolling-stock and parts thereof; railway or tramway track fixtures and fittings and parts thereof; mechanical (including electro-mechanical) traffic signalling equipment of all kinds                      
87      Alias for all codes in the group Vehicles; other than railway or tramway rolling stock, and parts and accessories thereof                                                                                                                                                  
88      Alias for all codes in the group Aircraft, spacecraft and parts thereof                                                                                                                                                                                                    
89      Alias for all codes in the group Ships, boats and floating structures                                                                                                                                                                                                      
90      Alias for all codes in the group Optical, photographic, cinematographic, measuring, checking, medical or surgical instruments and apparatus; parts and accessories                                                                                                         
91      Alias for all codes in the group Clocks and watches and parts thereof                                                                                                                                                                                                      
92      Alias for all codes in the group Musical instruments; parts and accessories of such articles                                                                                                                                                                               
93      Alias for all codes in the group Arms and ammunition; parts and accessories thereof                                                                                                                                                                                        
94      Alias for all codes in the group Furniture; bedding, mattresses, mattress supports, cushions and similar stuffed furnishings; lamps and lighting fittings, n.e.c.; illuminated signs, illuminated name-plates and the like; prefabricated buildings                        
95      Alias for all codes in the group Toys, games and sports requisites; parts and accessories thereof                                                                                                                                                                          
96      Alias for all codes in the group Miscellaneous manufactured articles                                                                                                                                                                                                       
97      Alias for all codes in the group Works of art; collectors' pieces and antiques                                                                                                                                                                                             
99      Alias for all codes in the group Commodities not specified according to kind                                                                                                                                                                                               
all     Alias for all codes                                                                                                                                                                                                                                                        

### API parameters

The tables provided withing our API contain at least one of these fields:

* Year (`y`) 
* Reporter ISO (`r`)
* Partner ISO (`p`)
* Product Code (`c`)

The most detailed table is `yrpc` that contains all bilateral flows at product level.

With respect to `y` you can pass any integer contained in $[1962,2018]$.

Both `r` and `p` accept any valid ISO code or alias contained in the [countries](https://api.tradestatistics.io/countries) table. For example, both `chl` (valid ISO code) and `c-am` (continent Americas, an alias) are valid API filtering parameters.

`c` takes any valid product code or alias from the [products](https://api.tradestatistics.io/products). For example, both `0101` (valid HS product code) and `01` (valid HS group code) are valid API filtering parameters.

By default the API takes `c = "all"` by default.

You can always skip `c`, but `y`, `r` and `p` are requiered to return data.

### Available reporters

The only applicable filter is by year.


```r
# Available reporters (filter by year)
as_tibble(fromJSON(
  "https://api.tradestatistics.io/reporters?y=2018"
))
```

```
## # A tibble: 226 x 1
##    reporter_iso
##    <chr>       
##  1 zwe         
##  2 zmb         
##  3 zaf         
##  4 yem         
##  5 wsm         
##  6 wlf         
##  7 vut         
##  8 vnm         
##  9 vgb         
## 10 ven         
## # … with 216 more rows
```

### YRPC (Year, Reporter, Partner and Product Code)

The applicable filters here are year, reporter, partner and product code.


```r
# Year - Reporter - Partner - Product Code

yrpc_1 <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrpc?y=1962&r=usa&p=mex&c=8703"
))

yrpc_1
```

```
## # A tibble: 1 x 6
##    year reporter_iso partner_iso product_code export_value_usd import_value_usd
##   <int> <chr>        <chr>       <chr>                   <int>            <int>
## 1  1962 usa          mex         8703                 72334238             7600
```

Columns definition:

* `reporter_iso`: Official ISO-3 code for the reporter (e.g. the country that reports X dollars in exports/imports from/to country Y)
* `partner_iso`: Official ISO-3 code for the partner
* `product_code`: Official Harmonized System rev. 2007 (HS07) product code (e.g. according to the \code{products} table in the API, 8703 stands for "Motor cars and other motor vehicles; principally designed for the transport of persons (other than those of heading no. 8702), including station wagons and racing cars")
* `export_value_usd`: Exports measured in nominal United States Dollars (USD)
* `import_value_usd`: Imports measured in nominal United States Dollars (USD)

### YRPC-GA (Year - Reporter - Partner - Product Code, Group Code Aggregated)

The applicable filters here are year, reporter, partner and group code. Here the group code is just an aggregation over product code.


```r
# Year - Reporter - Partner - Product Code, Group Code Aggregated

yrpc_ga <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrpc-ga?y=1962&r=usa&p=mex&g=87"
))

yrpc_ga
```

```
## # A tibble: 1 x 6
##    year reporter_iso partner_iso group_code export_value_usd import_value_usd
##   <int> <chr>        <chr>       <chr>                 <int>            <int>
## 1  1962 usa          mex         87                136983479           237587
```

Columns definition:

* `group_code`: Official Harmonized System rev. 2007 (HS07) group code (e.g. according to the \code{products} table in the API, 87 stands for "Vehicles; other than railway or tramway rolling stock, and parts and accessories thereof")

### YRPC-CA (Year - Reporter - Partner - Product Code, Community Code Aggregated)

The applicable filters here are year, reporter, partner and community code. Here the community code is just an aggregation over both product code and group code.


```r
# Year - Reporter - Partner - Product Code, Community Code Aggregated

yrpc_ca <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrpc-ca?y=1962&r=usa&p=mex&o=17"
))

yrpc_ca
```

```
## # A tibble: 1 x 6
##    year reporter_iso partner_iso community_code export_value_usd
##   <int> <chr>        <chr>       <chr>                     <int>
## 1  1962 usa          mex         17                    196732494
## # … with 1 more variable: import_value_usd <int>
```

Columns definition:

* `community_code`: Unofficial Harvard CID community code (e.g. according to the \code{communities} table in the API, 17 stands for "Transportation")

### YRPC-GCA (Year - Reporter - Partner - Product Code, Group Code and Community Code Aggregated)

The applicable filters here are year, reporter, partner and community code. Here the community code is just an aggregation over both product code and group code.


```r
# Year - Reporter - Partner - Product Code, Community Code Aggregated

yrpc_gca <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrpc-gca?y=1962&r=usa&p=mex&o=17"
))

yrpc_gca
```

```
## # A tibble: 4 x 7
##    year reporter_iso partner_iso group_code community_code export_value_usd
##   <int> <chr>        <chr>       <chr>      <chr>                     <int>
## 1  1962 usa          mex         86         17                     26210502
## 2  1962 usa          mex         87         17                    136983479
## 3  1962 usa          mex         88         17                     28529998
## 4  1962 usa          mex         89         17                      5008515
## # … with 1 more variable: import_value_usd <int>
```

### YRC (Year, Reporter and Product Code)

The only applicable filter is by year, reporter, product code and (optionally) product code length.


```r
# Year - Reporter - Product Code

yrc <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrc?y=1962&r=chl"
))

yrc
```

```
## # A tibble: 912 x 7
##     year reporter_iso product_code export_value_usd import_value_usd export_rca
##    <int> <chr>        <chr>                   <int>            <int>      <dbl>
##  1  1962 chl          0101                   289076            50833     0.534 
##  2  1962 chl          0102                    17685         22167651     0.0072
##  3  1962 chl          0103                        0            32399    NA     
##  4  1962 chl          0104                    12870           199519     0.062 
##  5  1962 chl          0105                        0           132064    NA     
##  6  1962 chl          0106                    20259            10548     0.210 
##  7  1962 chl          0201                    13075          3520194     0.0033
##  8  1962 chl          0203                        0           275764    NA     
##  9  1962 chl          0204                   387625            78528     0.349 
## 10  1962 chl          0206                    38106                0     0.0755
## # … with 902 more rows, and 1 more variable: import_rca <dbl>
```

Columns definition:

* `export_rca`:  Balassa Index or [Revealed Comparative Advantage](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#revealed-comparative-advantage-rca) of an exported product. 
* `import_rca`:  Balassa Index or [Revealed Comparative Advantage](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#revealed-comparative-advantage-rca) of and imported product. 

### YRP (Year, Reporter and Partner)

The only applicable filter is by year, reporter and partner.


```r
# Year - Reporter - Partner
yrp <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yrp?y=2018&r=chl&p=arg"
))

yrp
```

```
## # A tibble: 1 x 5
##    year reporter_iso partner_iso export_value_usd import_value_usd
##   <int> <chr>        <chr>                  <int>            <dbl>
## 1  2018 chl          arg                837640220       3768079208
```

### YC (Year and Product Code)

The only applicable filter is by year, product and (optionally) product code length.


```r
# Year - Product Code
yc <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yc?y=2018&c=0101"
))

yc
```

```
## # A tibble: 1 x 14
##    year product_code export_value_usd import_value_usd pci_fitness_met…
##   <int> <chr>                   <dbl>            <dbl>            <dbl>
## 1  2018 0101               4073362162       4073362162            0.293
## # … with 9 more variables: pci_rank_fitness_method <int>,
## #   pci_reflections_method <dbl>, pci_rank_reflections_method <int>,
## #   pci_eigenvalues_method <dbl>, pci_rank_eigenvalues_method <int>,
## #   top_exporter_iso <chr>, top_exporter_trade_value_usd <int>,
## #   top_importer_iso <chr>, top_importer_trade_value_usd <int>
```

Columns definition:

* `pci_fitness_method`: Product Complexity Index (PCI) computed by using the [Fitness Method](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#fitness-method).
* `pci_reflections_method`: Product Complexity Index (PCI) computed by using the [Reflections Method](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#reflections-method).
* `pci_eigenvalues_method`: Product Complexity Index (PCI) computed by using the [Eigenvalues Method](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#eigenvalues-method).
* `pci_rank_*_method`: The rank of a product given its PCI (e.g. the highest PCI obtains the #1)

### YR (Year and Reporter)

The only applicable filter is by year and reporter.


```r
## Year - Reporter
yr <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/yr?y=2018&r=chl"
))

yr
```

```
## # A tibble: 1 x 14
##    year reporter_iso export_value_usd import_value_usd cci_fitness_met…
##   <int> <chr>                   <dbl>            <dbl>            <dbl>
## 1  2018 chl               82917211883      83743442685            0.420
## # … with 9 more variables: cci_rank_fitness_method <int>,
## #   cci_reflections_method <dbl>, cci_rank_reflections_method <int>,
## #   cci_eigenvalues_method <dbl>, cci_rank_eigenvalues_method <int>,
## #   top_export_product_code <chr>, top_export_trade_value_usd <dbl>,
## #   top_import_product_code <chr>, top_import_trade_value_usd <dbl>
```

Columns definition:

* `cci_fitness_method`: Country Complexity Index (CCI) computed by using the [Fitness Method](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#fitness-method).
* `cci_reflections_method`: Country Complexity Index (CCI) computed by using the [Reflections Method](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#reflections-method).
* `cci_eigenvalues_method`: Country Complexity Index (CCI) computed by using the [Eigenvalues Method](https://docs.tradestatistics.io/the-mathematics-of-economic-complexity.html#eigenvalues-method).
* `cci_rank_*_method`: The rank of a product given its CCI (e.g. the highest CCI obtains the #1)

### Other group/community aggregated tables

As you might notice in [api.tradestatistics.io/tables](https://api.tradestatistics.io/tables), there are more tables:

* yrc-ga
* yrc-ca
* yrc-gca
* yr-short
* yr-ga
* yr-ca

These tables follow the same parameters as the examples above.

### Country rankings

The only applicable filter is by year.


```r
# Country rankings
country_rankings <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/country_rankings?y=2018"
))
```

### Product rankings

The only applicable filter is by year.


```r
# Product rankings
product_rankings <- as_tibble(fromJSON(
  "https://api.tradestatistics.io/product_rankings?y=2018"
))
```

## R Package

To ease API using, we provide an [R Package](https://ropensci.github.io/tradestatistics/). This package is a part of [ROpenSci](https://ropensci.org/) and its documentation is available on a separate [pkgdown site](https://ropensci.github.io/tradestatistics/).

Here's what the package does:

<div class="figure">
<img src="fig/data-diagram.svg" alt="R package flow"  />
<p class="caption">(\#fig:unnamed-chunk-10)R package flow</p>
</div>

## Dashboard (beta)

To ease API using, we provide a [Shiny Dashboard](https://shiny.tradestatistics.io/) that is still under improvements.

## RDS datasets

Please check the [md5sums](https://docs.tradestatistics.io/direct-download/md5sums.txt) to verify data integrity after downloading.


