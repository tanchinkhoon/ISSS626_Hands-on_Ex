# Take-home Ex02: Bus Mobility and Urban Trends

This folder contains the Quarto source, data, and outputs for Take-home Exercise 02 of ISSS626 Geospatial Analytics and Applications. The project examines how public bus trips vary across space and time in Singapore. It uses local spatial statistics to reveal neighbourhood clusters of high and low demand together with their temporal evolution.

## Learning goals

* Build a mainland hexagon grid that serves as regular traffic analysis zones for Singapore, and aggregate hourly bus origin trips into these cells.
* Apply global and local measures of spatial autocorrelation, such as Moran's I and Gi star, to detect clusters, outliers, and spatial patterns in bus ridership.
* Use emerging hot spot analysis to study how hot and cold areas of bus activity evolve between weekday morning, weekday evening, and weekend periods.
* Translate numerical outputs into publication-ready maps and figures that support transport planning discussions.

## Data

The analysis draws on three main inputs:

1. Bus stop point locations from the Singapore Land Transport Authority open data catalogue.
2. Master Plan 2019 planning subzone boundary without sea from the national open data portal.
3. Passenger volume by origin-destination bus stops table, with hourly trip counts for weekday and weekend or holiday conditions.

In this repository, these files are expected in the following folders:

* `data/geospatial` for the bus stop and planning boundary layers in shapefile or KML format.
* `data/aspatial` for the passenger origin-destination table in CSV format.

Raw data must be downloaded directly from the official providers in accordance with their terms of use.

## Repository content

* Quarto document that reproduces the full analytical report for Take-home Exercise 02.
* Figure files exported from the Quarto document, including selected peak period maps and hot spot outputs used in the written report.
* Data folders that contain the cleaned working copies of the official input data sets.
* This README file.

## Analytical workflow

The Quarto document follows the structure required in the exercise brief:

1. Define reproducibility settings and load all required R packages such as tidyverse, sf, sfdep, tmap, purrr, knitr, kableExtra, DT, plotly, and Kendall.
2. Import bus stops, planning boundaries, and passenger tables, project all spatial layers to the SVY21 coordinate reference system, and keep only the mainland extent.
3. Construct a regular hexagon grid, validate its geometry, and retain only active cells that intersect the mainland planning boundary.
4. Link passenger records to bus stops, attach them to hexagon cells, then create a balanced panel of origin trips by cell, day type, and hour of day.
5. Derive peak period totals for weekday morning, weekday evening, and weekend or holiday time windows and map their intensity with shared quantile classes.
6. Build spatial weights, compute global Moran I for each period, and generate local Moran I cluster maps and Moran scatterplots for the hexagon totals.
7. Compute local Gi star statistics and visualise their z scores with diverging colour schemes to highlight high and low concentration areas.
8. Run emerging hot spot analysis using the Gi star results and Mann Kendall trend testing to classify each cell, such as persistent, intensifying, diminishing, or sporadic hot and cold spots.
9. Summarise key spatial and temporal patterns and discuss planning implications for network design, resource allocation, and equity of access.

## How to run

1. Open the project in RStudio.
2. Ensure that all required packages listed in the Quarto document are installed.
3. Place the downloaded data sets in the `data/geospatial` and `data/aspatial` folders using the expected file names.
4. Render the main Quarto document to HTML using the Knit or Render button or the `quarto render` command.

The rendered document will reproduce all tables, maps, and interpretive text used in the written submission.

## Acknowledgements

This exercise is completed as part of ISSS626 Geospatial Analytics and Applications at the School of Computing and Information Systems, Singapore Management University. The project builds on in-class materials on local measures of spatial autocorrelation and emerging hot spot analysis, together with public transport data from the Singapore Land Transport Authority and national open data portals.
