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

**Sales by States during the period**
``` sql
SELECT 
distinct(location_state) AS state,
round(sum(paid_amt),2) AS yearly_sales
FROM retail.`retail transactions`
GROUP BY state;
```
![States Sales](https://user-images.githubusercontent.com/92436079/217491384-8071094c-45b7-4e37-ab59-173399814fd2.png)

**Sales by Cities during the period**
``` sql
SELECT
DISTINCT location_city,
location_state,
extract(year from transaction_date) AS year,
round(sum(paid_amt),2) AS total_sale
FROM retail.`retail transactions`
GROUP BY location_city ;
```
![Cities Sales (1)](https://user-images.githubusercontent.com/92436079/218033365-0418b399-dcd5-4c2b-ab27-fbb2a89c234c.png)


Atlanta Georgia had the highest number of sales followed by Miami Florida 

**Yearly Sales**

``` sql
SELECT 
extract(year from transaction_date) AS year,
round(sum(paid_amt),2) AS total_sales
FROM retail.`retail transactions`
GROUP BY year;
``` 
![Sheet 4](https://user-images.githubusercontent.com/92436079/217524865-043572ba-3cbe-4a80-bec0-4e0b74d32b10.png)
+ 2020 contributed 76.77% of the total sales from the transactions made while 2019 contributed only 23.23%
**Monthly Sales**

``` sql
SELECT 
extract(month from transaction_date) AS month,
round(sum(paid_amt),2) as monthly_sales,
extract(year from transaction_date) as year
FROM retail.`retail transactions`
GROUP BY month;
```
![Monthly Sales](https://user-images.githubusercontent.com/92436079/217526501-435bd09d-1b1b-4fd3-84f8-dbcc934fa271.png)
+ October recoreder the highest number of sales for the year 2020 

**Sales by time_of-day**
``` sql
SELECT
time_of_day,
round(sum(paid_amt),2) AS total_sales
FROM retail.`retail transactions`
GROUP BY time_of_day;
```
![Sheet 5](https://user-images.githubusercontent.com/92436079/217529187-5ada6510-a263-4cf7-9fc8-85bd427fbe3d.png)

+ The Evening recorded most sales,followed by late afternoon

## Time Series Decomposition
+ A time series decomposition is a process of breaking down a time series into its constituent parts such as trend, seasonality, and residuals. The decomposition can be performed using various statistical models, but one of the most common approaches is to use the STL (Seasonal and Trend decomposition using Loess) method.

**Trend**
+ Calculating the average total amount for each month, and then using a linear regression to estimate the trend.
``` sql
WITH raw_data AS(
SELECT 
extract(month FROM transaction_date) AS month,
round(SUM(paid_amt),2) AS total_amount
FROM retail.`retail transactions`
GROUP BY month
  ),
trend_data AS(
SELECT
rd.month,
AVG(total_amount) AS avg_total_amount, 
ROW_NUMBER() OVER (ORDER BY month) AS rn
FROM raw_data AS rd
GROUP BY month
),
trend_fit AS (
SELECT
td.month,
td.avg_total_amount,
td.rn, 
(rn - 1) * AVG(avg_total_amount) OVER () AS y_intercept,
(rn - 1) * AVG(avg_total_amount) OVER () / (rn - 1) AS slope
FROM trend_data AS td
GROUP BY month )
SELECT
tf.month,
tf.avg_total_amount, 
round(tf.slope * tf.rn + y_intercept,2) AS trend
FROM trend_fit AS tf 
GROUP BY month

```

![image](https://user-images.githubusercontent.com/92436079/217534454-6a2784b3-2543-4ece-8119-0745b651b92c.png)

+ The first CTE raw_data groups the data by month and calculates the total amount spent in each month.

+ The second CTE trend_data calculates the average total amount spent for each month and assigns a row number to each month based on the order of the months.

+ The third CTE trend_fit calculates the slope and y-intercept of the line of best fit(linear regression) for the average total amount spent for each month.

+ The fourth CTE trend_fit_with_y calculates the trend component of the time series data.

**Seasonality**
+ Calculating the deviation of each month's total amount from the average monthly amount, and then dividing the deviation by the seasonal index to get the seasonal fit.
``` sql
WITH raw_data AS(
SELECT 
extract(month FROM transaction_date) AS month,
round(SUM(paid_amt),2) AS total_amount
FROM retail.`retail transactions`
GROUP BY month
  ),
trend_data AS(
SELECT
rd.month,
AVG(total_amount) AS avg_total_amount, 
ROW_NUMBER() OVER (ORDER BY month) AS rn
FROM raw_data AS rd
GROUP BY month
),
trend_fit AS (
SELECT
td.month,
td.avg_total_amount,
td.rn, 
(rn - 1) * AVG(avg_total_amount) OVER () AS y_intercept,
(rn - 1) * AVG(avg_total_amount) OVER () / (rn - 1) AS slope
FROM trend_data AS td
GROUP BY month ),
trend_fit_with_y AS(
SELECT
tf.month,
tf.avg_total_amount, 
round(tf.slope * tf.rn + y_intercept,2) AS trend
FROM trend_fit AS tf 
GROUP BY month
),

seasonality_data AS(
SELECT
month(rd.month) AS month,
AVG(total_amount) OVER (ORDER BY extract(month from month)) AS avg_monthly_amount,
total_amount - AVG(total_amount) OVER (ORDER BY extract(month from month)) AS deviation
FROM raw_data AS rd
GROUP BY month),
seasonal_index_data AS(
SELECT 
extract(month from sd.month) as month_part,
AVG(deviation) OVER (PARTITION BY extract(month from month)) AS seasonal_index
FROM seasonality_data AS sd
GROUP BY month_part
)
SELECT
month_part,
deviation / seasonal_index AS seasonal_fit
FROM seasonality_data AS sd, seasonal_index_data AS sid
GROUP BY month

```

![image](https://user-images.githubusercontent.com/92436079/217536997-5d340f56-ca3e-4ac6-a12f-d34417a08e20.png)

+ CTE seasonality_data calculates the deviation of each month's total amount spent from the average monthly amount spent.

+ CTE seasonal_index_data calculates the seasonal index, which represents the deviation of each month's average amount spent from the overall average amount spent.

+ CTE seasonality_fit calculates the seasonal component of the time series data.

**Residuals**
``` sql
WITH raw_data AS(
SELECT 
extract(month FROM transaction_date) AS month,
round(SUM(paid_amt),2) AS total_amount
FROM retail.`retail transactions`
GROUP BY month
  ),
trend_data AS(
SELECT
rd.month,
AVG(total_amount) AS avg_total_amount, 
ROW_NUMBER() OVER (ORDER BY month) AS rn
FROM raw_data AS rd
GROUP BY month
),
trend_fit AS (
SELECT
td.month,
td.avg_total_amount,
td.rn, 
(rn - 1) * AVG(avg_total_amount) OVER () AS y_intercept,
(rn - 1) * AVG(avg_total_amount) OVER () / (rn - 1) AS slope
FROM trend_data AS td
GROUP BY month ),
trend_fit_with_y AS(
SELECT
tf.month,
tf.avg_total_amount, 
round(tf.slope * tf.rn + y_intercept,2) AS trend
FROM trend_fit AS tf 
GROUP BY month
),

seasonality_data AS(
SELECT
month(rd.month) AS month,
AVG(total_amount) OVER (ORDER BY extract(month from month)) AS avg_monthly_amount,
total_amount - AVG(total_amount) OVER (ORDER BY extract(month from month)) AS deviation
FROM raw_data AS rd
GROUP BY month),
seasonal_index_data AS(
SELECT 
extract(month from sd.month) as month_part,
AVG(deviation) OVER (PARTITION BY extract(month from month)) AS seasonal_index
FROM seasonality_data AS sd
GROUP BY month_part
),
seasonality_fit AS(
SELECT
month_part,
deviation / seasonal_index AS seasonal_fit
FROM seasonality_data AS sd, seasonal_index_data AS sid
GROUP BY month_part),
decomposed_data AS(
SELECT
rd.month,
rd.total_amount,
tfy.trend,
sf.seasonal_fit,
rd.total_amount - tfy.trend - sf.seasonal_fit AS residual
FROM raw_data rd, trend_fit_with_y  as tfy,seasonality_fit sf)
SELECT *
FROM decomposed_data
ORDER BY month;
```

![image](https://user-images.githubusercontent.com/92436079/218108866-0bd509d0-973a-48a8-8773-f917ccee4a19.png)

+ The "decomposed_data" CTE combines the trend, seasonality, and residual information for each month by subtracting the trend and seasonal fit from the total amount in each month.
