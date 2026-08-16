# Heart Disease Risk 2026

## Introduction
Cardiovascular disease is a rising cause of concern worldwide. According to the World Health Organization (WHO), there were  19.8 million deaths recorded in 2022 (WHO, 2025). While risk factors point to varying indicators, this data analysis project explores the correlation between various indicators associated with heart disease and will offer clear insight into those who are at risk of developing heart disease and those who are not, potential risk factors, and more.
This is a secondary dataset collected from Kaggle; it contains substantial data on heart risk in 2026 with 9000 rows and 27 columns. 

## My Role
As the Health Data Analyst, I uncover potential risk factors and causes of heart disease among different age groups, provide insights and recommendations that influence decision-making, promote healthy living, and provide health information. This data analytics covered descriptive, exploratory, and epidemiological analyses of heart disease risk for the year 2026.

## Analysis Questions
- What is the rate of heart disease across sex and age group, alongside average BMI?
- How do lipid panel values (LDL, HDL, cholesterol, triglycerides) and heart disease rate vary by sex and age group?
- Does the LDL/HDL ratio predict heart disease better than either value alone?
- How does chest pain type relate to heart disease across age group and sex?
- Does smoking status affect heart disease rate, and by how much relative to never-smokers?
- Is the association between smoking and heart disease statistically real, or could it be due to chance?
- How does heart disease rate vary by family history, further broken down by age group and smoking status?
- What is the independent risk of heart disease based on hereditary factors (family history alone)?
- Which individual clinical/lifestyle variables are most strongly correlated with heart disease across the full variable set?
- How does heart disease rate vary by blood pressure stage (using real AHA clinical thresholds) and age group?
- How does heart disease prevalence vary by glycemic status (diabetic/prediabetic/normal, using ADA thresholds) combined with triglyceride level?
- Among wearable owners specifically, how does heart disease prevalence vary by activity tier and sleep tier?
- Does the mean LDL/HDL ratio differ significantly between people with and without heart disease?

## Data Source 
Kaggle. This dataset is structured as cross-sectional epidemiological data, containing outcome, exposure, and demographic variables suitable for descriptive and inferential epidemiological analysis. 
It is treated as simulated data for analytical practice rather than verified clinical surveillance data.

## Tools
- Google Sheets - Data Preview 
- RStudio - Data Cleaning, Statistical and Epidemiological Analysis
- Tableau - Data Visualization

## Analytical Process
Data Cleaning & Wrangling --> Data Formatting --> Data Review & De-duplication --> Descriptive Analysis --> Exploratory Analysis --> Epidemiological Analysis.

The dataset was accessed via Kaggle and uploaded to Google Sheets for a first preview before ensuring consistency of records. All of these were done before uploading the dataset to RStudio. 

R Packages Used during the analysis:
library(tidyverse)
library(lubridate)
library(dplyr)
library(janitor)
library(skimr)
library(readr)
library(tibble)

## Descriptive Analysis
Before testing for associations or statistical significance, the dataset was first explored descriptively, summarizing heart disease rate and related clinical measures across key demographic and clinical subgroups. This step establishes the baseline patterns that the later exploratory and epidemiological analyses build on.

### Overall sample characteristics
- Total patients:                                    9,000
- Overall heart disease prevalence:                  30.3%, with a total of 2727 
- Heart disease rate by sex: 
  - Male:                                              35.4%  
  - Female:                                            24.7%
- Wearable device owners:
  - Owners:                                        3,985 (44.3%)
  - Non-owners:                                    5,015 (55.7%)

### Feature Engineering: Segmentation with case_when()
Three new categorical variables were engineered from continuous clinical measures, using established clinical thresholds rather than arbitrary cutoffs. This ensures the segmentation reflects real diagnostic standards, not convenience bins.

### Age Group
Patients were grouped into four bands aligned with cardiovascular risk stratification conventions:
```r
HRD_Grouped <- heart_disease_risk_2026 %>% 
  mutate(Age_Group = case_when(
    age >= 18 & age <= 39 ~ "Low_baseline",
    age >= 40 & age <= 59 ~ "On_set",
    age >= 60 & age <= 74 ~ "High_risk",
    age >= 75 & age <= 90 ~ "At_risk",
    TRUE ~ "Other"))
```
- Low_baseline (18–39): reference group, lowest expected baseline risk
- On_set (40–59): the age range where cardiovascular risk typically begins rising meaningfully, and where most clinical risk calculators (e.g., ASCVD: Atherosclerotic cardiovascular disease) start scoring.
- High_risk (60–74): older adult range, higher risk per clinical guidelines.
- At_risk (75–90): oldest group in the dataset.

### Blood Pressure Stage
Patients were classified in accordance with the American Heart Association (AHA) blood pressure staging thresholds:
```r
HRD_Grouped <- HRD_Grouped %>%
  mutate(BP_Stage = case_when(
    resting_bp_systolic >= 140 | resting_bp_diastolic >= 90 ~ "Stage_2_Hypertension",
    (resting_bp_systolic >= 130 & resting_bp_systolic <= 139) | 
      (resting_bp_diastolic >= 80 & resting_bp_diastolic <= 89) ~ "Stage_1_Hypertension",
    resting_bp_systolic >= 120 & resting_bp_systolic <= 129 & resting_bp_diastolic < 80 ~ "Elevated",
    resting_bp_systolic < 120 & resting_bp_diastolic < 80 ~ "Normal",
    TRUE ~ "Isolated_Systolic_Unclassified"))
  ```
This distribution was used across the full sample of 9000. This gave a count of:
                 
| BP Stage             |      Patients |               
|----------------------|---------------|
| Stage 1 Hypertension |	   3,015     |
| Stage 2 Hypertension |     2,527     |
| Normal	             |     2,093     |
| Elevated	           |     1,365     |

 N/B: No patient was recorded for Isolated Systolic Unclassified

### Glycemic State (Diabetes)
The 9000 Patients were classified according to the American Diabetes Association (ADA) diagnostic thresholds for HbA1c and fasting blood glucose. They were segmented as follows:
```r
HRD_Metabolic <- HRD_Grouped %>%
  mutate(
    Glycemic_State = case_when(
      hba1c >= 6.5 | fasting_blood_sugar >= 126 ~ "Diabetic",
      hba1c >= 5.7 | fasting_blood_sugar >= 100 ~ "Prediabetic",
      TRUE ~ "Normal"
    ),
    Lipid_Trig_State = if_else(
      triglycerides >= 150, 
      "High_TG", 
      "Normal_TG"))
```
With these segmentations, the dataset became 9000 obs and 32 variables, compared to 9000 obs and 29 variables in the initial dataset
HRD = Heart Disease

 - Heart disease rate and BMI by sex and age group
   
| Age Group          |     Sex       |  Average BMI  | Avg. HRD Rate           | Total HRD Rate               |  
|--------------------|---------------|---------------|-------------------------|------------------------------|
|  At_risk           |	   Female    |    25.6       | 0.49                    |120                           |
|  High_risk         |     Female    |   25.4        | 0.37                    |460                           |
|   On_set  	       |     Female    |   25.3        | 0.20                    |435                           |
| Low_baseline       |     FeMale    |   24.9        | 0.07                    | 39                           |
|   At_risk          |     Male      |    25.6       | 0.64                    | 180                          |
|   High_risk        |     Male      |   25.3        |  0.49                   | 649                          |
|   Onset            |     Male      |  25.3         |  0.30                   | 750                          |
|   Low_baseline     |     Male      |  25.9         |  0.14                   | 94                           |


### **Lipid panel values and heart disease rate by sex and age group**

<img width="681" height="256" alt="Screenshot (293)" src="https://github.com/user-attachments/assets/65393d80-4de3-4041-9339-82ea016968c3" />

### **Chest pain type by age group and sex**

  - _**Asymptomatic Chest Pain**_

<img width="525" height="234" alt="Screenshot (294)" src="https://github.com/user-attachments/assets/ad9466dc-eda0-4f54-b36c-aaa1819d6667" />

  - _**Atypical Angina**_

<img width="483" height="219" alt="Screenshot (295)" src="https://github.com/user-attachments/assets/ed00d24c-196b-4972-b8ab-bb56f419890b" />

 - _**Non-Anginal Paint**_

<img width="478" height="225" alt="Screenshot (296)" src="https://github.com/user-attachments/assets/5e78347c-5256-4186-9d00-3be25f08b4d1" />

 - _**Typical Angina**_

<img width="491" height="197" alt="Screenshot (297)" src="https://github.com/user-attachments/assets/f7e35c0e-ad17-4c9b-a965-513b5edfcf89" />

### **Blood pressure stage and heart disease rate by age group**

<img width="616" height="449" alt="Screenshot (298)" src="https://github.com/user-attachments/assets/80f1fbf0-c94c-40ef-a343-208eaa92cfee" />

### **Glycemic status, triglyceride level, and heart disease prevalence**

<img width="613" height="176" alt="Screenshot (299)" src="https://github.com/user-attachments/assets/b8d3b8dc-5337-4189-8b54-4463f65be424" />

### **Activity and sleep tier among wearable owners**

<img width="499" height="169" alt="Screenshot (300)" src="https://github.com/user-attachments/assets/5586ab69-3f8e-4690-96e7-5bf7cd7978bc" />


### Interpretation

The descriptive layer confirms the dataset behaves as expected for a cardiovascular risk dataset: heart disease prevalence is higher in males than females, and the AHA and ADA-based clinical classifications produced complete, non-overlapping patient groupings with no unclassified cases. These descriptive patterns form the foundation for the exploratory and epidemiological analyses that follow, where specific exposure-outcome relationships are tested.

## Exploratory Analysis
After establishing the descriptive aspects of the data, the patient data were examined across all available variables to identify the factors that share the strongest relationship to potential heart issues.
All of these were done to get a clear direction for hypothesis testing and exploratory analysis. This is a discovery step: it doesn't confirm causation or statistical significance; it identifies where the strongest signals are, so the epidemiological analysis that follows can focus on the relationships most worth testing rigorously.

### **Correlation Matrix**

For the exploratory analysis, a correlation matrix was built across all numeric clinical and lifestyle variables against has_heart_disease:

```r
numeric_vars <- HRD_Grouped %>% 
  select(age, resting_bp_systolic, resting_bp_diastolic, cholesterol_total, ldl, hdl, 
         triglycerides, resting_heart_rate, ldl_hdl_ratio,
         max_heart_rate_achieved, bmi, exercise_minutes_per_week, 
         daily_steps, sleep_hours, stress_score, has_heart_disease)

cor_matrix <- cor(numeric_vars, use = "complete.obs")


```
With the results from these numeric variables, a heatmap representing the correlation was created for visual interpretation:

```r
library(ggplot2)
library(reshape2)

cor_melted <- melt(cor_matrix)

ggplot(cor_melted, aes(x = Var1, y = Var2, fill = value)) +
  geom_tile() +
  scale_fill_gradient2(low = "blue", mid = "white", high = "red", midpoint = 0) +
  theme(axis.text.x = element_text(angle = 45, hjust = 1)) +
  labs(title = "Correlation Heatmap: Heart Disease Risk Factors",
       x = "", y = "", fill = "Correlation")
```
<img width="779" height="519" alt="Heart Disaease Correlation Heat Map" src="https://github.com/user-attachments/assets/c5111c98-6c6e-425a-b566-79c8ac9b857c" />

Heat Map Interpretation: The red cells indicate a positive correlation (both variables tend to rise together), Blue cells indicate a negative correlation (as one rises, the other falls), and pale/white cells indicate little to no linear relationship. The diagonal is always solid red, since every variable correlates perfectly with itself.

### LDL/HDL Ratio: 

Does the LDL/HDL Outperform Either Measure Alone?

A derived variable — the LDL/HDL ratio — was created to test whether the balance between "bad" and "good" cholesterol carries more information than either measure alone:

```r
HRD_Grouped <- HRD_Grouped %>% 
  mutate(ldl_hdl_ratio = ldl / hdl)

cor(HRD_Grouped$ldl, HRD_Grouped$has_heart_disease)
cor(HRD_Grouped$hdl, HRD_Grouped$has_heart_disease)
cor(HRD_Grouped$ldl_hdl_ratio, HRD_Grouped$has_heart_disease)
```
The Outcome of this syntax: 

| Variable      | Correlation with Heart Disease|
|---------------|-------------------------------|
| LDL alone     | 	 0.267                      |
| HDL alone	    |   -0.252                      |
| LDL/HDL ratio | 	 0.353                      |


The LDL/HDL ratio shows a stronger correlation with heart disease (0.353) than either LDL (0.267) or HDL (-0.252) individually. 
This suggests the balance between these two lipid measures carries more predictive information than either measure in isolation, consistent with established cardiovascular risk literature, where lipid ratios are commonly used for this reason. It also indicates that HDL's negative correlation (-0.252) reflects its role as a protective factor — as HDL rises, heart disease likelihood tends to fall. 
This is consistent with HDL's clinical designation as 'good cholesterol,' distinct from LDL's positive, harmful association.
All three correlations fall in the moderate range (0.25–0.35), meaning each is a real but partial signal, not a strong standalone predictor.

**The Interpretation**

This exploratory analysis confirmed that max_heart_rate_achieved shows the strongest relationship with heart disease of any variable in the dataset (r ≈ -0.58) — a strong negative association, meaning lower max heart rate achieved is strongly linked to higher heart disease likelihood. This is clinically sensible, reflecting reduced cardiovascular capacity as a marker of risk. It also validated that combining LDL and HDL into a ratio adds real analytical value over using either measure alone. These findings directly shaped which relationships were prioritized for formal statistical testing in the epidemiological analysis that follows — smoking status, family history, and the LDL/HDL ratio were each carried forward for significance testing based on the strength and clinical relevance of what this exploratory step revealed.




## References
**World Health Organization (2025). Cardiovascular diseases (CVDs). [online] World Health Organization. Available at: https://www.who.int/news-room/fact-sheets/detail/cardiovascular-diseases-(cvds) 
[Accessed 15 Aug. 2026].**
