# Retail-Transactions-Analysis
Retail analysis using Real World Fake Data(RWFD) in SQL  and Visuals in Tableau

[Data Source](https://data.world/markbradbourne/rwfd-real-world-fake-data/workspace/file?filename=Retail+Transactions.csv)
The data table contains 75620 rows and 10 columns
``` sql
SELECT *
FROM retail.`retail transaction`;
```


``` sql
Describe retail.`retail transaction`;
```

## Data Cleaning
The data cleaning process involved:
1. Checking of missing values
+ The rewards_member and rewards_number column had 31697rows of missing values,when a customer doesn't have a reward number then no record of being a member,in cases of having a reward number the customer can either be a member or not.

``` sql
SELECT 
rewards_member,
rewards_number
FROM retail.`retail transactions`;
```
+ 54678 rows of missing values in the coupon_flag and discount_amt,without the coupon flag the customer doesn't get a discount
``` sql
SELECT 
coupon_flag,
discount_amt
FROM retail.`retail transactions`
WHERE coupon_flag ='';
```
~Deleting the missing vakues wil alter the analysis therefore the data maintains as it is.

2. Checking for Unique values 
+ Location_state
``` sql
SELECT 
DISTINCT location_state
FROM retail.`retail transactions`;
``` 
~ the dataset contains 5 unique states

+ Location_city
``` sql
SELECT 
DISTINCT location_city
FROM retail.`retail transactions`;
```
~ the dataset contains 84 unique cities

+ reward_member
``` sql 
SELECT 
DISTINCT rewards_member
FROM retail.`retail transactions`;
``` 
~ A customer can either be a member or not and null recorder in cases of no record of reward_number

3. Transformation of columns

**The transformation of the order amount from text to numeric as well as removing the dollar sign**

``` sql
SELECT 
CAST(REPLACE(REPLACE(order_amt, "$", ""), ",", "") AS DECIMAL(10,2)) AS new_amt
FROM retail.`retail transactions`;
```

* Updating the table

``` sql
UPDATE retail.`retail transactions`
SET order_amt=CAST(REPLACE(REPLACE(order_amt, "$", ""), ",", "") AS DECIMAL(10,2));
```
* Confirming changes 
``` sql
SELECT order_amt
FROM retail.`retail transactions`;
```

**The transformation of the transaction_date column from text to date and changing from mm-ddd-yyy to yyy-mm-ddd** 

``` sql
SELECT
date_format(str_to_date(transaction_date,"%m/%e/%Y"),"%Y-%m-%e") as new_date
FROM retail.`retail transactions`;
```

* Update table

``` sql
UPDATE retail.`retail transactions`
SET transaction_date =date_format(str_to_date(transaction_date,"%m/%e/%Y"),"%Y-%m-%e");
```

+ Confrim the changes

``` sql
SELECT transaction_date
FROM retail.`retail transactions`;
```

**Transformation of transaction_date from text to date** 

``` sql
SELECT 
date_format(str_to_date(transaction_hour,"%l:%i %p"),"%l:%i %p")
FROM retail.`retail transactions`;
``` 
+ Update table
``` sql
UPDATE retail.`retail transactions`
SET transaction_hour =date_format(str_to_date(transaction_hour,"%l:%i %p"),"%l:%i %p");
```
+ Confirm Changes

``` sql
SELECT transaction_hour
FROM retail.`retail transactions`;
```
## Manipulation of Columns 
