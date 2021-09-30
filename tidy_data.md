tidy\_data
================
Xinyuan Liu
2021/9/30

## pivot longer

``` r
pulse_df =
  haven::read_sas("data/public_pulse_data.sas7bdat") %>% 
  janitor::clean_names()
```

    ## Warning in FUN(X[[i]], ...): strings not representable in native encoding will
    ## be translated to UTF-8

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C4>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00D6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E4>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00F6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00DF>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E6>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00D8>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00F8>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00C5>' to native encoding

    ## Warning in FUN(X[[i]], ...): unable to translate '<U+00E5>' to native encoding

let’s try to pivot

``` r
pulse_tidy = 
  pulse_df %>% 
  pivot_longer(
    bdi_score_bl:bdi_score_12m,
    names_to = "visit", 
    names_prefix = "bdi_score_",
    values_to = "bdi"
  ) %>% 
  mutate(
    visit = replace(visit, visit == "bl", "00m"),
    visit = factor(visit)
  )
```

## pivot\_wider

make up a result data table

``` r
analysis_df = 
  tibble(
    group = c("treatment", "treatment", "control", "control"),
    time = c("a", "b", "a", "b"),
    group_mean = c(4, 8, 3, 6)
  )
analysis_df %>% 
  pivot_wider(
    names_from = "time",
    values_from = "group_mean"
  ) %>% 
  knitr::kable()
```

| group     |   a |   b |
|:----------|----:|----:|
| treatment |   4 |   8 |
| control   |   3 |   6 |

## BIND ROWS

import the lord of the ring movie stuff

``` r
fellowship_df = 
  read_excel("data/LotR_Words.xlsx", range ="B3:D6") %>% 
  janitor::clean_names() %>% 
  mutate(movie = "fellowship_rings")
two_towers_df = 
  read_excel("data/LotR_Words.xlsx", range ="F3:H6") %>% 
  janitor::clean_names() %>% 
  mutate(movie = "two_towers")
return_king_df = 
  read_excel("data/LotR_Words.xlsx", range ="J3:L6") %>% 
  janitor::clean_names() %>% 
  mutate(movie = "return_king")

lotr_df = 
  bind_rows(fellowship_df, two_towers_df, return_king_df) %>% 
  janitor::clean_names() %>% 
  pivot_longer(
    female:male,
    names_to = "sex",
    values_to ="words"
  ) %>% 
  relocate(movie)
```

## joins

look at fas data

``` r
litters_df = 
  read_csv("data/FAS_litters.csv") %>% 
  janitor::clean_names() %>% 
  separate(group, into = c("dose", "day_of_tx"), 3) %>% 
  relocate(litter_number) %>% 
  mutate(dose = str_to_lower(dose))
```

    ## Rows: 49 Columns: 8

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (2): Group, Litter Number
    ## dbl (6): GD0 weight, GD18 weight, GD of Birth, Pups born alive, Pups dead @ ...

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
pups_df =
  read_csv("data/FAS_pups.csv") %>% 
  janitor::clean_names() %>% 
  mutate(sex = recode(sex, `1` = "male", `2` = "female"))
```

    ## Rows: 313 Columns: 6

    ## -- Column specification --------------------------------------------------------
    ## Delimiter: ","
    ## chr (1): Litter Number
    ## dbl (5): Sex, PD ears, PD eyes, PD pivot, PD walk

    ## 
    ## i Use `spec()` to retrieve the full column specification for this data.
    ## i Specify the column types or set `show_col_types = FALSE` to quiet this message.

let’s join these up

``` r
fas_df = 
  left_join(pups_df, litters_df, by = "litter_number")
```
