# Problem Solving 

In this section the focus was on generating the requierements from the email template using the previously created base tables. 

The requirements were tackled in this order:

**1.** Category Insights for the top 2 categories.

**2.** Film recommendations for both categories.

**3.** Actor insights. 

**4.** Actor film recommendations.

## Category Insights

An extra column (rental_date) was added to the base table in case there are ties for the calculated fields.

```sql
DROP TABLE IF EXISTS complete_join_dataset;
CREATE TEMP TABLE complete_join_dataset AS
SELECT 
  rental.customer_id,
  inventory.film_id,
  film.title,
  film_category.category_id,
  category.name AS category_name,
  rental.rental_date
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_category
  ON film.film_id = film_category.film_id
INNER JOIN dvd_rentals.category
  ON film_category.category_id = category.category_id;
  
SELECT *
FROM complete_join_dataset
LIMIT 5;
```

| customer_id | film_id | title           | category_id | category_name | rental_date              |
|-------------|---------|-----------------|-------------|---------------|--------------------------|
| 130         | 80      | BLANKET BEVERLY | 8           | Family        | 2005-05-24T22:53:30.000Z |
| 459         | 333     | FREAKY POCUS    | 12          | Music         | 2005-05-24T22:54:33.000Z |
| 408         | 373     | GRADUATE LORD   | 3           | Children      | 2005-05-24T23:03:39.000Z |
| 333         | 535     | LOVE SUICIDES   | 11          | Horror        | 2005-05-24T23:04:41.000Z |
| 222         | 450     | IDOLS SNATCHERS | 3           | Children      | 2005-05-24T23:05:21.000Z |

These are the calculated fields:

``category_name:`` The name of the top 2 ranking categories.

``rental_count:`` How many total films have they watched in this category.

``average_comparison:`` How many more films has the customer watched compared to the average DVD Rental Co customer.

``percentile:`` How does the customer rank in terms of the top X% compared to all other customers in this film category?

``category_percentage:`` What proportion of total films watched does this category make up?

| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|--------------------|------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                  | 1          | 19                  |
| 1           | 2                | Comedy        | 5            | 4                  | 2          | 16                  |
| 2           | 1                | Sports        | 5            | 3                  | 7          | 19                  |
| 2           | 2                | Classics      | 4            | 2                  | 11         | 15                  |

To generate the calculated fields, the agreggations and calculations were separated into temporary tables and then combined.

## Customer Rental Count

The first values to be calculated were the rental count for each customer.

```sql
DROP TABLE IF EXISTS category_rental_counts;
CREATE TEMP TABLE category_rental_counts AS
SELECT 
  customer_id,
  category_name,
  COUNT(*) AS rental_count,
  MAX(rental_date) AS latest_rental_date
FROM complete_join_dataset
GROUP BY 
  customer_id,
  category_name;
  
SELECT *
FROM category_rental_counts
ORDER BY 
    customer_id, 
    rental_count DESC, 
    latest_rental_date DESC
LIMIT 5;
```

| customer_id | category_name | rental_count | latest_rental_date       |
|-------------|---------------|--------------|--------------------------|
| 1           | Classics      | 6            | 2005-08-19T09:55:16.000Z |
| 1           | Comedy        | 5            | 2005-08-22T19:41:37.000Z |
| 1           | Drama         | 4            | 2005-08-18T03:57:29.000Z |
| 1           | Animation     | 2            | 2005-08-22T20:03:46.000Z |
| 1           | Sci-Fi        | 2            | 2005-08-21T23:33:57.000Z |

The ``rental_date`` column from the base table was used so that in case of any ties on the top 2 categories, the row with the most recent rental date would be choosen. 

## Total Customer Rentals 

In order to generate the ``category_percentage`` calculation the total rentals per customer had to be calculated. 

```sql
DROP TABLE IF EXISTS customer_total_rentals;
CREATE TEMP TABLE customer_total_rentals AS  
SELECT 
  customer_id,
  SUM(rental_count) AS total_rental_amount
FROM category_rental_counts
GROUP BY customer_id;

-- show output for first 5 customer_id values
SELECT *
FROM customer_total_rentals
LIMIT 5;
```
| customer_id | total_rental_amount |
|-------------|---------------------|
| 1           | 32                  |
| 2           | 27                  |
| 3           | 26                  |
| 4           | 22                  |
| 5           | 38                  |

## Average Category Rental Counts

The average rental count for every ``category_name`` was also calculated.

```sql
DROP TABLE IF EXISTS average_category_rental_counts;
CREATE TEMP TABLE average_category_rental_counts AS 
SELECT 
  category_name,
  AVG(rental_count) AS avg_rental_count
FROM category_rental_counts
GROUP BY category_name;

-- show output for first 5 values
SELECT *
FROM average_category_rental_counts
ORDER BY avg_rental_count DESC
LIMIT 5;
```
| category_name | avg_rental_count   |
|---------------|--------------------|
| Animation     | 2.3320000000000000 |
| Sports        | 2.2716763005780347 |
| Family        | 2.1876247504990020 |
| Action        | 2.1803921568627451 |
| Documentary   | 2.1739130434782609 |

To make the average values make more sense to the customers, all the average values were rounded down and updated in the table by using the ``UPDATE`` and ``SET`` command.

```sql
UPDATE average_category_rental_counts
SET avg_rental_count = FLOOR(avg_rental_count)
RETURNING *;  
```
| category_name | avg_rental_count |
|---------------|------------------|
| Sports        | 2                |
| Classics      | 2                |
| New           | 2                |
| Family        | 2                |
| Comedy        | 1                |
| Animation     | 2                |
| Travel        | 1                |
| Music         | 1                |
| Horror        | 1                |
| Drama         | 2                |
| Sci-Fi        | 2                |
| Games         | 2                |
| Documentary   | 2                |
| Foreign       | 2                |
| Action        | 2                |
| Children      | 1                |

## Percentile Values
The percentile values were calculated by using the window function ``PERCENT_RANK``. The percentiles values were multiplied by a 100 so that the percentiles values ranges were from 0 to 100. All the percentile values were rounded up with ``CEILING`` because there were percentile values near to 0 and if those values were rounded they would end up being 0 and being not understandable for the customers. 

At the end after the CTE, the percentile values of 0 were converted into 1 using a ``CASE`` statement.   

```sql
DROP TABLE IF EXISTS customer_category_percentiles;
CREATE TEMP TABLE customer_category_percentiles AS 
WITH calculated_cte AS (
SELECT
  customer_id,
  category_name,
  -- use ceiling to round up to nearest integer after multiplying by 100
  CEILING(
    100 * PERCENT_RANK() OVER (
      PARTITION BY category_name
      ORDER BY rental_count DESC)
    ) AS percentile
FROM category_rental_counts
)
SELECT 
  customer_id,
  category_name,
  -- if percentile is 0, it turns into 1
  CASE
    WHEN (percentile * 1) = 0 THEN 1
    ELSE (percentile * 1)
  END AS percentile  
FROM calculated_cte;

SELECT *
FROM customer_category_percentiles
WHERE customer_id = 506
LIMIT 2;
```

| customer_id | category_name | percentile |
|-------------|---------------|------------|
| 506         | Action        | 1          |
| 506         | Drama         | 4          |

## Joining Temporary Tables & Adding in Calculated Fields 

The temporary generated tables were combined and also the ``average_comparasion`` and ``category_percentage`` fields were calculated.

The ``category_rental_counts`` table was the base table for the table joins and the inner join was used for the table joins since the keys from the other tables were also in the ``category_rental_counts`` table.

```sql
DROP TABLE IF EXISTS customer_category_joint_table;
CREATE TEMP TABLE customer_category_joint_table AS 
SELECT
  t1.customer_id,
  t1.category_name,
  t1.rental_count,
  t1.latest_rental_date,
  t2.total_rental_amount,
  t3.avg_rental_count,
  t4.percentile,
  t1.rental_count - t3.avg_rental_count AS average_comparison,
  ROUND((t1.rental_count / t2.total_rental_amount) * 100) AS category_percentage
FROM category_rental_counts AS t1
INNER JOIN customer_total_rentals AS t2
  ON t1.customer_id = t2.customer_id
INNER JOIN average_category_rental_counts AS t3 
  ON t1.category_name = t3.category_name
INNER JOIN customer_category_percentiles AS t4 
  ON t1.customer_id = t4.customer_id 
  AND t1.category_name = t4.category_name;
  
SELECT *
FROM customer_category_joint_table
WHERE customer_id = 1
ORDER BY percentile
LIMIT 5;  
```  
| customer_id | category_name | rental_count | latest_rental_date       | total_rental_amount | avg_rental_count | percentile | average_comparison | category_percentage |
|-------------|---------------|--------------|--------------------------|---------------------|------------------|------------|---------------------|---------------------|
| 1           | Comedy        | 5            | 2005-08-22T19:41:37.000Z | 32                  | 1                | 1          | 4                   | 16                  |
| 1           | Classics      | 6            | 2005-08-19T09:55:16.000Z | 32                  | 2                | 1          | 4                   | 19                  |
| 1           | Drama         | 4            | 2005-08-18T03:57:29.000Z | 32                  | 2                | 4          | 2                   | 13                  |
| 1           | Music         | 2            | 2005-07-09T16:38:01.000Z | 32                  | 1                | 21         | 1                   | 6                   |
| 1           | New           | 2            | 2005-08-19T13:56:54.000Z | 32                  | 2                | 27         | 0                   | 6                   |


## Final Output Category Insights

The window function that helped on getting the final output was ``ROW_NUMBER``, this function adds row numbers for records within a group. For this case each group was a different ``customer_id`` and every group was numbered from 1 until the record that each ``customer_id`` had.

 The windows function was made within a CTE and then a filter was applied after the CTE to keep row number records which were less or equal to 2. 

```sql
DROP TABLE IF EXISTS top_categories_information;
CREATE TEMP TABLE top_categories_information AS (
WITH ordered_customer_category_joint_table AS (
  SELECT
    customer_id,
    ROW_NUMBER() OVER(
    PARTITION BY customer_id
    ORDER BY rental_count DESC, latest_rental_date DESC
    ) AS category_ranking,
    category_name,
    rental_count,
    average_comparison,
    percentile,
    category_percentage
  FROM customer_category_joint_table   
)

SELECT *
FROM ordered_customer_category_joint_table
WHERE category_ranking <= 2
);

--result for the first 3 customers
SELECT *
FROM top_categories_information
WHERE customer_id in (1,2,3)
ORDER BY customer_id, category_ranking;
```
| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|---------------------|------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                   | 1          | 19                  |
| 1           | 2                | Comedy        | 5            | 4                   | 1          | 16                  |
| 2           | 1                | Sports        | 5            | 3                   | 3          | 19                  |
| 2           | 2                | Classics      | 4            | 2                   | 2          | 15                  |
| 3           | 1                | Action        | 4            | 2                   | 5          | 15                  |
| 3           | 2                | Sci-Fi        | 3            | 1                   | 15         | 12                  |


# Category Recommendations


## Film Counts

Using the category insights base table or the ``complete_join_dataset`` table, a total film count table was created.

This output included the ``film_id``, ``category_name``, ``title``
and the ``rental_count`` for each distinct ``film_id``.

A windows function was used to calculate the rental count of each film. The reason for using the window function was so that the ``category name`` could be kept in the output.

```sql
DROP TABLE IF EXISTS film_counts;
CREATE TEMP TABLE film_counts AS 
SELECT DISTINCT
  film_id,
  title,
  category_name,
  COUNT(*) OVER(
    PARTITION BY film_id
  ) AS rental_count
FROM complete_join_dataset;  

SELECT *
FROM film_counts
LIMIT 5;
```
| film_id | title            | category_name | rental_count |
|---------|------------------|---------------|--------------|
| 655     | PANTHER REDS     | Sci-Fi        | 15           |
| 285     | ENGLISH BULWORTH | Sci-Fi        | 30           |
| 258     | DRUMS DYNAMITE   | Horror        | 13           |
| 809     | SLIPPER FIDELITY | Sports        | 16           |
| 883     | TEQUILA PAST     | Children      | 6            |

## Category Film Exclusions

A table was created with all the films that each customer already watched so that the recommendations were not from already watched films by the customer.

```sql
DROP TABLE IF EXISTS category_film_exclusions;
CREATE TEMP TABLE category_film_exclusions AS 
SELECT DISTINCT 
  customer_id,
  film_id
FROM complete_join_dataset;

SELECT *
FROM category_film_exclusions
ORDER BY customer_id
LIMIT 5;
```
| customer_id | film_id |
|-------------|---------|
| 1           | 308     |
| 1           | 480     |
| 1           | 709     |
| 1           | 997     |
| 1           | 579     |

## Final Category Recommendations

**In this example - the anti join would actually be ran after the inner join as it needs values from both of those table references, you can think of it as removing any records from the joined records which are part of the exclusions.**

**The inner join will happen first and then the anti join will take place. From the inner joined table, the row of the ``customer_id`` with the ``film_id`` that appears in the ``film_exclusions`` table will not be considered in the anti join and at the end we will have all the movies that each customer has not seen in their top categories.**

In this part 3 film recommendations for the top 2 categories of each customer were generated. To do this an inner join between the ``top_categories_information`` and ``film_counts`` tables was applied. Afterwards, an anti join between the previously combined tables and the ``film_exclussions`` was applied to remove any records which were part of the ``film_exclussions`` table.

Next, ``DENSE_RANK`` was used to rank the films by ``rental_count`` for each of the top 2 categories per customer. In case of any ``rental_count`` ties the order was by title alphabetically.

At the end a filter was applied to keep only the top 3 films for each top 2 category.

```sql
DROP TABLE IF EXISTS category_recommendations;
CREATE TEMP TABLE category_recommendations AS 
WITH ranked_films_cte AS (
  SELECT
    top_categories_information.customer_id,
    top_categories_information.category_name,
    top_categories_information.category_ranking,
    film_counts.film_id,
    film_counts.title,
    film_counts.rental_count,
    DENSE_RANK() OVER(
      PARTITION BY 
        top_categories_information.customer_id,
        top_categories_information.category_ranking
      ORDER BY
        film_counts.rental_count DESC,
        film_counts.title
    ) AS reco_rank
  FROM top_categories_information
  INNER JOIN film_counts
    ON top_categories_information.category_name = film_counts.category_name
  WHERE NOT EXISTS(
    SELECT 1
    FROM category_film_exclusions
    WHERE 
      category_film_exclusions.customer_id = top_categories_information.customer_id
      AND category_film_exclusions.film_id = film_counts.film_id
  )  

)
SELECT * 
FROM ranked_films_cte
WHERE reco_rank <= 3;

SELECT *
FROM category_recommendations
WHERE customer_id = 1
ORDER BY 
  category_ranking, 
  reco_rank;
```

| customer_id | category_name | category_ranking | film_id | title               | rental_count | reco_rank |
|-------------|---------------|------------------|---------|---------------------|--------------|-----------|
| 1           | Classics      | 1                | 891     | TIMBERLAND SKY      | 31           | 1         |
| 1           | Classics      | 1                | 358     | GILMORE BOILED      | 28           | 2         |
| 1           | Classics      | 1                | 951     | VOYAGE LEGALLY      | 28           | 3         |
| 1           | Comedy        | 2                | 1000    | ZORRO ARK           | 31           | 1         |
| 1           | Comedy        | 2                | 127     | CAT CONEHEADS       | 30           | 2         |
| 1           | Comedy        | 2                | 638     | OPERATION OPERATION | 27           | 3         |


# Actor Insights

## Actor Joint Table

As previously seen this is the base table for the actor insights. This table contains more rows than the base table for the category insights since there is a many-to-many relationship between ``film_id`` and ``actor_id``.

```sql
DROP TABLE IF EXISTS actor_joint_dataset;
CREATE TEMP TABLE actor_joint_dataset AS
SELECT 
  rental.customer_id,
  rental.rental_id,
  rental.rental_date,
  film.film_id,
  film.title,
  actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.rental
INNER JOIN dvd_rentals.inventory
  ON rental.inventory_id = inventory.inventory_id
  INNER JOIN dvd_rentals.film
  ON inventory.film_id = film.film_id
INNER JOIN dvd_rentals.film_actor
  ON film.film_id = film_actor.film_id
INNER JOIN dvd_rentals.actor
  ON film_actor.actor_id = actor.actor_id;

SELECT *
FROM actor_joint_dataset
LIMIT 5;
```

| customer_id | rental_id | rental_date              | film_id | title           | actor_id | first_name | last_name |
|-------------|-----------|--------------------------|---------|-----------------|----------|------------|-----------|
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 200      | THORA      | TEMPLE    |
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 193      | BURT       | TEMPLE    |
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 173      | ALAN       | DREYFUSS  |
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 16       | FRED       | COSTNER   |
| 459         | 2         | 2005-05-24T22:54:33.000Z | 333     | FREAKY POCUS    | 147      | FAY        | WINSLET   |

## Top Actor Counts 

The rental counts per actor were generated and only the top actor from each customer was kept by using a filter. 

The whole process was divided into CTEs and a temporal table to make things easier. In the first CTE the actor counts per customer were generated and in the second CTE the ranking of the top actors per customer was generated, the ranking was based on the rental counts and in case of any ties the values would be sorted by the latest rental date and also the name in alphabetical order.

Once the second CTE was done, the filter could be applied and only the first top actor of each customer was kept.

```sql
DROP TABLE IF EXISTS top_actor_counts;
CREATE TEMP TABLE top_actor_counts AS 
WITH actor_counts AS (
  SELECT 
    customer_id,
    actor_id,
    first_name,
    last_name,
    COUNT(*) AS rental_count,
    MAX(rental_date) AS latest_rental_date
    FROM actor_joint_dataset
    GROUP BY 
      customer_id,
      actor_id,
      first_name,
      last_name
),
ranked_actor_counts AS (
  SELECT
    actor_counts.*,
    DENSE_RANK() OVER(
      PARTITION BY customer_id
      ORDER BY 
        rental_count DESC, 
        latest_rental_date DESC,
        first_name,
        last_name
    ) AS actor_rank
  FROM actor_counts  

)
SELECT 
  customer_id,
  actor_id,
  first_name,
  last_name,
  rental_count
FROM ranked_actor_counts
WHERE actor_rank = 1;

SELECT *
FROM top_actor_counts
LIMIT 5;
```
| customer_id | actor_id | first_name | last_name | rental_count |
|-------------|----------|------------|-----------|--------------|
| 1           | 37       | VAL        | BOLGER    | 6            |
| 2           | 107      | GINA       | DEGENERES | 5            |
| 3           | 150      | JAYNE      | NOLTE     | 4            |
| 4           | 102      | WALTER     | TORN      | 4            |
| 5           | 12       | KARL       | BERRY     | 4            |

# Actor Recommendations

## Actor Film Counts
The aggregated total rental counts across all customers by ``actor_id`` and ``film_id`` was generated, so that it could be joined onto the ``top_actor_counts`` later on for the final actor recommendations.

To do this a split agregation was performed by doing a CTE with the film counts from the ``actor_joint_dataset`` and afterwards a left join was done between the ``actor_joint_dataset`` and the film_counts CTE to get the right ``rental_count`` values.

The ``DISTINCT`` for the ``film_id`` in the second part after the CTE, had to be used to get rid of duplicates because different ``rental_id``'s had the same ``film_id`` and ``actor_id`` values.

```sql
DROP TABLE IF EXISTS actor_film_counts;
CREATE TEMP TABLE actor_film_counts AS 
WITH film_counts AS (
  SELECT 
    film_id,
    COUNT(DISTINCT rental_id) AS rental_count
  FROM actor_joint_dataset
  GROUP BY film_id
)
SELECT DISTINCT
  actor_joint_dataset.film_id,
  actor_joint_dataset.actor_id,
  actor_joint_dataset.title,
  film_counts.rental_count
FROM actor_joint_dataset
LEFT JOIN film_counts
  ON actor_joint_dataset.film_id = film_counts.film_id;
  
SELECT *
FROM actor_film_counts
WHERE film_id = 80;  
```

| film_id | actor_id | title           | rental_count |
|---------|----------|-----------------|--------------|
| 80      | 16       | BLANKET BEVERLY | 12           |
| 80      | 173      | BLANKET BEVERLY | 12           |
| 80      | 193      | BLANKET BEVERLY | 12           |
| 80      | 200      | BLANKET BEVERLY | 12           |

## Actor Film Exclusions

The actor film exclusions was created by doing ``UNION`` between the films that the customers already watched and the film recommendations that were madesd for the customer's top category. 

By doing the actor film exclusions table, the recommendation of the same films is avoided later on for the final actor film recommendations.

```sql
DROP TABLE IF EXISTS actor_film_exclusions;
CREATE TEMP TABLE actor_film_exclusions AS 
(
  SELECT DISTINCT
    customer_id,
    film_id
  FROM complete_join_dataset  
)

-- we use a UNION to combine the previously watched and the recommended films!
UNION

(
  SELECT DISTINCT 
    customer_id,
    film_id
  FROM category_recommendations  
);

SELECT *
FROM actor_film_exclusions
LIMIT 5;
```
| customer_id | film_id |
|-------------|---------|
| 493         | 567     |
| 114         | 789     |
| 596         | 103     |
| 176         | 121     |
| 459         | 724     |

## Final Actor Recommendations

The final actor recommendations were generated by doing an inner join between the ``top_actor_counts`` and ``actor_film_counts`` tables. Afterwards, an anti join between the previously combined tables and the ``actor_film_exclussions`` table was applied to remove any records from the joined records which are part of the exclusions.

Next, the ``DENSE_RANK`` window function was used to rank the films by ``rental_count`` for each of the top actor per customer. In case of any ``rental_count`` ties the order was by title alphabetically.

At the end a filter was applied to keep only the top 3 films for the top actor of each customer.


```sql
DROP TABLE IF EXISTS actor_recommendations;
CREATE TEMP TABLE actor_recommendations AS 
WITH ranked_films_cte AS (
  SELECT 
    top_actor_counts.customer_id,
    top_actor_counts.first_name,
    top_actor_counts.last_name,
    top_actor_counts.rental_count,
    actor_film_counts.title,
    actor_film_counts.film_id,
    actor_film_counts.actor_id,
    DENSE_RANK() OVER(
      PARTITION BY 
        top_actor_counts.customer_id
      ORDER BY 
        actor_film_counts.rental_count DESC,
        actor_film_counts.title
    ) AS reco_rank
    FROM top_actor_counts
    INNER JOIN actor_film_counts
      ON top_actor_counts.actor_id = actor_film_counts.actor_id
    WHERE NOT EXISTS (
      SELECT 1
      FROM actor_film_exclusions
      WHERE
        actor_film_exclusions.customer_id = top_actor_counts.customer_id
        AND actor_film_exclusions.film_id = actor_film_counts.film_id
    )  
)
SELECT * 
FROM ranked_films_cte
WHERE reco_rank <= 3;

SELECT *
FROM actor_recommendations
ORDER BY customer_id, reco_rank
LIMIT 5;
```
| customer_id | first_name | last_name | rental_count | title             | film_id | actor_id | reco_rank |
|-------------|------------|-----------|--------------|-------------------|---------|----------|-----------|
| 1           | VAL        | BOLGER    | 6            | PRIMARY GLASS     | 697     | 37       | 1         |
| 1           | VAL        | BOLGER    | 6            | ALASKA PHANTOM    | 12      | 37       | 2         |
| 1           | VAL        | BOLGER    | 6            | METROPOLIS COMA   | 572     | 37       | 3         |
| 2           | GINA       | DEGENERES | 5            | GOODFELLAS SALUTE | 369     | 107      | 1         |
| 2           | GINA       | DEGENERES | 5            | WIFE TURN         | 973     | 107      | 2         |

## Final Output 
In the next section all of these individual outputs were combined into one dataset so that the marketing team can access to all the requierements from the email template in one place. 

Please click on the link below to go to the next section.

[![forthebadge](view-final-output.svg)](https://github.com)

