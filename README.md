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
Identifying Duplicates
</summary>
</details>

<details>
<summary>
Summary Statistics
</summary>
</details>

<details>
<summary>
Distribution Functions
</summary>
</details>
