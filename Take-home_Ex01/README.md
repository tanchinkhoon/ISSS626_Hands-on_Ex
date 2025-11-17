# Take-home Ex01: Geospatial Analytics for Public Good

This repository contains the full reproducible workflow for Take-home Ex01 in ISSS626 Geospatial Analytics and Applications. The study uses newly registered restaurant entities in Singapore to explore how geospatial methods can support evidence-based planning for town centres, food services licensing, and last-mile logistics.  

The analysis follows the exact sequence taught in ISSS626: from data tidying and geocoding, through first-order and second-order spatial point pattern analysis, to spatio-temporal kernel density estimation. All code is written in R and organised so that a new user can clone the repository, configure a small number of paths and tokens, then regenerate all tables and figures used in the report.

---

## Research objectives and questions

The substantive focus is on new restaurants classified under SSIC code 56111 registered between 1 Jan 2025 and 30 Jun 2025. The study links firm registrations to planning areas and subzones to examine the geography and dynamics of restaurant entry in Singapore.

The work addresses four main research questions

1. **Where and when**  
   Where do new restaurants concentrate within the study window, and how does this pattern evolve from month to month

2. **First order intensity**  
   What is the underlying intensity structure of new restaurant locations, and which locales exhibit persistent high density after smoothing

3. **Second order interaction**  
   At which distance scales do restaurant locations show statistically significant clustering or regularity when compared to complete spatial randomness

4. **Planning and management implications**  
   How should empirically validated hotspots and clustering scales inform planning of town centres, licensing cadence, amenity placement, and last-mile logistics near activity corridors and nodes

---

## Data sources

The analysis integrates four main data sources that are **not** distributed with this repository. Users must obtain them directly from the original providers.

* **SSIC 2020 report and detailed definitions**  
  Structured codes and descriptions of economic activities, used to identify restaurants under code 56111 and related food service classes  
  Provider SingStat Department of Statistics Singapore  

* **ACRA information on corporate entities**  
  Open data extract of firm-level registrations, including SSIC codes, addresses, entity type, and key dates  
  Provider data dot gov dot sg ACRA  

* **URA Master Plan 2019 subzone boundary no sea**  
  Official planning area and subzone polygons for Singapore  
  Format GeoJSON and KML  
  Provider data dot gov dot sg URA  

* **OneMap search and reverse geocode services**  
  Application programming interface for geocoding six-digit postal codes and returning Singapore SVY21 coordinates  
  Provider Singapore Land Authority OneMap  

You must create a OneMap account and obtain an API token before running the geocoding script.

---

## Analytical workflow

The project follows the same section order and coding style as the ISSS626 teaching materials

1. **Environment setup**

   * Configure R project options and working folders  
   * Install and load all required packages with `pacman::p_load`  
   * Set a fixed random seed for reproducible complete spatial randomness envelopes

2. **Raw data preprocessing for restaurants**

   * Import multiple ACRA CSV extracts using `list.files` and `purrr::map_dfr`  
   * Save the combined raw table as an RDS file for reuse  
   * Filter to SSIC 56111 restaurants, keep the first 24 columns, and standardise column names  
   * Convert incorporation dates to `Date`, derive year, month number, and month abbreviation  
   * Normalise postal codes to six digits with leading zeros and restrict to entities registered in the year 2025  
   * Save the tidy restaurant table as an RDS file

3. **Exploratory data analysis**

   * Inspect structure with `glimpse` to confirm data types and column contents  
   * Count new restaurants by year and by month to confirm that the sample matches the intended study window  
   * Create a monthly bar chart with a trend line for visual checks on short-term dynamics

4. **Geodata preparation and geocoding**

   * Extract unique postal codes and call the OneMap search endpoint through `httr::GET`  
   * Separate matched and unmatched postcodes into two tables for quality assurance  
   * Keep the core OneMap fields such as postal code, X, Y, longitude, latitude, and address text  
   * Join coordinates back to the tidy restaurant table using the postal code as the key  
   * Create an `sf` point layer in SVY21 EPSG code 3414 using the X and Y columns  
   * Save the geocoded dataset as an RDS file

5. **Boundary preparation and quality assurance mapping**

   * Read the URA Master Plan 2019 KML file and convert it to `sf`  
   * Parse the HTML in the Description field with `rvest` to extract the REGION name, the planning area name, the subzone name, and the subzone code  
   * Remove offshore groups such as Southern Group, Western Islands, and North Eastern Islands  
   * Dissolve subzones to planning areas and repair geometry with `st_make_valid`  
   * Build a static quality assurance map using `tmap` that overlays restaurant points on planning areas  
   * Build an interactive map in `tmap` view mode to visually confirm spatial alignment for dense urban cores

6. **Global first-order spatial point pattern analysis**

   * Convert subzones to a spatstat observation window using `as.owin`  
   * Convert restaurant points to a spatstat point pattern using `as.ppp` and clip them to the observation window  
   * Rescale coordinates from metres to kilometres for interpretation  
   * Estimate bandwidths with `bw.diggle`, `bw.CvL`, `bw.scott`, and `bw.ppl` to examine sensitivity to smoothing choice  
   * Compute kernel density surfaces for Diggle bandwidth, a fixed neighbourhood radius, and an adaptive kernel using `adaptive.density`  
   * Convert kernel images to `SpatRaster`, assign SVY21 projection, and map results with `tmap` in a consistent cartographic style

7. **Local first-order analysis by planning area**

   * Select four contrastive planning areas: Orchard, Tampines, Jurong West, and Punggol  
   * Convert each to a separate observation window and clip the point pattern accordingly  
   * Rescale to kilometres and compute KDE surfaces using a consistent bandwidth selector  
   * Compare densities and spatial concentration across these four local study areas

8. **Spatio-temporal kernel density estimation**

   * Attach the month of incorporation as a numeric mark and convert to a planar point pattern with temporal marks  
   * Use `sparr::spattemp.density` to estimate STKDE on a regular grid for months one to seven  
   * Plot monthly slices with aligned colour scales so that maps are comparable  
   * Attach day of year marks, run `sparr::BOOT.spattemp` to obtain data-driven spatial and temporal bandwidths, then recompute STKDE at daily granularity  
   * Interpret the evolution of hotspots through time, paying particular attention to the persistence of central clusters

9. **Second-order spatial point pattern analysis**

   * Focus on selected planning areas such as Tampines and Punggol for deeper interaction analysis  
   * Compute G, F, K, and L functions for each local point pattern, with primary emphasis on K and L  
   * Use `envelope` with a large number of complete spatial randomness simulations and global rank envelopes for rigorous testing  
   * Plot deviation of observed K or L curves from the theoretical expectation and interpret distances where clustering or inhibition is statistically significant

10. **Implications for planning and policy**

    * Summarise the joint evidence from intensity surfaces, spatio-temporal dynamics, and K or L functions  
    * Relate confirmed hotspots and clustering scales to town centre planning, phasing of food outlet licences, and supporting infrastructure such as transit and logistics hubs

---

## Repository structure

The repository is organised as follows. Paths are relative to the project root.

* `data\aspatial`  
  Unmodified input files, such as ACRA CSV tables and URA geospatial layers  

* `data_rds`  
  RDS copies of intermediate objects, including combined ACRA tables, tidy restaurant records, geocoded data, and parsed planning boundaries  

* `data_geospatial`  
  Geospatial vector files used in the analysis, for example, the Master Plan 2019 subzone KML  

* `quarto`  
  Quarto document that knits the complete report with narrative, code, tables, and figures  

Exact file names may vary in your local copy, but the logical structure and flow should remain the same.

---

## Software and R packages

The analysis uses R together with the following package set, loaded through `pacman::p_load`

* `tidyverse` for data wrangling and plotting  
* `stringr` for string operations  
* `lubridate` for date handling  
* `sf` for vector geospatial operations  
* `tmap` for static and interactive mapping  
* `terra` for raster handling and conversion from spatstat images  
* `sparr` for spatio-temporal kernel density estimation  
* `spatstat.geom` and `spatstat` for spatial point pattern analysis  
* `stpp` for further spatio-temporal point process tools  
* `rvest` for HTML parsing when extracting attributes from KML Description  
* `httr` for HTTP requests to OneMap  
* `plotly` for interactive graphics  
* `ggthemes` for additional ggplot styles  

You may also see standard helper packages such as `purrr`, `readr`, and `dplyr` through the tidyverse.

---

## How to run the analysis

1. Clone the repository from GitHub to your local machine.  
2. Open the R project in RStudio.  
3. Download and place the required source data files in the `data/aspatial` and `data/geospatial` folders following the naming conventions used in the quarto file (.qmd).  
4. Create a OneMap account and store your API token securely for use by the geocoding script within the .qmd file.  
5. Run the code chunk in the `.qmd` file in numeric order, or knit the Quarto document if it orchestrates the full workflow.  
6. Inspect intermediate RDS files being created in the 'data/rds' folder to confirm that each stage has completed successfully before moving to subsequent analysis.  

---

## Reproducibility notes

* All random operations, including complete spatial randomness envelopes, are seeded with `set.seed(626)` to ensure reproducible results.  
* The project assumes coordinates are in the SVY21 EPSG code 3414 for all spatial operations.  
* Any changes to folder paths should be made in a single configuration chunk or helper script so that the rest of the code remains unchanged.  
* Raw data is not committed to the public repository due to size and licensing considerations. You may use the documented sources to recreate the raw inputs.

---

## Citation and attribution

If you use this repository for learning or further research, please cite it in the spirit of scholarly work, for example

> Tan C K Take home Ex01 Geospatial Analytics for Public Good ISSS626 School of Computing and Information Systems Singapore Management University 2025

Please also credit the original data providers, ACRA, SingStat, URA, and the Singapore Land Authority OneMap when using their datasets.

---

## Acknowledgements

This project builds directly on the teaching materials and coding patterns developed by Professor Kam Tin Seong for ISSS626. The structure of the analysis, the choice of methods, and the interpretation style closely follow the in-class and hands-on exercises for spatial point pattern analysis and spatio-temporal geospatial analytics.  
