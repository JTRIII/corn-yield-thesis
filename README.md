# County-Level Corn Yield Modeling With Weather and Soil Data

**Author:** James T. Roberts III  
**Institution:** Indiana State University  
**Status:** Notebooks 01–06 complete; thesis manuscript in preparation

## Table of Contents

1. [Overview of Thesis](#overview-of-thesis)
2. [Environment Setup](#environment-setup)
3. [Running the Project](#running-the-project)
4. [Data Collection](#data-collection)
5. [Data Sources and Citations](#data-sources-and-citations)
6. [Completed Notebooks](#completed-notebooks)
7. [Final Dataset](#final-dataset)
8. [Feature Selection and Modeling](#feature-selection-and-modeling)
9. [Final Testing Results](#final-testing-results)
10. [Interpretation and Limitations](#interpretation-and-limitations)
11. [Thesis Writing](#thesis-writing)

## Overview of Thesis

This project examines whether selecting a smaller set of weather and soil features can maintain or improve county-level corn-yield predictions, and whether the benefit of feature selection depends on the prediction algorithm.

The completed analysis compares **Pearson correlation, Spearman correlation, and random forest feature importance** across **random forest, linear regression, and neural network regression**. Each algorithm is evaluated with all retained environmental features and with the top 15 environmental features from each selection method. A historical county-average yield baseline provides a reference for the predictions.

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

To rebuild the dataset, run Notebooks **01 → 02 → 03**. Then run **04 → 05 → 06** for exploration, feature rankings, and model evaluation.

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

If `data/processed/corn_belt_yield_weather_soil.csv` is already available, Notebooks 04–06 can run from that file without repeating the API downloads. Notebook 06 recomputes its feature selections and preprocessing within each training stage.

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

The six notebooks form the completed data collection and analysis workflow:

| Order | Notebook | Purpose |
|---:|---|---|
| 1 | [`01_county_level_corn_yield.ipynb`](./notebooks/01_county_level_corn_yield.ipynb) | Retrieves county-level NASS corn yields for 2000–2025, selects the study states, removes combined-county records, and creates county FIPS identifiers. |
| 2 | [`02_county_level_weather_daymet.ipynb`](./notebooks/02_county_level_weather_daymet.ipynb) | Associates counties with coordinates, retrieves daily Daymet weather, creates 84 monthly weather features, and merges weather with county-year yields. |
| 3 | [`03_county_level_soil.ipynb`](./notebooks/03_county_level_soil.ipynb) | Retrieves SSURGO records, computes nine county-level soil properties, and writes the final combined dataset. |
| 4 | [`04_exploratory_data_analysis.ipynb`](./notebooks/04_exploratory_data_analysis.ipynb) | Examines missing values, duplicates, zero yields, county coverage, and the distributions and patterns of yield, weather, and soil variables. |
| 5 | [`05_correlations_and_feature_importance.ipynb`](./notebooks/05_correlations_and_feature_importance.ipynb) | Removes constant environmental features and compares Pearson, Spearman, and random forest feature rankings using 2000–2020 observations. |
| 6 | [`06_model_evaluation_and_final_results.ipynb`](./notebooks/06_model_evaluation_and_final_results.ipynb) | Recomputes feature selection for each training stage, evaluates a baseline and 12 model/feature-set combinations on validation and testing periods, and presents final comparison tables and plots. |

Notebook 04 describes the full 2000–2025 dataset. Notebook 05 provides feature-ranking analysis for 2000–2020. Notebook 06 performs the predictive comparisons with the chronological splits described below.

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

Notebook 06 uses the following periods:

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
| Pearson-selected | 15 | 3 | 18 |
| Spearman-selected | 15 | 3 | 18 |
| RF-importance-selected | 15 | 3 | 18 |

Pearson and Spearman features are ranked by the **absolute magnitude** of their correlation with yield. The random forest method ranks features using the fitted forest's impurity-based `feature_importances_`. The top 15 environmental features are used for each selected set.

### Prediction Models

Each of the four input sets is evaluated with each of the three algorithms, giving **12 model combinations**:

| Algorithm | Implementation and settings used in Notebook 06 |
|---|---|
| Random forest | `RandomForestRegressor(random_state=110, n_jobs=-2)`, using the default 100 trees and other default estimator settings. |
| Linear regression | `LinearRegression()` with its default estimator settings. |
| Neural network | `MLPRegressor(hidden_layer_sizes=(100,), activation="relu", solver="adam", max_iter=1000, early_stopping=False, random_state=110)`. |

Linear regression and the neural networks use predictors standardized with `StandardScaler`. A separate scaler is fitted for each feature set using its training observations, and the same fitted scaler transforms the corresponding validation or testing inputs. Random forest uses unscaled inputs. Yield remains in bushels per acre for all models.

The neural network has one hidden layer with 100 neurons. The `max_iter=1000` setting is a training limit; it does not require every network to run for 1,000 epochs.

### Historical County-Average Baseline

The baseline predicts each county's average yield from the relevant training period. If a county has no training observations, it uses that state's training-period mean; an overall training-period mean is the final fallback. All baseline averages are calculated from training observations.

### Evaluation Metrics

The notebook reports mean absolute error (**MAE**), mean squared error (**MSE**), root mean squared error (**RMSE**), and **R²**. MAE and RMSE are measured in bushels per acre; MSE is measured in squared bushels per squared acre. Lower MAE, MSE, and RMSE indicate smaller prediction errors, and higher R² indicates better performance relative to predicting the evaluation set's mean.

Comparison tables are sorted by RMSE because it retains yield units and gives greater influence to large errors. MAE is also reported because it can rank models differently. Scores are pooled across county-year observations, with equal weight per observation and no harvested-acreage weighting.

## Final Testing Results

The following results are recorded in Notebook 06 after training on **2000–2020** and evaluating on **2021–2025**. All 13 rows use the same 4,164 testing observations. The table is ordered by RMSE and rounded for readability; the notebook retains the underlying calculated values.

| Model | Feature set | Inputs | MAE | MSE | RMSE | R² |
|---|---|---:|---:|---:|---:|---:|
| Neural network | Spearman | 18 | 15.778 | 427.183 | 20.668 | 0.6931 |
| Neural network | Pearson | 18 | 17.018 | 485.112 | 22.025 | 0.6515 |
| Neural network | RF importance | 18 | 16.752 | 492.397 | 22.190 | 0.6463 |
| Random forest | RF importance | 18 | 18.437 | 508.594 | 22.552 | 0.6347 |
| Random forest | Spearman | 18 | 18.408 | 513.687 | 22.665 | 0.6310 |
| Random forest | Pearson | 18 | 18.697 | 528.900 | 22.998 | 0.6201 |
| Random forest | All features | 93 | 20.003 | 594.063 | 24.373 | 0.5733 |
| Neural network | All features | 93 | 19.705 | 614.371 | 24.786 | 0.5587 |
| Linear regression | All features | 93 | 20.271 | 690.944 | 26.286 | 0.5037 |
| Linear regression | RF importance | 18 | 21.148 | 727.029 | 26.963 | 0.4778 |
| Linear regression | Pearson | 18 | 22.827 | 867.419 | 29.452 | 0.3769 |
| Linear regression | Spearman | 18 | 23.202 | 880.836 | 29.679 | 0.3673 |
| Historical county-average baseline | County mean with fallbacks | — | 31.920 | 1276.244 | 35.725 | 0.0833 |

MAE and RMSE are in **bushels per acre**. MSE is in **(bushels per acre)²**. R² is unitless and is not a percentage accuracy score.

### Main Findings

- The **Spearman-selected neural network** has the lowest testing MAE (**15.778**) and RMSE (**20.668**), with **R² = 0.6931**, using 18 inputs.
- All three selection methods improve testing MAE and RMSE for random forest and the neural network relative to their respective all-feature models.
- Linear regression achieves its lowest testing MAE and RMSE with all 93 inputs.
- Within random forest, RF importance selection gives the lowest testing RMSE (**22.552**), while Spearman selection gives the lowest testing MAE (**18.408**).
- Available water storage and summer temperature features rank prominently across the feature-selection methods. Their rankings describe statistical associations and contributions to fitted models.

The lowest validation RMSE was achieved by random forest with Spearman-selected features (**18.778**). The model with the lowest testing RMSE differs, reinforcing that the observed rankings depend on the evaluation period and the fitted models. The central finding is that the usefulness of feature selection depends on the prediction algorithm.

## Thesis Writing

The data collection, exploratory analysis, feature-ranking analysis, and model comparisons in Notebooks 01–06 are complete. The next stage is to write the thesis using the documented methods, saved outputs, and cited data sources.

The thesis will explain data collection and aggregation, the chronological evaluation design, the feature-selection methods, the differences among prediction algorithms, and the interpretation and limitations of the observed results.
