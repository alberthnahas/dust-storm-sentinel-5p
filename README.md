## Detecting Dust Storms using Sentinel-5P UV Aerosol Index


### 1. Introduction

Dust storms are significant meteorological events that can impact air quality, human health, transportation, and climate. Monitoring and detecting these events are crucial for early warning and impact assessment.

Sentinel-5P, part of the Copernicus program, carries the TROPOMI instrument, which provides valuable atmospheric composition data. One of its key products is the **UV Aerosol Index (AER_AI_354_388)**. This index is sensitive to the presence of UV-absorbing aerosols in the atmospheric column, such as desert dust, biomass burning smoke, and volcanic ash. Positive AI values generally indicate the presence of these aerosols, with higher values suggesting a greater abundance or optical thickness.

This Colab notebook demonstrates how to:
* Connect to the `openEO` platform to access Sentinel-5P data from the Copernicus Data Space Ecosystem.
* Retrieve and process daily mean UV Aerosol Index data for selected cities in Oman for a defined period (e.g., the year 2023).
* Analyze the AI time series to identify potential dust events based on different AI thresholds.
* Visualize the data through time series plots with event highlighting, histograms, and seasonal/monthly boxplots.

### 2. Setup

#### 2.1 Install Necessary Libraries
First, we need to install the required Python libraries. If you are running this in a new Colab environment, execute the following cell: 
```openeo xarray matplotlib pandas seaborn netcdf4 h5netcdf```
#### 2.2 Authenticate with openEO (Copernicus Data Space Ecosystem)
To access Sentinel-5P data via openEO, you need an account on the [Copernicus Data Space Ecosystem](https://dataspace.copernicus.eu/).
When you run the ```openeo.connect(...).authenticate_oidc()``` command for the first time, it will typically guide you through an authentication process in your browser.

### 3. Understanding the Script

The Python script is divided into several logical sections:

#### 3.1 Configuration (Section A)
This is where you set the main parameters for your analysis:
* **`cities_coordinates`**: A Python dictionary defining the cities you want to analyze (currently Omani cities) along with their longitude and latitude.
    ```python
    # Example:
    # "Muscat": [58.54, 23.61],
    ```
* **`temporal_extent`**: A list specifying the start and end dates for data retrieval (e.g., `["YYYY-MM-DD", "YYYY-MM-DD"]`).
* **`product_band`**: Defines the specific Sentinel-5P UV Aerosol Index band to be used (`AER_AI_354_388`).
* **`dust_event_thresholds_list`**: A list of Aerosol Index (AI) values. The script will report how many days exceed each of these thresholds. These are also plotted as reference lines on histograms and boxplots.
* **`threshold_yellow_min`, `threshold_orange_min`, `threshold_red_min`**: These AI values define the ranges for color-coding events on the timeseries plot (yellow for AI > 1.0 to ≤ 1.5, orange for AI > 1.5 to ≤ 2.0, and red for AI > 2.0), providing a visual guide to event intensity.

#### 3.2 Data Acquisition (Section B) 
This section handles fetching the data:
* It connects to the Copernicus Data Space Ecosystem using the `openeo` library.
* A data cube (`dataset`) is defined for Sentinel-5P Level 2 data, filtered by your chosen time period, a geographical area around each city (defined by `buffer_degrees`), and the specific AI band.
* **`aggregate_temporal_period(reducer="mean", period="day")`**: This openEO process computes the average AI value for each day within the retrieved data.
* **`aggregate_spatial(reducer="mean", geometries=...)`**: This further averages the daily AI values over the precise point geometry defined for each city, resulting in a single AI value per day for each city.
* **`execute_batch(...)`**: This command sends the entire processing chain (data loading, filtering, and aggregation) as a job to the openEO backend. The backend performs the computation, and the script then downloads the resulting data as a NetCDF file. **Note:** This step can take several minutes for each city, depending on the data volume and backend load.

#### 3.3 Data Processing (Section C) 
Once the data is downloaded for a city:
* The NetCDF file (`.nc`) is loaded using the `xarray` library.
* The script intelligently tries to identify the time coordinate and the AI data variable within the file.
* The relevant data is then converted into a `pandas` DataFrame, which is a convenient format for time series analysis and plotting with `matplotlib` and `seaborn`.

#### 3.4 Plotting and Analysis (Sections D-G) 
For each city, the script generates and saves several analytical plots:
* **Timeseries Plot (Intensity Highlighted)**:
    * Displays daily AI values (grey dots) and a 7-day rolling mean (blue line).
    * Highlights days with AI values in specific ranges using different colors:
        * Yellow: `1.0 < AI <= 1.5`
        * Orange: `1.5 < AI <= 2.0`
        * Red: `AI > 2.0`
    * Includes a text box with key summary statistics (mean, median, max AI) and the count of days exceeding various AI thresholds from `dust_event_thresholds_list`.
* **Histogram**:
    * Shows the frequency distribution of daily AI values.
    * Includes vertical lines indicating the mean, median, and the AI thresholds from `dust_event_thresholds_list` for context.
* **Seasonal Boxplot**:
    * Illustrates the distribution of AI values for each meteorological season (Winter, Spring, Summer, Fall). This helps in identifying seasonal patterns in aerosol activity.
* **Monthly Boxplot**:
    * Similar to the seasonal plot, but breaks down the AI distribution by month, offering a more granular view of temporal patterns.


### 4. Interpreting the Results 

* **Timeseries Plot**:
    * **Grey dots**: Represent the raw daily AI values.
    * **Blue dashed line**: The 7-day rolling mean helps visualize short-to-medium term trends by smoothing out daily noise.
    * **Colored markers** (Yellow, Orange, Red): These provide an immediate visual classification of potential dust event intensity based on the AI value.
        * **Yellow**: AI > 1.0 and ≤ 1.5 (suggests potential light to moderate aerosol events).
        * **Orange**: AI > 1.5 and ≤ 2.0 (suggests moderate to significant aerosol events).
        * **Red**: AI > 2.0 (suggests significant or strong aerosol events).
    * The **statistics box** offers quantitative data: the overall average, median, and maximum AI, plus how many days exceeded specific AI thresholds. This is useful for comparing aerosol load across different periods or cities.

* **Histogram**:
    * This plot shows how often different AI values occurred. A distribution skewed towards higher values might suggest frequent or intense aerosol events.
    * The **vertical lines** for mean, median, and defined thresholds help you see where the bulk of AI values lie in relation to these references.

* **Boxplots (Seasonal and Monthly)**:
    * Each box shows the **interquartile range (IQR)** (from 25th to 75th percentile), with a line inside marking the **median** (50th percentile). Whiskers typically extend to 1.5 times the IQR from the box edges, and points beyond that are often considered outliers.
    * These plots are excellent for identifying which seasons or months consistently show higher AI values (indicating more frequent or intense dust/aerosol activity) or greater variability. The horizontal reference lines for the statistical thresholds add further context.


### 5. Outputs 

For each city processed, the script will save the following files into your Google Colab environment's current working directory:

* **A NetCDF file (`.nc`)**: Contains the daily AI time series data for the city (e.g., `AerosolIndex354_388_2023_Muscat.nc`).
* **Four PNG image files** for the plots:
    * `..._timeseries_intensity_CITY_YEAR.png`
    * `..._histogram_CITY_YEAR.png`
    * `..._seasonal_boxplot_CITY_YEAR.png`
    * `..._monthly_boxplot_CITY_YEAR.png`

You can download these files from the file browser panel on the left side of the Colab interface.


### 6. Further Exploration and Considerations 

* **Validation**: Remember, the Aerosol Index is an *indicator*. For definitive dust storm confirmation and impact assessment, it's highly recommended to correlate high AI events with other data sources:
    * Ground-based PM₂.₅ or PM₁₀ measurements from air quality monitoring stations.
    * Visibility reports from meteorological stations.
    * True-color satellite imagery (e.g., from MODIS, VIIRS, Sentinel-2, or Sentinel-3 SLSTR) to visually confirm the presence and extent of dust plumes.
    * News reports or official advisories from meteorological agencies.
* **Limitations of AI**:
    * **Clouds**: Sentinel-5P AI retrievals are significantly affected by cloud cover. Data may be missing or less reliable on heavily clouded days.
    * **Other Absorbing Aerosols**: While highly sensitive to dust, the UV AI can also be elevated by other UV-absorbing aerosols like smoke from biomass burning or volcanic ash. Regional knowledge and ancillary data are often needed for accurate attribution.
    * **Altitude Information**: AI is a column-integrated product. It indicates the presence of aerosols in the atmospheric column but does not directly provide information on the altitude of the aerosol layer or, crucially, its surface concentration.
* **Customization**:
    * Modify the `cities_coordinates` dictionary to analyze different regions or add more cities.
    * Adjust the `temporal_extent` to study different years, specific dust seasons, or known event periods.
    * Experiment with the various AI `thresholds` in Section A to better align with regional characteristics or your specific definition of a dust event or different intensity levels.
    * Explore other Sentinel-5P products (like NO₂, SO₂, CO) or different openEO collections if relevant to your research.


### 7. Conclusion 

The Sentinel-5P UV Aerosol Index provides a powerful, freely accessible dataset for monitoring and detecting the presence of UV-absorbing aerosols, including desert dust. By leveraging `openEO` for efficient data access and Python for robust analysis and visualization (as demonstrated in this notebook), researchers, air quality forecasters, and public health officials can gain valuable insights into the frequency, intensity, and seasonality of dust events over specific regions. While AI is a strong indicator, always consider integrating complementary data sources for a comprehensive understanding of these complex atmospheric phenomena.
