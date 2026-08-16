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
Data Cleaning & Wrangling --> Data Formatting --> Data Review & De-duplication --> Descriptive Analysis --> Exploraory Analysis --> Epidemiological Analysis.

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
With these segmentations, the dataset became 9000 obs and 32 variables, compared to 9000 obs and 29 variables as per the initial dataset

Heart disease rate and BMI by sex and age group
| Age Group          |     Sex       |  Average BMI  | Avg Heart Disease Rate  | Total      |  
|--------------------|---------------|---------------|-------------------------|------------|
|  At_risk           |	   Female    |    25.6       |                         |            |
|  High_risk         |     Female    |   25.4        |                         |            |
|   On_set  	       |     Female    |   25.3        |                         |            |
| Low_baseline       |     FeMale    |   24.9        |                         |            |
|   At_risk          |     Male      |               |                         |            |
|   High_risk        |     Male      |               |                         |            |
|   Onset            |     Male      |               |                         |            |
|   Low_baseline     |     Male      |               |                         |            |


Lipid panel values and heart disease rate by sex and age group

(Insert Lipoprotein table output here — avg_LDL, avg_HDL, avg_cholesterol, avg_triglyceride, PercentHrd by sex and Age_Group)

Chest pain type by age group and sex

(Insert Chestpain table output here — Total_HRD, Avg_HRD by chest_pain_type, Age_Group, sex)

Blood pressure stage and heart disease rate by age group

(Insert Hypertension_Staging table output here — Disease_Rate, Total_Patients by BP_Stage, Age_Group)

Glycemic status, triglyceride level, and heart disease prevalence

(Insert Metabolic_Profiles table output here — Patient_Count, Heart_Disease_Prevalence, Avg_Stress by Glycemic_State, Lipid_Trig_State)

Activity and sleep tier among wearable owners

(Insert Lifestyle_Protection_Matrix table output here — Prevalence, Avg_Stress, Volume by Activity_Tier, Sleep_Tier)

Note: this table currently describes patterns within wearable owners only. A full comparison against non-owners — needed to properly test whether wearable ownership is associated with lower risk — is planned as a next step.

Interpretation

The descriptive layer confirms the dataset behaves as expected for a cardiovascular risk dataset: heart disease prevalence is higher in males than females, and the AHA and ADA-based clinical classifications produced complete, non-overlapping patient groupings with no unclassified cases. These descriptive patterns form the foundation for the exploratory and epidemiological analyses that follow, where specific exposure-outcome relationships are tested f







## References
**World Health Organization (2025). Cardiovascular diseases (CVDs). [online] World Health Organization. Available at: https://www.who.int/news-room/fact-sheets/detail/cardiovascular-diseases-(cvds) 
[Accessed 15 Aug. 2026].**
