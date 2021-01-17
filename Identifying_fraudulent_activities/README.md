Background:
Company XXX is an e-commerce site that sells hand-made clothes. We are asked to build a model that predicts whether a user has a high probability of using the site to perform some illegal activity or not
We only have information about the user first transaction on the site and based on that we have to make our classification ("fraud/no fraud").
Goal：
Build a machine learning model that predicts the probability that the first transaction of a new user is fraudulent.
Describe what kinds of users are more likely to be classified as at risk? What are their characteristics?
From the product perspective, how can we use this model.
Methodology summary:
Overall flow:
EDA > Data preprocessing > Baseline model > Feature engineering > Feature selection > Modeling
Metrics for evaluation: ROC-AUC, F1 score and confusion matrix.
Algorithms to test: Logistic regression, Random Forest, XGBoost.

Raw input data overview:
The raw input data has two tables:
Transaction data for 2015 (151112 observations)
IP address data (138846 observations)
There are no missing data nor duplicated observations.
11 independent variables, target variable is ‘class’.
Imbalanced class with fraud ratio 9.36%.
There are 3 unique sources, 5 unique browsers and 182 unique countries.
The same device_id has been used up to 20 times.

Feature engineering:
Based on the EDA, we transformed and created a few more features explained below:

1. Time difference (Purchase time – signup time): the amount of time between signing up new account and making purchase. Quick purchase: the time difference is less than 30s.

2. IP address frequency: the frequency of the same IP address has been used to signup new account and make purchase.

3. Device ID frequency: the frequency of the same device ID has been used to signup new account and make purchase.

4. Signup/Purchase time variants: the hour of the day, the day of the week, the month of the year, etc.

5. Regions with high fraud rate: calculate the fraud rate per country from the training data and divide the data into different groups.

The feature importance rank based on the correlation to the dependent variable matches the observations in scatter plots:

1. Quick_purchase
2. Device_frequency
3. Ip_frequency
4. Purchase_month
5. Time_difference_hour

Feature selection:
Two methods are tested to solve the Imbalanced class problem:
1. SMOTE + RUD(RandomUnderSampler)
2. Class weight

Three methods of feature selection are tested:
1. Filter method: Pearson correlation (selected 16 features)
                             Pearson correlation to remove colinear features (selected 14 features)
2. Wrapper method: Recursive feature elimination (selected 8 features)
3. Embedded method: Lasso regression (selected 8 features)

The ROC-AUC from validation/test set is compared here

|              | Baseline | Pearson correlation | Pearson corr. remove multi-collinearity | REA   | Lasso regression |
|--------------|----------|---------------------|-----------------------------------------|-------|------------------|
| SMOTE+RUD    | 0.746    | 0.811               | 0.781                                   | 0.835 | 0.838            |
| Class weight | 0.759    | 0.842               | 0.796                                   | 0.836 | 0.835            |


### Different modeling algorithms are tuned and compared in the following 2 tables.
### ROC-AUC and weighted F1 are compared:
|                     | Default |             | Hyperparameter tuning |             |
|---------------------|---------|-------------|-----------------------|-------------|
|                     | ROC-AUC | Weighted F1 | ROC-AUC               | Weighted F1 |
| Logistic regression | 0.842   | 0.917       | N/A                   | N/A         |
| Random forest       | 0.835   | 0.947       | 0.839                 | 0.920       |
| XGBoost             | 0.839   | 0.950       | 0.836                 | 0.951       |

### Fraud precision and recall are compared:

|                     | Default         |              | Hyperparameter tuning |              |
|---------------------|-----------------|--------------|-----------------------|--------------|
|                     | Fraud precision | Fraud recall | Fraud precision       | Fraud recall |
| Logistic regression | 0.518           | 0.706        | N/A                   | N/A          |
| Random forest       | 0.928           | 0.541        | 0.534                 | 0.705        |
| XGBoost             | 0.980           | 0.537        | 0.999                 | 0.535        |

1. Based on the modeling tests, the best ROC-AUC achieved is around 0.84, and weighted F1 around 0.951.
2. The threshold could be further tuned depending on the specific business objective, either we should emphasize on the precision more or the recall.

Modeling – feature importance
Feature rank summary from the modeling*:

1. Quick_purchase
2. Device_frequency
3. Ip_frequency
4. Purchase_month
5. Source_direct
6. Risk_country
7. Purchase_value

High-risk user characteristics 
Based on what have been learned from the EDA, feature examination and modeling output. The high-risk user characteristics include:

1. Make quick purchase: less than 30s after signup.
2. Use the same device to repeat signing up and purchase.
3. Use the same IP address to repeat signing up and purchase.
4. Signup and purchase in January. (need to verify whether it’s a seasonality or a one-time characteristic)
5. Comes from direct visit.

Model application strategy
We now have a model to predict the probability of first-time users committing a fraudulent activity, we may create different user experience to minimize fraudulent loss but maximize experience for normal users: 
1. If a user fraudulent probability is less than a threshold X1, the user will have normal experience; 
2. If a user fraudulent probability is larger than X1 but lower than a highly-suspicious threshold X2, the user will be asked for additional verifications; 
3. If a user fraudulent probability is larger than X2, then this user should be prohibited or at least put on hold to check his / her login data for further decision.

How to measure the impact or ROI
After the fraud detection system online, there are a few key performance indicators need to be tracked:
1. Order approval rates: A transaction passed the system will need to go through the card issuers’ system. If a card is reported stolen, the transaction will be declined.
2. Chargeback rates: A transaction goes through the system, but the card owner filed claim for unauthorized transaction, the money will be refunded.
3. User rating and feedback: Information can be extracted from the user feedback, review to check for model performance.




