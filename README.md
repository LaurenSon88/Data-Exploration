# Data-Exploration
### The following is a compilation of SQL queries takening from [Serious SQL](https://www.datawithdanny.com/courses/serious-sql) to show my understanding of SQL

<details>
<summary>
SELECT statement and Sort
</summary>

Select all columns:        `SELECT *`

OR

Select specific columns:   `SELECT column_name_1, column_name_2...`

Where is the data?         `FROM schema_name.table_name`

In what order:             `ORDER BY column_name_1, column_name_2...DESC`

  Descending order `DESC`
  
How many records?          `LIMIT number`

1. What is the name of the category with the highest category_id in the dvd_rentals.category table?
```sql
SELECT name, category_id
FROM dvd_rentals.category
ORDER BY category_id desc
LIMIT 1;
```

2. For the films with the longest length, what is the title of the “R” rated film with the lowest replacement_cost in dvd_rentals.film table?
```sql
SELECT title, rating, length, replacement_cost
FROM dvd_rentals.film
GROUP BY replacement_cost,length,rating, title
ORDER BY length desc, replacement_cost;
```

3. Who was the manager of the store with the highest total_sales in the dvd_rentals.sales_by_store table?
```sql
SELECT manager, total_sales
FROM dvd_rentals.sales_by_store
ORDER BY total_sales DESC;
```

4. What is the postal_code of the city with the 5th highest city_id in the dvd_rentals.address table?
```sql
SELECT postal_code, city_id
FROM dvd_rentals.address
ORDER BY city_id DESC
LIMIT 5;
```
</details>

<details>
<summary>
COUNT and DISTINCT
</summary>

  `COUNT` returns the number of records/rows in a particular column
  
  `DISTINCT` returns unique values if there are duplicate values
  
  `COUNT DISTINCT` returns the number of unique records in a particular column
  
  We also use `AS` to create an alias for a new output column
  
1. How many rows are there in the film_list table?
```sql
SELECT COUNT(*) AS row_count
FROM dvd_rentals.film_list;
```
2. What are the unique values for the rating column in the film table?
```sql
SELECT DISTINCT rating
FROM dvd_rentals.film_list;
```

3. How many unique category values are there in the film_list table?
```sql
SELECT COUNT(DISTINCT category) AS unique_category_count
FROM dvd_rentals.film_list;
```
  
### Apply Aggregate Count Function & Single Column Value Counts
  
  We can also sort our data using the `GROUP BY` clause
  
4. What is the frequency of values in the rating column in the film_list table?
```sql
SELECT
  rating,
  COUNT(*) AS frequency
FROM
  dvd_rentals.film_list
GROUP BY
  rating
ORDER BY
  frequency DESC;
```

### Adding a Percentage Column
  
5. What percentage does each rating hold in the film_list table
```sql
SELECT
  rating,
  COUNT(*) AS frequency,
  ROUND(
    100 * COUNT(*) :: NUMERIC / SUM(COUNT(*)) OVER (),
    2
  ) AS percentage
FROM
  dvd_rentals.film_list
GROUP BY
  rating
ORDER BY
  frequency DESC;
```
> A few things to note:
  1. We use `ROUND` to round off to a number of decimal points i.e. 2 decimal points in the example
  2. We use `::NUMERIC` to cast an integter as a numeric data type to avoid [floor division](https://www.educative.io/answers/floor-division)
  3. `OVER()` is a window funtion
  4. We first count the number of ratings and the divide by the total number `SUM` of the ratings.
  
### Counts For Multiple Column Combinations
 
6.1. What are the 5 most frequent rating and category combinations in the film_list table?
```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM
  dvd_rentals.film_list
GROUP BY
  rating,
  category
ORDER BY
  frequency DESC
LIMIT
  5;
```
> NOTE: We need to group by the same selected columns
  
6.2. Group by ordinal syntax (instead of column name)
```sql
SELECT
  rating,
  category,
  COUNT(*) AS frequency
FROM
  dvd_rentals.film_list
GROUP BY
  1,2
ORDER BY
  frequency DESC
LIMIT
  5;
```
</details>

<details>
<summary>
Identifying Duplicate Records
</summary>

## Dealing with duplicate records
  
#### 1. Using a `SELECT COUNT(*)` will return the total number of rows in the dataset.
  <img width="595" alt="count star" src="https://user-images.githubusercontent.com/111830926/204182009-38d04ebb-0bf0-47ee-b0a7-bc76d5fb8ded.png">

#### 2. Using `SELECT DISTINCT *` returns all the unique rows in the datatset, i.e. removing duplicate rows.
  <img width="1157" alt="distinct" src="https://user-images.githubusercontent.com/111830926/204182247-301a075c-c737-49a9-8f00-29b5f60cfa04.png">

  
> **A problem arises when we want to count the number of distinct/unique rows. PostgreSQL does not allow for this:**
  <img width="1150" alt="count-distinct" src="https://user-images.githubusercontent.com/111830926/204182501-e45771b9-3b55-420f-9220-f5b9510b5f1e.png">

### There are 3 ways to get around this:

  #### a. Subqueries
```sql
SELECT COUNT(*)
FROM (
  SELECT DISTINCT *
  FROM health.user_logs) AS subquery;
 ```
  
  #### b. CTE (Common table expression)
```sql
WITH cte_dedups AS (
  SELECT distinct *
  FROM health.user_logs)
SELECT COUNT(*)
FROM cte_dedups;
```
  
  #### c. Temp Tables
```sql
DROP TABLE IF EXISTS deduplicated_user_logs;

CREATE TEMP TABLE deduplicated_user_logs AS
SELECT DISTINCT *
FROM health.user_logs;

SELECT COUNT(*)
FROM deduplicated_user_logs;
  ```
  
#### 3. Compare counts
  The row count in the original table/dataset vs. the row count of the deduplicated table.
  
  In this example the original table has 43891 rows and the deduplicated table has 31004 row, therefore we can conclude that there are duplicate records.
  
  
## Other ways to identify duplicate records
  
  ### Group by counts across all columns
```sql
 SELECT 
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic,
  COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
ORDER BY frequency DESC;
```

#### Using the `WHERE` clause to show records that appear more than once `> 1`, and excluding those that only appear once.
  ```sql
WITH groupby_count AS (
SELECT 
   id,
   log_date,
   measure,
   measure_value,
   systolic,
   diastolic,
   COUNT(*) AS frequency
FROM health.user_logs
GROUP BY 
   id,
   log_date,
   measure,
   measure_value,
   systolic,
   diastolic)
SELECT *
FROM groupby_count
WHERE frequency > 1
ORDER BY frequency DESC;
```
  
#### Applying a condition using the `HAVING` clause to return the duplicate records and there frequencies
  
```sql
DROP TABLE IF EXISTS unique_duplicate_records;

CREATE TEMPORARY TABLE unique_duplicate_records AS
SELECT *
FROM health.user_logs
GROUP BY
  id,
  log_date,
  measure,
  measure_value,
  systolic,
  diastolic
HAVING COUNT(*) > 1;

SELECT *
FROM unique_duplicate_records
LIMIT 10;
```
  
> NOTES:
  1. We use `DISTINCT` to remove duplicate records from a dataset
  2. To calculate unique record counts we can use either CTEs or subqueries, however CTEs are better to use in terms of readability.
  3. To detect the presence of duplicate records compare the basic record counts with the unique counts
  4. We use the `GROUP BY` clause to identify the exact duplicate records across all columns in a table
  5. We use the `HAVING` clause to filter records. NB we cannot use the alias name for an aggregate function in the `HAVING` clause i.e. we must use `COUNT(*)` eg. `COUNT(*) > 1` 
 
</details>

<details>
<summary>
Summary Statistics
</summary>
  
  ### Mean, median and mode
 
  #### Mean 
```sql
SELECT 
  ROUND(AVG(measure_value),2) AS average_weight
FROM health.user_logs
WHERE measure = 'weight'
 AND measure_value > 0
 AND measure_value < 201;
```
<img width="263" alt="mean" src="https://user-images.githubusercontent.com/111830926/204466495-1437325c-cd65-495f-ba82-dd06d353065d.png">
                   
#### Median                         
```sql
SELECT 
  ROUND(
  CAST(PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_value) AS NUMERIC),
  2) AS median_weight
FROM health.user_logs
WHERE measure = 'weight'
 AND measure_value > 0
 AND measure_value < 201;
```      
 <img width="249" alt="median" src="https://user-images.githubusercontent.com/111830926/204466540-6e60e384-db1d-4c51-8013-c04a929b5835.png">
                      
#### Mode
```sql
SELECT 
  ROUND(
   MODE() WITHIN GROUP (ORDER BY measure_value), 
   2) AS mode_weight
FROM health.user_logs
WHERE measure = 'weight'
 AND measure_value > 0
 AND measure_value < 201;
 ```
  <img width="209" alt="mode" src="https://user-images.githubusercontent.com/111830926/204466582-db837e68-5b53-4f26-8dd9-277b0969bf4b.png">
             
  ### Max and min to get a range
```sql
SELECT 
  MIN(measure_value) AS min_weight,
  MAX(measure_value) AS max_weights,
  MAX(measure_value) - MIN(measure_value) AS weight_range
FROM health.user_logs
WHERE measure = 'weight'
 AND measure_value > 0
 AND measure_value < 201;
```
<img width="525" alt="min max" src="https://user-images.githubusercontent.com/111830926/204466843-538245a3-e39b-4801-a2d1-b5e892fab5d1.png">


  ### Variance and standard deviation
```sql
SELECT ROUND(STDDEV(measure_value,2) AS standard_deviation
FROM health.user_logs
WHERE measure = 'weight'
```

```sql
SELECT ROUND(VARIANCE(measure_value,2) AS variance_value
FROM health.user_logs
WHERE measure = 'weight'
```
  ### 
   
</details>

<details>
<summary>
Distribution Functions
</summary>
  

  ### Cumulative Distribution Function F(V)
  
#### SQL reverse engineering 
  
| Percentile | floor_value | ceiling_value | percentile_count |
|------------|-------------|---------------|------------------|
|      1     |     min     |      max      |     frequency    |
|     100    |             |               |                  |
  
#### Data algorithm:
  1. Sort values ascending
  2. Assign 1 - 100 percentile value 
  3. For each percentile
     * calculate floor and ceiling values
     * calculate record count

```sql  
WITH percentile_value AS (
  SELECT 
    measure_value,
    NTILE(100) OVER(ORDER BY measure_value) AS percentile 
  FROM health.user_logs
  WHERE measure = 'weight'
  )
SELECT 
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM percentile_value
GROUP BY percentile
ORDER BY percentile;
```
> You need to inspect for outliers
  - Look at 1st and 100th percentile
<figure>
<img width="408" img src="https://user-images.githubusercontent.com/111830926/204946911-6ba19228-b0b9-4e92-bba4-fa292c35b255.png"/>
</figure>
<figure>
<img width="408" img src="https://user-images.githubusercontent.com/111830926/204947074-f04aafa0-e6cc-4202-bb7f-ed8be10926f4.png"/>
</figure>


  

  - Remove outliers

  ```sql
DROP TABLE IF EXISTS clean_weight_logs;
CREATE TABLE clean_weight_logs AS(
  SELECT *
  FROM health.user_logs
  WHERE measure = 'weight'
    AND measure_value > 0
    AND measure_value < 201);
WITH percentile_value AS (
  SELECT 
    measure_value,
    NTILE(100) OVER(ORDER BY measure_value) AS percentile 
  FROM clean_weight_logs
  )
SELECT 
  percentile,
  MIN(measure_value) AS floor_value,
  MAX(measure_value) AS ceiling_value,
  COUNT(*) AS percentile_count
FROM percentile_value
GROUP BY percentile
ORDER BY percentile                            
  ```          
| percentile | floor_value | ceiling_value | frequency |
|------------|-------------|---------------|-----------|
| 1          | 1.814368    | 29.48348      | 28        |
| 2          | 29.48348    | 32.4771872    | 28        |
| 3          | 32.658623   | 35.380177     | 28        |
| 4          | 35.380177   | 36.74095      | 28        |
| 5          | 36.74095    | 37.194546     | 28        |
| ...        | ...         | ...           | ...       |
| 95         | 129.77278   | 130.52802     | 27        |
| 96         | 130.5389    | 131.54168     | 27        |
| 97         | 131.54169   | 132.6599      | 27        |
| 98         | 132.736     | 133.765       | 27        |
| 99         | 133.80965   | 136.0776      | 27        |
| 100        | 136.0776    | 200.487664    | 27        |
                             
### Visualize the cumulative distribution
#### 15% of users are under 60kg
<img width="825" alt="1st percentile" src="https://user-images.githubusercontent.com/111830926/204723518-851aeb57-27fb-4182-be11-ec38d7316cac.png">

#### 99% of users are under 134kg
<img width="825" alt="99th percentile" src="https://user-images.githubusercontent.com/111830926/204723549-df722571-53c4-4b35-8e26-aabe05105891.png">

### Histogram/Frequency Plots

#### WIDTH_BUCKET function
```sql
SELECT WIDTH_BUCKET(measure_value,0,200,50) AS buckets,
       AVG(measure_value) AS mean_value,
       COUNT(*) AS frequency
FROM clean_weight_logs
GROUP BY bucket
ORDER BY bucket;
```
<img width="747" alt="width_bucket" src="https://user-images.githubusercontent.com/111830926/204728396-26fde2d8-61ff-4fb9-8af5-9740a15eae78.png">

</details>
