# County Level Corn Yield, Weather, and Soil Dataset


## Table of Contents

1. [Overview of Thesis](#overview-of-thesis)
2. [How to set up environment](#how-to-set-up-environment)
3. [Data Collection](#data-collection)
4. [Notebooks](#notebooks)
5. [Final Dataset](#final-dataset)
6. [What's Next?](#whats-next)


## Overview of Thesis

This project creates a county-level dataset for studying and predicting corn
yield using weather and soil data over the states in the Corn Belt. These states include: Illinois, Indiana, Iowa, Kansas, Kentucky, Michigan, Minnesota, Missouri, Nebraska, North Dakota, Ohio, South Dakota, and Wisconsin.

The dataset combines:

- Corn yield from USDA NASS
- Weather from Daymet
- Soil properties from USDA NRCS SSURGO

The project is being completed as part of my master's degree in Computer
Science at Indiana State University.


## How to set up environment

I am using Anaconda with Jupyter Lab to run my python code. I have a general guide I use to setup an Anaconda evironment everytime I setup a new project:
- [Anaconda Environment Setup for a Jupyter Lab/Notebook Project](./anaconda_env_setup.md). 

I have also added the code below to copy and paste to setup the same environment I am using:

```bash
conda create --name corn_belt_yield python=3.12

conda activate corn_belt_yield

conda install pandas numpy matplotlib seaborn scikit-learn requests python-dotenv jupyterlab ipykernel

python -m ipykernel install --user --name corn_belt_yield --display-name "Corn Belt Yield"
```

This environment contains the Python packages required to collect, clean, combine, visualize, and model county-level corn yield, weather, and soil data.


### USDA NASS API Key

The corn-yield notebook requires a USDA NASS API key.

Create a file named `.env` and add:

```text
USDA_NASS_API_KEY=your_api_key_here
```


## Data Collection


### Corn Belt Yield Data

Corn yield comes from
[USDA NASS Quick Stats](https://www.nass.usda.gov/Quick_Stats/).

The data contain annual county-level corn grain yield measured in bushels per
acre.


### Corn Belt Weather Data

Weather comes from [Daymet](https://daymet.ornl.gov/).

A representative latitude and longitude are used for each county. Daymet
returns daily weather from the nearest 1 km grid cell.

Daily weather is converted into monthly features for:

- Precipitation
- Solar radiation
- Day length
- Snow-water equivalent
- Maximum temperature
- Minimum temperature
- Vapor pressure


### Corn Belt Soil Data

Soil data comes from
[USDA NRCS SSURGO](https://www.nrcs.usda.gov/resources/data-and-reports/soil-survey-geographic-database-ssurgo)
through the
[Soil Data Access service](https://sdmdataaccess.nrcs.usda.gov/Query.aspx).

The soil data include:

- Sand
- Silt
- Clay
- Organic matter
- pH
- Bulk density
- Hydraulic conductivity
- Available water storage
- Slope

Detailed SSURGO records are combined to create one representative soil row for
each county.


#### How the Soil Values Are Calculated

A county contains many soil map units, components, and soil layers.

For soil properties in the upper 30 cm, each record is weighted by:

$$
\text{soil weight}
=
\text{county map-unit acres}
\times
\frac{\text{component percent}}{100}
\times
\text{layer thickness}
$$

The county value is then calculated as:

$$
\text{county average}
=
\frac{
\sum(\text{soil value} \times \text{soil weight})
}{
\sum(\text{soil weight})
}
$$

This gives more influence to soil records that represent a larger portion of
the county.


## Notebooks

Run the notebooks in this order:

| Order | Notebook | Purpose |
|---:|---|---|
| 1 | `county_level_corn_yield.ipynb` | Downloads and cleans corn-yield data |
| 2 | `county_level_weather_daymet.ipynb` | Downloads and summarizes weather data |
| 3 | `county_level_soil.ipynb` | Downloads and summarizes soil data |
| 4 | `corn_belt_analysis.ipynb` | Planned data analysis and machine learning |


## Final Dataset

The final dataset is:

[`corn_belt_yield_weather_soil.csv`](./County%20Corn%20Yield%20Data/corn_belt_yield_weather_soil.csv)


### Dataset Summary

| Item | Value |
|---|---:|
| County-year observations | 24,644 |
| Columns | 106 |
| Counties | 1,124 |
| States | 13 |
| Years | 2000–2025 |
| Monthly weather variables | 84 |
| Soil variables | 9 |
| Missing values | 0 |
| Duplicate county-year records | 0 |

Each row represents one county during one year. The dataset contains county identifiers, annual corn yield, monthly weather measurements, and county-level soil characteristics.


### County and Yield Columns

| Column | Meaning |
|---|---|
| `state_name` | Full state name. |
| `state_alpha` | Two-letter state abbreviation. |
| `state_ansi` | Two-digit ANSI state code. This should be treated as text so leading zeros are preserved. |
| `county_ansi` | Three-digit ANSI county code within the state. This should also be treated as text. |
| `fips` | Five-digit county FIPS code created by combining the state and county ANSI codes. This is the main county identifier. |
| `county_name` | County name. |
| `year` | Year of the corn-yield and weather observation. |
| `short_desc` | USDA NASS description of the yield measurement. |
| `unit_desc` | Unit used for corn yield. All observations use bushels per acre (`BU / ACRE`). |
| `statisticcat_desc` | USDA NASS statistic category. All observations are categorized as `YIELD`. |
| `Value` | Reported county-level corn yield in bushels per acre. This is the target variable for machine-learning models. |
| `latitude` | Representative county latitude in decimal degrees used to retrieve Daymet weather data. |
| `longitude` | Representative county longitude in decimal degrees used to retrieve Daymet weather data. |


### Monthly Weather Columns

Weather columns end with a month number:

| Ending | Month |
|---|---|
| `_m01` | January |
| `_m02` | February |
| `_m03` | March |
| `_m04` | April |
| `_m05` | May |
| `_m06` | June |
| `_m07` | July |
| `_m08` | August |
| `_m09` | September |
| `_m10` | October |
| `_m11` | November |
| `_m12` | December |

For example, `tmax_mean_c_m07` is the mean daily maximum temperature during July.

| Column Pattern | Meaning |
|---|---|
| `prcp_total_mm_m01` through `prcp_total_mm_m12` | Total precipitation during each month, measured in millimeters. Daily precipitation values are summed for the month. |
| `radiation_total_mj_m2_m01` through `radiation_total_mj_m2_m12` | Total incoming shortwave solar energy during each month, measured in megajoules per square meter. |
| `dayl_mean_s_m01` through `dayl_mean_s_m12` | Mean daily daylight duration during each month, measured in seconds per day. |
| `swe_mean_kg_m2_m01` through `swe_mean_kg_m2_m12` | Mean snow water equivalent during each month, measured in kilograms per square meter. This represents the amount of water contained in the snowpack. |
| `tmax_mean_c_m01` through `tmax_mean_c_m12` | Mean daily maximum temperature during each month, measured in degrees Celsius. |
| `tmin_mean_c_m01` through `tmin_mean_c_m12` | Mean daily minimum temperature during each month, measured in degrees Celsius. |
| `vp_mean_pa_m01` through `vp_mean_pa_m12` | Mean atmospheric water vapor pressure during each month, measured in pascals. Higher values generally represent more moisture in the air. |

The weather data comes from the Daymet grid cell nearest to each county's representative latitude and longitude. Therefore, the values represent one location near the center of the county rather than an average across the entire county.


### Soil Columns

The soil columns are county-level weighted averages created from SSURGO soil records. Most properties describe the top 30 cm of soil. Soil values are treated as constant across years for the same county.

| Column | Meaning |
|---|---|
| `sand_pct_0_30cm` | Weighted average percentage of sand in the top 30 cm of soil. |
| `silt_pct_0_30cm` | Weighted average percentage of silt in the top 30 cm of soil. |
| `clay_pct_0_30cm` | Weighted average percentage of clay in the top 30 cm of soil. |
| `organic_matter_pct_0_30cm` | Weighted average percentage of organic matter in the top 30 cm. Organic matter can influence soil fertility, structure, and water retention. |
| `ph_0_30cm` | Weighted average soil pH in the top 30 cm. Lower values indicate more acidic soil, while higher values indicate more alkaline soil. |
| `bulk_density_g_cm3_0_30cm` | Weighted average soil bulk density in grams per cubic centimeter. Higher values generally represent more compact soil. |
| `ksat_um_s_0_30cm` | Weighted average saturated hydraulic conductivity in micrometers per second. This describes how quickly water can move through saturated soil. |
| `available_water_storage_cm_0_100cm` | Weighted average amount of plant-available water that can be stored in the top 100 cm of soil, measured in centimeters of water. |
| `slope_pct` | Weighted average land slope expressed as a percentage. Larger values represent steeper land. |

The soil weighting process accounts for:

1. the amount of each soil map unit that overlaps the county,
2. the percentage of each soil component within the map unit, and
3. the thickness of each soil horizon within the selected depth.

This prevents a small soil component or thin soil horizon from having the same influence as a soil that covers a large portion of the county.


## What's Next?

Since I have collected data and built a dataset, the next step will be to create a final analysis notebook that explores the data, creates visualizations, engineers machine-learning features, and compares
prediction models.