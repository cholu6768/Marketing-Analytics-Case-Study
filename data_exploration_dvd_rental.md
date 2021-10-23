# Data Exploration

In this section the data of each table from the ERD was explored.

![erd](erd_dvd_rental.png)

There are 7 tables in the ERD:
- ``rental``
- ``inventory``
- ``film``
- ``film_category``
- ``category``
- ``film_actor``
- ``actor``


## Table #1 - rental 

This is the first table of the ERD. 

The output was limited to only 5 rows to get a preview of the ``rental`` table. 

```sql
SELECT * 
FROM dvd_rentals.rentals 
LIMIT 5;
```

| rental_id | rental_date              | inventory_id | customer_id | return_date              | staff_id | last_update              |
|-----------|--------------------------|--------------|-------------|--------------------------|----------|--------------------------|
| 1         | 2005-05-24T22:53:30.000Z | 367          | 130         | 2005-05-26T22:04:30.000Z | 1        | 2006-02-15T21:30:53.000Z |
| 2         | 2005-05-24T22:54:33.000Z | 1525         | 459         | 2005-05-28T19:40:33.000Z | 1        | 2006-02-15T21:30:53.000Z |
| 3         | 2005-05-24T23:03:39.000Z | 1711         | 408         | 2005-06-01T22:12:39.000Z | 1        | 2006-02-15T21:30:53.000Z |
| 4         | 2005-05-24T23:04:41.000Z | 2452         | 333         | 2005-06-03T01:43:41.000Z | 2        | 2006-02-15T21:30:53.000Z |
| 5         | 2005-05-24T23:05:21.000Z | 2079         | 222         | 2005-06-02T04:33:21.000Z | 1        | 2006-02-15T21:30:53.000Z |

This table contains the customer's rental history.

There is a unique sequential ``rental_id`` for each record in the table which corresponds to an individual ``customer_id`` and renting a specific item with an ``inventory_id``.

There is also information about the ``rental_date`` and ``return_date`` as well as which staff members served the customers.

In the ERD there is a linkage between this ``rental`` table with the ``inventory`` table via the ``inventory_id`` field.

A ``COUNT`` was used to do a row count of the ``rental`` table.

```sql
SELECT 
  COUNT(*) AS count_rentals
FROM dvd_rentals.rental;
```  
| count_rentals |
|---------------|
| 16044         |

There were a total of 16,044 rentals.

## Table #2 - inventory

This is the second table of the ERD.

The output was limited to only 5 rows to get a preview of the ``inventory`` table.

```sql
SELECT *
FROM dvd_rentals.inventory
LIMIT 5;
```
| inventory_id | film_id | store_id | last_update              |
|--------------|---------|----------|--------------------------|
| 1            | 1       | 1        | 2006-02-15T05:09:17.000Z |
| 2            | 1       | 1        | 2006-02-15T05:09:17.000Z |
| 3            | 1       | 1        | 2006-02-15T05:09:17.000Z |
| 4            | 1       | 1        | 2006-02-15T05:09:17.000Z |
| 5            | 1       | 2        | 2006-02-15T05:09:17.000Z |

The ``inventory`` table consists of specific items available for rental at each store. On the table above it can be seen that there can be multiple inventory items for a specific film at a unique store.

The ``inventory`` table is linked to the previous ``rental`` table via the ``inventory_id`` and also has a linkage with the ``film`` table via the ``film_id`` field.

A ``COUNT`` was used to do a row count of the ``inventory`` table.

```sql
SELECT
  COUNT(*) AS count_items
FROM dvd_rentals.inventory;
```  
| count_items |
|-------------|
| 4581        |

There were a total of 4581 items in the ``inventory`` table.

## Table #3 - film

This is the third table of the ERD.

The output was limited to only 5 rows to get a preview of the ``film`` table.

```sql
SELECT *
FROM dvd_rentals.film
LIMIT 5;
```
| film_id | title            | description                                                                                                             | release_year | language_id | original_language_id | rental_duration | rental_rate | length | replacement_cost | rating | last_update              | special_features                 | fulltext                                                                                                                                                                           |
|---------|------------------|-------------------------------------------------------------------------------------------------------------------------|--------------|-------------|----------------------|-----------------|-------------|--------|------------------|--------|--------------------------|----------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 1       | ACADEMY DINOSAUR | A Epic Drama of a Feminist And a Mad Scientist who must Battle a Teacher   in The Canadian Rockies                      | 2006         | 1           |                      | 6               | 0.99        | 86     | 20.99            | PG     | 2006-02-15T05:03:42.000Z | Deleted Scenes,Behind the Scenes | 'academi':1 'battl':15 'canadian':20 'dinosaur':2 'drama':5 'epic':4   'feminist':8 'mad':11 'must':14 'rocki':21 'scientist':12 'teacher':17                                      |
| 2       | ACE GOLDFINGER   | A Astounding Epistle of a Database Administrator And a Explorer who must   Find a Car in Ancient China                  | 2006         | 1           |                      | 3               | 4.99        | 48     | 12.99            | G      | 2006-02-15T05:03:42.000Z | Trailers,Deleted Scenes          | 'ace':1 'administr':9 'ancient':19 'astound':4 'car':17 'china':20   'databas':8 'epistl':5 'explor':12 'find':15 'goldfing':2 'must':14                                           |
| 3       | ADAPTATION HOLES | A Astounding Reflection of a Lumberjack And a Car who must Sink a   Lumberjack in A Baloon Factory                      | 2006         | 1           |                      | 7               | 2.99        | 50     | 18.99            | NC-17  | 2006-02-15T05:03:42.000Z | Trailers,Deleted Scenes          | 'adapt':1 'astound':4 'baloon':19 'car':11 'factori':20 'hole':2   'lumberjack':8,16 'must':13 'reflect':5 'sink':14                                                               |
| 4       | AFFAIR PREJUDICE | A Fanciful Documentary of a Frisbee And a Lumberjack who must Chase a   Monkey in A Shark Tank                          | 2006         | 1           |                      | 5               | 2.99        | 117    | 26.99            | G      | 2006-02-15T05:03:42.000Z | Commentaries,Behind the Scenes   | 'affair':1 'chase':14 'documentari':5 'fanci':4 'frisbe':8   'lumberjack':11 'monkey':16 'must':13 'prejudic':2 'shark':19 'tank':20                                               |
| 5       | AFRICAN EGG      | A Fast-Paced Documentary of a Pastry Chef And a Dentist who must Pursue a   Forensic Psychologist in The Gulf of Mexico | 2006         | 1           |                      | 6               | 2.99        | 130    | 22.99            | G      | 2006-02-15T05:03:42.000Z | Deleted Scenes                   | 'african':1 'chef':11 'dentist':14 'documentari':7 'egg':2 'fast':5   'fast-pac':4 'forens':19 'gulf':23 'mexico':25 'must':16 'pace':6 'pastri':10   'psychologist':20 'pursu':17 |

The ``film`` table consists of information about the availble films. For example, there is information about the title, description, and rental rate of the films.

The ``film`` table is linked to the  ``inventory``, ``film_category`` and ``film_actor`` tables via the ``film_id`` field.

A ``COUNT`` was used to do a row count of the ``film`` table.

```sql
SELECT 
  COUNT(*) AS count_films
FROM dvd_rentals.film;
```
| count_films |
|-------------|
| 1000        |

There were 1000 different films in the ``film`` table.

## Table #4 - film_category

This is the fourth table of the ERD.

The output was limited to only 5 rows to get a preview of the ``film_category`` table.

```sql
SELECT * 
FROM dvd_rentals.film_category
LIMIT 5;
```

| film_id | category_id | last_update              |
|---------|-------------|--------------------------|
| 1       | 6           | 2006-02-15T05:07:09.000Z |
| 2       | 11          | 2006-02-15T05:07:09.000Z |
| 3       | 6           | 2006-02-15T05:07:09.000Z |
| 4       | 11          | 2006-02-15T05:07:09.000Z |
| 5       | 8           | 2006-02-15T05:07:09.000Z |

The ``film_category`` table shows the ``category_id`` for the ``film_id``.

The ``film_category`` table is linked to the ``film`` table via the ``film_id`` field and also has a link with the ``category`` table via the ``category_id`` field.

## Table #5 - category

This is the fifth table of the ERD.

The output was limited to only 5 rows to get a preview of the ``category`` table.

```sql
SELECT * 
FROM dvd_rentals.category
LIMIT 5;
```

| category_id | name      | last_update              |
|-------------|-----------|--------------------------|
| 1           | Action    | 2006-02-15T04:46:27.000Z |
| 2           | Animation | 2006-02-15T04:46:27.000Z |
| 3           | Children  | 2006-02-15T04:46:27.000Z |
| 4           | Classics  | 2006-02-15T04:46:27.000Z |
| 5           | Comedy    | 2006-02-15T04:46:27.000Z |

The ``category`` table contains a one to one mapping between the ``category_id`` and the name of each category.

The ``category`` table has a link with the ``film_category`` table via the ``category_id`` field.

A ``COUNT`` was used to do a row count of the ``categories`` table.

```sql
SELECT
  COUNT(name) AS categories
FROM dvd_rentals.category;
```  
| categories |
|------------|
| 16         |

There were a total of 16 categories in the ``category`` table.

## Table #6 - film_actor

This is the sixth table of the ERD.

The output was limited to only 5 rows to get a preview of the ``film_actor`` table.

```sql
SELECT *
FROM dvd_rentals.film_actor
LIMIT 5;
```

| actor_id | film_id | last_update              |
|----------|---------|--------------------------|
| 1        | 1       | 2006-02-15T05:05:03.000Z |
| 1        | 23      | 2006-02-15T05:05:03.000Z |
| 1        | 25      | 2006-02-15T05:05:03.000Z |
| 1        | 106     | 2006-02-15T05:05:03.000Z |
| 1        | 140     | 2006-02-15T05:05:03.000Z |

The ``film_actor`` table shows the actors who appeared in the films.

The ``film_actor`` table has a link with the ``film`` table via the ``film_id`` field and also has a linkage with the ``actor`` table via the ``actor_id`` field.

The actors that starred for ``film_id`` 23 were shown to check more about the relationship in the data.

```sql
SELECT *
FROM dvd_rentals.film_actor
WHERE film_id = 23;
```

| actor_id | film_id | last_update              |
|----------|---------|--------------------------|
| 1        | 23      | 2006-02-15T05:05:03.000Z |
| 4        | 23      | 2006-02-15T05:05:03.000Z |
| 22       | 23      | 2006-02-15T05:05:03.000Z |
| 150      | 23      | 2006-02-15T05:05:03.000Z |
| 164      | 23      | 2006-02-15T05:05:03.000Z |

As seen in the two tables above, films can have multiple actors and an actor can be in different films which showed a many-to-many relationship. 

## Table #7 - actor

This is the seventh and last table of the ERD.

The output was limited to only 5 rows to get a preview of the ``actor`` table.

```sql
SELECT *
FROM dvd_rentals.actor
LIMIT 5;
```

| actor_id | first_name | last_name    | last_update              |
|----------|------------|--------------|--------------------------|
| 1        | PENELOPE   | GUINESS      | 2006-02-15T04:34:33.000Z |
| 2        | NICK       | WAHLBERG     | 2006-02-15T04:34:33.000Z |
| 3        | ED         | CHASE        | 2006-02-15T04:34:33.000Z |
| 4        | JENNIFER   | DAVIS        | 2006-02-15T04:34:33.000Z |
| 5        | JOHNNY     | LOLLOBRIGIDA | 2006-02-15T04:34:33.000Z |

The ``actor`` table contains the first and last names of the actors, also each actor has their own unique ``actor_id``.

The ``actor`` table has a link with the previous table (film_actor) via the ``actor_id ``field.

## Problem Approach
The next section talked about how the problem is going to be approached by using reverse engineering. 

Please make sure to click on the link below to go to the next section. 

[![forthebadge](view-problem-approach.svg)](https://github.com)