
<!-- README.md is generated from README.Rmd. Please edit that file -->

# microViz <a href='https://david-barnett.github.io/microViz/index.html'><img src="man/figures/microViz-smaller.png" align="right" height="139"/></a>

<!-- badges: start -->

[![R-CMD-check](https://github.com/david-barnett/microViz/workflows/R-CMD-check/badge.svg)](https://github.com/david-barnett/microViz/actions)
[![codecov](https://codecov.io/gh/david-barnett/microViz/branch/main/graph/badge.svg?token=C1EoVkhnxA)](https://codecov.io/gh/david-barnett/microViz)
[![DOI](https://zenodo.org/badge/307119750.svg)](https://zenodo.org/badge/latestdoi/307119750)
[![Docker Cloud Build
Status](https://img.shields.io/docker/cloud/build/barnettdavid/microviz-rocker-verse)](https://hub.docker.com/r/barnettdavid/microviz-rocker-verse)
[![status](https://joss.theoj.org/papers/4547b492f224a26d96938ada81fee3fa/status.svg)](https://joss.theoj.org/papers/4547b492f224a26d96938ada81fee3fa)

<!-- badges: end -->

## Overview

:package: `microViz` is an R package for analysis and visualization of
microbiome sequencing data.

:hammer: `microViz` functions are intended to be easy to use and
flexible.

:microscope: `microViz` extends and complements popular microbial
ecology packages like `phyloseq`, `vegan`, & `microbiome`.

## Learn more

:paperclip: This website is the best place for documentation and
examples: <https://david-barnett.github.io/microViz/>

-   **[This ReadMe](https://david-barnett.github.io/microViz/)** shows a
    few example analyses

-   **The
    [Reference](https://david-barnett.github.io/microViz/reference/index.html)
    page** lists all functions and links to help pages and examples

-   **The
    [Changelog](https://david-barnett.github.io/microViz/news/index.html)**
    describes important changes in new microViz package versions

-   **The Articles pages** give tutorials and further examples

    -   [Working with phyloseq
        objects](https://david-barnett.github.io/microViz/articles/web-only/phyloseq.html)

    -   [Fixing your taxa table with
        tax\_fix](https://david-barnett.github.io/microViz/articles/web-only/tax-fixing.html)

    -   [Creating ordination
        plots](https://david-barnett.github.io/microViz/articles/web-only/ordination.html)
        (e.g. PCA or PCoA)

    -   [Interactive ordination plots with
        ord\_explore](https://david-barnett.github.io/microViz/articles/web-only/ordination-interactive.html)

    -   [Visualising taxonomic compositions with
        comp\_barplot](https://david-barnett.github.io/microViz/articles/web-only/compositions.html)

    -   [Modelling and plotting individual taxon associations with
        taxatrees](https://david-barnett.github.io/microViz/articles/web-only/modelling-taxa.html)

    -   More coming soon(ish)! Post on [GitHub
        discussions](https://github.com/david-barnett/microViz/discussions)
        if you have questions/requests

## Installation

You can install the latest available microViz package version using the
following instructions.

``` r
# Installing from github requires the devtools package
install.packages("devtools") 

# To install the latest "released" version of this package
devtools::install_github("david-barnett/microViz@0.8.0") # check 0.8.0 is the latest release

# To install the very latest version:
devtools::install_github("david-barnett/microViz")
# If you encounter a bug please try the latest version & let me know if the bug persists!

# If the Bioconductor dependencies don't automatically install you can install
# them yourself like this:
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager") 
BiocManager::install(c("phyloseq", "microbiome", "ComplexHeatmap"))
```

:computer: **Windows users** - will need to have RTools installed so
that your computer can build this package (follow instructions here:
<http://jtleek.com/modules/01_DataScientistToolbox/02_10_rtools/>)

**:apple: macOS** **users** - might need to install
[xquartz](https://www.xquartz.org/) to make the heatmaps work (to do
this with homebrew, run the following command in your mac’s Terminal:
`brew install --cask xquartz`

:package: I highly recommend using
[renv](https://rstudio.github.io/renv/index.html) for managing your R
package installations across multiple projects.

:whale: Alternatively, for Docker users an image with the main branch
installed is available at:
<https://hub.docker.com/r/barnettdavid/microviz-rocker-verse>

:date: microViz is tested to work with R version 4 on Windows, MacOS,
and Ubuntu 18 and 20. R version 3.6.\* should probably work, but I don’t
formally test this.

## Interactive ordination exploration

``` r
library(microViz)
```

microViz provides a Shiny app for an easy way to start exploring your
microbiome data: all you need is a phyloseq object.

``` r
# example data from corncob package
pseq <- corncob::ibd_phylo %>% tax_fix() %>% phyloseq_validate()
```

``` r
ord_explore(pseq) # gif generated with microViz version 0.7.4 (plays at 1.75x speed)
```

![](vignettes/web-only/images/20210429_ord_explore_x175.gif)

## Example analyses

``` r
library(phyloseq)
library(dplyr)
#> 
#> Attaching package: 'dplyr'
#> The following objects are masked from 'package:stats':
#> 
#>     filter, lag
#> The following objects are masked from 'package:base':
#> 
#>     intersect, setdiff, setequal, union
library(ggplot2)
```

``` r
# get some example data
data("dietswap", package = "microbiome")

# create a couple of numerical variables to use as constraints or conditions
dietswap <- dietswap %>%
  ps_mutate(
    weight = recode(bmi_group, obese = 3, overweight = 2, lean = 1),
    female = if_else(sex == "female", true = 1, false = 0)
  )
# add a couple of missing values to show how microViz handles missing data
sample_data(dietswap)$female[c(3, 4)] <- NA
```

### Looking at your data

You have quite a few samples in your phyloseq object, and would like to
visualise their compositions. Perhaps these example data differ across
BMI groups?

``` r
dietswap %>%
  tax_filter(min_prevalence = 1) %>%
  comp_barplot(
    tax_level = "Genus",  n_taxa = 12, 
    merge_other = FALSE 
    # set merge_other = TRUE (the default) to remove outlines from inside "other" category
  ) +
  facet_wrap("bmi_group", nrow = 3, scales = "free") +
  labs(x = NULL, y = NULL) +
  theme(
    axis.text.x = element_blank(),
    axis.ticks.x = element_blank()
  )
#> Registered S3 method overwritten by 'seriation':
#>   method         from 
#>   reorder.hclust vegan
```

<img src="man/figures/README-unnamed-chunk-2-1.png" width="100%" />

### Example ordination plot workflow

Maybe visually inspecting all your samples isn’t quite what you want.
Ordination methods can also help you to visualise if overall microbial
ecosystem composition differs markedly between groups, e.g. BMI.

Here is one option as an example:

1.  Filter out rare taxa (e.g. remove Genera not present in at least 10%
    of samples) - use `tax_filter()`
2.  Aggregate the taxa into bacterial families (for example) - use
    `tax_agg()`
3.  Transform the microbial data with the centre-log-ratio
    transformation - use `tax_transform()`
4.  Perform PCA with the clr-transformed features (equivalent to
    aitchison distance PCoA) - use `ord_calc()`
5.  Plot the first 2 axes of this PCA ordination, colouring samples by
    group and adding taxon loading arrows to visualise which taxa
    generally differ across your samples - use `ord_plot()`
6.  Customise the theme of the ggplot as you like and/or add features
    like ellipses or annotations

``` r
# perform ordination
unconstrained_aitchison_pca <-
  dietswap %>%
  tax_filter(min_prevalence = 0.1, tax_level = "Genus") %>%
  tax_agg("Family") %>%
  tax_transform("clr") %>%
  ord_calc() 
#> Proportional min_prevalence given: 0.1 --> min 23/222 samples.
# ord_calc will automatically infer you want a "PCA" here
# specify explicitly with method = "PCA", or you can pick another method

# create plot
pca_plot <- unconstrained_aitchison_pca %>%
  ord_plot(
    plot_taxa = 1:6, colour = "bmi_group", size = 1.5,
    tax_vec_length = 0.325, 
    tax_lab_style = tax_lab_style(max_angle = 90, aspect_ratio = 0.5),
    auto_caption = 8
  )

# customise plot
customised_plot <- pca_plot +
  stat_ellipse(aes(linetype = bmi_group, colour = bmi_group), size = 0.3) +
  scale_colour_brewer(palette = "Set1") +
  theme(legend.position = "bottom") +
  coord_fixed(ratio = 0.5, clip = "off") # makes rotated labels align correctly

# show plot
customised_plot
```

<img src="man/figures/README-ordination-plot-1.png" width="100%" />

### PERMANOVA

You visualised your ordinated data in the plot above. Below you can see
how to perform a PERMANOVA to test the significance of BMI’s association
with overall microbial composition. This example uses the Family-level
aitchison distance to correspond with the plot above.

``` r
# calculate distances
aitchison_dists <-
  dietswap %>%
  tax_filter(min_prevalence = 0.1, tax_level = "Genus") %>%
  tax_agg("Family") %>%
  tax_transform("identity") %>% 
  dist_calc("aitchison")
#> Proportional min_prevalence given: 0.1 --> min 23/222 samples.

# the more permutations you request, the longer it takes
# but also the more stable and precise your p-values become
aitchison_perm <- dist_permanova(
  data = aitchison_dists,
  seed = 1234, # for set.seed to ensure reproducibility of random process
  n_perms = 99, # you should use at least 999!
  n_processes = 1,
  variables = "bmi_group + female"
)
#> Dropping samples with missings: 2
#> 2021-11-08 11:42:58 - Starting PERMANOVA with 99 perms with 1 processes
#> 2021-11-08 11:42:58 - Finished PERMANOVA
# view the permanova results
perm_get(aitchison_perm) %>% as.data.frame()
#>            Df   SumOfSqs         R2        F Pr(>F)
#> bmi_group   2  106.32182 0.04332620 5.008911   0.01
#> female      1   53.22231 0.02168812 5.014697   0.01
#> Residual  216 2292.46559 0.93418103       NA     NA
#> Total     219 2453.98430 1.00000000       NA     NA
# view the info stored about the distance calculation
info_get(aitchison_perm)
#> ps_extra info:
#> tax_agg = Family 
#> tax_transform = identity 
#> tax_scale = NA 
#> distMethod = aitchison 
#> ordMethod = NA 
#> constraints = NA 
#> conditions = NA
```

### Constrained ordination

You could visualise the effect of the (numeric/logical) variables in
your permanova directly using the ord\_plot function with constraints.

``` r
perm2 <- dist_permanova(
  data = aitchison_dists, variables = c("weight", "female"), seed = 321
)
#> Dropping samples with missings: 2
#> 2021-11-08 11:42:58 - Starting PERMANOVA with 999 perms with 1 processes
#> 2021-11-08 11:42:58 - Finished PERMANOVA
perm_get(perm2)
#> Permutation test for adonis under reduced model
#> Marginal effects of terms
#> Permutation: free
#> Number of permutations: 999
#> 
#> vegan::adonis2(formula = formula, data = metadata, permutations = n_perms, by = by, parallel = parall)
#>           Df SumOfSqs      R2      F Pr(>F)    
#> weight     1    56.06 0.02284 5.1927  0.001 ***
#> female     1    55.83 0.02275 5.1714  0.001 ***
#> Residual 217  2342.73 0.95466                  
#> Total    219  2453.98 1.00000                  
#> ---
#> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```

``` r
ord_calc(perm2, constraints = c("weight", "female")) %>%
  ord_plot(
    colour = "sex", size = 1.5,
    constraint_vec_length = 1,
    constraint_vec_style = vec_constraint(size = 1.15, colour = "black"),
    constraint_lab_style = constraint_lab_style(
      max_angle = 90, size = 3, aspect_ratio = 0.75, colour = "black"
      )
  ) +
  stat_ellipse(aes(colour = sex), size = 0.3) + 
  scale_color_brewer(palette = "Dark2") +
  coord_fixed(ratio = 0.75)
#> 
#> Centering (mean) and scaling (sd) the constraints and conditions:
#>  weight
#>  female
```

<img src="man/figures/README-constrained-ord-plot-1.png" width="100%" />

### Heatmaps

microViz heatmaps are powered by `ComplexHeatmap` and annotated with
taxa prevalence and/or abundance.

``` r
# set up the data with numerical variables and filter to top taxa
psq <- dietswap %>%
 ps_mutate(
   weight = recode(bmi_group, obese = 3, overweight = 2, lean = 1),
   female = if_else(sex == "female", true = 1, false = 0),
   african = if_else(nationality == "AFR", true = 1, false = 0)
 ) %>% 
  tax_filter(
    tax_level = "Genus", min_prevalence = 1 / 10, min_sample_abundance = 1 / 10
  ) %>% 
  tax_transform("identity", rank = "Genus")
#> Proportional min_prevalence given: 0.1 --> min 23/222 samples.

# randomly select 30 taxa from the 50 most abundant taxa (just for an example)
set.seed(123)
taxa <- sample(microbiome::top_taxa(ps_get(psq))[1:50], size = 30)
# actually draw the heatmap
cor_heatmap(
  data = psq, taxa = taxa, 
  anno_tax = tax_anno(undetected = 50, size = 50, rel_sizes = 1:2)
)
```

<img src="man/figures/README-heatmap-1.png" width="100%" />

## Citation

:innocent: If you find any part of microViz useful to your work, please
consider citing the JOSS article:

Barnett et al., (2021). microViz: an R package for microbiome data
visualization and statistics. Journal of Open Source Software, 6(63),
3201, <https://doi.org/10.21105/joss.03201>

## Contributing

Bug reports, questions, suggestions for new features, and other
contributions are all welcome. Feel free to create a [GitHub
Issue](https://github.com/david-barnett/microViz/issues) or write on the
[Discussions](https://github.com/david-barnett/microViz/discussions)
page. Alternatively you could also contact me (David) on Twitter
[@\_david\_barnett\_](https://twitter.com/_david_barnett_) .

This project is released with a [Contributor Code of
Conduct](https://david-barnett.github.io/microViz/CODE_OF_CONDUCT.html)
and by participating in this project you agree to abide by its terms.

## Session info

``` r
sessionInfo()
#> R version 4.1.2 (2021-11-01)
#> Platform: x86_64-pc-linux-gnu (64-bit)
#> Running under: Ubuntu 18.04.6 LTS
#> 
#> Matrix products: default
#> BLAS:   /usr/lib/x86_64-linux-gnu/openblas/libblas.so.3
#> LAPACK: /usr/lib/x86_64-linux-gnu/libopenblasp-r0.2.20.so
#> 
#> locale:
#>  [1] LC_CTYPE=en_GB.UTF-8       LC_NUMERIC=C               LC_TIME=en_GB.UTF-8       
#>  [4] LC_COLLATE=en_GB.UTF-8     LC_MONETARY=en_GB.UTF-8    LC_MESSAGES=en_GB.UTF-8   
#>  [7] LC_PAPER=en_GB.UTF-8       LC_NAME=C                  LC_ADDRESS=C              
#> [10] LC_TELEPHONE=C             LC_MEASUREMENT=en_GB.UTF-8 LC_IDENTIFICATION=C       
#> 
#> attached base packages:
#> [1] stats     graphics  grDevices utils     datasets  methods   base     
#> 
#> other attached packages:
#> [1] ggplot2_3.3.5   dplyr_1.0.7     phyloseq_1.36.0 microViz_0.8.0  devtools_2.4.2  usethis_2.1.3  
#> [7] pkgdown_1.6.1  
#> 
#> loaded via a namespace (and not attached):
#>   [1] Rtsne_0.15             colorspace_2.0-2       rjson_0.2.20           ellipsis_0.3.2        
#>   [5] rprojroot_2.0.2        circlize_0.4.13        markdown_1.1           XVector_0.32.0        
#>   [9] GlobalOptions_0.1.2    fs_1.5.0               gridtext_0.1.4         ggtext_0.1.1          
#>  [13] clue_0.3-60            rstudioapi_0.13        farver_2.1.0           remotes_2.4.1         
#>  [17] fansi_0.5.0            xml2_1.3.2             codetools_0.2-18       splines_4.1.2         
#>  [21] doParallel_1.0.16      cachem_1.0.6           knitr_1.36             pkgload_1.2.3         
#>  [25] ade4_1.7-18            jsonlite_1.7.2         Cairo_1.5-12.2         cluster_2.1.2         
#>  [29] png_0.1-7              compiler_4.1.2         assertthat_0.2.1       Matrix_1.3-4          
#>  [33] fastmap_1.1.0          cli_3.1.0              htmltools_0.5.2        prettyunits_1.1.1     
#>  [37] tools_4.1.2            igraph_1.2.8           gtable_0.3.0           glue_1.4.2            
#>  [41] GenomeInfoDbData_1.2.6 reshape2_1.4.4         Rcpp_1.0.7             Biobase_2.52.0        
#>  [45] vctrs_0.3.8            Biostrings_2.60.2      rhdf5filters_1.4.0     multtest_2.48.0       
#>  [49] ape_5.5                nlme_3.1-152           iterators_1.0.13       xfun_0.27             
#>  [53] stringr_1.4.0          ps_1.6.0               testthat_3.1.0         lifecycle_1.0.1       
#>  [57] zlibbioc_1.38.0        MASS_7.3-54            scales_1.1.1           TSP_1.1-11            
#>  [61] parallel_4.1.2         biomformat_1.20.0      rhdf5_2.36.0           RColorBrewer_1.1-2    
#>  [65] ComplexHeatmap_2.8.0   yaml_2.2.1             memoise_2.0.0          stringi_1.7.5         
#>  [69] highr_0.9              S4Vectors_0.30.1       desc_1.4.0             foreach_1.5.1         
#>  [73] permute_0.9-5          seriation_1.3.1        BiocGenerics_0.38.0    pkgbuild_1.2.0        
#>  [77] shape_1.4.6            GenomeInfoDb_1.28.4    rlang_0.4.12           pkgconfig_2.0.3       
#>  [81] bitops_1.0-7           matrixStats_0.59.0     evaluate_0.14          lattice_0.20-45       
#>  [85] purrr_0.3.4            Rhdf5lib_1.14.2        labeling_0.4.2         processx_3.5.2        
#>  [89] tidyselect_1.1.1       plyr_1.8.6             magrittr_2.0.1         R6_2.5.1              
#>  [93] IRanges_2.26.0         generics_0.1.1         DBI_1.1.1              pillar_1.6.4          
#>  [97] withr_2.4.2            mgcv_1.8-38            survival_3.2-13        RCurl_1.98-1.5        
#> [101] tibble_3.1.5           corncob_0.2.0          crayon_1.4.2           utf8_1.2.2            
#> [105] microbiome_1.14.0      rmarkdown_2.11         GetoptLong_1.0.5       grid_4.1.2            
#> [109] data.table_1.14.2      callr_3.7.0            vegan_2.5-7            digest_0.6.28         
#> [113] tidyr_1.1.4            stats4_4.1.2           munsell_0.5.0          registry_0.5-1        
#> [117] sessioninfo_1.2.0
```
