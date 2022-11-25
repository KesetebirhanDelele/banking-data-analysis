# banking-data-analysis

## This project demonstrates data exploration and cleaning using imputation to fill in missing values

# Practice Exercise: Cleaning data & Transforming columns/features
## Context:
- The data is based on real anonymized Czech bank transactions and account info.
- We'll be focusing on practicing the data cleaning, columns transformations, and other techniques that we've learned in the course. 
- But here is the original task description of the dataset publishers:

*The bank wants to improve their services. For instance, the bank managers have only vague idea, who is a good client (whom to offer some additional services)   and who is a bad client (whom to watch carefully to minimize the bank losses). Fortunately, the bank stores data about their clients, the accounts (transactions within several months), the loans already granted, the credit cards issued. The bank managers hope to improve their understanding of customers and seek specific actions to improve services.*
## Dataset Description:

We'll work on three datasets (in three separate csv files):

  - **account**: each record describes static characteristics of an account
  - **transaction**: each record describes one transaction on an account
  - **district**: each record describes districtgraphic characteristics of a district
  
In reality, the organizations like banks often have data stored in multiple datasets. Assume we want to study the transactional level data, we'll need to combine these three datasets together to have transactions data with account and district data.
## Objective: 
   - Examine/clean the individual dataset
   - Combine them into a single dataset, which is subject to more cleaning
   - Create new columns based on existing columns

By the end, the new dataset is ready for more analysis.
### 1. Import the libraries
import pandas as pd
import numpy as np
### 2. Import the data from three csv files as DataFrames `account`, `district`, `trans`
Hint: 
- the `read_csv` function can automatically infer and load zip file, read its documentation of parameter `compression` if you are interested in details
- you may ignore the warning when reading the `trans.csv.zip` file. It is optional to follow the warning instructions to remove it.
account=pd.read_csv('account.csv')
district=pd.read_csv('district.csv')
trans=pd.read_csv('trans.csv')
### 3. Look at the info summary, head of each DataFrame
account.head()
district.head()
trans.head()
### 4. Check for the unique values and their counts in each column for the three DataFrames
account.info()
district.info()
trans.info()
### 5. Check for duplicates in the three DataFrames
account.nunique()
account[account.duplicated()]
district.nunique()
district[district.duplicated()]
trans.nunique()
trans[trans.duplicated()]
### 6. Convert column `account_open_date` in `account` and column `date` in `trans` into datetime dtypes
account["account_open_date"]=pd.to_datetime(account["account_open_date"], format='%Y-%m-%d')
account.info()
trans["trans_date"]=pd.to_datetime(trans["date"], format='%Y-%m-%d')
trans.info()
### 7. Convert the columns `region` and `district_name` in `district` to all uppercase
district['region'] = district['region'].str.upper()
list[set(district['region'])]
district['district_name'] = district['district_name'].str.upper()
list[set(district['district_name'])]
### 8. Check for missing data by columns in `account` using the `isna` method
account.isna().sum()
### 9. Check for missing data by columns in `district` using the `isna` method
# Missing values
district.isna().sum()
# % Missing values
district.isna().mean()*100
`district` has numeric features that could have relationships with each other. Let's use iterative imputation on them.
#### Use `IterativeImputer` in `sklearn` to impute based on columns `population`, `average_salary`, `unemployment_rate`, `num_committed_crimes`
##### Import libraries
# Import IterativeImputer from sklearn.impute

from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer
##### Build a list of columns that will be used for imputation, which are `population`, `average_salary`, `unemployment_rate`, `num_committed_crimes`
These are the columns that might be related to each other 
#Define a subset of the dataset
numeric_cols = district.filter(['population', 'average_salary', 'unemployment_rate', 'num_committed_crimes'], axis=1).copy()
numeric_cols
##### Create `IterativeImputer` object and set its `min_value` and `max_value` parameters to be the minumum and maximum of corresponding columns
iter_imp = IterativeImputer(min_value=numeric_cols.min(), max_value=numeric_cols.max())
##### Apply the imputer to fit and transform the columns to an imputed NumPy array
imputed_cols = iter_imp.fit_transform(numeric_cols)
##### Assign the imputed array back to the original DataFrame's columns
cols_to_impute=['population','average_salary','unemployment_rate','num_committed_crimes']
district[cols_to_impute] = imputed_cols
##### Double check that the columns are imputed
# Missing values after imputation
district.isna().sum()
### 10. Check for missing data by columns in `trans` using the `isna` method
# Missing values
trans.isna().sum()
#### Divide the columns into numeric columns and categorical columns, then use the `fillna` method to fill numeric columns with -999, fill categorical columns with 'UNKNOWN'
# Identify data types
trans.info()
# Differentiating columns into numerical and categorical data
trans_numeric_cols=['operation_type']
trans_categorical_cols=['description','partner_bank','partner_account']

trans[trans_numeric_cols]=trans[trans_numeric_cols].fillna(-999)
trans[trans_categorical_cols]=trans[trans_categorical_cols].fillna('Unknown')
trans.isna().sum()
### 11. Check for outliers in `district` using the `describe` method, then look at the histograms of the suspicious columns
district.describe()
#### Explore the outliers in the dataset
import matplotlib.pyplot as plt
district[['population', 'average_salary', 'unemployment_rate', 'num_committed_crimes']].hist(bins=20, figsize=(15, 10))
### 12. Check for outliers in `trans` using the `describe` method, then look at the histograms of the suspicious columns
trans.describe()
#### Explore the outliers in the dataset
trans[['amount','balance']].hist(bins=20,figsize=(10,5))
The DataFrame `account` doesn't have any columns that could have outliers, so we are not exploring it.
### 13. Merge (left join) `account` and `district` into a new DataFrame called `account_district` using their common columns
account2=pd.read_csv('account.csv')
district2=pd.read_csv('district.csv')
account_district = pd.merge(account2, district2, on="district_id", how='left')
### 14. Check the information summary of `account_district`, any missing data?
account_district.isna().sum()
#### Look at the rows with missing data in `account_district`
account_district.isnull() 
account_district[account_district['average_salary'].isnull()]
# do the same to see records with missing values for unemployment_rate, and num_committed_crimes
#### Use `SimpleImputer` from `sklearn` to impute the missing data in columns `population`, `average_salary`, `unemployment_rate`, `num_committed_crimes` with their means
from sklearn.impute import SimpleImputer
imputer = SimpleImputer(missing_values=np.NaN, strategy='mean')
account_district.population = imputer.fit_transform(account_district['population'].values.reshape(-1,1))[:,0]
account_district.average_salary = imputer.fit_transform(account_district['average_salary'].values.reshape(-1,1))[:,0]
account_district.unemployment_rate = imputer.fit_transform(account_district['unemployment_rate'].values.reshape(-1,1))[:,0]
account_district.num_committed_crimes = imputer.fit_transform(account_district['num_committed_crimes'].values.reshape(-1,1))[:,0]

account_district.isnull().sum()
#### Use `fillna` method to impute the missing data in columns `district_name` and `region` with 'UNKNOWN'
account_district['district_name'].fillna('Uknown',inplace=True)
account_district['region'].fillna('Uknown',inplace=True)
account_district.isna().sum()
### 15. Merge (left join) `trans` and `account_district` into a new DataFrame called `all_data` using their common columns
trans.columns
all_data=pd.merge(trans,account_district, on='account_id', how='left')
#### Check the information summary of `all_data`
all_data.info()
### 16. Create a new column `account_open_year` and assign it as the year from column `account_open_date`
all_data['account_open_year']=pd.DatetimeIndex(all_data['account_open_date']).year
all_data['trans_year']=pd.DatetimeIndex(all_data['date']).year
print(set(all_data['account_open_year']))
print(set(all_data['trans_year']))
### 17. Calculate the difference between columns `date` (transaction date) and `account_open_date`
set(all_data['trans_year']-all_data['account_open_year'])
### 18. Create a new column `account_age_days` and assign it as the difference in days between columns `date` (transaction date) and `account_open_date`
all_data.info()
all_data.info()
all_data['date']=pd.to_datetime(all_data['date'], format='%Y-%m-%d')
all_data['account_open_date']=pd.to_datetime(all_data['account_open_date'], format='%Y-%m-%d')
all_data['account_age_days']=(all_data['date']-all_data['account_open_date']) / pd.Timedelta(days=1)
all_data['account_age_days'].head()
### 19. Create a new column `amount_category` by cutting the column `amount` into 3 equal-sized bins, and label the bins as 'low_amount', 'medium_amount', 'high_amount'
bins=3
labels=['low_amount', 'medium_amount', 'high_amount']
all_data['amount_category'] = pd.cut(all_data['amount'], bins, labels=labels)
#### Verify the categories and their counts in `amount_category`
all_data['amount_category'].value_counts()
### 20. Create a new column `account_age_days_category` by cutting the column `account_age_days` into 5 equal-width bins
all_data['account_age_days_category'] = pd.cut(all_data['account_age_days'], bins=5)
#### Verify the categories and their counts in `account_age_days_category`
all_data['account_age_days_category'].value_counts()
#### Print out the first 20 rows of `all_data` to look at the newly added columns
all_data.head(20)