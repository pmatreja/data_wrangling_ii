strings\_and\_factors
================
Priyal
10/16/2018

Tidyverse automatically loads packages: ggplot2, tibble, tidyr, readr, purr, dplyr, stringr and forcats.

### Regex

``` r
string_vec1 = c("my", "name", "is", "jeff")
str_detect(string_vec1, "jeff")
```

    ## [1] FALSE FALSE FALSE  TRUE

``` r
str_replace(string_vec1, "jeff", "Jeff")
```

    ## [1] "my"   "name" "is"   "Jeff"

It is basically scanning through to look for "jeff". str\_detect is case sensitive.

str\_replace will scan to look for "jeff" and replace it with "Jeff"

``` r
string_vec2 = c(
  "i think we all rule for participating",
  "i think i have been caught",
  "i think this will be quite fun actually",
  "it will be fun, i think"
  )
str_detect(string_vec2, "i think")
```

    ## [1] TRUE TRUE TRUE TRUE

``` r
str_detect(string_vec2, "^i think")
```

    ## [1]  TRUE  TRUE  TRUE FALSE

``` r
str_detect(string_vec2, "i think$")
```

    ## [1] FALSE FALSE FALSE  TRUE

^ represents --&gt; begin with $ represents ---&gt; end with

``` r
string_vec3 = c(
  "Y'all remember Pres. HW Bush?",
  "I saw a green bush",
  "BBQ and Bushwalking at Molonglo Gorge",
  "BUSH!!"
  )

str_detect(string_vec3,"[Bb]ush")
```

    ## [1]  TRUE  TRUE  TRUE FALSE

Using the square bracket; multiple options for the match were given; B or b

``` r
string_vec4 = c(
  '7th inning stretch',
  '1st half soon to begin. Texas won the toss.',
  'she is 5 feet 4 inches tall',
  '3AM - cant sleep :('
  )

str_detect(string_vec4,"[0-9][a-zA-Z]")
```

    ## [1]  TRUE  TRUE FALSE  TRUE

Giving a range of values in the square bracket. str-detect now scans for numbers ranging from 0 to 9 followed by a-z or A-Z.

``` r
string_vec5 = c(
  'Its 7:11 in the evening',
  'want to go to 7-11?',
  'my flight is AA711',
  'NetBios: scanning ip 203.167.114.66'
  )

str_detect(string_vec5, "7.11")
```

    ## [1]  TRUE  TRUE FALSE  TRUE

. can match anything. Here the third example didn't match as there was nothing between 7 and 11.

Dealing with special characters

``` r
string_vec6 = c(
  'The CI is [2, 5]',
  ':-]',
  ':-[',
  'I found the answer on pages [6-7]'
  )

str_detect(string_vec6, "\\[")
```

    ## [1]  TRUE FALSE  TRUE  TRUE

If we want to search for \[ and \], ( and ), and . we have to indicate it is special with `\`, but `\` itself is special so we have to put `\\[`

### PULSE data

Tidying the Pulse dataset

``` r
pulse_data = haven::read_sas("./data/public_pulse_data.sas7bdat") %>%
  janitor::clean_names() %>%
  gather(key = visit, value = bdi, bdi_score_bl:bdi_score_12m) %>%
  mutate(visit = str_replace(visit, "bdi_score_", ""),
         visit = str_replace(visit, "bl", "00m"), 
         visit = fct_relevel(visit, str_c(c("00", "01", "06", "12"), "m"))) %>% 
  arrange(id, visit)
```

    ## Warning: attributes are not identical across measure variables;
    ## they will be dropped

### NSDUH data

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"
drug_use_xml = read_html(url)

table_marj = (drug_use_xml %>% html_nodes(css = "table")) %>% 
  .[[1]] %>%
  html_table() %>%
  slice(-1) %>%
  as_tibble()
```

Cleaning up the imported data

``` r
data_marj = 
  table_marj %>%
  select(-contains("P Value")) %>%
  gather(key = key, value = percent, -State) %>%
  separate(key, into = c("age", "year"), sep = "\\(") %>%
  mutate(year = str_replace(year, "\\)", ""),
         percent = str_replace(percent, "[a-c]$", ""),
         percent = as.numeric(percent)) %>%
  filter(!(State %in% c("Total U.S.", "Northeast", "Midwest", "South", "West")))
```

Making some plots with the tidy data

``` r
data_marj %>%
  filter(age == "12-17") %>% 
  mutate(State = fct_reorder(State, percent)) %>% 
  ggplot(aes(x = State, y = percent, color = year)) + 
    geom_point() + 
    theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

<img src="strings_and_factors_files/figure-markdown_github/unnamed-chunk-10-1.png" width="90%" />

Here states are reordered by the percent variable.

### Toothbrush reviews

Reading a collection of pages from the web

``` r
url_base = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber="

urls = str_c(url_base, 1:5)
```

### Factors...

Can get confusing..

Taking an example

``` r
vec_sex = factor(c("male", "male", "female", "female"))
as.numeric(vec_sex)
```

    ## [1] 2 2 1 1

R follows alphabetical order and thus denotes female as category 1.

``` r
vec_sex = relevel(vec_sex, ref = "male")
vec_sex
```

    ## [1] male   male   female female
    ## Levels: male female

``` r
as.numeric(vec_sex)
```

    ## [1] 1 1 2 2

### WEATHER DATA

``` r
weather_df = 
  rnoaa::meteo_pull_monitors(c("USW00094728", "USC00519397", "USS0023B17S"),
                      var = c("PRCP", "TMIN", "TMAX"), 
                      date_min = "2017-01-01",
                      date_max = "2017-12-31") %>%
  mutate(
    name = recode(id, USW00094728 = "CentralPark_NY", 
                      USC00519397 = "Waikiki_HA",
                      USS0023B17S = "Waterhole_WA"),
    tmin = tmin / 10,
    tmax = tmax / 10) %>%
  select(name, id, everything())
weather_df
```

    ## # A tibble: 1,095 x 6
    ##    name           id          date        prcp  tmax  tmin
    ##    <chr>          <chr>       <date>     <dbl> <dbl> <dbl>
    ##  1 CentralPark_NY USW00094728 2017-01-01     0   8.9   4.4
    ##  2 CentralPark_NY USW00094728 2017-01-02    53   5     2.8
    ##  3 CentralPark_NY USW00094728 2017-01-03   147   6.1   3.9
    ##  4 CentralPark_NY USW00094728 2017-01-04     0  11.1   1.1
    ##  5 CentralPark_NY USW00094728 2017-01-05     0   1.1  -2.7
    ##  6 CentralPark_NY USW00094728 2017-01-06    13   0.6  -3.8
    ##  7 CentralPark_NY USW00094728 2017-01-07    81  -3.2  -6.6
    ##  8 CentralPark_NY USW00094728 2017-01-08     0  -3.8  -8.8
    ##  9 CentralPark_NY USW00094728 2017-01-09     0  -4.9  -9.9
    ## 10 CentralPark_NY USW00094728 2017-01-10     0   7.8  -6  
    ## # ... with 1,085 more rows

Alphabetically ordered levels

``` r
weather_df %>% 
    mutate(name = factor(name)) %>% pull(name)
```

    ##    [1] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##    [5] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##    [9] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [13] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [17] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [21] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [25] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [29] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [33] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [37] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [41] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [45] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [49] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [53] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [57] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [61] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [65] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [69] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [73] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [77] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [81] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [85] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [89] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [93] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##   [97] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [101] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [105] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [109] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [113] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [117] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [121] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [125] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [129] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [133] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [137] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [141] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [145] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [149] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [153] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [157] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [161] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [165] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [169] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [173] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [177] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [181] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [185] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [189] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [193] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [197] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [201] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [205] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [209] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [213] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [217] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [221] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [225] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [229] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [233] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [237] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [241] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [245] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [249] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [253] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [257] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [261] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [265] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [269] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [273] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [277] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [281] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [285] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [289] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [293] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [297] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [301] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [305] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [309] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [313] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [317] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [321] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [325] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [329] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [333] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [337] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [341] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [345] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [349] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [353] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [357] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [361] CentralPark_NY CentralPark_NY CentralPark_NY CentralPark_NY
    ##  [365] CentralPark_NY Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [369] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [373] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [377] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [381] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [385] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [389] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [393] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [397] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [401] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [405] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [409] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [413] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [417] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [421] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [425] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [429] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [433] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [437] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [441] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [445] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [449] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [453] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [457] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [461] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [465] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [469] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [473] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [477] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [481] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [485] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [489] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [493] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [497] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [501] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [505] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [509] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [513] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [517] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [521] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [525] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [529] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [533] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [537] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [541] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [545] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [549] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [553] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [557] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [561] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [565] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [569] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [573] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [577] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [581] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [585] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [589] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [593] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [597] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [601] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [605] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [609] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [613] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [617] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [621] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [625] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [629] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [633] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [637] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [641] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [645] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [649] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [653] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [657] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [661] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [665] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [669] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [673] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [677] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [681] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [685] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [689] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [693] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [697] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [701] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [705] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [709] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [713] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [717] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [721] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [725] Waikiki_HA     Waikiki_HA     Waikiki_HA     Waikiki_HA    
    ##  [729] Waikiki_HA     Waikiki_HA     Waterhole_WA   Waterhole_WA  
    ##  [733] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [737] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [741] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [745] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [749] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [753] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [757] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [761] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [765] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [769] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [773] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [777] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [781] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [785] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [789] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [793] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [797] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [801] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [805] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [809] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [813] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [817] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [821] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [825] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [829] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [833] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [837] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [841] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [845] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [849] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [853] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [857] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [861] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [865] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [869] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [873] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [877] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [881] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [885] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [889] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [893] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [897] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [901] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [905] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [909] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [913] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [917] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [921] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [925] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [929] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [933] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [937] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [941] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [945] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [949] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [953] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [957] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [961] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [965] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [969] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [973] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [977] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [981] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [985] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [989] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [993] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ##  [997] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1001] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1005] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1009] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1013] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1017] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1021] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1025] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1029] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1033] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1037] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1041] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1045] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1049] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1053] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1057] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1061] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1065] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1069] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1073] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1077] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1081] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1085] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1089] Waterhole_WA   Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## [1093] Waterhole_WA   Waterhole_WA   Waterhole_WA  
    ## Levels: CentralPark_NY Waikiki_HA Waterhole_WA

Reordering the stations

``` r
weather_df %>%
  mutate(name = forcats::fct_relevel(name, c("Waikiki_HA", "CentralPark_NY", "Waterhole_WA"))) %>% 
  ggplot(aes(x = name, y = tmax)) + 
  geom_violin(aes(fill = name), color = "blue", alpha = .5) + 
  theme(legend.position = "bottom")
```

    ## Warning: Removed 3 rows containing non-finite values (stat_ydensity).

<img src="strings_and_factors_files/figure-markdown_github/unnamed-chunk-16-1.png" width="90%" />

Factor reorder

It is going to reorder the variable we care about as per the 2nd variable we input(median is the default).

``` r
weather_df %>%
  mutate(name = fct_reorder(name, tmax)) %>% 
  ggplot(aes(x = name, y = tmax)) + 
  geom_violin(aes(fill = name), color = "blue", alpha = .5) + 
  theme(legend.position = "bottom")
```

    ## Warning: Removed 3 rows containing non-finite values (stat_ydensity).

<img src="strings_and_factors_files/figure-markdown_github/unnamed-chunk-17-1.png" width="90%" />

Interpretating the models and how it depends on the factor levelling.

``` r
weather_df %>%
  mutate(name = forcats::fct_relevel(name, c("Waikiki_HA", "CentralPark_NY", "Waterhole_WA"))) %>% 
  lm(tmax ~ name, data = .)
```

    ## 
    ## Call:
    ## lm(formula = tmax ~ name, data = .)
    ## 
    ## Coefficients:
    ##        (Intercept)  nameCentralPark_NY    nameWaterhole_WA  
    ##              29.66              -12.29              -22.18

### NYC Restaurent Inspections

``` r
data(rest_inspec)

rest_inspec %>% 
  group_by(boro, grade) %>% 
  summarize(n = n()) %>% 
  spread(key = grade, value = n)
```

    ## # A tibble: 6 x 8
    ## # Groups:   boro [6]
    ##   boro              A     B     C `Not Yet Graded`     P     Z `<NA>`
    ##   <chr>         <int> <int> <int>            <int> <int> <int>  <int>
    ## 1 BRONX         13688  2801   701              200   163   351  16833
    ## 2 BROOKLYN      37449  6651  1684              702   416   977  51930
    ## 3 MANHATTAN     61608 10532  2689              765   508  1237  80615
    ## 4 Missing           4    NA    NA               NA    NA    NA     13
    ## 5 QUEENS        35952  6492  1593              604   331   913  45816
    ## 6 STATEN ISLAND  5215   933   207               85    47   149   6730

``` r
rest_inspec =
  rest_inspec %>%
  filter(grade %in% c("A", "B", "C"), boro != "Missing") %>% 
  mutate(boro = str_to_title(boro))
```

Pizza places

``` r
rest_inspec %>% 
  filter(str_detect(dba, "Pizza")) %>% 
  group_by(boro, grade) %>% 
  summarize(n = n()) %>% 
  spread(key = grade, value = n)
```

    ## # A tibble: 5 x 3
    ## # Groups:   boro [5]
    ##   boro              A     B
    ##   <chr>         <int> <int>
    ## 1 Bronx             9     3
    ## 2 Brooklyn          6    NA
    ## 3 Manhattan        26     8
    ## 4 Queens           17    NA
    ## 5 Staten Island     5    NA

Remember str\_detect is case sensitive!

``` r
rest_inspec %>% 
  filter(str_detect(dba, "[Pp][Ii][Zz][Zz][Aa]")) %>% 
  group_by(boro, grade) %>% 
  summarize(n = n()) %>% 
  spread(key = grade, value = n)
```

    ## # A tibble: 5 x 4
    ## # Groups:   boro [5]
    ##   boro              A     B     C
    ##   <chr>         <int> <int> <int>
    ## 1 Bronx          1170   305    56
    ## 2 Brooklyn       1948   296    61
    ## 3 Manhattan      1983   420    76
    ## 4 Queens         1647   259    48
    ## 5 Staten Island   323   127    21

Visualizing the data

``` r
library(viridis)
```

    ## Loading required package: viridisLite

``` r
rest_inspec %>% 
  filter(str_detect(dba, "[Pp][Ii][Zz][Zz][Aa]")) %>%
  ggplot(aes(x = boro, fill = grade)) + 
  geom_bar() + 
  scale_fill_viridis(discrete = TRUE)
```

<img src="strings_and_factors_files/figure-markdown_github/unnamed-chunk-23-1.png" width="90%" />

Reordering...

``` r
rest_inspec %>% 
   filter(str_detect(dba, "[Pp][Ii][Zz][Zz][Aa]")) %>%
  mutate(boro = fct_infreq(boro)) %>% 
  ggplot(aes(x = boro, fill = grade)) + 
  geom_bar() + 
  scale_fill_viridis(discrete = TRUE)
```

<img src="strings_and_factors_files/figure-markdown_github/unnamed-chunk-24-1.png" width="90%" />

Changing factor values

Because boro is a fcator variable replacing it isn't allowed.

Really change factor values--&gt; Use recode function

See stringer::str\_ functions
