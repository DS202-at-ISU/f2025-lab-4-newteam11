Lab 4 Progress Report - HabigerDan
================

## Load Required Libraries

``` r
library(tidyverse)
library(rvest)
library(readr)
```

## Scrape the Data from Baseball Reference

``` r
url <- "https://www.baseball-reference.com/awards/hof_2025.shtml"
html <- read_html(url)
tables <- html_table(html, fill = TRUE)
raw <- tables[[1]]
```

## Fix Column Names

The first row contains the actual column names.

``` r
col_names <- raw[1, ] |> unlist() |> as.character()
colnames(raw) <- col_names

data <- raw[-1, ]
colnames(data) <- make.names(colnames(data), unique = TRUE)
```

## Convert to Numeric

``` r
data$X.vote <- parse_number(data$X.vote)

num_cols <- c("Votes", "WAR", "WAR7", "JAWS", "HOFm", "HOFs")
data[num_cols] <- lapply(data[num_cols], readr::parse_number)
```

## Clean Names

Remove special characters from player names.

``` r
data$Name <- gsub("\\+", "", data$Name)
data$Name <- gsub("\\*", "", data$Name)
```

## Add Required Variables

``` r
data$inducted <- ifelse(data$X.vote >= 75, "Y", "N")

data$yearID <- 2025
```

## Create Final Dataset

``` r
HallOfFame_2025 <- data
```

## Export to CSV

``` r
write_csv(HallOfFame_2025, "HallOfFame_2025.csv")
```

## Summary

``` r
cat("Total candidates:", nrow(HallOfFame_2025), "\n")
```

    ## Total candidates: 28

``` r
cat("Players inducted:", sum(HallOfFame_2025$inducted == "Y"), "\n")
```

    ## Players inducted: 3

``` r
# Show inducted players
HallOfFame_2025 |>
  filter(inducted == "Y") |>
  select(Name, Votes, X.vote)
```

    ## # A tibble: 3 × 3
    ##   Name          Votes X.vote
    ##   <chr>         <dbl>  <dbl>
    ## 1 Ichiro Suzuki   393   99.7
    ## 2 CC Sabathia     342   86.8
    ## 3 Billy Wagner    325   82.5
