# Part 1: Data Exploration and Evaluation
- annual income, dti, and revolving balance are right skewed and have outliers on the higher end of the spectrum - see distribution table and histograms below. These are handled by using log transformations in section 1.2
- loan funded amount and loan amount have 99% correlation - see correlation table below. This is handled using a ratio transformation in section 1.2
- better grade loans reflect higher annual income of borrower, and correspond to a lower interest rate of loan - see aggregation by grade below
- there has been an exponential growth of loan originations - see histogram of loan origination dates in section 1.3
- 36 month term loans have a higher likelihood of being paid off. 27% of 36 month loans are fully paid, whereas 15% of 60 month loans are fully paid - see term table in section 1.3
- Data cleaning: 
> - issue_d: change to datetime field and create year and month variable
> - loan_status: change categorical values with redundant prefix
> - annual_inc: impute values for missing values and self reported 0 annual income, log transformation to handle outliers
> - monthly_debt_payments: calculate monthly debt payments from self reported income and dti, log transformation to handle outliers
> - revol_bal: log transformation to handle outliers
> - dti: log transformation to handle outliers
> - funded_amnt: remove correlation with loan_amnt by ratio transformation
> - assumption: when doing log transformations, log of 0 is undefined - so the median of the log transformation within each cohort is imputed. A cohort is defined as a grade in a given year. Assume that this imputation will add minimal noise. Also assume that the distributions of the model training set is the same as the distribution of the entire population 
# Part 2: Business Analysis
- 78% of loans have been fully paid
- Grade G loans originating in 2007 have the highest rate of default
- These loans have an annuallized return rate of -10.4%
# Part 3: Modeling
- transform categorical variables to binary variables
- use random forest model for variable selection
- fit a logistic regression model on the top variables
- evaluation: 
> - The model is somewhat effective at identifying defaulting loans.
> - A 30% in-time testing sample was held out from the training population and this testing sample was used to evaluate the model.
> - Applying this model to the testing sample results in a KS of 22.6% and an ROC of 70%
> - The gains table indicates that this model is good at ranking defaults and prioritizing defaults -
> - When there is a 50% threshold for default for probability of default (> 50% - default classification and < 50% non default classification), the model has a precision of 57% and recall of 13%. This means that 57% of the loans identified as default will actually default and 13% of all defaults are actually captured in the model. This means that a substantially large amount (87%) of the defaults are missed in default classification and there is also a 43% chance of turning away a good loan.
> - In order to find an optimal precision/recall and false negative/positive rate, the threshold of default classification would have to be changed
> - Although the presision and recall scores are low, the model identifies 39% of defaulters within the first two deciles of the testing population This is a lift of 2x better than random selection (see gains table) and it corresponds to identifying 44.7% of all loan losses from defaults (~$10M). This means that the model is good at prioritizing high loan losses

> - In order to refine this model and improve its performance:

>> - Current 60 month loans add noise to the target variable and they should not be treated as 36 month loans
>> - Delinquent loans (late, grace period) should not be treated as default loans because they have not defaulted yet. This adds noise to the model and assumes that all loans that are currently delinquent or have been delinquent will result in a loss
>> - Use a FICO score or similar credit scoring to replace the grade, which is a bucketed version of a score
>> - Use client's cross-product data: e.g. payment history from other loans
>> - Use random forest algorithm instead of logistic regression to yield better performance
