# Crime Incident Clustering in Bangladesh with K-Means

Unsupervised segmentation of crime incidents in Bangladesh using K-Means. The project combines careful data cleaning (including domain-based validation of impossible values), PCA for dimensionality reduction, silhouette-based selection of the number of clusters, and business-style profiling of each resulting segment.

**Result:** 6,169 incidents were segmented into **2 clusters** (silhouette score **0.51**): a larger non-metropolitan group and a smaller, dense metropolitan group with distinct crime patterns.

## Key Objectives

- **EDA & Data Cleaning:** Remove redundant columns and duplicates, fix inconsistent categories, and detect impossible values.
- **Domain-Based Validation:** Treat physically implausible values as errors instead of keeping them as outliers.
- **Missing Value Handling:** Use context-appropriate strategies per column instead of a single global rule.
- **Feature Engineering:** Encode categorical variables, transform skewed features, scale, and reduce dimensionality with PCA.
- **Modeling & Tuning:** Fit a baseline K-Means, then select the number of clusters using inertia and silhouette analysis.
- **Cluster Profiling:** Describe each cluster so that the segments are interpretable.

## Dataset

Bangladesh crime incident data: 6,574 rows and 26 columns, covering incident timing (month, week, weekday, part of day), location (district, division), weather (precipitation, visibility, heat index, season), regional demographics and infrastructure (population, literacy rate, density, schools, police stations, etc.), and the crime type.

## Data Cleaning & Preprocessing

| Issue | Handling |
|-------|----------|
| Redundant `Unnamed: 0` column (identical to the index) | Dropped |
| 267 duplicate rows | Removed |
| `incident_division` inconsistent due to casing and whitespace | Lowercased and stripped |
| `police_station` negative (1 row) | Row removed |
| `density_per_kmsq` with impossible values (from about 1e-61 to 1e+56) | Kept only 30 to 60,000 people/km², removing 137 rows (about 2%) |
| Missing `incident_month` | Imputed from the median month of the same `incident_week` |
| Missing `part_of_the_day` | New category `unknown` |
| Missing `incident_district` | Mode within the same division |
| Missing `literacy_rate` | KNN Imputer (k = 5) on related, robust-scaled features |

After cleaning, 6,169 incidents remained.

**Feature selection:** Highly correlated or redundant features (male/female population, school, college, park, police station, religious institutions, household size, density, incident week) were dropped to reduce multicollinearity, which K-Means is sensitive to.

**Feature engineering**
- One-hot encoding for part of day, season, weekday, division, district, and crime type
- `log1p` transform on skewed features (`precip`, `total_population`)
- `RobustScaler` for scaling
- PCA to 3 components (selected with the scree plot, eigenvalue criterion)

## Modeling & Tuning

- **Baseline:** K-Means with k = 3, silhouette score 0.4353
- **Tuning:** k from 2 to 10 evaluated with inertia, silhouette score, and silhouette plots

| k | Silhouette |
|---|-----------|
| **2** | **0.5100** |
| 3 | 0.4353 |
| 4 | 0.4082 |
| 5 | 0.4199 |
| 6 | 0.4158 |
| 7 | 0.4169 |
| 8 | 0.4192 |
| 9 | 0.4139 |
| 10 | 0.3769 |

k = 2 was selected: it has the highest silhouette score, and the silhouette plots show misassigned points at k = 3 but not at k = 2.

## Cluster Profiles

| | Cluster 0 | Cluster 1 |
|---|-----------|-----------|
| Size | 4,749 incidents (77%) | 1,420 incidents (23%) |
| Type | Non-metropolitan (rural / semi-urban) | Dense metropolitan |
| Population & density | Small to medium, lower density | Very large, extreme density |
| Literacy & infrastructure | Mid-level literacy, limited facilities | Higher literacy, many schools, colleges, parks, and police stations |
| Dominant crime | Murder | Body found |
| Typical timing | Night, rainy season | Night, concentrated in city centers such as Dhaka |

## Key Findings

- Incidents peak at **night**, and the **rainy season** records the most cases, especially murder and body-found cases.
- Body-found cases stay high in the morning, which suggests incidents occurring at night are often discovered later.
- Daytime hours (noon to evening) have the fewest incidents.
- Dhaka accounts for the largest share of incidents in the dataset.

## Conclusion

This project shows an end-to-end clustering workflow, from data validation to an interpretable segmentation. Cleaning decisions were based on domain logic (for example, removing physically impossible population densities rather than imputing them), and the number of clusters was chosen by comparing silhouette scores and silhouette plots instead of relying on the elbow method alone. The result is a clear two-segment view that separates metropolitan from non-metropolitan incident profiles.

Possible extensions include comparing K-Means with other algorithms (e.g. Gaussian Mixture or DBSCAN), testing the stability of clusters across random seeds, and clustering with fewer location-specific features to look for patterns beyond geography.

## Tech Stack

- **Language:** Python
- **Machine Learning:** scikit-learn (KNNImputer, RobustScaler, OneHotEncoder, PCA, KMeans, silhouette metrics)
- **Data Handling & Visualization:** Pandas, NumPy, Matplotlib, Seaborn
