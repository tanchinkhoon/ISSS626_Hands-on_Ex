# Take-home Ex03 README

This project provides a complete, reproducible Quarto workflow for modeling HDB resale prices in Singapore using global and local spatial methods. The study builds a Multiple Linear Regression (MLR) hedonic model and a Geographically Weighted model (GWmodel) to examine localised effects. Outputs include publication-grade regression tables, diagnostics, residual spatial tests, and cartographic products such as Local R-squared and coefficient maps. All computations run from a single Quarto source using relative paths. Spatial operations use EPSG 3414 with metre units.

## Folder layout

1. take home ex03.qmd Quarto source
2. `data/aspatial` folder for CSV and lookups
3. `data/geospatial` folder for sf layers
4. `data/rds` for RDS artefacts
5. figs folder for saved images, plots, and maps
6. outputs folder for tables and intermediate files
7. HTML folder for the rendered site if used

## Software and packages

1. R version 4.3 or newer
2. Quarto version 1.5 or newer
3. R packages

```r
packages <- c(
  "tidyverse", "sf", "sfdep", "tmap", "GWmodel",
  "gtsummary", "performance", "see", "qqplotr",
  "olsrr", "corrplot", "ggpubr", "RColorBrewer"
)
install.packages(setdiff(packages, installed.packages()[, "Package"]))
```

## Data requirements

1. HDB resale transactions with geometry POINT in EPSG 3414
2. Planning polygons mpsz in EPSG 3414
3. Target point layers for location predictors in EPSG 3414

   * MRT stations
   * Parks
   * Eldercare
   * Hawker centres
   * Supermarkets
   * Shopping malls
   * Primary schools with an elite school filter
4. Service points for buffer counts

   * Kindergartens
   * Childcare centres
   * Bus stops
   * Primary schools

Distances are in metres. Counts use buffers of 350 metres or 1000 metres as stated in the report.

## Run order

1. Open the project in RStudio
2. Create folders data/aspatial, data/geospatial, and data/rds if missing
3. Download the source data, as indicated in Table 1, and place the files in the correct folders
4. Open take-home_ex03.qmd
5. Click Render or run the command below

```bash
quarto render "take home ex03.qmd"
```

## Modelling specification

Dependent variable

* RESALE_PRICE

Multiple model predictors

* FLOOR_LEVEL
* REMAINING_LEASE_MTH
* AGE_2025
* DIST_CBD_M
* DIST_MRT_M
* DIST_PARK_M
* DIST_ELDERCARE_M
* DIST_HAWKER_M
* DIST_SUPERMKT_M
* DIST_MALL_M
* DIST_ELITEPRI_M
* CNT_KINDER_350
* CNT_CHILD_350
* CNT_BUS_350
* CNT_PRIMARY_1000

## Section 9 Hedonic Pricing Modelling

1. Prepare a correlation matrix of the predictors using corrplot AOE order upper triangle
2. Fit the full Multiple model on RESALE_PRICE with the exact 15 predictors
3. Revise the model if required, and keep the revised fit as the reference
4. Produce the table with gtsummary tbl regression, including intercept, t, then add glance source note with R-squared, adjusted R-squared, AIC, F statistic, overall p value, and sigma
5. Diagnostics

   * performance check on collinearity and plot the result
   * see check model for linearity and normality
   * performance check normality with QQ plot
6. Residual spatial test

   * build neighbours using st dist band upper 1500
   * weights style W with st weights
   * run global Moran perm with 499 simulations on the residual vector
7. Residual map

   * tmap plot mode
   * legend inside the frame
   * quantile classification

## Section 10 Geographically Weighted model

1. Compute fixed bandwidth with cross-validation, then fit GWR based on the same predictors
2. Compute adaptive bandwidth with cross-validation, n, then fit GWR basic with adaptive TRUE
3. Convert output SDF to sf and enforce EPSG 3414
4. Map Local R-squared with quantile breaks and a legend inside the frame
5. Optionally map selected coefficients and standard errors with consistent styling

## Case study template for Tampines

1. Select the Tampines polygon from mpsz using the planning area field
2. Subset GWR points within the polygon using st within
3. Map boundary and points coloured by Local R-squared with quantile style
4. Summarise Local R-squared using n min, quartiles, median, mean, and max for narrative reporting

## Rendering notes for interactivity

Static reporting

```r
tmap::tmap_mode("plot")
```

Interactive checks inside RStudio

```r
tmap::tmap_mode("view")
```

Return to plot mode before knitting.

## Troubleshooting

1. No map appears after a chunk

   * Print the tmap object on its own line or use tmap arrange
   * Ensure tmap mode is set to plot when knitting
2. Moran test error that x is not numeric

   * Confirm residual column exists and is numeric
   * Ensure nb and wt come from the same sf object used for x
3. Colour scale warnings

   * Use tm scale with explicit RColorBrewer values and style quantile
4. CRS mismatch

   * Verify both layers are EPSG 3414 using st crs

## Reproducibility notes

1. All spatial work uses EPSG 3414 in metres
2. Set seed before permutation tests for determinism

```r
set.seed(626)
```

3. Save key artefacts as RDS for reuse

```r
saveRDS(analysis_sf, "data/rds/analysis_sf.rds")
```

## Session info

Add this at the end of the document to record versions

```r
sessionInfo()
```

## How to cite

Include citations for official data sources and R packages. Create package citations with

```r
citation("GWmodel")
citation("sf")
citation("tmap")
```

## Contact

Open a repository issue with the exact chunk label and the console message if assistance is required.
