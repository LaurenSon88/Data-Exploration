# Data-Exploration
The following is a compilation of SQL queries takening from [Serious SQL](https://www.datawithdanny.com/courses/serious-sql)

This is to show my understanding of SQL

## Select and Sort
SELECT statement and ORDER BY clause

Select all columns:        SELECT *

OR

Select specific columns:   SELECT column_name_1, column_name_2...

Where is the data?         FROM schema_name.table_name

In what order:             ORDER BY column_name_1, column_name_2...

How many records?          LIMIT number

1. What is the name of the category with the highest category_id in the dvd_rentals.category table?
```sql
SELECT name, category_id
FROM dvd_rentals.category
ORDER BY category_id desc
LIMIT 1
```

2. For the films with the longest length, what is the title of the “R” rated film with the lowest replacement_cost in dvd_rentals.film table?
```sql
SELECT title, rating, length, replacement_cost
FROM dvd_rentals.film
GROUP BY replacement_cost,length,rating, title
ORDER BY length desc, replacement_cost
```

3. Who was the manager of the store with the highest total_sales in the dvd_rentals.sales_by_store table?
```sql
SELECT manager, total_sales
FROM dvd_rentals.sales_by_store
ORDER BY total_sales DESC 
```

4. What is the postal_code of the city with the 5th highest city_id in the dvd_rentals.address table?
```sql
SELECT postal_code, city_id
FROM dvd_rentals.address
ORDER BY city_id DESC
LIMIT 5
```
