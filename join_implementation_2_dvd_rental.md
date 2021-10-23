# Join Implementation Part 2

The focus for the second part was on generating a base table for the actor insights.

| customer_id | rental_id | rental_date              | film_id | title           | actor_id | first_name | last_name |
|-------------|-----------|--------------------------|---------|-----------------|----------|------------|-----------|
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 200      | THORA      | TEMPLE    |
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 193      | BURT       | TEMPLE    |
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 173      | ALAN       | DREYFUSS  |
| 130         | 1         | 2005-05-24T22:53:30.000Z | 80      | BLANKET BEVERLY | 16       | FRED       | COSTNER   |
| 459         | 2         | 2005-05-24T22:54:33.000Z | 333     | FREAKY POCUS    | 147      | FAY        | WINSLET   |

The steps for generating the table above were exactly the same as for the first part. This join implementation was faster because some of the work done in the first part created a head start for the table joining journey of this part.

## Table Journey

The table joins for this part were from table 1 to table 7 but tables 4 and 5 were not consired for the join.

![erd](erd_dvd_rental.png)

**The table joining journey for the actor insights base table:**

| Join Journey Part | Start         | End           | Foreign Key  |
|-------------------|---------------|---------------|--------------|
| Part 1            | ``rental``        | ``inventory``     | ``inventory_id`` |
| Part 2            | ``inventory``     | ``film``          | ``film_id``      |
| Part 3            | ``film``          | ``film_actor`` | ``film_id``      |
| Part 4            | ``film_actor`` | ``actor``      | ``actor_id``  |

The analysis begins from part 3 because parts 1 and 2 of the joining journey were already analyzed in the join implementation - part 1.

## Join Journey Part 3

### 1. What is the purpose of joining these two tables?

Match the films on ``film_id`` to obtain the ``actor_id`` from each film.

### a. & b. What contextual hypotheses do we have about the data and how can we validate it?

**Hypothesis 1:**

> **The number of unique ``film_id`` records will be equal in both ``film`` and ``film_actor`` tables**

**Film Table**

```sql
SELECT 
  COUNT(DISTINCT film_id)
FROM dvd_rentals.film;
```
| count      |
|------------|
| 1000        |

**Film_actor Table**

```sql
SELECT 
  COUNT(DISTINCT film_id)
FROM dvd_rentals.film_actor;
```
| count      |
|------------|
| 997        |

**Findings**

The ``film_actor`` table had 3 ``film_id`` values less than the film table 

**Hypothesis 2:**

> **There will be a multiple records per unique ``film_id`` in the ``film_actor`` table**

```sql
-- generate group by counts on the target_column_values column
WITH counts_base AS(
  SELECT 
    film_id,
    COUNT(*) AS row_counts
  FROM dvd_rentals.film_actor
  GROUP BY film_id
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT 
  row_counts,
  COUNT(film_id) AS count_film_ids
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | count_film_ids |
|------------|----------------|
| 1          | 21             |
| 2          | 69             |
| 3          | 119            |
| 4          | 137            |
| 5          | 195            |
| 6          | 150            |
| 7          | 119            |
| 8          | 90             |
| 9          | 49             |
| 10         | 21             |
| 11         | 14             |
| 12         | 6              |
| 13         | 6              |
| 15         | 1              |

**Findings**

There were multiple records per ``film_id`` value in the ``film_actor`` table.

### 2. What is the distribution of foreign keys within each table?

**Film Table**

```sql
-- generate group by counts on the target_column_values column
WITH counts_base AS(
  SELECT 
    film_id,
    COUNT(*) AS row_counts
  FROM dvd_rentals.film
  GROUP BY film_id
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT 
  row_counts,
  COUNT(film_id) AS count_film_ids
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | unique_film_id_values |
|------------|-----------------------|
| 1          | 1000                  |

**Film_actor Table**

```sql
-- generate group by counts on the target_column_values column
WITH counts_base AS(
  SELECT 
    film_id,
    COUNT(*) AS row_counts
  FROM dvd_rentals.film_actor
  GROUP BY film_id
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT 
  row_counts,
  COUNT(film_id) AS count_film_ids
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | count_film_ids |
|------------|----------------|
| 1          | 21             |
| 2          | 69             |
| 3          | 119            |
| 4          | 137            |
| 5          | 195            |
| 6          | 150            |
| 7          | 119            |
| 8          | 90             |
| 9          | 49             |
| 10         | 21             |
| 11         | 14             |
| 12         | 6              |
| 13         | 6              |
| 15         | 1              |

**Findings**

There is a 1-to-1 relationship for the ``film_id`` in the ``film`` table and there is a 1-to-many relationship for the ``film_id`` in the ``film_actor`` table.

### 3. How many overlapping and missing unique foreign key values are there between the two tables?

**Film Table**

By using an anti join, the foreign keys which only exist in the ``film`` table and not in the ``film_actor`` table can be seen.

```sql
-- how many foreign keys only exist in the left table and not in the right?
SELECT
  COUNT(DISTINCT film_id)
FROM dvd_rentals.film
WHERE NOT EXISTS (
  SELECT film_id
  FROM dvd_rentals.film_actor
  WHERE film.film_id = film_actor.film_id
);  
```
| count |
|-------|
| 3     |

These 3 values were:

| film_id | title            | description                                                                                                      |
|---------|------------------|------------------------------------------------------------------------------------------------------------------|
| 257     | DRUMLINE CYCLONE | A Insightful Panorama of a Monkey And a Sumo Wrestler who must Outrace a   Mad Scientist in The Canadian Rockies |
| 323     | FLIGHT LIES      | A Stunning Character Study of a Crocodile And a Pioneer who must Pursue a   Teacher in New Orleans               |
| 803     | SLACKER LIAISONS | A Fast-Paced Tale of a A Shark And a Student who must Meet a Crocodile in   Ancient China                        |

These 3 films have missing data regarding its actors.

**Film_actor Table**

The unique foreign keys that only exists in the ``film_actor`` table and not in the ``film`` table were also checked

```sql
SELECT
  COUNT(DISTINCT film_id)
FROM dvd_rentals.film_actor
WHERE NOT EXISTS (
  SELECT film_id
  FROM dvd_rentals.film
  WHERE film_actor.film_id = film.film_id 
);  
```

| count |
|-------|
| 0     |

**Findings**

The ``film_id`` values in the ``film_actor`` table also exist in the ``film`` table but there are 3 ``film_id`` values from the ``film`` table that do not exist in the ``film_actor`` table.

### Left Join or Inner Join?

Both left and inner joins were implemented to validate that the raw row counts output would be different for the inner and left join.

```sql
DROP TABLE IF EXISTS left_film_join;
CREATE TEMP TABLE left_film_join AS
SELECT 
  film.film_id,
  film_actor.actor_id
FROM dvd_rentals.film
LEFT JOIN dvd_rentals.film_actor
  ON film.film_id = film_actor.film_id;
  
DROP TABLE IF EXISTS inner_film_join;
CREATE TEMP TABLE inner_film_join AS 
SELECT 
  film.film_id,
  film_actor.actor_id
FROM dvd_rentals.film
INNER JOIN dvd_rentals.film_actor
  ON film.film_id = film_actor.film_id;
  
-- check the counts for each output
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_key_values
  FROM left_film_join
)

UNION 

(
  SELECT
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT film_id) AS unique_key_values
  FROM inner_film_join
)
```


| join_type  | record_count | unique_key_values |
|------------|--------------|-------------------|
| inner join | 5462         | 997               |
| left join  | 5465         | 1000              |

For this join it made more sense to use the inner join because the left join would include the films that had missing data from the actors. 

## Join Journey Part 4

### 1. What is the purpose of joining these two tables?

Match the actors on ``actor_id`` to obtain the ``first_name`` and ``last_name`` from each actor.

### a. & b. What contextual hypotheses do we have about the data and how can we validate it?

**Hypothesis 1:**

> **The number of unique ``actor_id`` records will be equal in both ``film_actor`` and ``actor`` tables**

**Film_actor Table**

```sql
SELECT 
  COUNT(DISTINCT actor_id)
FROM dvd_rentals.film_actor;
```
| count      |
|------------|
| 200        |

**Actor Table**

```sql
SELECT 
  COUNT(DISTINCT actor_id)
FROM dvd_rentals.actor;
```
| count      |
|------------|
| 200        |

**Findings**

Both of the tables had the same distinct ``film_id`` values.

**Hypothesis 2:**

> **There will be a multiple records per unique ``actor_id`` in the ``film_actor`` table**

```sql
-- generate group by counts on the target_column_values column
WITH counts_base AS(
  SELECT 
    actor_id,
    COUNT(*) AS row_counts
  FROM dvd_rentals.film_actor
  GROUP BY actor_id
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT 
  row_counts,
  COUNT(actor_id) AS count_actor_ids
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```
| row_counts | count_actor_ids |
|------------|-----------------|
| 14         | 1               |
| 15         | 2               |
| 16         | 1               |
| 18         | 2               |
| 19         | 4               |
| 20         | 9               |
| 21         | 5               |
| 22         | 10              |
| 23         | 8               |
| 24         | 14              |
| 25         | 19              |
| 26         | 14              |
| 27         | 17              |
| 28         | 11              |
| 29         | 11              |
| 30         | 16              |
| 31         | 16              |
| 32         | 10              |
| 33         | 13              |
| 34         | 5               |
| 35         | 6               |
| 36         | 1               |
| 37         | 1               |
| 39         | 1               |
| 40         | 1               |
| 41         | 1               |
| 42         | 1               |

**Findings**

There were multiple records per ``actor_id`` value in the ``film_actor`` table.

### 2. What is the distribution of foreign keys within each table?

**Film_actor Table**

```sql
-- generate group by counts on the target_column_values column
WITH counts_base AS(
  SELECT 
    actor_id,
    COUNT(*) AS row_counts
  FROM dvd_rentals.film_actor
  GROUP BY actor_id
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT 
  row_counts,
  COUNT(film_id) AS count_actor_ids
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | count_actor_ids |
|------------|-----------------|
| 14         | 1               |
| 15         | 2               |
| 16         | 1               |
| 18         | 2               |
| 19         | 4               |
| 20         | 9               |
| 21         | 5               |
| 22         | 10              |
| 23         | 8               |
| 24         | 14              |
| 25         | 19              |
| 26         | 14              |
| 27         | 17              |
| 28         | 11              |
| 29         | 11              |
| 30         | 16              |
| 31         | 16              |
| 32         | 10              |
| 33         | 13              |
| 34         | 5               |
| 35         | 6               |
| 36         | 1               |
| 37         | 1               |
| 39         | 1               |
| 40         | 1               |
| 41         | 1               |
| 42         | 1               |

**Actor Table**

```sql
-- generate group by counts on the target_column_values column
WITH counts_base AS(
  SELECT 
    actor_id,
    COUNT(*) AS row_counts
  FROM dvd_rentals.actor
  GROUP BY film_id
)
-- summarize the group by counts above by grouping again on the row_counts from counts_base CTE part
SELECT 
  row_counts,
  COUNT(actor_id) AS count_film_ids
FROM counts_base
GROUP BY row_counts
ORDER BY row_counts;
```

| row_counts | count_film_ids |
|------------|----------------|
| 1          | 200            |

**Findings**

There is a 1-to-many relationship for the ``actor_id`` in the ``film_actor`` table and there is a 1-to-1 relationship for the ``actor_id`` in the ``actor`` table.

### 3. How many overlapping and missing unique foreign key values are there between the two tables?

**Film_actor Table**

By using an anti join, the foreign keys which only exist in the ``film_actor`` table and not in the ``actor`` table can be seen.

```sql
-- how many foreign keys only exist in the left table and not in the right?
SELECT
  COUNT(DISTINCT actor_id)
FROM dvd_rentals.film_actor
WHERE NOT EXISTS (
  SELECT actor_id
  FROM dvd_rentals.actor
  WHERE film_actor.actor_id = film_actor.actor_id
);  
```
| count |
|-------|
| 0     |

**Actor Table**

The unique foreign keys that only exists in the ``actor`` table and not in the ``film_actor`` table were also checked

```sql
SELECT
  COUNT(DISTINCT actor_id)
FROM dvd_rentals.actor
WHERE NOT EXISTS (
  SELECT actor_id
  FROM dvd_rentals.film_actor
  WHERE actor.actor_id = film_actor.actor_id
);   
```
| count |
|-------|
| 0     |

**Findings**

The same ``actor_id`` values that were in the ``film_category`` table were also in the ``actor`` table and vice versa.

### Left Join or Inner Join?

The left and inner joins were implemented to validate that the raw row counts output was the same.

```sql
DROP TABLE IF EXISTS left_film_actor_join;
CREATE TEMP TABLE left_film_actor_join AS 
SELECT 
  film_actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.film_actor
LEFT JOIN dvd_rentals.actor
  ON film_actor.actor_id = actor.actor_id;
  
DROP TABLE IF EXISTS inner_film_actor_join;
CREATE TEMP TABLE inner_film_actor_join AS 
SELECT 
  film_actor.actor_id,
  actor.first_name,
  actor.last_name
FROM dvd_rentals.film_actor
INNER JOIN dvd_rentals.actor
  ON film_actor.actor_id = actor.actor_id;
  
(
  SELECT
    'left join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT actor_id) AS unique_key_values 
  FROM left_film_actor_join
)  

UNION ALL 

(
  SELECT 
    'inner join' AS join_type,
    COUNT(*) AS record_count,
    COUNT(DISTINCT actor_id) AS unique_key_values
  FROM inner_film_actor_join  
)
```

| join_type  | record_count | unique_key_values |
|------------|--------------|-------------------|
| left join  | 5462         | 200               |
| inner join | 5462         | 200               |

As seen in the table above, the output was the same for both joins.

## Final Ouput 

The table joins to create the base table for the actor insights is shown below

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

## Problem Solving
In the next section all the requierement outputs were generated with the use of the base tables made with these join implementations.

Please click on the link below to go to the next section.

[![forthebadge](view-problem-solving.svg)](https://github.com)