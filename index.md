---
title: "Virginia 2021 Governor Election Project"
author: "Aaron Kessler"
date: "4/19/2022"
output: 
  rmdformats::downcute:
    keep_md: true
---



# Getting 2020 Presidential results 

Data available here: <https://historical.elections.virginia.gov/elections/view/144567/>

A little column cleaning and we'll load in the data file.


```r
prez_2020 <- read_csv("processed_data/va_2020_prez_cleaned.csv")
```

```
## Rows: 134 Columns: 4
## ── Column specification ────────────────────────────────────────────────────────
## Delimiter: ","
## chr (1): locality
## 
## ℹ Use `spec()` to retrieve the full column specification for this data.
## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```

Let's see what we have


```r
head(prez_2020)
```

```
## # A tibble: 6 × 4
##   locality         biden trump total_votes_2021_prez
##   <chr>            <dbl> <dbl>                 <dbl>
## 1 Accomack County   7578  9172                 16962
## 2 Albemarle County 42466 20804                 64657
## 3 Alexandria City  66240 14544                 82508
## 4 Alleghany County  2243  5859                  8203
## 5 Amelia County     2411  5390                  7893
## 6 Amherst County    5672 11041                 17005
```

Calculating percentage of the vote


```r
prez_2020 %>% 
  mutate(
    biden_pct = biden/total_votes_2021_prez
  )
```

```
## # A tibble: 134 × 5
##    locality           biden trump total_votes_2021_prez biden_pct
##    <chr>              <dbl> <dbl>                 <dbl>     <dbl>
##  1 Accomack County     7578  9172                 16962     0.447
##  2 Albemarle County   42466 20804                 64657     0.657
##  3 Alexandria City    66240 14544                 82508     0.803
##  4 Alleghany County    2243  5859                  8203     0.273
##  5 Amelia County       2411  5390                  7893     0.305
##  6 Amherst County      5672 11041                 17005     0.334
##  7 Appomattox County   2418  6702                  9268     0.261
##  8 Arlington County  105344 22318                130699     0.806
##  9 Augusta County     10840 30714                 42278     0.256
## 10 Bath County          646  1834                  2501     0.258
## # … with 124 more rows
```

Now let's do some rounding and move that decimal point


```r
prez_2020 %>% 
  mutate(
    biden_pct = janitor::round_half_up(biden / total_votes_2021_prez * 100, 1)
  )
```

```
## # A tibble: 134 × 5
##    locality           biden trump total_votes_2021_prez biden_pct
##    <chr>              <dbl> <dbl>                 <dbl>     <dbl>
##  1 Accomack County     7578  9172                 16962      44.7
##  2 Albemarle County   42466 20804                 64657      65.7
##  3 Alexandria City    66240 14544                 82508      80.3
##  4 Alleghany County    2243  5859                  8203      27.3
##  5 Amelia County       2411  5390                  7893      30.5
##  6 Amherst County      5672 11041                 17005      33.4
##  7 Appomattox County   2418  6702                  9268      26.1
##  8 Arlington County  105344 22318                130699      80.6
##  9 Augusta County     10840 30714                 42278      25.6
## 10 Bath County          646  1834                  2501      25.8
## # … with 124 more rows
```

Now trump too


```r
prez_2020 <- prez_2020 %>% 
  mutate(
    biden_pct = janitor::round_half_up(biden / total_votes_2021_prez * 100, 2),
    trump_pct = janitor::round_half_up(trump / total_votes_2021_prez * 100, 2)
  )

head(prez_2020)
```

```
## # A tibble: 6 × 6
##   locality         biden trump total_votes_2021_prez biden_pct trump_pct
##   <chr>            <dbl> <dbl>                 <dbl>     <dbl>     <dbl>
## 1 Accomack County   7578  9172                 16962      44.7      54.1
## 2 Albemarle County 42466 20804                 64657      65.7      32.2
## 3 Alexandria City  66240 14544                 82508      80.3      17.6
## 4 Alleghany County  2243  5859                  8203      27.3      71.4
## 5 Amelia County     2411  5390                  7893      30.6      68.3
## 6 Amherst County    5672 11041                 17005      33.4      64.9
```

# Getting 2021 Virginia governor results 

Data available from the state here: <https://results.elections.virginia.gov/vaelections/2021%20November%20General/Site/Statewide.html>

In JSON format only


```r
#build the url
url <- "https://results.elections.virginia.gov/vaelections/2021%20November%20General/Json/Governor.json"

get_info <- GET(url)

this.raw.content <- rawToChar(get_info$content)
this.content <- fromJSON(this.raw.content)

#dataframe from just the 6 content 
content_df <- as.data.frame(this.content[[6]])
```

Where are candidates themselves? They are "nested" inside. We'll use `unnest()`to expand things.


```r
#unnest
results <- content_df %>%
  unnest(cols = Candidates)

head(results)
```

```
## # A tibble: 6 × 9
##   Locality$LocalityNa… PrecinctsReport… PrecinctsPartic… LastModified BallotName
##   <chr>                           <int>            <int> <chr>        <chr>     
## 1 ACCOMACK COUNTY                    19               19 2021-11-08T… Glenn A. …
## 2 ACCOMACK COUNTY                    19               19 2021-11-08T… Terry R. …
## 3 ACCOMACK COUNTY                    19               19 2021-11-08T… Princess …
## 4 ACCOMACK COUNTY                    19               19 2021-11-08T… Write In  
## 5 ALBEMARLE COUNTY                   33               33 2021-11-08T… Glenn A. …
## 6 ALBEMARLE COUNTY                   33               33 2021-11-08T… Terry R. …
## # … with 5 more variables: Locality$LocalityCode <chr>, BallotOrder <int>,
## #   Votes <int>, Percentage <chr>, PoliticalParty <chr>
```

Unnest again on locality


```r
results <- results %>%
  unnest(cols = Locality)

head(results)
```

```
## # A tibble: 6 × 10
##   LocalityName     LocalityCode PrecinctsReporting PrecinctsPartic… LastModified
##   <chr>            <chr>                     <int>            <int> <chr>       
## 1 ACCOMACK COUNTY  001                          19               19 2021-11-08T…
## 2 ACCOMACK COUNTY  001                          19               19 2021-11-08T…
## 3 ACCOMACK COUNTY  001                          19               19 2021-11-08T…
## 4 ACCOMACK COUNTY  001                          19               19 2021-11-08T…
## 5 ALBEMARLE COUNTY 003                          33               33 2021-11-08T…
## 6 ALBEMARLE COUNTY 003                          33               33 2021-11-08T…
## # … with 5 more variables: BallotName <chr>, BallotOrder <int>, Votes <int>,
## #   Percentage <chr>, PoliticalParty <chr>
```

Great.

We'll give it a new name to make it easier for us and clean the column names, remove extraneous ones.


```r
gov_2021 <- results %>% 
  clean_names() %>% 
  select(-precincts_reporting,
         -precincts_participating,
         -last_modified,
         -ballot_order)

head(gov_2021)
```

```
## # A tibble: 6 × 6
##   locality_name    locality_code ballot_name    votes percentage political_party
##   <chr>            <chr>         <chr>          <int> <chr>      <chr>          
## 1 ACCOMACK COUNTY  001           Glenn A. Youn…  7878 61.08%     Republican     
## 2 ACCOMACK COUNTY  001           Terry R. McAu…  4948 38.37%     Democratic     
## 3 ACCOMACK COUNTY  001           Princess L. B…    67 0.52%      Liberation     
## 4 ACCOMACK COUNTY  001           Write In           4 0.03%      Write-In       
## 5 ALBEMARLE COUNTY 003           Glenn A. Youn… 19141 37.21%     Republican     
## 6 ALBEMARLE COUNTY 003           Terry R. McAu… 31919 62.05%     Democratic
```

What's the issue we may have here?

This is actually tidy data which is good - but for comparing the candidates we'd want them on the same row.

## Reshaping

Enter pivot_wider().

We'll get rid of everything we don't need first.


```r
gov_2021 <- gov_2021 %>% 
  filter(ballot_name %in% c("Glenn A. Youngkin", "Terry R. McAuliffe")) %>% 
  select(-locality_code,
         -political_party)
  
gov_2021
```

```
## # A tibble: 266 × 4
##    locality_name    ballot_name        votes percentage
##    <chr>            <chr>              <int> <chr>     
##  1 ACCOMACK COUNTY  Glenn A. Youngkin   7878 61.08%    
##  2 ACCOMACK COUNTY  Terry R. McAuliffe  4948 38.37%    
##  3 ALBEMARLE COUNTY Glenn A. Youngkin  19141 37.21%    
##  4 ALBEMARLE COUNTY Terry R. McAuliffe 31919 62.05%    
##  5 ALEXANDRIA CITY  Glenn A. Youngkin  14013 24.02%    
##  6 ALEXANDRIA CITY  Terry R. McAuliffe 43866 75.20%    
##  7 ALLEGHANY COUNTY Glenn A. Youngkin   4530 74.52%    
##  8 ALLEGHANY COUNTY Terry R. McAuliffe  1518 24.97%    
##  9 AMELIA COUNTY    Glenn A. Youngkin   4720 74.19%    
## 10 AMELIA COUNTY    Terry R. McAuliffe  1617 25.42%    
## # … with 256 more rows
```

Now we'll do the spreading out to reshape.


```r
gov_2021_wide <- gov_2021 %>% 
  pivot_wider(names_from = ballot_name, values_from = c(votes, percentage))

gov_2021_wide
```

```
## # A tibble: 133 × 5
##    locality_name     `votes_Glenn A. Youngkin` `votes_Terry R…` `percentage_Gl…`
##    <chr>                                 <int>            <int> <chr>           
##  1 ACCOMACK COUNTY                        7878             4948 61.08%          
##  2 ALBEMARLE COUNTY                      19141            31919 37.21%          
##  3 ALEXANDRIA CITY                       14013            43866 24.02%          
##  4 ALLEGHANY COUNTY                       4530             1518 74.52%          
##  5 AMELIA COUNTY                          4720             1617 74.19%          
##  6 AMHERST COUNTY                         9731             3897 71.00%          
##  7 APPOMATTOX COUNTY                      5971             1438 80.26%          
##  8 ARLINGTON COUNTY                      21548            73013 22.63%          
##  9 AUGUSTA COUNTY                        26196             7231 77.93%          
## 10 BATH COUNTY                            1539              396 79.04%          
## # … with 123 more rows, and 1 more variable:
## #   `percentage_Terry R. McAuliffe` <chr>
```

Nice.

This is giving us some pretty long column names. we can change them after the fact using `rename()`. But first let's clean the names to make it easier.


```r
gov_2021_wide <- gov_2021_wide %>% 
  clean_names()

head(gov_2021_wide)
```

```
## # A tibble: 6 × 5
##   locality_name    votes_glenn_a_youngkin votes_terry_r_mc_aul… percentage_glen…
##   <chr>                             <int>                 <int> <chr>           
## 1 ACCOMACK COUNTY                    7878                  4948 61.08%          
## 2 ALBEMARLE COUNTY                  19141                 31919 37.21%          
## 3 ALEXANDRIA CITY                   14013                 43866 24.02%          
## 4 ALLEGHANY COUNTY                   4530                  1518 74.52%          
## 5 AMELIA COUNTY                      4720                  1617 74.19%          
## 6 AMHERST COUNTY                     9731                  3897 71.00%          
## # … with 1 more variable: percentage_terry_r_mc_auliffe <chr>
```

Now let's rename, and we'll use similar names to what we had earlier in our 2021 results.


```r
gov_2021_wide <- gov_2021_wide %>% 
  rename(
    youngkin = votes_glenn_a_youngkin,
    mcauliffe = votes_terry_r_mc_auliffe,
    pct_youngkin = percentage_glenn_a_youngkin,
    pct_mcauliffe = percentage_terry_r_mc_auliffe
  )

head(gov_2021_wide)
```

```
## # A tibble: 6 × 5
##   locality_name    youngkin mcauliffe pct_youngkin pct_mcauliffe
##   <chr>               <int>     <int> <chr>        <chr>        
## 1 ACCOMACK COUNTY      7878      4948 61.08%       38.37%       
## 2 ALBEMARLE COUNTY    19141     31919 37.21%       62.05%       
## 3 ALEXANDRIA CITY     14013     43866 24.02%       75.20%       
## 4 ALLEGHANY COUNTY     4530      1518 74.52%       24.97%       
## 5 AMELIA COUNTY        4720      1617 74.19%       25.42%       
## 6 AMHERST COUNTY       9731      3897 71.00%       28.43%
```

Bingo.

There's still one potential issue here. Can you see it?

The percentage columns are actually text values, not numbers. And they have that `%` sign in the text too. Let's fix that using a handy function from the readr package, `parse_number()`.


```r
gov_2021_wide <- gov_2021_wide %>% 
  mutate(
    pct_youngkin = readr::parse_number(pct_youngkin),
    pct_mcauliffe = readr::parse_number(pct_mcauliffe)
  )

head(gov_2021_wide)
```

```
## # A tibble: 6 × 5
##   locality_name    youngkin mcauliffe pct_youngkin pct_mcauliffe
##   <chr>               <int>     <int>        <dbl>         <dbl>
## 1 ACCOMACK COUNTY      7878      4948         61.1          38.4
## 2 ALBEMARLE COUNTY    19141     31919         37.2          62.0
## 3 ALEXANDRIA CITY     14013     43866         24.0          75.2
## 4 ALLEGHANY COUNTY     4530      1518         74.5          25.0
## 5 AMELIA COUNTY        4720      1617         74.2          25.4
## 6 AMHERST COUNTY       9731      3897         71            28.4
```

Perfect. Problem solved.

# Putting things together - joining 

Now we want to actually join things up.

Can we do that? Let's check out our two tables


```r
gov_2021_wide
```

```
## # A tibble: 133 × 5
##    locality_name     youngkin mcauliffe pct_youngkin pct_mcauliffe
##    <chr>                <int>     <int>        <dbl>         <dbl>
##  1 ACCOMACK COUNTY       7878      4948         61.1          38.4
##  2 ALBEMARLE COUNTY     19141     31919         37.2          62.0
##  3 ALEXANDRIA CITY      14013     43866         24.0          75.2
##  4 ALLEGHANY COUNTY      4530      1518         74.5          25.0
##  5 AMELIA COUNTY         4720      1617         74.2          25.4
##  6 AMHERST COUNTY        9731      3897         71            28.4
##  7 APPOMATTOX COUNTY     5971      1438         80.3          19.3
##  8 ARLINGTON COUNTY     21548     73013         22.6          76.7
##  9 AUGUSTA COUNTY       26196      7231         77.9          21.5
## 10 BATH COUNTY           1539       396         79.0          20.3
## # … with 123 more rows
```


```r
prez_2020
```

```
## # A tibble: 134 × 6
##    locality           biden trump total_votes_2021_prez biden_pct trump_pct
##    <chr>              <dbl> <dbl>                 <dbl>     <dbl>     <dbl>
##  1 Accomack County     7578  9172                 16962      44.7      54.1
##  2 Albemarle County   42466 20804                 64657      65.7      32.2
##  3 Alexandria City    66240 14544                 82508      80.3      17.6
##  4 Alleghany County    2243  5859                  8203      27.3      71.4
##  5 Amelia County       2411  5390                  7893      30.6      68.3
##  6 Amherst County      5672 11041                 17005      33.4      64.9
##  7 Appomattox County   2418  6702                  9268      26.1      72.3
##  8 Arlington County  105344 22318                130699      80.6      17.1
##  9 Augusta County     10840 30714                 42278      25.6      72.6
## 10 Bath County          646  1834                  2501      25.8      73.3
## # … with 124 more rows
```

Few quick things to deal with... first let's get rid of the total votes in 2021, we don't need that anymore. But also the locality isn't the same case as in our governor's results, so let's uppercase it.


```r
prez_2020 <- prez_2020 %>% 
  mutate(
    locality = str_to_upper(locality)
  ) %>% 
  select(-total_votes_2021_prez)

prez_2020
```

```
## # A tibble: 134 × 5
##    locality           biden trump biden_pct trump_pct
##    <chr>              <dbl> <dbl>     <dbl>     <dbl>
##  1 ACCOMACK COUNTY     7578  9172      44.7      54.1
##  2 ALBEMARLE COUNTY   42466 20804      65.7      32.2
##  3 ALEXANDRIA CITY    66240 14544      80.3      17.6
##  4 ALLEGHANY COUNTY    2243  5859      27.3      71.4
##  5 AMELIA COUNTY       2411  5390      30.6      68.3
##  6 AMHERST COUNTY      5672 11041      33.4      64.9
##  7 APPOMATTOX COUNTY   2418  6702      26.1      72.3
##  8 ARLINGTON COUNTY  105344 22318      80.6      17.1
##  9 AUGUSTA COUNTY     10840 30714      25.6      72.6
## 10 BATH COUNTY          646  1834      25.8      73.3
## # … with 124 more rows
```

Great. Now we're all ready to join. Or are we?

Look closely at the record counts for our two tables. It looks like the presidential results has an extra record perhaps? Let's investigate.

We can use `anti_join()` to see what's in one table but not the other.


```r
anti_join(prez_2020, gov_2021_wide, by = c("locality" = "locality_name"))
```

```
## # A tibble: 2 × 5
##   locality                biden   trump biden_pct trump_pct
##   <chr>                   <dbl>   <dbl>     <dbl>     <dbl>
## 1 KING AND QUEEN COUNTY    1590    2450      38.6      59.5
## 2 TOTALS                2413568 1962430      54.1      44
```

There's a total row in there! Well we don't want that. Let's take it out.

Also though notice a second county didn't match...want to be that might be because it's differently worded? Let's see by doing the anti-join in reverse now.


```r
anti_join(gov_2021_wide, prez_2020, by = c("locality_name" = "locality"))
```

```
## # A tibble: 1 × 5
##   locality_name       youngkin mcauliffe pct_youngkin pct_mcauliffe
##   <chr>                  <int>     <int>        <dbl>         <dbl>
## 1 KING & QUEEN COUNTY     2112      1130         64.8          34.6
```

Yep that's it. So let's fix both of those things in the prez table.


```r
prez_2020 <- prez_2020 %>% 
  filter(locality != "TOTALS") %>% 
  mutate(
    locality = str_replace(locality, "KING AND QUEEN", "KING & QUEEN")
  )
```

Now we're ready. Let's join.


```r
inner_join(prez_2020, gov_2021_wide, by = c("locality" = "locality_name"))
```

```
## # A tibble: 133 × 9
##    locality      biden trump biden_pct trump_pct youngkin mcauliffe pct_youngkin
##    <chr>         <dbl> <dbl>     <dbl>     <dbl>    <int>     <int>        <dbl>
##  1 ACCOMACK CO…   7578  9172      44.7      54.1     7878      4948         61.1
##  2 ALBEMARLE C…  42466 20804      65.7      32.2    19141     31919         37.2
##  3 ALEXANDRIA …  66240 14544      80.3      17.6    14013     43866         24.0
##  4 ALLEGHANY C…   2243  5859      27.3      71.4     4530      1518         74.5
##  5 AMELIA COUN…   2411  5390      30.6      68.3     4720      1617         74.2
##  6 AMHERST COU…   5672 11041      33.4      64.9     9731      3897         71  
##  7 APPOMATTOX …   2418  6702      26.1      72.3     5971      1438         80.3
##  8 ARLINGTON C… 105344 22318      80.6      17.1    21548     73013         22.6
##  9 AUGUSTA COU…  10840 30714      25.6      72.6    26196      7231         77.9
## 10 BATH COUNTY     646  1834      25.8      73.3     1539       396         79.0
## # … with 123 more rows, and 1 more variable: pct_mcauliffe <dbl>
```

Alright! It worked.

## Comparing gov vs. prez results

Now that things are join, let's actually go ahead and start making columns to compare the two elections and how the candidates did this time compared with last time.

Where should we go from here....?










