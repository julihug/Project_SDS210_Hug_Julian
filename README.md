# Assessing Urban Heat

## Research Question and Overview
This project analyzes long‑term environmental change in Zürich using multi spectral data from Landsat composites from 1985–2024. For each year in the time series of 1985-2024, the Normalized Difference Vegetation Index (NDVI) and Land Surface Temperature (LST) gets calculated and then compiled into a multi-year raster dataset stack (ndvi_stack and lst_stack). These outputs are used to investigate the following research questions:

How drastically has the average land surface temperature changed since the mid-1980s? Is there a measurable correlation between warmer urban areas and lower vegetation or high build-up signals? How do spectral indices help us interpret the physical causes of these heat patterns? Are specific neighborhoods within the city warming faster than others?

## Repository Structure

```bash
Project_SDS210_Hug_Julian/
├── data/
│   └── raw/
├── notebooks/
│   └── Time_Series_Analysis.ipynb
├── outputs/
├── README.md
└── .gitignore
```   

### Folder explanations

* data/raw/      contains the 14 Landsat composite TIFFs (1985–2024).
* notebooks/     contains the main notebook that executes the full analysis.
* outputs/       contains all exported figures (RGB composites, NDVI/LST maps, trend maos, correlation plot)

## Data Sources

14 cloud-free Landsat composites were collected. These composites are arranged in a time series between 1985 and 2024 and were stored as GeoTIFF files in the data/raw directory. Each TIFF contains 7 spectral bands: Blue, Green, Red, NIR, SWIR1, SWIR2, TIR1(thermal infrared) and the surface reflectance. Like this the the Normalized Difference Vegetation Index (NDVI) and Land Surface Temperature (LST) were able to be computed.

## Setup Instructions

1. Create the environment: 
conda env create -sds-env
conda activate sds-env

2. Start JupyterLab:
jupyter lab

3. Place the raw data:
data/raw

4. Execution Order:
notebooks/Time_Series_Analysis

Clone the repository: 

To download the project, run the following commands in your terminal:

```bash
git clone https://github.com/julihug/Project_SDS210_Hug_Julian
cd Project_SDS210_Hug_Julian
```

## Workflow Overview

#### 1. Data Loading & Preparation
1.1 Spectral Band Inspection & RGB Visualization
1.2 NDVI Computation & Visualization
1.3 LST Computation & Visualization
1.4 Yearly Statistics & Time Series
   
#### 2. Trend Analysis
2.1 NDVI Trend
2.2 LST Trend
  
#### 3. Early vs. Late Change Detection
3.1 NDVI Change Map
3.2 LST Change Map

#### 4. NDVI-LST Correlation Analysis
4.1 Scatterplot
4.2 Correlation Map

## Summary for the Workflow

#### 1. Data Loading & Preparation

- Load 14 Landsat composite TIFFs (1985–2024) from data/raw/.
- This creates a list of file paths pointing to each Landsat composite raster that will be processed. Each file represents one year in the time series dataset.
```bash
files = ["./data/raw/Landsat/LandsatComposite_Zurich_1985.tif", ...]
``` 
- Read each file using rasterio and store all 7 spectral bands.
- Build a 4D data stack with shape: (year, band, row, col)
- Define the years covered with years:
```bash
np.arange(1985, 1985 + stack.shape[0])
```
- Extract year labels for indexing and plotting.

#### 1.1 Spectral Band Inspection & RGB Visualization

1.1.1 Spectral Band Analysis

- Open the 2024 TIFF with raster.io.
- Print: Number of bands and the Band descriptions.
- Verify correct band order (Blue to TIR1).
- Visualize all 7 bands in grayscale with a loop that loops from band 1 to 7 and reads the band with src.read(i).
  
1.1.2 Simple RGB Visualization (All Years)

- Create a function using bands 1–3 (Blue, Green, Red), which are the first three bands
- Clip negative values and normalize the image so all pixels fall in the range [0,1].
- Display all 14 years in a 3×5 grid with a loop that loops over all years that seltects the full 7-Band Landsat image for year i:
```bash
arr = stack[i]
```
- This is convertet to a simple RGB composite using simple_rgb(), and display it in a 3×5 subplot grid.

1.1.3 RGB with Percentile Stretch (All Years)

- Use true‑color mapping with function get_rgb: R = NIR (band 4), G = Red (band 3), B = Green (band 2).
- Apply 2–98% percentile stretch with function def stretch(x).
- Display enhanced RGB composites for all years with a loop. It iterates through all 14 years:
```bash
arr = stack[1]. 
```

1.1.4 Reflectance Distribution (2024)

- Select the 2024 image and take the first 3 Bands and rearrange.
```bash
rgb = stack[13][:3].transpose(1, 2, 0).
```
- Clip negative reflectance values:
```bash
rgb = np.clip(rgb, 0, None).
```
- Flatten all pixel values.
- Plot histogram on log scale to inspect reflectance range.

1.1.5 Optimized RGB Composite (2024)

- Extract 2024 image and take first 3 Bands and rearrange:
```bash
year = stack[13], rgb = year[:3].transpose(1, 2, 0).
```
- Create unoptimized RGB composite with rgb_raw, normalizing all values to range [0,1].
- Create optimized RGB composite with rgb_opt.
- Apply 0.5–99.5% stretch according to the values observed in the histogram in step 1.14.
- Apply gamma correction, so image becomes clearer and more natural.
- Export final RGB composite to outputs/.

#### 1.2 NDVI Computation & Visualization

1.2.1 NDVI Computation (All Years)

- Create a 3D empty array with ndvi_stack, because NDVI is singleband product.
- Loop through all 14 years and extract Red (band 3) and NIR (band 4).
- Compute and store it in the ndvi_stack. NDVI formula:
```bash
NDVI=\frac{NIR-Red}{NIR+Red}
```
1.2.2 NDVI Histograms

- Extract NDVI for first and last year to Plot NDVI distributions for: 1985 and 2024.
- Flatten each NDVI image (vals_ndvi_1985 and vals_ndvi_2024) to a 1D array and clean values (remove NaNs).
- Plot the histograms to analyse the data distributions. 

1.2.3 NDVI Maps

- Plot NDVI maps for 1985 and 2024 using a shared color scale, according to the values observed in the histograms in step      1.2.2.

#### 1.3 LST Computation & Visualization

1.3.1 LST Computation (All Years)

- Define physical constants for computation
- Create a 3D array empty stack with lst_stack, because LST is singleband product.
- Loop through all 14 years and extract thermal band (TIR1) and apply the LST correction formula:
```bash
lst = bt / (1 + (lambda_  * bt / rho) * np.log(emissivity))
```
1.3.2 LST Histograms

- Extract LST for first and last year to Plot LST distributions for: 1985 and 2024.
- Flatten each LST image (vals_lst_1985 and vals_lst_2024) to a 1D array and clean values (remove NaNs).
- Plot the histograms to analyse the data distributions. 

1.3.3 LST Maps

- Plot LST maps for 1985 and 2024 using a shared color scale, according to the values observed in the histograms in step     1.3.2.

#### 1.4 Yearly Statistics & Time Series

1.4.1 Computing yearly statistics for NDVI and LST (each year)
- Compute yearly:
- Mean NDVI: the average vegetation greenness, Standard deviation: the variability in vegetation, minimum NDVI: darkest /      least vegetated, maximum NDVI: brightest / densest vegetation.
- Print the values Mean NDVI for years 1985 and 2024.
- Mean LST (converted to °C) with lst_celsius = lst_stack - 273.15: average surface temperature, Standard deviation: thermal   variability, Minimum LST: coolest, Maximimum LST: hottest.
- Print the values Mean LST for years 1985 and 2024.

1.4.2 Visualizing the NDVI mean and LST mean Time Series (1985-2024)

- Generate array of years: years = np.arange(1985, 1985 + ndvi_stack.shape[0]) and create Figure. 
- Plot NDVI and LST mean time series (1985–2024).

#### 2. Trend Analysis

#### 2.1 NDVI Trend

- Define time axis:
```bash
years = np.array([...]), assert ndvi_stack.shape[0] == len(years)
```
- Ensures the number of Years matches the NDVI stack. Because years are not evenly spaced.
- Prepare empty trend map with 2D array, each pixel will store one slope value:
```bash
ndvi_trend = np.full((rows, cols),np.nan) lst_trend  = np.full((rows, cols), np.nan)
```
- For each pixel, extract its NDVI time series with loop over every pixel. For each pixel its 14-year NDVI time series gets extracted.
- Computing the linear trend: First fit a linear regression across all 14 years and store slope change per year (a) in a 2D NDVI trend map. Positive slope: vegetation increasing, Negative slope: vegetation decreasing, Near zero: stable vegetation
- Plot histogram of NDVI trend values to analyse the distribution of the values.

2.1.1 Plotting the NDVI Trend for 1985 to 2024

- Load spatial metadata: Bounds: the geographic extent of the raster, (left, right, bottom, top in projected coordinates), transform: the affine transform
- These values allow to georeference the trend map so the axes show real‑world coordinates (Easting/Northing).
- create figure.
- Display the NDVI trend raster:
- ndvi_trend: the 2D array of NDVI slopes (change per year).
- Red = negative trend (vegetation loss), Yellow = stable, Green = positive trend (vegetation gain):
```bash
cmap="RdYlGn" 
``` 
- Georeferences the image using map coordinates: extent=[...]
- vmin / vmax: These values come from 2.5% and 97.5% percentile, which remove extreme outliers.
- Add coordingate ticks: ticks every 1000 meters, making the map easier to interpret spatially.
- Display it and Layout

#### 2.2 LST Trend

- Same procedure for LST as in step 2.1 for NDVI.
- Compute pixel‑wise LST slope (°C/year).
- Plot histogram of LST trend values to analyse the distribution of the values.

2.2.1 Plotting the LST Trend for 1985 to 2024

- create figure.
- Display the LST trend raster similar to the description in step 2.1.1.
- Display it and Layout.

#### 3. Early vs. Late Change Detection

3.1.1 Plotting the NDVI change map

- Create figure
- Display NDVI change raster:
- ndvi_change: The 2D array containing NDVI differences (late minus early).
- Red = vegetation loss, Yellow = stable, Green = vegetation gain:
```bash
cmap="RdYlGn"
```
- extent=[...]: Georeferences the image using the raster bounds so the axes show real‑world coordinates (Easting/Northing in meters).
- vmin / vmax: These values come from 2.5% and 97.5% percentiles of the NDVI change distribution. They remove extreme outliers and make the map visually interpretable.

#### 3.1 NDVI Change Map
- Select early and late NDVI periods: Early period: 1985–1994 and Late period: 2015–2024. Take the first 7 NDVI images: early period. Take the last 7 NDVI images: late period
- Compute mean NDVI for each period. Collapse each 7‑year block into a single NDVI image. Each pixel now represents the average vegetation in that period
- Subtract early from late to create NDVI change map. Positive values: greening, Negative values: vegetation loss,Near zero: stable vegetation.
- This produces a 2D NDVI change map.
- Plot histogram to see distribution of the values to help identify outliers and choose colorbar limits.

### 3.2 LST Change Map
- Same procedure for LST (converted to °C).
- Compute LST change (late − early).
- Plot histogram to see distribution of the values to help identify outliers and choose colorbar limits.

3.2.1 Plotting LST Change Map

- create figure.
- Display the LST change raster similar to the description in step 2.1.1.
- Display it and Layout.

#### 4. NDVI–LST Correlation Analysis

#### 4.1 Scatterplot

- Flatten NDVI and LST stacks into millions of pixel‑year pairs:
```bash
ndvi_flat = ndvi_stack.flatten()
lst_flat = lst_stack.flatten()
```
- This converts the 3D stacks (years, rows, cols) into 1D arrays. Each element now represents one pixel at one time.
- Convert LST from Kelvin to Celcius
- Remove Invalid values: Remove NaNs and infinities. Ensure regression and correlation work properly. Prints the number of valid pixel pairs.
- Subsample 20,000 points for visualization, because millions of points would make the scatterplot unreadable.
- Fit a linear regression: computes the best fit line. Slope = how much LST changes per NDVI unit. Intercept = baseline temperature.
- Compute:
- Pearson correlation (r): Negative: vegetation cools the surface., Positive: vegetation warms the surface (rare). Near zero: weak relationship. 
- R²: Fraction of LST variation explained by NDVI, Higher = stronger vegetation–temperature link
- Plot NDVI–LST scatterplot.
  
#### 4.2 Correlation Map

- Ensure NDVI and LST stacks match: Both stacks must have shape (years, rows, cols). Guarantees pixel‑wise correlation is valid
- Create empty correlation map: Output is a 2D raster. Each pixel stores one correlation value. Same spatial size as the NDVI/LST images.
- Loop through every pixel: Extract its NDVI time series (14 values). Extract its LST time series (14 values). Remove invalid values (NaN, inf).
- For each pixel, compute correlation between NDVI and LST across all years. Requires at least 3 valid years. Compute the Pearson correlation coefficient (r). Store it in the correlation map. If too few valid values: pixel becomes NaN.
- Interpretation of r:
+ r < 0: NDVI up, LST down (vegetation cools surface)
+ r > 0: NDVI up, LST down (rare, often urban artifacts)
+ r ≈ 0: no meaningful relationship
- Create a 2D correlation map showing spatial NDVI–LST relationships, with vmin=-1, vmax=1 showing full correlation range
and extent=[...]: georeferences the map using real‑world coordinates.




  






