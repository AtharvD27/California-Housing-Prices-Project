# **California Housing Market Analysis (SQL-based)**

## Project Overview

In this project, we aim to perform a comprehensive analysis of the Housing Market in California using the 1990 census data. The data includes features such as geographical location, housing age, number of rooms and bedrooms, household information, and proximity to the ocean. The main objective is to identify patterns and insights that influence house prices across California.

### Dataset Description

The dataset includes the following columns:

- **longitude**: Geographical coordinate (longitude).
- **latitude**: Geographical coordinate (latitude).
- **housing_median_age**: The median age of houses in each district.
- **total_rooms**: Total number of rooms in each district.
- **total_bedrooms**: Total number of bedrooms in each district.
- **population**: Population of the district.
- **households**: Number of households in the district.
- **median_income**: Median income in the district.
- **median_house_value**: Median house value in the district.
- **ocean_proximity**: Distance of houses from the ocean.

### Tools and Disclaimer

This analysis is predominantly based on SQL. We used SQL to clean and analyze the dataset while leveraging Python for visualization. A separate file (`EDA.ipynb`) provides Python code for the Exploratory Data Analysis (EDA), and the visualizations are detailed in `plots.ipynb`.

## Data Cleaning

At the start of the project, we checked for any null or invalid values in the dataset and replaced them with the median of the respective columns to maintain data integrity.

```sql
-- Check for missing or invalid values in the numeric columns
SELECT *
FROM housing_data
WHERE housing_median_age IS NULL OR housing_median_age < 0
   OR total_rooms IS NULL OR total_rooms < 0
   OR total_bedrooms IS NULL OR total_bedrooms < 0
   OR population IS NULL OR population < 0
   OR households IS NULL OR households < 0
   OR median_income IS NULL OR median_income < 0
   OR median_house_value IS NULL OR median_house_value < 0;
```

```sql
-- Fill missing values with the median for each column
WITH median_values AS (
    SELECT
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY housing_median_age) AS median_age,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_rooms) AS median_rooms,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY total_bedrooms) AS median_bedrooms,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY population) AS median_population,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY households) AS median_households,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY median_income) AS median_income,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY median_house_value) AS median_value
    FROM housing_data
)
UPDATE housing_data
SET housing_median_age = COALESCE(housing_median_age, (SELECT median_age FROM median_values)),
    total_rooms = COALESCE(total_rooms, (SELECT median_rooms FROM median_values)),
    total_bedrooms = COALESCE(total_bedrooms, (SELECT median_bedrooms FROM median_values)),
    population = COALESCE(population, (SELECT median_population FROM median_values)),
    households = COALESCE(households, (SELECT median_households FROM median_values)),
    median_income = COALESCE(median_income, (SELECT median_income FROM median_values)),
    median_house_value = COALESCE(median_house_value, (SELECT median_value FROM median_values))
WHERE housing_median_age IS NULL OR total_rooms IS NULL
   OR total_bedrooms IS NULL OR population IS NULL
   OR households IS NULL OR median_income IS NULL
   OR median_house_value IS NULL;
```

## Data Analysis

After cleaning the data, we moved on to the actual analysis. Below are the results of our analysis, ordered in a meaningful progression, with insights that lead us toward a final conclusion.

### 1. Correlation Between Income and House Value

```sql
-- Select income and house value to visualize their correlation
SELECT median_income, median_house_value
FROM housing_data
ORDER BY median_income DESC;
```

![Image](Plots/Hexbin%20Plot%20of%20Median%20Income%20vs%20House%20Value.png)

**Key Insights**:

- A strong positive correlation exists between median income and house prices. Higher income areas are generally associated with higher housing prices.
- The data is skewed towards lower-income and lower house value, suggesting a large portion of the population resides in areas with more affordable housing.
- However, in high-income regions, house prices tend to cap around $500,000, likely due to market saturation or policy-driven price ceilings.

### 2. Income Segmentation and House Prices

```sql
-- Segmenting areas by median income and calculating average house value
SELECT CASE
           WHEN median_income < 2 THEN 'Very Low'
           WHEN median_income >= 2 AND median_income < 3 THEN 'Low'
           WHEN median_income >= 3 AND median_income < 4 THEN 'Median'
           WHEN median_income >= 4 AND median_income < 5 THEN 'High'
           ELSE 'Very High'
       END AS income_category, 
       AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY income_category
ORDER BY avg_house_value DESC;
```

![Image](Plots/Average%20House%20Value%20by%20Income%20Segmentation.png)

**Key Insights**:

- There is a substantial gap in house prices between income categories, with high-income areas having much higher house values.
- The steep increase in house prices for high-income areas highlights potential housing affordability challenges for lower-income groups, as the gap between median and high-income groups is more significant.

### 3. Average House Value by Housing Age Range

```sql
-- Average house value by housing age range
SELECT ROUND(CAST(housing_median_age AS NUMERIC), -1) AS age_group, AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY age_group
ORDER BY avg_house_value DESC;
```

![Image](Plots/Avg%20House%20Value%20by%20Housing%20Age%20Range.png)

**Key Insights**:

- Older houses (50+ years) have higher values, likely due to historical significance or prime location.
- Houses aged 10-20 years tend to have lower values, possibly due to repair needs or less appeal compared to newer constructions.
- Newer and much older homes are valued higher, while moderately aged homes see lower demand.

### 4. Room and Bedroom Counts Impact on House Prices

#### By Bedroom Count

```sql
-- Average house value by number of total bedrooms
SELECT ROUND(CAST(total_bedrooms AS NUMERIC), -1) AS bedroom_group, AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY bedroom_group
ORDER BY avg_house_value DESC;
```

![Image](Plots/Avg%20House%20Value%20by%20Bedroom%20Count.png)

#### By Room Count

```sql
-- Average house value by number of total rooms
SELECT ROUND(CAST(total_rooms AS NUMERIC), -1) AS room_group, AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY room_group
ORDER BY avg_house_value DESC;
```

![Image](Plots/Avg%20House%20Value%20by%20Room%20Count.png)

**Key Insights**:

- Both room and bedroom counts have a direct relationship with house prices, with larger homes generally commanding higher values.
- However, beyond a certain point, adding more rooms or bedrooms leads to diminishing returns in house prices. This trend indicates that buyers are not willing to pay significantly more for homes with a surplus of rooms, possibly due to practical constraints or reduced demand for excessively large properties.

### 5. Average House Value by Ocean Proximity

```sql
-- Average house value by ocean proximity
SELECT ocean_proximity, AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY ocean_proximity
ORDER BY avg_house_value DESC;
```

![Image](Plots/Avg%20House%20Value%20by%20Ocean%20Proximity.png)

**Key Insights**:

- Proximity to water significantly raises house values, with island properties having the highest average prices.
- Inland properties are more affordable, while coastal and ocean-proximate homes command premium prices.
- Location near the ocean is a key driver of high property values.

### 6. Ocean Proximity and House Prices

```sql
-- Average house value by ocean proximity
SELECT ocean_proximity, AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY ocean_proximity
ORDER BY avg_house_value DESC;
```

![Image](Plots/Income%20to%20House%20Price%20Ratio%20by%20Ocean%20Proximity.png)

**Key Insights**:

- Proximity to water bodies, particularly island locations, has a significant impact on house prices, with these properties commanding the highest values.
- Inland properties, by contrast, have considerably lower values, confirming the well-known desirability of oceanfront real estate.

### 7. Z-Score Distribution of House Prices

```sql
-- Detecting price outliers using z-score
WITH price_stats AS (
    SELECT AVG(median_house_value) AS avg_value, 
           STDDEV(median_house_value) AS stddev_value
    FROM housing_data
)
SELECT *, 
       (median_house_value - (SELECT avg_value FROM price_stats)) / 
       (SELECT stddev_value FROM price_stats) AS z_score
FROM housing_data;
```

![Image](Plots/Z-score%20Distribution%20of%20House%20Prices.png)

**Key Insights**:

- The z-score distribution shows that most house prices are close to the mean, with a few outliers (both high and low). Properties with a z-score beyond ±2 can be considered over- or under-valued, which may indicate specific local factors or extreme market behaviors.

### 8. Price Distribution by Location (Longitude and Latitude)

```sql
-- Average house value by latitude and longitude ranges
SELECT ROUND(CAST(longitude AS NUMERIC), 1) AS rounded_longitude, 
       ROUND(CAST(latitude AS NUMERIC), 1) AS rounded_latitude,
       AVG(median_house_value) AS avg_house_value
FROM housing_data
GROUP BY rounded_longitude, rounded_latitude
ORDER BY avg_house_value DESC;
```

![Image](Plots/Price%20Distribution%20by%20Location.png)

**Key Insights**:

- This heatmap visualization highlights the spatial distribution of housing prices, where coastal and urban areas show much higher values than inland or rural regions.
- The closer the properties are to major urban centers and coastlines, the higher the average house values, confirming that location is a key determinant of property prices.

## Conclusion

Through this analysis, it is evident that house prices in California are heavily influenced by several factors:

- **Income Levels:** Higher income strongly correlates with higher house prices. This relationship is particularly stark in high-income areas, highlighting significant housing affordability gaps between different economic segments.

- **House Size:** Larger homes with more rooms and bedrooms command higher prices. However, diminishing returns occur beyond a certain point, indicating that buyers place limits on the premium they are willing to pay for extra space.

- **House Age:** Both very old and very new houses tend to be valued higher, suggesting that historical significance and modern features both drive up property values. In contrast, mid-aged homes are less desirable, perhaps due to maintenance issues or lack of modern amenities.

- **Ocean Proximity:** Properties located near the ocean or on islands have significantly higher house prices, reflecting the desirability of waterfront properties. Inland homes are more affordable, but they lack the market premium associated with ocean views and access.

- **Geographical Distribution:** High house values are concentrated around coastal and urban areas, while inland regions see much lower average prices, highlighting location as a major determinant of property value.
