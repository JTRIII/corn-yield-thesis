# County-Level Corn Yield, Weather, and Soil Dataset


**Author:** James T. Roberts III  
**Institution:** Indiana State University  
**Status:** Master's thesis research in progress


## Table of Contents

1. [Overview of Thesis](#overview-of-thesis)
2. [How to Set Up the Environment](#how-to-set-up-environment)
3. [Data Collection](#data-collection)
4. [Data Sources and Citations](#data-sources-and-citations)
5. [Notebooks](#notebooks)
6. [Final Dataset](#final-dataset)
7. [What's Next?](#whats-next)


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

I use Anaconda and JupyterLab to run the Python code in this project. The following guide explains how to set up the project environment:
- [Anaconda Environment Setup for a Jupyter Lab/Notebook Project](./docs/environment_setup.md). 

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

Weather comes from [Daymet](https://daymet.ornl.gov/) through the [Single Pixel Extraction Tool](https://daymet.ornl.gov/single-pixel-tool-guide).

A representative [latitude and longitude](https://simplemaps.com/data/us-counties) are used for each county. Daymet
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
\text{soil weight} = \text{county map-unit acres} \times \frac{\text{component percent}}{100} \times \text{layer thickness}
$$

The county value is then calculated as:

$$
\text{county average} = \frac{\sum(\text{soil value} \times \text{soil weight})}{\sum(\text{soil weight})}
$$

This gives more influence to soil records that represent a larger portion of
the county.


## Data Sources and Citations

The final dataset is a derived research dataset created by collecting, cleaning, aggregating, and merging information from the sources below. Each original data provider should be cited separately when this dataset is used.

### USDA NASS Corn-Yield Data

Annual county-level corn grain yield data were obtained from the United States Department of Agriculture National Agricultural Statistics Service (USDA NASS) Quick Stats database through the Quick Stats API. The query selected county-level corn grain yield measured in bushels per acre for 2000–2025.

**Citation:**

> United States Department of Agriculture, National Agricultural Statistics Service. (n.d.). *Quick Stats database: County-level corn grain yield, 2000–2025* [Data set]. Accessed September 4, 2026. https://quickstats.nass.usda.gov/

USDA NASS requests acknowledgment when its public-domain information is reproduced or used in another work.

### Daymet Weather Data

Daily weather data were obtained from Daymet Version 4 R1 using the Daymet Single Pixel Extraction Tool. Daily observations were collected for 2000–2025 and converted into monthly precipitation, solar radiation, day length, snow-water equivalent, maximum temperature, minimum temperature, and vapor-pressure features.

**Daymet dataset citation:**

> Thornton, M. M., Shrestha, R., Wei, Y., Thornton, P. E., and Kao, S.-C. (2022). *Daymet: Daily Surface Weather Data on a 1-km Grid for North America, Version 4 R1* (Version 4.1) [Data set]. ORNL Distributed Active Archive Center. https://doi.org/10.3334/ORNLDAAC/2129

**Single Pixel Extraction Tool citation:**

> Thornton, M. M., and Devarakonda, R. (2011). *Daymet Single Pixel Extraction Tool*. ORNL Distributed Active Archive Center, Oak Ridge, Tennessee, USA. https://doi.org/10.3334/ORNLDAAC/2361

Daymet also recommends citing the following publications for its methodology. The radiation and humidity references are particularly relevant because this dataset includes shortwave solar radiation and vapor pressure.

> Thornton, P. E., Shrestha, R., Thornton, M., Kao, S.-C., Wei, Y., and Wilson, B. E. (2021). Gridded daily weather data for North America with comprehensive uncertainty quantification. *Scientific Data, 8*, 190. https://doi.org/10.1038/s41597-021-00973-0

> Thornton, P. E., Running, S. W., and White, M. A. (1997). Generating surfaces of daily meteorological variables over large regions of complex terrain. *Journal of Hydrology, 190*, 214–251. https://doi.org/10.1016/S0022-1694(96)03128-9

> Thornton, P. E., Hasenauer, H., and White, M. A. (2000). Simultaneous estimation of daily solar radiation and humidity from observed temperature and precipitation: An application over complex terrain in Austria. *Agricultural and Forest Meteorology, 104*, 255–271. https://doi.org/10.1016/S0168-1923(00)00170-2

> Thornton, P. E., and Running, S. W. (1999). An improved algorithm for estimating incident daily solar radiation from measurements of temperature, humidity, and precipitation. *Agricultural and Forest Meteorology, 93*, 211–228. https://doi.org/10.1016/S0168-1923(98)00126-9

### USDA NRCS SSURGO Soil Data

Soil information was obtained from the Soil Survey Geographic Database (SSURGO), maintained by the USDA Natural Resources Conservation Service. Tabular records were retrieved through the Soil Data Access REST service and aggregated into county-level soil features.

**Citation:**

> Soil Survey Staff, Natural Resources Conservation Service, United States Department of Agriculture. (n.d.). *Soil Survey Geographic (SSURGO) Database* [Data set]. Soil Data Access. Accessed September 4, 2026. https://sdmdataaccess.nrcs.usda.gov/

The soil records were processed into county-level weighted averages based on map-unit area, component percentage, and soil-horizon thickness. These calculations and resulting county-level values are derived products and are not original SSURGO measurements.

### County Coordinate Data

Representative county latitude and longitude coordinates were obtained from the Basic version of the SimpleMaps United States Counties Database. Only county identifiers and coordinate fields were used. The records were subset to the 13 Corn Belt states and merged with the yield data using county FIPS codes.

**Citation:**

> SimpleMaps. (2026). *United States Counties Database, Basic version* [Data set]. https://simplemaps.com/data/us-counties

The SimpleMaps Basic database is identified by its provider as licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). The required attribution and link to the original database are provided above.

### Derived Dataset Notice

The file `data/processed/corn_belt_yield_weather_soil.csv` is a derived dataset produced for this research project. It combines and transforms data from USDA NASS, Daymet, USDA NRCS SSURGO, and SimpleMaps. The inclusion of these sources does not imply that the original data providers endorse this project or its conclusions.


## Notebooks

Run the notebooks in this order:

| Order | Notebook | Purpose |
|---:|---|---|
| 1 | [`01_county_level_corn_yield.ipynb`](./notebooks/01_county_level_corn_yield.ipynb) | Downloads and cleans corn-yield data |
| 2 | [`02_county_level_weather_daymet.ipynb`](./notebooks/02_county_level_weather_daymet.ipynb) | Downloads and summarizes weather data |
| 3 | [`03_county_level_soil.ipynb`](./notebooks/03_county_level_soil.ipynb) | Downloads and summarizes soil data |
| 4 | `04_corn_belt_analysis.ipynb` | Planned data analysis and machine learning |


## Final Dataset

The final dataset is:

[`corn_belt_yield_weather_soil.csv`](./data/processed/corn_belt_yield_weather_soil.csv)


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

A Thesis direction could be called "County-Level Corn Yield Prediction Using Machine Learning, Weather, and Soil Data". 

Some potential questions:

1. How accurately can machine-learning models predict county-level corn yield?
2. How much do soil variables improve predictions compared with weather alone?
3. How early in the growing season can a reliable prediction be made?
4. Which weather and soil variables have the greatest influence on predictions?

I could use machine learning techniques like Linear Regression, Support Vector Regression, and Random Forest to do some predictions with. 


