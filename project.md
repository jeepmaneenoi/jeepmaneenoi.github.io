<img align="right" width="220" height="220" src="/assets/IMG/template_logo.png">

# Predicting Burned Area in Thailand during March 2024

This project focuses on predicting the burned area in Thailand during March 2024 using Visible Infrared Imaging Radiometer Suite (VIIRS) fire detection sensors from three different satellites: National Oceanic and Atmospheric Administration-20 (NOAA-20), National Oceanic and Atmospheric Administration-21 (NOAA-21), and Suomi National Polar-orbiting Partnership (Suomi NPP or Suomi). The analysis investigates the relationships between burned area, Fire Radiative Power (FRP), and dominant crop types, with the goal of assessing how these factors improve model predictions and comparing the performance of the three satellites.

## 1. Introduction

Fire activity in Thailand, including both forest fire and crop burning, is a significant contributor to air pollution and greenhouse gas emissions. Burned area serves as a critical metric for fire emissions and is widely used in emission inventories for air quality modeling (Wiedinmyer et al., 2023). However, processing burned area data presents challenges due to its high-resolution nature and reliance on satellite retrievals, which involve color-change detection algorithms and often require manual adjustments using GIS tools. This makes burned area computation both time-intensive and resource-demanding (Paluang et al., 2024).

Visible Infrared Imaging Radiometer Suite (VIIRS) sensors provide fire detection data at a resolution of 375m x 375m, with satellites covering the same regions twice daily. These detections are integral to fire emissions inventories, such as the FINNv2.5 Model (FINN version 2.5), which estimates pollutant concentrations, including particulate matter and gaseous emissions (Wiedinmyer et al., 2023). Validating burned area estimations derived from VIIRS data is essential for ensuring the reliability of air quality models.

This study applies supervised machine learning methods—Ridge Regression, Support Vector Regression (SVR), and Decision Tree Regression—to predict burned areas using fire detection data from three VIIRS-equipped satellites: NOAA-20, NOAA-21, and Suomi. It also examines the relationships between Fire Radiative Power (FRP), crop types, and prediction accuracy to evaluate and compare satellite performance. Results show that Ridge Regression was the most effective model for this dataset, demonstrating moderate correlations (r-values: 0.47–0.52) between VIIRS-derived predictors and GISTDA burned areas. In contrast, Decision Tree Regression underperformed, likely due to the dataset’s small size and limited predictors. The negligible impact of the FRP predictor underscores the dominant role of VIIRS-derived burned area in predictions. Additionally, the coarse spatial resolution and lack of consideration for topography and mixed crop influences may have obscured the significance of crop types. These findings highlight the need for improved data inputs and model refinements to enhance predictive accuracy.

## 2. Data Preprocessing

The Geo-Informatics and Space Technology Development Agency (GISTDA), a public organization in Thailand, contributed Landsat-8 OLI/TRI imagery with a 10 m resolution for the burned area product for March 2024 as GIS Shapefile. This dataset serves as the basis for burned area estimation training, which included labeled burned area polygons categorized by crop type: Forest, Maize, Rice, Sugarcane, and General agriculture. This project also utilized active fire detection data from the VIIRS sensor onboard the NOAA-20, NOAA-21 and SUOMI satellites, sourced from NASA's Fire Information for Resource Management System (FIRMS). The data covered March 2024 for an area of interest spanning 84.7°N to -3.5°S and 116.4°E to 33.7°W. The dataset included: fire locations (latitude/longitude), date and time of detections, Fire Radiative Power (FRP) of the fire detections.

### 2.1 Gridded Actual Burned Area and Dominant Crop Type

The GISTDA burned area shapefile served as the foundation for creating a reference burned area dataset for Thailand. The burned area resolution was downscaled by aggregating burned area polygons within a 0.1° latitude/longitude grid, resulting in units of square kilometers (km²) (Fig. 1a). The gridded reference burned areas were transformed using a log10 scale to address data imbalance, as the majority of the grids had values close to zero. Within each grid, the dominant crop type (i.e., the crop covering the largest summed burned areas) was identified and assigned as the representative crop type. This approach simplified integration into predictive models. The resulting map of dominant crop types is shown in Figure 1b.

<img align="center" width="900" height="550" src="/assets/IMG/Fig1.png"> 

### 2.2 Summed Burned Area and Averaged FRP from VIIRS

For each satellite, VIIRS fire detection data for March 2024 was processed by aggregating detections within each 0.1° latitude/longitude grid. The total burned area was estimated by summing detection counts per grid and multiplying by the pixel resolution (375m x 375m) (Fig. 2). These burned areas were converted to square meters (m²) and transformed using a log10 scale to address data imbalance (Fig. 3). 

<img align="center" width="900" src="/assets/IMG/Fig2.png"> 
<img align="center" width="900" src="/assets/IMG/Fig3.png"> 

Similarly, gridded FRP data was aggregated over the same grids and time period, with values averaged rather than summed for each grid (Fig. 4). The averaged FRP data was then converted from megawatts to watts and transformed using a log10 scale to achieve data balance and suitability for model fitting (Fig. 5).

<img align="center" width="900" src="/assets/IMG/Fig4.png"> 
<img align="center" width="900" src="/assets/IMG/Fig5.png"> 

## 3. Modeling

The primary goal of this project is to evaluate different machine learning algorithms to determine the best fit for the dataset as well as comparing satellite performance. Ridge Linear Regression, Support Vector Regression (SVR), and Decision Tree Regression—commonly used for quantitative prediction tasks—were employed to predict burned areas (Section 2.1) using VIIRS-derived burned area data and averaged FRP data (Section 2.2).


### 3.1 Ridge Linear Regression

Ridge Linear Regression is a linear model that is particularly effective when the predictors are highly correlated, as it incorporates a regularization term to penalize large coefficients. Ridge Regression assumes that the linear relationship between the GISTDA burned area and predictors (VIIRS burned area and averaged FRP data). 

### 3.2 Support Vector Regression (SVR)

Support Vector Regression (SVR) is well-suited for capturing non-linear and complex relationships within data. It uses a kernel function to map input features into a higher-dimensional space, allowing it to model non-linear patterns effectively. For small datasets in this study, SVR is particularly advantageous as it focuses on fitting a robust hyperplane while minimizing overfitting, making it ideal for scenarios where data size or variability is limited.

### 3.3 Decision Tree Regression

Decision Tree Regression is a flexible and interpretable model that divides the feature space into regions and predicts outcomes based on these divisions. This approach should work well if burned areas and  VIIRS-derived burned area data and averaged FRP data have complex and non-linear relationships, as it does not assume a specific form for the relationship between predictors and the target variable. However, it can overfit on small datasets.

StandardScaler() was applied to both the target variable (Y) and predictors (X) to normalize the data prior to model fitting, ensuring consistency and improved performance. For each approach, 30% of the dataset was allocated to the testing set, and 70% to the training set and random_state was set to 42 to ensure reproducibility by fixing the randomization process. Each machine learning algorithm was instantiated and trained on the scaled predictors and target. After training, predictions are made on the test set. RMSE is calculated to evaluate each model's performance. Lastly, REC Curve (Relative Error Curve) was computed to help compare models by plotting the percentage of predictions within a given error tolerance.

## 4. Results

<img align="center" width="900" src="/assets/IMG/Fig6.png"> 

## 5. Discussion

### Model performance

Based on the results in section 4, both Ridge Regression and SVR demonstrated comparable performance across all satellites, with Ridge performing slightly better with smallest RMSE for NOAA-21 and SUOMI, while SVR showed marginally better results with smallest RMSE for NOAA-20. Decision Tree Regression consistently exhibited the highest RMSE values and the lowest REC curves, highlighting its limitations for this task. In all models, the predictions improve in accuracy as the tolerance for absolute error increases. By allowing an absolute error of around 3, the percentage of correct predictions reaches 100% across all three models given that the data range is approximately 5 to 8 on a logarithmic scale. However, Ridge Regression and SVR demonstrate higher accuracy at lower error thresholds, achieving nearly 100% correct predictions with an absolute error of 2. In contrast, Decision Tree Regression requires a larger error tolerance of 3 to achieve the same level of accuracy, highlighting its comparatively lower precision. These shortcomings for Decision Tree Regression are likely attributed to the small dataset and the use of only two predictors, which reduce the potential for distinct splits. Consequently, Decision Tree Regression becomes prone to overfitting, leading to less reliable and accurate predictions.

Among the models tested, Ridge Regression emerged as the most effective overall. As a linear model with regularization, it is well-suited to situations where the relationship between predictors and the target variable is linear, as is expected with the GISTDA burned area data and predictors (VIIRS burned area and averaged FRP data). However, for NOAA-20, SVR marginally outperformed Ridge, suggesting that SVR's ability to handle complex patterns and noise may offer advantages in certain cases.

### Comparison of Satellite Performance and Predictor Importance

As discussed earlier, Ridge Regression performed best among all models. Scatter plots were generated to compare predicted burned areas from the Ridge Regression model with GISTDA burned areas, allowing evaluation of the predicted burned areas from each satellite VIIRS-derived data. Moderate positive correlations were observed between the predicted burned area (using VIIRS-derived predictors) and the actual burned area (from GISTDA-referenced data). The r-values ranged from 0.47 to 0.52, with NOAA-21 (0.47), SUOMI (0.51), and NOAA-20 (0.52). These suggest that only approximately 47-52% of the variability in the burned area predictions aligns positively with the observed burned areas, while the remaining variability is unexplained or influenced by other factors. While NOAA-20 shows the best correlation among the satellites tested, the moderate r-value reflects room for significant improvement in the predictive model and data inputs. Furthermore, across all satellite datasets, the FRP term appears to have minimal influence, as its coefficient is consistently close to zero. This indicates that the VIIRS-derived burned area is the dominant predictor in the model.

The scatter plots from ridge regression model, color-coded by crop type, show no clear patterns linking specific crop types to systematic underprediction or overprediction of burned areas. This indicates that the model does not consistently overestimate or underestimate burned areas for any particular crop type. While crop type is potentially relevant for understanding fire activity, it does not appear to be a significant standalone factor for prediction accuracy within the current model framework. This unclear relationship may be due in part to the resolution simplification to 0.1-degree latitude and longitude grids, which might be too coarse to capture finer spatial details. Additionally, the analysis did not account for topographical influences, such as mountainous terrain, or the potential mixed influence of neighboring crop types within adjacent grids.

A key limitation of this project is its reliance on the GISTDA burned area dataset, which provides monthly data aggregated from the start to the end of each month without capturing dynamic changes in fire activity between these points. While the VIIRS-derived satellite data aggregates detections from twice-daily observations over a month, limitations such as cloud cover obscuring active fires and algorithmic errors in detection could lead to underpredictions in some cases. These factors highlight the challenges of accurately modeling and predicting burned areas using the available datasets.

## Conclusion

From this study, the following conclusions can be drawn:
* For this dataset, Ridge Regression and SVR showed similar performance across satellites, while Decision Tree Regression consistently performed worse, likely due to limitations from having a small dataset with only two predictors. Overall, Ridge Regression emerged as the most effective model, leveraging its regularization capabilities to handle linear relationships between predictors and burned area.
* Ridge Regression provided moderate positive correlations (r-values: 0.47–0.52) between VIIRS-derived predictors and GISTDA burned areas, meaning only 47-52% of variability was explained, leaving room for improvement.
* The FRP predictor had minimal impact, as its coefficient was near zero across models, making VIIRS-derived burned area the dominant predictor.
* No systematic trends of overprediction or underprediction for specific crop types was observed.
The coarse resolution (0.1° grids) and lack of accounting for topographical and mixed crop type influences may obscure the potential significance of crop type on prediction accuracy.


## Works cited

Paluang, P., Thavorntam, W., & Phairuang, W. (2024). The spatial–temporal emission of air pollutants from biomass burning during haze episodes in northern Thailand. Fire, 7(4), 122. https://doi.org/10.3390/fire7040122

Wiedinmyer, C., Kimura, Y., McDonald-Buller, E. C., Emmons, L. K., Buchholz, R. R., Tang, W., Seto, K., Joseph, M. B., Barsanti, K. C., Carlton, A. G., & Yokelson, R. (2023). The Fire Inventory from NCAR version 2.5: An updated global fire emissions model for climate and chemistry applications. Geoscientific Model Development, 16(13), 3873–3891. https://doi.org/10.5194/gmd-16-3873-2023


***


