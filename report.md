## 1. Introduction
- Dataset used

    The data used is **SLSTR (Sea Land Surface Temperature Radiometer)** Level 1 Data from **Sentinel 3 satellite**. It has nine spectral bands that detect tops of atmosphere (TOA) radiation in the **VNIR/SWIR/TIR** regions. 

- Purpose of NDVI analysis

    The **Normalized Difference Vegetation Index (NDVI)** is used to analyze vegetation health by assessing how plants reflect electromagnetic radiation in different wavelengths.

## 2. Methodology

- Data preprocessing steps

    * In data processing, the first step is to load data using xarray. 
    * After loading the data, all the required params **solar_zenith_tn, solar_irradiance and toa_radiance** are read from the respective files.
    * Next step is to calculate TOA Reflectance.        
    * The computed reflectance values for each band are then stored in netCDF files in their respective folders.

- Dask parallelization approach

    * **Data processing** :  I have applied parallel processing for the reflectance computation task.
    * **NDVI computation** : Similarly, I have also used parallel processing for ndvi computation part for each imagery.

    For both I used the dask's delayed and parallel processing using **'delayed' and 'compute'** feature. 

## 3. Results
- NDVI visualization:

    I have used **matplotlib** to plot the images for visualization with required labels for vegetation.

- Optional: Visual interpretation of the NDVI map

    The NDVI visualization indices values ranges from **-1 to 1** and it can identify vegetation and non-vegetation areas. 

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
