## 1. Introduction
- Dataset used

    The data used is **SLSTR (Sea Land Surface Temperature Radiometer)** Level 1 Data from **Sentinel 3 satellite**. It has nine spectral bands that detect tops of atmosphere (TOA) radiation in the **VNIR/SWIR/TIR** regions. It provides observations from two different views:

    - **Nadir View**: 1400 km across track
    - **Oblique View**: 740 km across track

    The radiance in visible and near-infrared bands are directly measured by sensors and must be converted to reflectance.

    The data are available for different tracks:

    - **Forward track (f)**
    - **Along track (a)**
    - **Intermediate (i)**
    - **Across (b)**

    This notebook processes the bands (s1-s6) across all the above-mentioned tracks and views.

- Purpose of NDVI analysis

    The **Normalized Difference Vegetation Index (NDVI)** is used to analyze vegetation health by assessing how plants reflect electromagnetic radiation in different wavelengths.  
    Healthy vegetation absorbs most of the visible red light and strongly reflects near-infrared (NIR) light.  Unhealthy vegetation reflects more red light and has lower NIR reflectance.

    Based on SLSTR documentation, for this dataset, S2 and S3 bands are used for vegetation analysis. The formula to calculate NDVI is:

    **NDVI = (NIR - Red) / (NIR + Red)**

    The value ranges from -1 to 1 and the values represent following info:

    * Healthy (> 0.6)
    * Moderate (0.2 - 0.6)
    * Poor Health (0 - 0.2)
    * No vegetation (< 0)

    This index helps in monitoring vegetation health, detecting deforestation, identifying drought conditions, and supporting agricultural planning.

## 2. Methodology

- Data preprocessing steps

    * In data processing, the first step is to load data using xarray. 

    * After loading the data, all the required params **solar_zenith_tn, solar_irradiance and toa_radiance** are read from the respective files.

        * radiance: `S<n>_radiance_<vv>.nc`
        * irradiance: `S<n>_quality_<vv>.nc`
        * solar zenith angle: `geometry_tn`

    * Next step is to calculate TOA Reflectance:
        The reflectance is calculated using the following formula:
        $$
        \text{Reflectance} = \pi \times \frac{\text{ToA Radiance}}{\text{Solar Irradiance} \times \cos(\text{Solar Zenith Angle})}
        $$

        where:
        - **ToA Radiance**: The radiance values from the SLSTR bands.
        - **Solar Irradiance**: Provided in the ‘quality’ dataset for each channel.
        - **Solar Zenith Angle**: Available in the ‘geometry’ dataset.
        - $\pi$ **(Pi)**: A normalization factor.
    * The computed reflectance values for each band are then stored in netCDF files in their respective folders in **output/reflectances** path.

- Dask parallelization approach

    * **Data processing** :  I have applied parallel processing for the reflectance computation task.
    * **NDVI computation** : Similarly, I have also used parallel processing for ndvi computation part for each imagery.

    For both I used the dask's delayed and parallel processing using **'delayed' and 'compute'** feature. 

## 3. Results
- NDVI visualization:

    The visualization are saved in **output/ndvi** folder in netCDF
    format. 
    
    I have used **matplotlib** to plot the images for visualization with required labels for vegetation. The pixel value could be converted to lat lng and displayed but I did not get much time to apply that.

- Optional: Visual interpretation of the NDVI map

    The NDVI visualization indices values ranges from **-1 to 1** and it can identify vegetation and non-vegetation areas. The initial visualization results without normalizing the reflectance values seemed to be inaccurate, with most of the area being coded as red(no vegetation). After I **scaled the reflectance values between 0 to 1**, the vegetation and non vegetation area was correctly identified.

    Comparatively, the first image has large non-vegetation area than 
    the second image.

## 4. Challenges & Lessons Learned

- Issues faced and solutions applied
    * The solar zenith angle values had cosequtive **nan** values in rows and cols, this was causing the radiance data loss.
      * **Soln:** I tried using interpolation but it did not work.  So, I used mean values to fill the nan pixels as the second solution. Then the radiance values were retained.
    * The **radiance and solar zenith** dataset had different spatial resolutions.        
      * **Soln:** I downsampled the radiance data to match solar zenith data resolution.
    * Due to mismatched dimensions of the **'quality'** files, I was not able to load **nc files** to xarray using **xr.open_mfdataset**.
        * **Soln:** Did not have time to solve it. 

## 5. Conclusion

- Summary of insights

* SLSTR level 1 dataset has **S1-S9** bands and **S1-S6** has the radiance information. It has to be corrected by converting to **toa reflectance** to remove cosine effect due to **variations in solar zenith angles**.
* **NDVI** values are very useful indices for vegetation, forests, grass land and other land type monitoring.
* Using a **land cover mask** would help to better identify **non-vegetated land area** with confidence.
* Besides Red and NIR we can use other bands also for various other purposes such as **fire, snow, cloud detection** etc.
