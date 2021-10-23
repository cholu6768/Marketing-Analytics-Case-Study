# Final Ouput 

As talked in the problem approach section, the final output would have to look like this:

| customer_id | cat_1    | cat_1_reco_1        | cat_1_reco_2      | cat_1_reco_3      | cat_2     | cat_2_reco_1   | cat_2_reco_2   | cat_2_reco_3        | actor          | actor_reco_1      | actor_reco_2         | actor_reco_3    | insight_cat_1                                                                                                                   | insight_cat_2                                                                    | insight_actor                                                                                                         |
|-------------|----------|---------------------|-------------------|-------------------|-----------|----------------|----------------|---------------------|----------------|-------------------|----------------------|-----------------|---------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------|
| 1           | Classics | Timberland Sky      | Gilmore Boiled    | Voyage Legally    | Comedy    | Zorro Ark      | Cat Coneheads  | Operation Operation | Val Bolger     | Primary Glass     | Alaska Phantom       | Metropolis Coma | You’ve watched 6 Classics films,   that’s 4 more than the DVD Rental Co average and puts you in the top 1% of   Classics gurus! | You’ve watched 5 Comedy films   making up 16% of your entire viewing history!    | You’ve watched 6 films featuring   Val Bolger! Here are some other films Val stars in that might interest you!        |
| 2           | Sports   | Gleaming Jawbreaker | Talented Homicide | Roses Treasure    | Classics  | Frost Head     | Gilmore Boiled | Voyage Legally      | Gina Degeneres | Goodfellas Salute | Wife Turn            | Dogma Family    | You’ve watched 5 Sports films,   that’s 3 more than the DVD Rental Co average and puts you in the top 7% of   Sports gurus!     | You’ve watched 4 Classics films   making up 15% of your entire viewing history!  | You’ve watched 5 films featuring   Gina Degeneres! Here are some other films Gina stars in that might interest   you! |
| 3           | Action   | Rugrats Shakespeare | Suspects Quills   | Handicap Boondock | Animation | Juggler Hardly | Dogma Family   | Storm Happiness     | Jayne Nolte    | English Bulworth  | Sweethearts Suspects | Dancing Fever   | You’ve watched 4 Action films,   that’s 2 more than the DVD Rental Co average and puts you in the top 14% of   Action gurus!    | You’ve watched 3 Animation films   making up 12% of your entire viewing history! | You’ve watched 4 films featuring   Jayne Nolte! Here are some other films Jayne stars in that might interest   you!   |

The tables below which were created on the previous section were used to generate the final output.

**``top_categories_information``**

| customer_id | category_ranking | category_name | rental_count | average_comparison | percentile | category_percentage |
|-------------|------------------|---------------|--------------|---------------------|------------|---------------------|
| 1           | 1                | Classics      | 6            | 4                   | 1          | 19                  |
| 1           | 2                | Comedy        | 5            | 4                   | 1          | 16                  |
| 2           | 1                | Sports        | 5            | 3                   | 3          | 19                  |
| 2           | 2                | Classics      | 4            | 2                   | 2          | 15                  |
| 3           | 1                | Action        | 4            | 2                   | 5          | 15                  |
| 3           | 2                | Sci-Fi        | 3            | 1                   | 15         | 12                  |

**``top_actor_counts``**

| customer_id | actor_id | first_name | last_name | rental_count |
|-------------|----------|------------|-----------|--------------|
| 1           | 37       | VAL        | BOLGER    | 6            |
| 2           | 107      | GINA       | DEGENERES | 5            |
| 3           | 150      | JAYNE      | NOLTE     | 4            |
| 4           | 102      | WALTER     | TORN      | 4            |
| 5           | 12       | KARL       | BERRY     | 4            |

**``category_recommendations``**

| customer_id | category_name | category_ranking | film_id | title               | rental_count | reco_rank |
|-------------|---------------|------------------|---------|---------------------|--------------|-----------|
| 1           | Classics      | 1                | 891     | TIMBERLAND SKY      | 31           | 1         |
| 1           | Classics      | 1                | 358     | GILMORE BOILED      | 28           | 2         |
| 1           | Classics      | 1                | 951     | VOYAGE LEGALLY      | 28           | 3         |
| 1           | Comedy        | 2                | 1000    | ZORRO ARK           | 31           | 1         |
| 1           | Comedy        | 2                | 127     | CAT CONEHEADS       | 30           | 2         |
| 1           | Comedy        | 2                | 638     | OPERATION OPERATION | 27           | 3         |

**``actor_recommendations``**

| customer_id | first_name | last_name | rental_count | title             | film_id | actor_id | reco_rank |
|-------------|------------|-----------|--------------|-------------------|---------|----------|-----------|
| 1           | VAL        | BOLGER    | 6            | PRIMARY GLASS     | 697     | 37       | 1         |
| 1           | VAL        | BOLGER    | 6            | ALASKA PHANTOM    | 12      | 37       | 2         |
| 1           | VAL        | BOLGER    | 6            | METROPOLIS COMA   | 572     | 37       | 3         |
| 2           | GINA       | DEGENERES | 5            | GOODFELLAS SALUTE | 369     | 107      | 1         |
| 2           | GINA       | DEGENERES | 5            | WIFE TURN         | 973     | 107      | 2         |

## Final Output Script

The final output was generated using a combination of CTE's and joins inside a ``TEMP TABLE``. The first and second CTE's generated the insights for the top two categories and the third CTE generated the actor insights. In all of these 3 CTE's queries used ``CONCAT`` to concatenate text and the values from the columns. The last 4 CTE's focused on generating the categories and actor film recommendations. In the 4th CTE the title of the film recommendations was capitalized and the necessary columns for being able to do the next CTE were addded. The 5th or next CTE  pivoted the rows (using a MAX and CASE) from the films into columns, this was done so that each ``customer_id`` row included all the category film recommendations. This same process was applied for the actor film recommendations which was done in the 6th and 7th CTE.

On the final CTE the first 3 CTEs (first_category, second_category, top_actor_counts) and the 5th (wide_category_recommendations) and 7th (wide_actor_recommendations) CTE were joined to generate the final dataset. 

```sql
DROP TABLE IF EXISTS final_data_asset;
CREATE TEMP TABLE final_data_asset AS
WITH first_category AS (
  SELECT 
    customer_id,
    category_name,
    CONCAT(
      'You''ve watched ', rental_count, ' ', category_name,
      ' films,', ' that''s ', average_comparison, 
      ' more than the DVD Rental Co average and puts you in the top ',
      percentile, '% of ', category_name, ' gurus!'
    ) AS insight
  FROM top_categories_information
  WHERE category_ranking = 1
),

second_category AS (
  SELECT 
    customer_id,
    category_name,
    CONCAT(
      'You''ve watched ', rental_count, ' films, making up ',
      category_percentage, '% of your entire viewing history!'
    ) AS insight
  FROM top_categories_information
  WHERE category_ranking = 2  
), 

top_actor AS (
  SELECT 
    customer_id,
    -- use INITCAP to transform names into Title case
    CONCAT(INITCAP(first_name), ' ', INITCAP(last_name)) AS actor_name,
    CONCAT(
      'You''ve watched ', rental_count, ' films featuring ',
      INITCAP(first_name), ' ', INITCAP(last_name), '! ', 
      'Here are some films ', INITCAP(first_name), 
      ' stars in that might interest you!') AS insight
  FROM top_actor_counts    
),

adjusted_title_case_category_recommendations AS (
  SELECT 
    customer_id,
    INITCAP(title) AS title,
    category_ranking,
    reco_rank
  FROM category_recommendations  
),

wide_category_recommendations AS (
  SELECT 
    customer_id,
    -- Use MAX and CASE to pivot the row
    MAX(CASE WHEN category_ranking = 1 AND reco_rank = 1
    THEN title END) AS cat_1_reco_1,
    MAX(CASE WHEN category_ranking = 1 AND reco_rank = 2
    THEN title END) AS cat_1_reco_2,
    MAX(CASE WHEN category_ranking = 1 AND reco_rank = 3
    THEN title END) AS cat_1_reco_3,
    MAX(CASE WHEN category_ranking = 2 AND reco_rank = 1
    THEN title END) AS cat_2_reco_1,
    MAX(CASE WHEN category_ranking = 2 AND reco_rank = 2
    THEN title END) AS cat_2_reco_2,
    MAX(CASE WHEN category_ranking = 2 AND reco_rank = 3
    THEN title END) AS cat_2_reco_3
  FROM adjusted_title_case_category_recommendations
  GROUP BY customer_id
),

adjusted_name_case_actor_recommendations AS (
  SELECT  
    customer_id,
    INITACAP(title) AS title,
    reco_rank
  FROM actor_recommendations  
),

wide_actor_recommendations AS (
  SELECT 
    customer_id,
    MAX(CASE WHEN reco_rank = 1
    THEN title END) AS actor_reco_1,
    MAX(CASE WHEN reco_rank = 2
    THEN title END) AS actor_reco_2,
    MAX(CASE WHEN reco_rank = 3
    THEN title END) AS actor_reco_3
  FROM adjusted_name_case_actor_recommendations
  GROUP BY customer_id
),

final_output AS (
  SELECT 
    t1.customer_id,
    t1.category_name AS cat_1,
    t4.cat_1_reco_1,
    t4.cat_1_reco_2,
    t4.cat_1_reco_3,
    t2.category_name AS cat_2,
    t4.cat_2_reco_1,
    t4.cat_2_reco_2,
    t4.cat_2_reco_3,
    t3.actor_name,
    t5.actor_reco_1,
    t5.actor_reco_2,
    t5.actor_reco_3,
    t1.insight AS insight_cat_1,
    t2.insight AS insight_cat_2,
    t3.insight AS insight_actor
  FROM first_category AS t1 
  INNER JOIN second_category AS t2
    ON t1.customer_id = t2.customer_id
  INNER JOIN top_actor AS t3 
    ON t1.customer_id = t3.customer_id
  INNER JOIN wide_category_recommendations AS t4 
    ON t1.customer_id = t4.customer_id
  INNER JOIN wide_actor_recommendations AS t5 
    ON t1.customer_id = t5.customer_id  
)

SELECT *
FROM final_output;

SELECT *
FROM final_data_asset
LIMIT 5;
```

| customer_id | cat_1    | cat_1_reco_1        | cat_1_reco_2      | cat_1_reco_3      | cat_2     | cat_2_reco_1      | cat_2_reco_2     | cat_2_reco_3        | actor_name     | actor_reco_1         | actor_reco_2          | actor_reco_3           | insight_cat_1                                                                                                                 | insight_cat_2                                                         | insight_actor                                                                                                 |
|-------------|----------|---------------------|-------------------|-------------------|-----------|-------------------|------------------|---------------------|----------------|----------------------|-----------------------|------------------------|-------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| 1           | Classics | Timberland Sky      | Gilmore Boiled    | Voyage Legally    | Comedy    | Zorro Ark         | Cat Coneheads    | Operation Operation | Val Bolger     | Primary Glass        | Alaska Phantom        | Metropolis Coma        | You've watched 6 Classics films, that's 4 more than the DVD Rental Co   average and puts you in the top 1% of Classics gurus! | You've watched 5 films, making up 16% of your entire viewing history! | You've watched 6 films featuring Val Bolger! Here are some films Val   stars in that might interest you!      |
| 2           | Sports   | Gleaming Jawbreaker | Talented Homicide | Roses Treasure    | Classics  | Frost Head        | Gilmore Boiled   | Voyage Legally      | Gina Degeneres | Goodfellas Salute    | Wife Turn             | Dogma Family           | You've watched 5 Sports films, that's 3 more than the DVD Rental Co   average and puts you in the top 3% of Sports gurus!     | You've watched 4 films, making up 15% of your entire viewing history! | You've watched 5 films featuring Gina Degeneres! Here are some films Gina   stars in that might interest you! |
| 3           | Action   | Rugrats Shakespeare | Suspects Quills   | Handicap Boondock | Sci-Fi    | Goodfellas Salute | English Bulworth | Graffiti Love       | Jayne Nolte    | Sweethearts Suspects | Dancing Fever         | Invasion Cyclone       | You've watched 4 Action films, that's 2 more than the DVD Rental Co   average and puts you in the top 5% of Action gurus!     | You've watched 3 films, making up 12% of your entire viewing history! | You've watched 4 films featuring Jayne Nolte! Here are some films Jayne   stars in that might interest you!   |
| 4           | Horror   | Pulp Beverly        | Family Sweet      | Swarm Gold        | Drama     | Hobbit Alien      | Harry Idaho      | Witches Panic       | Walter Torn    | Curtain Videotape    | Lies Treatment        | Nightmare Chill        | You've watched 3 Horror films, that's 2 more than the DVD Rental Co   average and puts you in the top 8% of Horror gurus!     | You've watched 2 films, making up 9% of your entire viewing history!  | You've watched 4 films featuring Walter Torn! Here are some films Walter   stars in that might interest you!  |
| 5           | Classics | Timberland Sky      | Frost Head        | Gilmore Boiled    | Animation | Juggler Hardly    | Dogma Family     | Storm Happiness     | Karl Berry     | Virginian Pluto      | Stagecoach Armageddon | Telemark Heartbreakers | You've watched 7 Classics films, that's 5 more than the DVD Rental Co   average and puts you in the top 1% of Classics gurus! | You've watched 6 films, making up 16% of your entire viewing history! | You've watched 4 films featuring Karl Berry! Here are some films Karl   stars in that might interest you!     |

