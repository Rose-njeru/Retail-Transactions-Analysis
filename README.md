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

![image](https://user-images.githubusercontent.com/92436079/217490719-f6b9138e-96a3-4eb9-9800-7cdbb14a0ae7.png)


## Data Cleaning
The data cleaning process involved:
1.Checking of missing values**
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
~Deleting the missing values will alter the analysis therefore treating every row as unique transaction.

2. Checking for Unique values** 
+ Location_state
``` sql
SELECT 
DISTINCT location_state
FROM retail.`retail transactions`;
``` 

![image](https://user-images.githubusercontent.com/92436079/217489654-f118ee9b-2fa9-4c98-9af2-dcc6f22258d5.png)

~ the dataset contains 5 unique states

+ Location_city
``` sql
SELECT 
DISTINCT location_city
FROM retail.`retail transactions`;
```

![image](https://user-images.githubusercontent.com/92436079/217489965-41c9a690-189b-47cc-813a-9f5a243fb423.png)

~ the dataset contains 84 unique cities

+ reward_member
``` sql 
SELECT 
DISTINCT rewards_member
FROM retail.`retail transactions`;
``` 

![image](https://user-images.githubusercontent.com/92436079/217490505-3693a6d7-82c1-468b-ab7c-3c98b7912e43.png)

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
This involves adding more columns to help with analysis and visualization

**Creating a new column time_of_day**

+ Group the transaction_hour interms of morning,afternoon,late_afternoon and evening

``` sql
SELECT transaction_hour,
CASE 
WHEN transaction_hour BETWEEN "11:00 AM" AND "11:59 AM" THEN "Morning"
WHEN transaction_hour BETWEEN "12:00 PM" AND "2:59 PM" THEN "Afternoon"
WHEN transaction_hour BETWEEN "3:00 PM" AND "6:00 PM" THEN "Late_afternoon"
ELSE "Evening"
END AS hours
FROM retail.`retail transactions`;
```
+ Alter the table to add the column

``` sql
ALTER TABLE retail.`retail transactions`
ADD COLUMN time_of_day VARCHAR(50) ;
```
+ Update the table
```sql
UPDATE retail.`retail transactions`
SET time_of_day = CASE 
WHEN transaction_hour BETWEEN "11:00 AM" AND "11:59 AM" THEN "Morning"
WHEN transaction_hour BETWEEN "12:00 PM" AND "2:59 PM" THEN "Afternoon"
WHEN transaction_hour BETWEEN "3:00 PM" AND "6:00 PM" THEN "Late_afternoon"
ELSE "Evening"
END;
``` 
+ Confirm the changes
```
SELECT time_of_day
FROM retail.`retail transactions`;
```
**Creating a new column paid_amt that is the amount paid by customers offered the discount**

``` sql 
SELECT 
(order_amt-discount_amt) AS paid_amt
FROM retail.`retail transactions`;
```
+ Alter the table to add the column and due to the decimals in both discount_amt and paid_amt the column type will be float.
``` sql
ALTER TABLE retail.`retail transactions`
ADD COLUMN paid_amt FLOAT;
```
+ Update the table
``` sql
UPDATE retail.`retail transactions`
SET paid_amt =(order_amt-discount_amt);
```
+ Confirm the changes 
 ``` sql
UPDATE retail.`retail transactions`
SET paid_amt =(order_amt-discount_amt);
```

## Analysis

**The Total Amount paid to the store**

``` sql
SELECT 
round(sum(paid_amt),2) AS total_sales
FROM retail.`retail transactions`;
```
![image](https://user-images.githubusercontent.com/92436079/217489440-9d348c64-683e-4f19-843b-e23a70f02c76.png)

**The average amount paid daily**
``` sql
SELECT
round(AVG(paid_amt),2) as average_daily_amount
FROM retail.`retail transactions`;
```
![image](https://user-images.githubusercontent.com/92436079/217489029-e103c017-587b-4b73-9d3f-85f727b97793.png)

