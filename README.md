# County-Level Corn Yield Modeling With Weather and Soil Data

**Author:** James T. Roberts III  
**Institution:** Indiana State University  
**Status:** Notebooks 01–07 complete; thesis manuscript being revised

## Table of Contents

1. [Overview of Thesis](#overview-of-thesis)
2. [Environment Setup](#environment-setup)
3. [Running the Project](#running-the-project)
4. [Data Collection](#data-collection)
5. [Data Sources and Citations](#data-sources-and-citations)
6. [Completed Notebooks](#completed-notebooks)
7. [Final Dataset](#final-dataset)
8. [Feature Selection and Modeling](#feature-selection-and-modeling)
9. [Feature Count Comparison (Notebook 07)](#feature-count-comparison-notebook-07)
10. [Final Testing Results](#final-testing-results)
11. [Interpretation and Limitations](#interpretation-and-limitations)
12. [Thesis Writing](#thesis-writing)

## Overview of Thesis

This project examines whether selecting a smaller set of weather and soil features can maintain or improve county-level corn-yield predictions, and whether the benefit of feature selection depends on the prediction algorithm.

The analysis compares **Pearson correlation, Spearman correlation, and random forest feature importance** across **random forest, linear regression, and neural network regression**. Notebook 06 documents the initial fixed **15-feature** experiment. Notebook 07 extends this work by comparing **1–20 selected environmental features**, choosing one common count from validation data, and reporting updated final-test results for **6 selected features** (plus three context variables). The full environmental set and a historical county-average yield baseline remain as comparisons.

The study combines USDA NASS corn yields, Daymet weather, and USDA NRCS SSURGO soil data for **2000–2025**. The study region consists of 13 states: Illinois, Indiana, Iowa, Kansas, Kentucky, Michigan, Minnesota, Missouri, Nebraska, North Dakota, Ohio, South Dakota, and Wisconsin. This is the project's operational definition of the Corn Belt study region.

The research questions are:

1. Which weather and soil features rank highly under Pearson correlation, Spearman correlation, and random forest importance?
2. Can the selected feature sets maintain or improve predictions for later years compared with using all retained environmental features?
3. Does the effect of feature selection differ among random forest, linear regression, and neural network regression?

Because the inputs include weather from all 12 months of the yield year, the experiment evaluates **retrospective yield estimation using observed weather**. It does not establish an in-season forecast that can be issued before all of those observations are available.

The project is part of my master's degree in Computer Science at Indiana State University.

## Environment Setup

I use Anaconda, Python 3.12, and JupyterLab. Additional setup instructions are available in [Anaconda Environment Setup for a Jupyter Lab/Notebook Project](./docs/environment_setup.md).

```bash
conda create --name corn_belt_yield python=3.12
conda activate corn_belt_yield
conda install pandas numpy matplotlib seaborn scikit-learn requests python-dotenv jupyterlab ipykernel
python -m ipykernel install --user --name corn_belt_yield --display-name "Corn Belt Yield"
```

These commands install the packages used for data collection, preparation, visualization, feature selection, and modeling. They do not pin every package version; matching the original data and software environment is necessary when reproducing exact numerical results.

### USDA NASS API Key

Notebook 01 requires a USDA NASS Quick Stats API key. Create a `.env` file in the repository root containing:

```text
USDA_NASS_API_KEY=your_api_key_here
```

Keep the `.env` file out of version control.

## Running the Project

To rebuild the dataset, run Notebooks **01 → 02 → 03**. Then run **04 → 05 → 06 → 07** for exploration, feature rankings, the original model comparison, and the follow-up feature-count comparison. Notebook 07 repeats the modeling workflow from Notebook 06; it recomputes feature rankings and models rather than loading saved evaluation results from Notebook 06.

Before running the collection notebooks:

1. Add the USDA NASS API key to `.env`.
2. Download the Basic county coordinate CSV from [SimpleMaps](https://simplemaps.com/data/us-counties) and place it at `data/raw/uscounties.csv`. Notebook 02 reads its `county_fips`, `lat`, and `lng` columns.
3. Ensure the data folders exist. From the repository root, run:

```bash
mkdir -p data/raw/daymet_counties data/raw/soil_counties data/processed
jupyter lab
```

Open the notebooks in the `notebooks/` directory and select the **Corn Belt Yield** kernel. Their `../data/...` paths assume the notebook working directory is `notebooks/`.

The weather and soil notebooks save separate county CSVs and skip files that already exist. To refresh a county's source data, its cached file must be replaced or removed before rerunning the download. Data collection requires internet access and the providers' current access requirements.

If `data/processed/corn_belt_yield_weather_soil.csv` is already available, Notebooks 04–07 can run from that file without repeating the API downloads. Notebooks 06 and 07 recompute feature selections and preprocessing within each training stage. **Notebook 07 contains the updated final results used for the current thesis revision**; Notebook 06 is retained as the original 15-feature comparison.

For a final reproducibility check, restart the kernel, run each analysis notebook from top to bottom, and save its outputs. Record the package versions used for the thesis results; for example, export the active environment from the repository root:

```bash
conda env export --no-builds > environment.yml
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

Precipitation and solar energy are summed by month; day length, snow-water equivalent, temperatures, and vapor pressure are averaged by month. Daily solar energy is calculated as `srad × dayl / 1,000,000`, converting daylight-average radiation flux and daylight duration to MJ/m²/day before monthly aggregation.


### Corn Belt Soil Data

Soil data come from
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

Layer thickness is restricted to the portion of each horizon between 0 and 30 cm. Each property uses only records with a nonmissing value and a positive soil weight. For sand, silt, and clay, all three texture values must be present and their sum must be greater than zero; a zero value for one texture fraction can still be retained.

This gives more influence to soil records that represent a larger portion of the county's upper 30 cm.

**Available water storage and slope use a different weighting calculation.** These fields are already summarized by NRCS at the map-unit level. Notebook 03 keeps one record per map unit within each county and weights those values only by the map-unit acres inside that county. Available water storage describes 0–100 cm; the horizon-based properties describe 0–30 cm.

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

Horizon-level soil properties were processed into county-level weighted averages using map-unit area, component percentage, and the thickness of the horizon within 0–30 cm. Available water storage and slope were weighted by county map-unit area only. These calculations and resulting county-level values are derived products and are not original SSURGO measurements.

### County Coordinate Data

Representative county latitude and longitude coordinates were obtained from the Basic version of the SimpleMaps United States Counties Database. Only county identifiers and coordinate fields were used. The records were subset to the 13 Corn Belt states and merged with the yield data using county FIPS codes.

**Citation:**

> SimpleMaps. (2026). *United States Counties Database, Basic version* [Data set]. https://simplemaps.com/data/us-counties

The SimpleMaps Basic database is identified by its provider as licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/). The required attribution and link to the original database are provided above.

### Derived Dataset Notice

The file `data/processed/corn_belt_yield_weather_soil.csv` is a derived dataset produced for this research project. It combines and transforms data from USDA NASS, Daymet, USDA NRCS SSURGO, and SimpleMaps. The inclusion of these sources does not imply that the original data providers endorse this project or its conclusions.

### Modeling and Evaluation References

The model implementations and preprocessing follow the scikit-learn interfaces used in Notebook 06:

- [RandomForestRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.RandomForestRegressor.html)
- [LinearRegression](https://scikit-learn.org/stable/modules/generated/sklearn.linear_model.LinearRegression.html)
- [MLPRegressor](https://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPRegressor.html)
- [StandardScaler](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.StandardScaler.html)

For discussion of RMSE and MAE:

> Hodson, T. O. (2022). Root-mean-square error (RMSE) or mean absolute error (MAE): when to use them or not. *Geoscientific Model Development, 15*, 5481–5487. https://doi.org/10.5194/gmd-15-5481-2022

## Completed Notebooks

The seven notebooks form the completed data collection and analysis workflow:

| Order | Notebook | Purpose |
|---:|---|---|
| 1 | [`01_county_level_corn_yield.ipynb`](./notebooks/01_county_level_corn_yield.ipynb) | Retrieves county-level NASS corn yields for 2000–2025, selects the study states, removes combined-county records, and creates county FIPS identifiers. |
| 2 | [`02_county_level_weather_daymet.ipynb`](./notebooks/02_county_level_weather_daymet.ipynb) | Associates counties with coordinates, retrieves daily Daymet weather, creates 84 monthly weather features, and merges weather with county-year yields. |
| 3 | [`03_county_level_soil.ipynb`](./notebooks/03_county_level_soil.ipynb) | Retrieves SSURGO records, computes nine county-level soil properties, and writes the final combined dataset. |
| 4 | [`04_exploratory_data_analysis.ipynb`](./notebooks/04_exploratory_data_analysis.ipynb) | Examines data quality, coverage, and yield/weather/soil distributions, including exploratory yield normality checks by evaluation period. |
| 5 | [`05_correlations_and_feature_importance.ipynb`](./notebooks/05_correlations_and_feature_importance.ipynb) | Removes constant environmental features and compares Pearson, Spearman, and random forest feature rankings using 2000–2020 observations. |
| 6 | [`06_model_evaluation_and_final_results.ipynb`](./notebooks/06_model_evaluation_and_final_results.ipynb) | Original fixed 15-feature experiment: computes training-only feature rankings and evaluates the baseline and 12 model/feature-set combinations across chronological validation and testing periods. |
| 7 | [`07_feature_count_comparison.ipynb`](./notebooks/07_feature_count_comparison.ipynb) | Follow-up analysis testing selected feature counts 1–20 on 2018–2020 validation data; selects one common count (6), refits rankings and models on 2000–2020, and reports updated 2021–2025 test results. |

Notebook 04 describes the full 2000–2025 dataset. Notebook 05 provides feature-ranking analysis for 2000–2020. Notebook 06 records the initial fixed 15-feature modeling experiment. **Notebook 07 is the updated experiment and source of the final testing results below.**

## Final Dataset

The final dataset is:

[`corn_belt_yield_weather_soil.csv`](./data/processed/corn_belt_yield_weather_soil.csv)


### Dataset Summary

The completed notebooks report the following summary for the dataset used in the experiments:

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
| Reported zero-yield observations retained | 18 |

Each row represents one county during one year. The dataset contains county identifiers, annual corn yield, monthly weather measurements, and county-level soil characteristics. County coverage varies by year, so not every county has all 26 years of observations. The 18 reported zero yields are retained in the analysis.

The 106-column CSV includes identifiers, descriptive fields, and the target as well as potential predictors. It is not a 106-predictor model input. The three constant summer snow-water-equivalent columns remain documented in the dataset and are removed when preparing the feature-ranking and modeling inputs.


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

The weather data come from the Daymet grid cell nearest to each county's representative latitude and longitude. Therefore, the values represent one location near the center of the county rather than an average across the entire county.


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

For the seven horizon-level properties, the soil weighting process accounts for:

1. the amount of each soil map unit that overlaps the county,
2. the percentage of each soil component within the map unit, and
3. the thickness of each soil horizon within the selected depth.

Available water storage and slope use county map-unit area weights only. Soil values represent county aggregates and are reused for every year of that county.

## Feature Selection and Modeling

### Chronological Evaluation

Notebooks 06 and 07 use the following periods:

| Stage | Years | County-year observations | Purpose |
|---|---|---:|---|
| Development training | 2000–2017 | 17,919 | Fit preprocessing, feature rankings, and models for validation comparisons. |
| Validation | 2018–2020 | 2,561 | Compare the model and feature-set combinations. |
| Final training | 2000–2020 | 20,480 | Refit preprocessing, feature rankings, and all model combinations before testing. |
| Testing | 2021–2025 | 4,164 | Report later-year performance of the final fitted models. |

Final training combines the development training and validation observations. The same testing observations are used for every final model and the final baseline.

The variance filter, feature rankings, and scalers are fitted on the relevant training period. The fitted transformations and selected columns are then applied to validation or testing observations. Feature rankings are recomputed when moving from the 2000–2017 training period to the 2000–2020 final training period.

### Feature Sets

The starting environmental inputs contain 84 monthly weather features and nine soil features. A `VarianceThreshold(threshold=0.0)` filter removes three constant columns: `swe_mean_kg_m2_m07`, `swe_mean_kg_m2_m08`, and `swe_mean_kg_m2_m09`. This leaves **90 environmental features**.

Each model also receives the same three context variables: **year, latitude, and longitude**. These variables are appended to each input set and are excluded from the environmental feature rankings.

| Feature set | Environmental features | Context variables | Total model inputs |
|---|---:|---:|---:|
| All retained features | 90 | 3 | 93 |
| Pearson-selected (Notebook 07) | 6 | 3 | 9 |
| Spearman-selected (Notebook 07) | 6 | 3 | 9 |
| RF-importance-selected (Notebook 07) | 6 | 3 | 9 |

Pearson and Spearman features are ranked by the **absolute magnitude** of their correlation with yield. The random forest method ranks features using the fitted forest's impurity-based `feature_importances_`. Notebook 07 selects the top **six** features from each ranking (based on the validation experiment described below), while Notebook 06 originally used 15. Selected feature columns are alphabetically sorted before model fitting to ensure that identical selected subsets are supplied in an identical order.

The final **Pearson and Spearman subsets have the same six features**, although in different ranking orders: July and August maximum temperature, June solar radiation, June and July precipitation, and available water storage (0–100 cm). The final **random forest importance subset** contains July and August maximum temperature, June minimum temperature, June vapor pressure, available water storage, and saturated hydraulic conductivity.

### Prediction Models

Each of the four input sets is evaluated with each of the three algorithms, giving **12 model combinations** in the final evaluation. The feature-count search separately fits 20 × 3 × 3 = **180 validation configurations**:

| Algorithm | Implementation and settings used in Notebook 07 |
|---|---|
| Random forest | `RandomForestRegressor(random_state=110, n_jobs=-2)`, using the default 100 trees and other default estimator settings. |
| Linear regression | `LinearRegression()` with its default estimator settings. |
| Neural network | `MLPRegressor(hidden_layer_sizes=(100,), activation="relu", solver="adam", max_iter=2000, early_stopping=False, random_state=110)`. |

Linear regression and the neural networks use predictors standardized with `StandardScaler`. A separate scaler is fitted for each feature set using its training observations, and the same fitted scaler transforms the corresponding validation or testing inputs. Random forest uses unscaled inputs. Yield remains in bushels per acre for all models.

The neural network has one hidden layer with 100 neurons. In Notebook 07, `max_iter=2000` is an upper training limit; convergence may stop training earlier. Notebook 06 used the earlier 1,000-iteration setting.

### Historical County-Average Baseline

The baseline predicts each county's average yield from the relevant training period. If a county has no training observations, it uses that state's training-period mean; an overall training-period mean is the final fallback. All baseline averages are calculated from training observations.

### Evaluation Metrics

The notebook reports mean absolute error (**MAE**), mean squared error (**MSE**), root mean squared error (**RMSE**), and **R²**. MAE and RMSE are measured in bushels per acre; MSE is measured in squared bushels per squared acre. Lower MAE, MSE, and RMSE indicate smaller prediction errors, and higher R² indicates better performance relative to predicting the evaluation set's mean.

Comparison tables are sorted by RMSE because it retains yield units and gives greater influence to large errors. MAE is also reported because it can rank models differently. Scores are pooled across county-year observations, with equal weight per observation and no harvested-acreage weighting.

## Feature Count Comparison (Notebook 07)

Notebook 07 adds a **follow-up feature-count sensitivity experiment** to justify how many environmental predictors to retain. It tests each integer count from **1 to 20**, for each of the **three ranking methods** and **three prediction models**: 180 validation configurations. It does not use saved model performance from Notebook 06 to compute the search results.

- **Feature ranking and fitting:** 2000–2017 training observations.
- **Feature-count comparison:** 2018–2020 validation observations.
- **Selection rule:** The count with the **lowest unweighted mean validation RMSE** across the nine combinations of ranking method × prediction model. The goal is one shared count, not a separate optimum for each model.
- **Refit and testing:** Recalculate feature rankings and refit models using 2000–2020; evaluate the resulting models on 2021–2025. The final test period is **not used to select the feature count**.

| Selected environmental features | Mean validation RMSE (bu/acre) |
|---:|---:|
| **6** | **23.614** |
| 7 | 23.661 |
| 10 | 23.854 |
| 15 | 23.878 |
| 20 | 24.718 |

Six features produced the smallest mean validation RMSE **among the 20 tested counts**. The difference between 6 and 7 was small (approximately 0.047 bu/acre), and results may differ across individual models. This experiment does **not** establish a universal optimal subset size. The 15-feature model results from Notebook 06 were studied earlier; the count search should therefore be reported transparently as a **follow-up analysis**, not as a model-selection rule established before any test-period results were examined.

For the final six-feature subsets, each model also includes **year, latitude, and longitude**, giving **9 total predictors** per selected model compared with **93** in the full-feature model.

## Final Testing Results

The table below is from **Notebook 07**, after refitting on **2000–2020** and evaluating on **2021–2025** (4,164 county-year observations). It supersedes the earlier **15-feature** final-test table in Notebook 06 for the updated thesis. The models were not selected using test RMSE.

| Model | Feature set | Inputs | MAE | MSE | RMSE | R² |
|---|---|---:|---:|---:|---:|---:|
| Random forest | Pearson | 9 | 18.149 | 495.380 | **22.257** | **0.6442** |
| Random forest | Spearman | 9 | 18.149 | 495.380 | **22.257** | **0.6442** |
| Random forest | RF importance | 9 | 18.322 | 512.408 | 22.636 | 0.6319 |
| Neural network | RF importance | 9 | 17.837 | 528.190 | 22.982 | 0.6206 |
| Neural network | Pearson | 9 | **17.169** | 528.203 | 22.983 | 0.6206 |
| Neural network | Spearman | 9 | **17.169** | 528.203 | 22.983 | 0.6206 |
| Random forest | All features | 93 | 20.003 | 594.063 | 24.373 | 0.5733 |
| Neural network | All features | 93 | 19.705 | 614.371 | 24.786 | 0.5587 |
| Linear regression | All features | 93 | 20.271 | 690.944 | 26.286 | 0.5037 |
| Linear regression | Pearson | 9 | 23.723 | 912.002 | 30.199 | 0.3449 |
| Linear regression | Spearman | 9 | 23.723 | 912.002 | 30.199 | 0.3449 |
| Linear regression | RF importance | 9 | 24.078 | 922.147 | 30.367 | 0.3376 |
| Historical county-average baseline | County mean with fallbacks | — | 31.920 | 1276.244 | 35.725 | 0.0833 |

MAE and RMSE are in **bushels per acre**; MSE is in **(bushels per acre)²**. R² is unitless. All configurations were evaluated on the same final test observations.

### Main Findings

- The **Pearson/Spearman-selected random forest** models tied for the lowest final-test RMSE (**22.257 bu/acre**) and highest reported R² (**0.6442**) with nine total inputs.
- The **Pearson/Spearman-selected neural networks** obtained the lowest final-test MAE (**17.169 bu/acre**) among the reported configurations. The best RMSE and best MAE therefore favor different algorithms.
- The Pearson and Spearman methods selected **identical sets of six environmental variables** for final training; after standardizing column order, corresponding models give identical results. This is **agreement**, not evidence that one correlation method outperformed the other.
- Six-feature selection improved both RMSE and MAE for random forest and neural network models compared with their own full-feature baselines. For linear regression, the full 93-input model remained better.
- High-ranked environmental variables included summer maximum temperature and soil available water storage. These rankings indicate **association or fitted-model importance, not causality**.
- Model comparisons depend on both the **choice of error metric** and the **evaluation period**. A small numerical difference should not be interpreted as a demonstrated statistically significant improvement without a separate uncertainty analysis.

## Interpretation and Limitations

These results support investigating feature ranking and nonlinear prediction algorithms **together**, rather than claiming one model family is universally best. The analysis is a retrospective comparison using **all twelve months of observed weather for each yield year**; it is not an operational pre-harvest forecast. Additional limitations include:

- **Feature-count scope:** Only counts **1–20** were evaluated, and the common count was chosen by the mean validation RMSE over nine configurations. This favors a common practical budget but may not produce each individual model's lowest RMSE.
- **Temporal evaluation:** Training and validation/testing periods are chronological, but the study does **not** implement repeated walk-forward evaluation, so performance stability across many annual training windows has not been established.
- **Prior test examination:** The earlier fixed-15 experiment in Notebook 06 includes 2021–2025 test results that were examined before Notebook 07. Describe Notebook 07 as a follow-up sensitivity analysis, and avoid treating the test set as completely unexamined during all study development.
- **Weather representation:** Each county uses weather from a **single representative Daymet grid cell**, not a county-wide spatial average. Soil values are county-level aggregates treated as static across years.
- **Inputs and models:** Management practices and other potentially relevant information are not modeled; no exhaustive feature-count, model-hyperparameter, or model-randomness optimization was performed. Statistical significance of performance differences was not tested.

Notebook 04 additionally includes an **exploratory distribution and normality assessment of corn yield by the chronological data periods**. Because Shapiro–Wilk tests on large samples can be highly sensitive to deviations from normality (and its reported p-values may be unreliable for samples larger than 5,000), the plots and descriptive summaries should be interpreted alongside, not replaced by, the test statistics. This normality check does **not** establish that regression residuals follow a normal distribution.

## Thesis Writing

The collection, exploratory analysis, feature-ranking, initial model comparison, and updated feature-count analysis in **Notebooks 01–07** are complete. The thesis draft needs to be synchronized with Notebook 07 before submission:

1. **Methodology:** State that a follow-up comparison evaluated feature counts **1–20**, using **mean validation RMSE across nine configurations** to select six features. Explain the chronological training/validation/test periods and the consistent column ordering.
2. **Results:** Replace the earlier **15-feature final comparison** with the **six-feature Notebook 07 results**, and include the feature-count comparison graph/table and final selected-feature lists.
3. **Discussion:** Interpret the model-dependent effects of feature selection, the Pearson/Spearman overlap, and the difference between the lowest **RMSE** model (random forest) and the lowest **MAE** model (neural network).
4. **Limitations and conclusion:** Acknowledge the bounded feature search, the earlier examination of the test period, no walk-forward evaluation, spatial representation of weather, and the retrospective use of full-year weather.
5. **Abstract and introduction:** Update the research design, selected count, results, and contribution to match the finished experiment.

A later research extension could evaluate wider feature budgets, repeated chronological folds, model initialization sensitivity, or in-season prediction using only weather available at the forecast date.
