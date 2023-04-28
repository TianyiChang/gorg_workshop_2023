# **Explore CheckM2 and GUNC outputs using R**

## Load R packages
```r
library(tidyverse)
```

## Set the working directory
```r
setwd("/mnt/storage/data/daily-data/day2/uncurated_SAGs")
```

## Import GUNC and CheckM outputs
```r
import_outputs <- function(path, outfile) {

    combined_df <- list.files(
        path = {{path}},
        pattern = {{outfile}},
        recursive = TRUE,
        full.names = TRUE
    ) %>%
        map_dfr(read_tsv)
    
    return(combined_df)

}

checkm_df <- import_outputs("checkm2_output", "quality_report.tsv")
gunc_df <- import_outputs("gunc_output", "GUNC.progenomes_2.1.maxCSS_level.tsv")
```
## Check the data frames
```r
head(checkm_df)
head(gunc_df)

names(checkm_df)
names(gunc_df)
```
## Merge data frames
```r
merged_df <- gunc_df %>%
    mutate(
        Name = str_replace(genome, "\\.", ""), # make SAG IDs be consistent between dfs
        plate = str_replace(Name, "-\\w+_contigs", "")
    ) %>%
    select(-genome) %>%
    left_join(checkm_df, by = "Name")
```
## Explore the metrics
### The distribution of checkm-estimated contamination
```
ggplot(merged_df, aes(x=Contamination, fill=plate)) +
    geom_histogram() +
    theme_classic()
```
### The distribution of GUNC's CSS
```r
binwidth <- c(0.01)

ggplot(merged_df, aes(x=clade_separation_score, fill=plate)) +
    geom_histogram(
        aes(y=100*binwidth*..density..),
            alpha=0.5, position='identity',
            binwidth=binwidth
            ) +
    geom_vline(xintercept=0.45, color="black",
            linetype="dashed", size=0.5) +
    labs(y="Percentage", x="Clade Separation Score")+
    theme_classic()
```
### Identify which genomes have higher estimated contamination level
```r
sags_w_high_est_contam <- merged_df %>%
    filter(
        clade_separation_score >= 0.85 |
        Contamination >= 10
        )
```
### Check the metrics of interests
```r
sags_w_high_est_contam %>%
    select(Name, clade_separation_score, Contamination, Completeness)
```