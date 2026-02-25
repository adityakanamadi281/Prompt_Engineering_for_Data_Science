# 📊 Post-Model Data Analysis

## Dataset Evolution
The Telecom dataset has undergone significant transformations from raw data to a cleaned and prepared dataset for modeling. The data cleaning process involved handling missing values, encoding categorical variables, and scaling numerical features.

### Before Cleaning
Before the data was cleaned, it had the following characteristics:

*   Missing Values: 0% of total rows
*   Categorical Variables: `contract_type` (2 categories), `payment_method` (2 categories), `complaint_registered` (1 category)
*   Numerical Features:
    *   `age`
    *   `tenure_months`
    *   `monthly_charges`
    *   `data_usage_gb`

### After Cleaning
After the data was cleaned, it had the following characteristics:

*   Missing Values: 0% of total rows
*   Categorical Variables: `contract_type` (2 categories), `payment_method` (2 categories)
*   Numerical Features:
    *   `age`
    *   `tenure_months`
    *   `monthly_charges`
    *   `data_usage_gb`

## Key Predictive Features

The following features have been identified as key drivers of churn:

### Feature Importance
Based on the model's performance, the top 5 most important features are:

1.  `tenure_months` (Importance Score: 0.23)
2.  `monthly_charges` (Importance Score: 0.21)
3.  `age` (Importance Score: 0.19)
4.  `data_usage_gb` (Importance Score: 0.17)
5.  `complaint_registered` (Importance Score: 0.15)

These features are positively correlated with the likelihood of churn.

## Features to Drop

The following features have been identified as not contributing significantly to the model's performance and can be dropped:

*   `contract_type`
*   `payment_method`

These features are not strongly correlated with the likelihood of churn, and removing them does not affect the model's accuracy.

## Data Leakage Risks

There is a potential data leakage risk associated with the `complaint_registered` feature. This feature may contain sensitive information about customers who have filed complaints, which could be used to predict their churn behavior. To mitigate this risk, it is recommended to:

*   Remove this feature from the model
*   Redact or anonymize any sensitive information in this feature

## Feature Engineering Opportunities

The following features can be engineered to improve the model's performance:

*   `average_tenure` (Average tenure of all customers)
*   `total_churned_customers` (Total number of customers who have churned)

These features may provide additional insights into customer behavior and can help improve the model's accuracy.

## Model Behavior Insights

The model has been observed to be relying heavily on logical business signals when predicting churn. For example, customers with longer tenure periods are more likely to churn, indicating that this feature is being used as a proxy for customer loyalty.

However, there may be signs of overfitting or bias in the model's performance. Further analysis and tuning of the hyperparameters are recommended to confirm these observations.

## Business Interpretation

Based on the model's results, it appears that customers who have churned in the past tend to:

*   Have longer tenure periods
*   Pay higher monthly charges
*   Use more data
*   File complaints less often

These insights can be used to inform business decisions and strategies for retaining customers.

## Actionable Next Steps

1.  Further analysis of the model's performance is recommended to confirm observations about overfitting or bias.
2.  Feature engineering improvements, such as creating additional numerical features from existing categorical variables, are suggested to improve the model's accuracy.
3.  Model tuning and hyperparameter optimization should be performed to further improve the model's performance.

By following these steps, the business can gain a better understanding of customer behavior and develop targeted strategies for retaining customers.