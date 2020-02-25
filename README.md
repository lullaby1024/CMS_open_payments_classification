# CMS_open_payments_classification
## Introduction
This repository contains source codes and analyses for the 2017 open payments classification task. The overall goal is to predict whether a payment by a company to a medical doctor or facility was made as part of a research project or not.

## Task 1: Identifying Features
- Two datasets (6.8G) were assembled to form the explanatory (`x`) and response (`y`) variables.
  - `x`: 14 features
  - `y`: binary feature, with `1` indicating research payments and `0` otherwise.
- Relevant features were identified for the prediction task. Excluded features include:
  - Features that are highly correlated with `y`
    - Distinct features for each class of `y` (i.e., research/non-research payments)
    - Common features with distinct missing patterns
  - Features with too many missing values (i.e., >50%) as they are not so informative.
  - Features irrelevant for the prediction task
    - Features for ID, physician or company names
    - Features that have (almost) consistent values across all records
    - Redundant geographical features
- The unique categories of the categorical features of general/research payments were also checked. 
  - For the same categorical feature, if there exists a distinct factor level under either of the two labels, that level will be deterministic in predicting the label it belongs to, thus becoming a strongly influential feature in model prediction. This could be seen by the following model fitting part, where the resulted baseline model is nearly perfect.

## Task 2: Preprocessing and Baseline Model
- Preprocessing
  - Cleaning up data types
  - Imputing missing values
  - OneHotEncoding for categorical variables
  - Undersampling for imbalanced dataset
- Baseline model: logistic regression model with initial feature selection

## Task 3: Feature Engineering
- Improved preprocessing
  - Target-based encoding for high-cardinality categorical features (>=10)
  - Scaling for continuous features
  
## Task 4: Any Model
- Four classifiers were experimented with parameters tuned using gridsearch, including
  - SVM
  - Decision Tree
  - Random Forest
  - Gradient Boosting
- The following table presents a summary of the model performances.

| Model  | Preprocessing | Tuning | ROC-AUC | Average Precision | 
| ------------- | ------------- | ------------- | ------------- | ------------- |
| Baseline: Logistic Regression | OneHotEncoder, Imputation | No | 0.986185418669472 | 0.9477666083828806 |
| Improved Baseline: Logistic Regression | OneHotEncoder, TargetEncoder, Imputation, StandardScaler | No | 0.9876607203751065 | 0.9565228823913399 |
| SVM | OneHotEncoder, TargetEncoder, Imputation, StandardScaler | Yes | 0.9899102596909489 | 0.9492819554691636 |
| Decision Tree | OneHotEncoder, TargetEncoder, Imputation, StandardScaler | Yes | 0.9773117605531414 | 0.8035644506506301 |
| Random Forest | OneHotEncoder, TargetEncoder, Imputation, StandardScaler | Yes | 0.9923797314086081 | 0.9542236084333124 |
| Gradient Boosting | OneHotEncoder, TargetEncoder, Imputation, StandardScaler | Yes | 0.994767508942391 | 0.9590947493623678 |
- **Gradient Boosting** has the best performance in terms of ROC-AUC. 
  - Note that it also has the highest average precision among the models we've fitted. However, for specific business insights where both precision and recall should be taken into consideration, the baseline model Logistic Regression gives out the highest f-score compared to all other models. In addition, if we are only considering the the model performance, Random Forest is also a good model with ROC-AUC scores close to those of Gradient Boosting.

## Task 5: Feature Selections
<img src="https://github.com/lullaby1024/CMS_open_payments_classification/blob/master/img/output_56_0.png" width="90%">
<img src="https://github.com/lullaby1024/CMS_open_payments_classification/blob/master/img/output_58_0.png" width="90%">

- Five important features were identified by the Gradient Boosting model, including
  - `Name_of_Drug_or_Biological_or_Device_or_Medical_Supply_1`
  - `Total_Amount_of_Payment_USDollars`
  - `x1_Covered Recipient Physician`
  - `x1_Non-covered Recipient Entity`
  - `Applicable_Manufacturer_or_Applicable_GPO_Making_Payment_State`
- These features are also agreed by Random Forest.
- A refit of the model using only these features returns ROC-AUC of 0.994767508942391 and average precision of 0.9590947493623678, which were both improved.

## Task 6: An Explainable Model
- Finally, an interpretable logistic regression model with 4 out of the 64 features was established. It achieved the same ROC-AUC (0.995) as the best gradient boosting model.
- Feature importance was examined using coefficients:
<img src="https://github.com/lullaby1024/CMS_open_payments_classification/blob/master/img/output_66_0.png" width="90%">

- Notice that `Covered_Recipient_Type_Covered Recipient Physician` and `Covered_Recipient_Type_Non-covered Recipient Entity` come from the same categorical feature and strongly contrast to each other. Since the OneHotEncoder generates one more dummy variables statistically, the coeficient magnitudes are reasonable as shown above. Such a result also proves that the feature `Covered_Recipient_Type` is a strong predictor for this classification.
- The rest festures are consistent to the Gradient Boosting model.
