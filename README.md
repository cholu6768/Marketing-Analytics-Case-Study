# DVD Rental Co - Marketing Analytics Case Study 💿

> **This case study can be found in the [Serious SQL](https://www.datawithdanny.com) course made by [Danny Ma](https://www.linkedin.com/in/datawithdanny/).**

# Table of contents

- 💿 [About](#️-about)

- 📋 [Requirements](#-requirements)
    * [Requirement #1](#-requirement_#1:_top_2_categories)
    * [Requirement #2](#-requirement_#2:_category_film_recommendations)
    * [Requirements #3 & #4](#-requirements_#3_&_#4:_individual_customer_insights)
    * [Requirement #5](#-requirement_#5:_favorite_actor_recommendation)

- 🔎 [Data Exploration](#-data-exploration)

- ⚙️ [Problem Approach](#-problem-approach)

- 🧱 [Join Implementation](#-join-implementation)
    - [Part 1](#-part-1)
    - [Part 2](#-part-2)
   
- 🔧 [Problem Solving](#️-problem-solving)

- 🔮 [Final Output](#-final-output)


# 💿 About 

The DVD Rental Co customer analytics  asked for our support. The team needed to generate  the necessary data points required to populate specific parts of this first-ever customer email campaign.

# 📋 Requirements

The marketing team shared with us an email template of what they wanted to send to their customers.

![template](email_template.png)

## 📋 Requirement #1: Top 2 Categories
The top 2 categories for each customer needs to be identified based off their past rental history.

![requirement-1](requirement_1.png)

## 📋 Requirement #2: Category Film Recommendations

The marketing team need to recommend the 3 most popular films for each customer's top 2 categories. 

⚠️In case only one film can be recommended, the marketing team would be fine with it.

❗️ A film can not be recommended if the customer has already watched it.

❗️ If the customer does not have any film recommendations for either category, the customer must be excluded from the email campaign.

![requirement-2](requirement_2.png)

## 📋 Requirements #3 & #4: Individual Customer Insights 

For the requirement #3, the following insights are needed on the first category:

**1.** How mny total films have they watched in their top category?

**2.** How many more films has the customer watched compared to the average DVD Rental Co customer?

**3.** How does the customer rank in terms of the top X% compared to all other customers in this film category?

For the requirement #4, the following insights are needed on the second category:

**1.** How many total films has the customer watched in this category?

**2.** What proportion of each customers's total films watched does the count make?

![requirement-3-4](requirement_3_4.png)

## 📋 Requirement #5: Favorite Actor Recommendation

Marketing has also requested for the top actor film recommendations where up to 3 more films are included in the recommendations list as well as the count of films by the top actor.

❗️ If there are any ties the actors should be choosen in alphabetical order.

❗️ Any films that have already been recommended in the top 2 categories must not be included as actor recommendations.

❗️ If the customer does not have at least 1 film recommendation - they also need to be flagged with a separate actor exclusion flag.

![requirement-5](requirement_5.png)

# 🔎 Data Exploration

The analytics team provided us with an entity-relationship diagram (ERD).

<iframe height="400" width="100%"
src='https://dbdiagram.io/embed/5fe1cb6e9a6c525a03bbf839'>
</iframe>

The 7 tables from the ERD were explored in the link below.

[![forthebadge](view-data-exploration.svg)](https://github.com)

# ⚙️ Problem Approach 

In this section the problem approach was shown and the key columns from the ERD were identified for proceeding with the table joins.

[![forthebadge](view-problem-approach.svg)](https://github.com)

# 🧱 Join Implementation

In this section the approach on doing the joins is shown and also how the base tables for the category and actor insights were generated.

There are two parts, the first part shows the base table for the category insights while the second part shows the base table for the actor insights.

### Part 1

[![forthebadge](view-join-implementation-part-1.svg)](https://github.com)

### Part 2

[![forthebadge](view-join-implementation-part-2.svg)](https://github.com)

# 🔧 Problem Solving 
In this section all of the requirements outputs were generated using the base tables previously made in the join implementations.

[![forthebadge](view-problem-solving.svg)](https://github.com)

# 🔮 Final Ouput
In this final section all of the individual outputs previously made in the problem solving section were combined into one dataset so that the marketing team can have access to all the requierements from the email template in one place. 

[![forthebadge](view-final-output.svg)](https://github.com)
