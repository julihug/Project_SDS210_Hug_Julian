# Project_SDS210_Hug_Julian


Full Workflow for the Zurich LST–NDVI Time‑Series Analysis

## **1. Load the required libraries**
importing the tools you need:

- `rasterio` → to read the TIFFs  
- `matplotlib` → to visualize them  
- `numpy` → to compute NDVI, LST, and trends  


---

## **2. Define the list of TIFF files**
# creating a list containing the file paths to all Landsat composites.  
# This ensures the images are loaded in the correct chronological order.

---

## **3. Visually inspect the TIFFs**
# Before doing any calculations, looping through the files and display each image.  

This step confirms:
- all images load correctly  
- they have the same resolution  
- they are aligned  
- no corrupted or missing data  
- the band structure is consistent

# Before performing any calculations, I visually inspected all TIFF files to ensure that the dataset was consistent and suitable for time‑series analysis. Using a simple loop, each image was loaded and displayed individually. This allowed me to verify that all files opened correctly, shared the same spatial resolution, and were properly aligned. The inspection also confirmed that no images contained corrupted pixels, missing data, or unexpected artifacts. Since all TIFFs showed a consistent structure and appearance, the dataset was deemed ready for NDVI, LST, and trend analysis without requiring additional preprocessing.

---

## **4. Load all TIFFs into memory**
Once known the data is consistent, you load all images into a list or a 4D array.  
This creates a **time‑series data cube**:

```
(years, bands, height, width)
```
# After confirming that all TIFF files were consistent and free of errors, the next step was to load the entire dataset into memory. Each image was opened using rasterio and its spectral bands were read into a NumPy array. All yearly arrays were then stacked along a new time dimension, resulting in a 4D data cube with the structure (years, bands, height, width). The stacked dataset contains 14 temporal layers, each corresponding to one year of Landsat observations. The stack itself does not store explicit year labels; instead, the temporal order is preserved through the sequence of file names used during loading. Thus, the first element of the stack represents the earliest year in the dataset, the second element the next year, and so on. This structure allows the data to be processed efficiently as a time series while maintaining a clear link between each array slice and its corresponding year.This unified representation of the dataset forms the basis for all subsequent analyses, including NDVI computation, LST extraction, and temporal trend evaluation. 

# So the dataset contains:
- 14 time steps
- 7 bands per year
- each band is a 252×297 raster
---

## **5. Identify the band order**

# 5.1 Before calculating NDVI and Land Surface Temperature (LST), it was necessary to determine the spectral band order of the Landsat composites. For this purpose, one representative TIFF file was opened with rasterio, and the metadata of the raster bands was inspected. In particular, the band descriptions and the number of bands were examined to identify which indices corresponded to the red, near‑infrared (NIR), and thermal bands. This step ensured that the correct spectral information was used for the NDVI calculation (which requires red and NIR reflectance) and for the extraction of surface temperature from the thermal band. Once the band indices were identified and confirmed to be consistent across all years, they were used as fixed references in the subsequent NDVI and LST computations.

# 5.2 I visualized the bands. - Band 4 (NIR) → vegetation is very bright - Band 3 (Red) → vegetation is medium bright - Band 6 (Thermal) → blurry, low‑resolution look
# To better understand the spectral structure of the dataset, each band of a representative TIFF file was visualized individually. Since the dataset contains seven spectral bands (Blue, Green, Red, NIR, SWIR1, SWIR2, TIR1), the corresponding band names were assigned manually and displayed in the plot titles. This allowed a clear interpretation of the spatial patterns in each spectral range and ensured that the correct bands were used later for NDVI (Red and NIR) and LST (TIR1) calculations.

# 5.3 When previously visually inspecting the TIFF files, each image was displayed in grayscale because only a single spectral band was shown at a time. This is standard practice, as individual bands contain one intensity value per pixel and are therefore represented as grayscale images. The grayscale preview does not indicate that the data is monochromatic; the Landsat composites are multispectral and contain several bands (e.g., red, green, blue, near‑infrared, thermal). Displaying one band per image is sufficient for checking spatial alignment, resolution consistency, and data integrity before performing further analysis. Then the RGB composite was created by selecting the first three spectral bands (Blue, Green, Red) and transposing the array from band‑first to channel‑last format. This ensures each pixel contains three color values corresponding to visible wavelengths, allowing correct visualization as a true‑color image.

# This is essential because NDVI and LST formulas depend on the correct bands.

---

## **6. Compute NDVI for each year**

Using the Red and NIR bands, computing NDVI:

# NDVI was calculated for each year using the red and near‑infrared (NIR) spectral bands of the Landsat composites. Based on the band structure of the dataset (Blue, Green, Red, NIR, SWIR1, SWIR2, TIR1), the red band corresponds to Band 3 and the NIR band to Band 4. For every year in the time series, NDVI was computed using the standard formula

NDVI=\frac{NIR-RED}{NIR+RED}

# The calculation was applied pixel‑wise to all 14 annual images, and the resulting NDVI layers were stored in a new three‑dimensional array with the structure (years, height, width) in the "ndvi_stack". This NDVI time series forms the basis for analyzing vegetation dynamics and long‑term ecological trends in the study area.
# Although NDVI requires two spectral bands (Red and NIR) for its computation, the resulting NDVI image is a single-band product. Therefore, after computing NDVI for each year, the band dimension collapses and the data are stored as 3‑D stacks (years × height × width). The stac is then flattened to align all pixel pairs for further analysis later.

# This gives me the first time‑series array.

# 6.1 A first look at the NDVIs for the first year (1985) and the last year (2024) in the time series was done. For visualization, NDVI maps were displayed using a fixed color scale ranging from –1 to +1. This ensures that all years are directly comparable and that the color representation remains consistent across the entire time series.

# 6.2  NDVI values theoretically range from –1 to +1, but in practice the values observed in the study area (Zürich) fall within a much narrower interval, typically between 0.1 and 0.7. When visualizing NDVI with a fixed color scale of –1 to +1, the effective dynamic range is therefore compressed, which reduces visual contrast between individual years. This is expected behavior and does not indicate a lack of temporal variation. For exploratory analysis, a narrower visualization range (e.g., 0 to 0.8) was used to enhance contrast and reveal spatial patterns more clearly. However, all final NDVI maps presented in the report use the standardized –1 to +1 scale to ensure comparability across years and to maintain consistency with established NDVI interpretation guidelines.

---

## **7. Compute LST for each year**

# Land Surface Temperature (LST) was derived from the thermal infrared band (TIR1) of the Landsat composites. Since the thermal band is provided as brightness temperature (BT) in Kelvin, LST was computed using the standard emissivity‑corrected Planck equation:
LST=\frac{BT}{1+\left( \frac{\lambda \cdot BT}{\rho }\right) \ln (\varepsilon )}
# where \lambda =10.895\, \mu m is the effective wavelength of the Landsat thermal band, \rho =1.438\times 10^{-2}\, mK is the Planck constant term, and \varepsilon =0.97 is the assumed surface emissivity for mixed urban–vegetation environments.

# The computation was applied pixel‑wise to all 14 annual thermal images, resulting in a three‑dimensional LST time series with the structure (years, height, width) and stored in the lst_stack. The resulting LST maps were subsequently converted from Kelvin to degrees Celsius for interpretation and visualization.
# LST is derived from the thermal band. Therefore, after computing LST for each year, the band dimension collapses and the data are stored as 3‑D stacks (years × height × width). The stack is then flattened to align all pixel pairs for later analysis.

# This gives me the second time‑series array.

# 7.1 After computing the annual Land Surface Temperature (LST) layers, each year was visualized to enable spatial interpretation of thermal patterns across the study area. The LST values were first converted from Kelvin to degrees Celsius to facilitate interpretation. For visualization, the inferno colormap was used, as it provides a perceptually uniform gradient that is well suited for temperature data and highlights thermal hotspots clearly. All LST maps were displayed using a consistent color scale across the entire time series to ensure comparability between years. This standardized visualization approach allowed the identification of spatially persistent warm areas, cooler vegetated zones, and potential temporal changes in surface temperature.

---

## **8. Computing yearly statistics**

# For each year, summary statistics were calculated for both NDVI and LST to quantify the overall vegetation and thermal conditions of the study area. The statistics included the mean, minimum, maximum, and standard deviation, computed across all pixels within each annual raster. These metrics provide a compact representation of the temporal evolution of vegetation greenness and surface temperature and serve as a basis for identifying long‑term trends and interannual variability. LST statistics were computed after converting the temperature values from Kelvin to degrees Celsius to facilitate interpretation.
# Each TIFF in the dataset represents a yearly median composite derived from all cloud‑free Landsat acquisitions within that year. Therefore, the dataset contains one LST image per year rather than monthly observations. Although each year is represented by a single composite, the raster still contains tens of thousands of pixel‑level temperature values because of its resolution of 252 × 297 = 74,844 pixels per year. To summarize the thermal conditions for each year in every raster image, descriptive statistics (mean, minimum, maximum, and standard deviation) were computed across all pixels in the annual LST raster. These yearly statistics provide a compact representation of the thermal state of the study area and form the basis for subsequent temporal and trend analyses.

---

## **9. Visualize the time series with the years means**

# To visualize long‑term environmental changes in Zürich, the yearly mean NDVI and LST values were plotted as time series covering the period 1985–2024. The NDVI time series illustrates the evolution of vegetation greenness, while the LST time series shows how surface temperatures have changed over the same period. Both metrics were derived from the annual raster composites by averaging all pixel values within each year.
# The NDVI time series was plotted using a green line to reflect vegetation dynamics, and the LST time series was plotted using a red line to highlight thermal behavior. A consistent temporal axis was used for both variables to enable direct comparison. These visualizations reveal interannual variability as well as long‑term trends, providing an intuitive overview of environmental change in the study area.

# Short first interpretation of the both mean time series: 

# NDVI:
# The NDVI curve shows relatively stable vegetation greenness across the entire time span, with only moderate fluctuations between years. This indicates that the overall vegetation cover in the study area has remained largely consistent, without major long‑term increases or declines. Short‑term variations are visible and likely reflect differences in annual climate conditions, seasonal timing of satellite acquisitions, or small‑scale land‑use changes, but no strong directional trend is apparent.
# LST:
# In contrast, the LST time‑series displays a clearer upward tendency. Although individual years show variability, the overall pattern suggests a gradual warming of surface temperatures in the study area. This aligns with broader regional and global warming trends and may also reflect local urban development, increased impervious surfaces, or reduced evapotranspiration in built‑up zones. The rise in mean LST over time indicates that the urban environment has become progressively warmer, even though vegetation levels remained relatively stable.
# Combined interpretation:
# When viewed together, the two curves suggest that Zürich has experienced a warming trend that is not directly accompanied by a corresponding decline in vegetation greenness. This implies that the observed increase in surface temperature is likely driven more by climatic warming and urbanization processes than by large‑scale vegetation loss.

---

## **10. Create spatial trend and change map from 1985 to 2024 of NDVI and LST**

# 10.1 To identify spatial patterns of long‑term environmental change, I computed pixel‑wise linear trends for both NDVI and LST across the full time series (1985–2024). For each pixel location, a simple linear regression was fitted to its 14 annual NDVI values and 14 annual LST values. The resulting regression slopes represent the yearly rate of change in vegetation and surface temperature.
# The NDVI trend map highlights areas where vegetation cover is increasing or declining, while the LST trend map reveals where surface temperatures are warming or cooling. These spatial trend maps provide geographic context to the statistical NDVI–LST relationship established in Step 10 and help identify neighborhoods experiencing the strongest vegetation loss or surface warming.

# Interpretation of NDVI Trend Map (1985–2024)
# The NDVI trend map illustrates the spatial pattern of vegetation change across the study area over the 39‑year period. Each pixel represents the linear rate of NDVI change per year, derived from annual composites between 1985 and 2024. The predominance of orange and light yellow tones indicates that most areas have experienced little to no significant change in vegetation cover, suggesting stable land‑use conditions. Isolated green patches correspond to zones with positive NDVI trends, reflecting gradual greening—likely due to reforestation, park expansion, or increased vegetation density. Conversely, scattered reddish areas mark negative NDVI trends, indicating vegetation loss or conversion to built‑up surfaces. These localized declines are typically associated with urban densification or infrastructure development. Overall, the map reveals that vegetation cover in Zürich has remained largely stable, with only minor spatial pockets of gain and loss, providing a spatial complement to the statistical NDVI–LST correlation analysis in Step 10.

# Interpretation of LST Trend Map (1985–2024)
# The LST trend map depicts the spatial pattern of long‑term surface temperature change across the study area between 1985 and 2024. Each pixel represents the linear rate of temperature change per year, derived from annual LST composites. The color scale ranges from dark purple (no or minimal change) to bright yellow (strong positive warming trend). The predominance of purple and orange tones indicates that most areas have experienced moderate warming, typically between 0.1 °C and 0.3 °C per year. Isolated yellow patches highlight zones with accelerated temperature increase, often corresponding to dense built‑up or industrial surfaces where heat retention is strongest. In contrast, darker areas with near‑zero slopes mark regions with stable or slightly cooling conditions, likely associated with vegetated or water‑influenced zones. Overall, the map reveals a clear spatial pattern of urban warming, consistent with the urban heat‑island effect, and complements the NDVI trend map by showing that areas with vegetation decline tend to coincide with stronger positive LST trends.

---

## **11. Compare NDVI and LST correlation**

# 11.1 To quantify the relationship between vegetation and surface temperature, a pixel‑wise correlation analysis was conducted using the NDVI and LST stacks. Both datasets were stored as 3‑D arrays (years × height × width). These stacks were flattened into 1‑D arrays so that every pixel from every year formed a single NDVI–LST observation pair.
# LST values were converted from Kelvin to degrees Celsius to ensure interpretability. A validity mask was applied to remove all pixels containing non‑finite values (NaN or ±inf) in either dataset. This step ensures that only spatially aligned and numerically valid pixel pairs are included in the analysis.
# To maintain plot readability, a random subsample of 20,000 valid pixel pairs was selected. A linear regression model was fitted to these data using np.polyfit, which returns the slope and intercept of the best‑fit line describing the NDVI–LST relationship. The regression line was generated by evaluating this model across the observed NDVI range.
# The Pearson correlation coefficient (r) and coefficient of determination (R²) were computed to quantify the strength and explanatory power of the relationship. The resulting scatterplot, combined with the regression line, visualizes the spatial co‑variation between NDVI and LST across all years. A negative slope and negative correlation indicate that higher vegetation density is associated with lower surface temperatures, demonstrating the cooling effect of vegetated areas

# 11.2 NDVI–LST Correlation Map (1985–2024)
# To examine how vegetation and surface temperature interact spatially across Zürich, a pixel‑wise correlation map was computed using the full NDVI and LST time series (1985–2024). For each pixel, the Pearson correlation coefficient r was calculated between its NDVI values and corresponding LST values across all available years. This approach reveals how strongly vegetation dynamics and thermal behavior are linked at the local scale, rather than only at the city‑wide level.
# The resulting map shows clear spatial patterns. Strong negative correlations (blue tones) indicate areas where higher vegetation consistently corresponds to lower surface temperatures, reflecting the cooling effect of parks, forests, and lakeshore vegetation. Weak or near‑zero correlations appear in mixed or transitional land‑use zones, where vegetation cover fluctuates or where surface materials vary. Slightly positive correlations (red tones) occur in dense urban areas or industrial surfaces, where vegetation is sparse and thermal behavior is dominated by built‑up materials rather than vegetation dynamics.
# This spatial correlation map complements the global scatterplot analysis by showing where the NDVI–LST relationship is strongest. It highlights the spatial heterogeneity of the urban heat–vegetation interaction and confirms that the cooling effect of vegetation is not uniform across the city but concentrated in specific green and peri‑urban areas

# 11.3 Creating a Bivariate Map for the NDVI and LST combined**
# The NDVI–LST bivariate map visualizes the spatial relationship between vegetation density and surface temperature across Zürich. NDVI and LST values were each classified into three categories—low, medium, and high—based on quantile thresholds, and then combined to form nine possible NDVI–LST pairings. The resulting categorical scale ranges from cool bare/water surfaces to warm vegetated zones, as shown in the color legend.
# Areas dominated by low NDVI and high LST (hot built‑up zones) correspond to dense urban surfaces and industrial areas, indicating strong heat‑island effects. Conversely, high NDVI and low LST regions (cool vegetated zones) represent parks, forested areas, and lakeshores that maintain lower surface temperatures. Intermediate categories reflect transitional land‑use zones with moderate vegetation and temperature levels.
# This map provides a spatial complement to the NDVI–LST correlation analysis, revealing where the negative relationship between vegetation cover and surface temperature is most pronounced. The categorical color scale directly explains each NDVI–LST combination, enabling intuitive interpretation of how vegetation and built‑up intensity influence local heat patterns.

---

## **12. Interpret the results**


This is where research questions gets answered:

- How much has Zurich warmed?  
- How has vegetation changed?  
- Are heat and vegetation loss correlated?  
- Which areas show the strongest changes?  

---


