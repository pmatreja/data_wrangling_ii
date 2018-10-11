data\_wrangling\_ii
================
Priyal
10/11/2018

Scrape a table
--------------

``` r
url = "http://samhda.s3-us-gov-west-1.amazonaws.com/s3fs-public/field-uploads/2k15StateFiles/NSDUHsaeShortTermCHG2015.htm"

drug_use_xml = read_html(url)
```

Get the tables from the html

All the objects labelled as table will be extracted.

``` r
drug_use_xml %>%
  html_nodes(css = "table")
```

    ## {xml_nodeset (15)}
    ##  [1] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [2] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [3] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [4] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [5] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [6] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [7] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [8] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ##  [9] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [10] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [11] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [12] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [13] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [14] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...
    ## [15] <table class="rti" border="1" cellspacing="0" cellpadding="1" width ...

We have extracted 15 tables.

Extracting 1st table using square brackets.

``` r
drug_use_xml %>%
  html_nodes(css = "table") %>%
  .[[1]] %>% 
  html_table() %>%
  slice(-1) %>% 
  as_tibble()
```

    ## # A tibble: 56 x 16
    ##    State `12+(2013-2014)` `12+(2014-2015)` `12+(P Value)` `12-17(2013-201…
    ##    <chr> <chr>            <chr>            <chr>          <chr>           
    ##  1 Tota… 12.90a           13.36            0.002          13.28b          
    ##  2 Nort… 13.88a           14.66            0.005          13.98           
    ##  3 Midw… 12.40b           12.76            0.082          12.45           
    ##  4 South 11.24a           11.64            0.029          12.02           
    ##  5 West  15.27            15.62            0.262          15.53a          
    ##  6 Alab… 9.98             9.60             0.426          9.90            
    ##  7 Alas… 19.60a           21.92            0.010          17.30           
    ##  8 Ariz… 13.69            13.12            0.364          15.12           
    ##  9 Arka… 11.37            11.59            0.678          12.79           
    ## 10 Cali… 14.49            15.25            0.103          15.03           
    ## # ... with 46 more rows, and 11 more variables: `12-17(2014-2015)` <chr>,
    ## #   `12-17(P Value)` <chr>, `18-25(2013-2014)` <chr>,
    ## #   `18-25(2014-2015)` <chr>, `18-25(P Value)` <chr>,
    ## #   `26+(2013-2014)` <chr>, `26+(2014-2015)` <chr>, `26+(P Value)` <chr>,
    ## #   `18+(2013-2014)` <chr>, `18+(2014-2015)` <chr>, `18+(P Value)` <chr>

Slice was used to remove the first row.

Learning Assessment
-------------------

``` r
url = "https://www.bestplaces.net/cost_of_living/city/new_york/new_york"

cost_of_living_ny = read_html(url)
```

``` r
 cost_of_living_ny %>%
  html_nodes(css = "table") %>% 
  .[[1]] %>% 
  html_table(header = TRUE) 
```

    ##     COST OF LIVING New York New York      USA
    ## 1          Overall      180      122      100
    ## 2          Grocery      125    111.6      100
    ## 3           Health      110      109      100
    ## 4          Housing      313      145      100
    ## 5 Median Home Cost $662,100 $282,000 $216,200
    ## 6        Utilities      128      118      100
    ## 7   Transportation      107      110      100
    ## 8    Miscellaneous      120      110      100

CSS Selectors
-------------

The information on the Harry Potter Saga page from IMDB does not exist in the form of table. Use Selector Gadget.

``` r
title_saga = read_html("https://www.imdb.com/list/ls000630791/") %>% 
  html_nodes(css = ".lister-item-header a") %>% 
  html_text()

gross_rev = read_html("https://www.imdb.com/list/ls000630791/") %>% 
  html_nodes(css = ".text-small:nth-child(7) span:nth-child(5)") %>% 
  html_text()

hp_saga_df = tibble(title = title_saga, rev = gross_rev)
```

``` r
url = "https://www.amazon.com/product-reviews/B00005JNBQ/ref=cm_cr_arp_d_viewopt_rvwer?ie=UTF8&reviewerType=avp_only_reviews&sortBy=recent&pageNumber=1"

dynamite_html = read_html(url)
review_titles = dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-title") %>%
  html_text()

review_stars = dynamite_html %>%
  html_nodes("#cm_cr-review_list .review-rating") %>%
  html_text()

review_text = dynamite_html %>%
  html_nodes(".review-data:nth-child(4)") %>%
  html_text()

review_df = tibble(
  stars = review_stars,
  title = review_titles,
  text = review_text
  )
```

APIs
----

Get the water data

Here we have used; API--&gt; CSV

``` r
nyc_water = GET("https://data.cityofnewyork.us/resource/waf7-5gvc.csv") %>% 
content("parsed")
```

    ## Parsed with column specification:
    ## cols(
    ##   new_york_city_population = col_double(),
    ##   nyc_consumption_million_gallons_per_day = col_double(),
    ##   per_capita_gallons_per_person_per_day = col_integer(),
    ##   year = col_integer()
    ## )

Using API--&gt; JSON It gives you more information than just the rectangle you extracted with CSV

``` r
nyc_water = GET("https://data.cityofnewyork.us/resource/waf7-5gvc.json") %>% 
  content("text") %>% 
   jsonlite::fromJSON() %>% 
  as_tibble()
```
