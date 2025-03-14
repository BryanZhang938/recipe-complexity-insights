# Unraveling Culinary Complexity: A Data-Driven Analysis of Recipe Complexity and User Satisfaction

## Bryan Zhang

*March 13th, 2025*

---

## Introduction

Food plays a crucial role in our daily lives, and cooking serves as a medium for self-expression—allowing individuals to expand their culinary skills and experiment with different cuisines. When browsing the internet or cookbooks for recipes, we often rely on reviews and ratings to gauge their quality. For those who share their recipes online, maintaining a positive reputation is essential as it helps them engage with a wider audience in the food community.

One key factor that food creators consider is the complexity of a recipe, which may influence how well it is received. In this project, we aim to answer the question: **Does the complexity of a recipe significantly influence user ratings?** We define complexity based on three factors: preparation time, the number of ingredients, and the number of steps required to complete a recipe.

To explore this relationship, we will analyze two datasets from *food.com*, an online recipe-sharing platform with over 500,000 user-generated and chef-created dishes. These datasets were originally created for a machine learning model that generates recipes, as highlighted in the research paper *Generating Personalized Recipes from Historical User Preferences* by Majumder et al. One dataset, `raw_interactions`, contains a list of reviews for recipes on the website, while the other dataset, `raw_recipes`, contains detailed information regarding the recipes themselves—including descriptions, ratings, and nutritional information. A more detailed description of the datasets is provided below.

### Interactions Dataset

| Column       | Type     | Description                                          |
|--------------|----------|------------------------------------------------------|
| `user_id`    | `int`    | The ID of the user posting a review.                 |
| `recipe_id`  | `int`    | The ID of the recipe the review is posted under.     |
| `date`       | `Timestamp` | The date when the review was posted.                |
| `rating`     | `int`    | The rating given to the recipe.                      |
| `review`     | `str`    | The content of the user's review.                    |

*731,927 rows, 5 columns*

### Recipes Dataset

| Column           | Type       | Description                                                                                                                                             |
|------------------|------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`           | `str`      | The name of the recipe.                                                                                                                                 |
| `id`             | `int`      | The ID of the recipe.                                                                                                                                   |
| `minutes`        | `int`      | The time required to prepare the recipe (in minutes).                                                                                                  |
| `contributor_id` | `int`      | The ID of the user who posted the recipe.                                                                                                               |
| `submitted`      | `Timestamp`| The date when the recipe was posted.                                                                                                                    |
| `tags`           | `list`     | A list of tags associated with the recipe.                                                                                                              |
| `nutrition`      | `list`     | Nutritional information in the format: \[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)\]. PDV = percentage of daily value. |
| `n_steps`        | `int`      | The number of steps needed to prepare the recipe.                                                                                                       |
| `steps`          | `list`     | A list of instructions for each step, indexed in sequential order.                                                                                      |
| `description`    | `str`      | A user-provided description of the recipe.                                                                                                               |
| `ingredients`    | `list`     | A list of ingredients required for the recipe.                                                                                                          |
| `n_ingredients`  | `int`      | The number of ingredients used in the recipe.                                                                                                           |

*83,782 rows, 12 columns*

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning
Before we begin analyzing our dataset, we must first clean our dataset first and ensure that it is ready for analysis. Thus, we have conducted the following cleaning steps.

1. Since we currently have two datasets to work with, `raw_interactions` and `raw_recipes`, we would rather work with a singular dataset that contains information from both of these datasets. Thus, we will perform a left merge on `raw_recipes` to merge these DataFrames together, as we want to ensure that all of the recipes are preserved in the merged dataset. We shall refer to this merged dataset as `recipes`, which now contains $234,429$ rows and $18$ columns.

2. Upon inspection of the possible values of the `ratings` column of `recipes`, we see that it can take on the possible numerical values: $\{0, 1, 2, 3, 4, 5\}$. Since ratings are only defined from the range $[1,5]$, we will replace all values of $0$ with `np.nan` to avoid bias in its distribution.

3. We will add an additional column `avg_rating`, which will contain the average rating given by users for a particular recipe. This will allow for a more comprehensive understanding of the favorability of a recipe.

4. The `list` columns in our DataFrame, `steps`, `ingredients`, `nutrition`, and `tags`, are not stored as a `list` object in our DataFrame. To ensure that these are the correct type, we will convert each of these columns into `list` objects.

5. Next, as indicated earlier, the entries in the `nutrition` column are of the format `[calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]`. For more effective information retrieval and display, we will split each of the entires of this list into an individual column that represents the nutritional info, respectively. 

6. We will also ensure that `recipe_id` and `user_id` are represented as an integer, to further emphasize the cleanliness of the data.

7. We have that the columns `n_ingredients` and `n_steps` reflect the number of elements in the `list` objects in the columns `ingredients` and `steps`, respectively. To further emphasize the cleanliness of the data, we will ensure that the values in `n_ingredients` and `n_steps` reflect the correct number of elements in the lists in `ingredients` and `steps`.

8. To be able to analyze the complexity of a recipe, we will introduce a new column `complexity_score` and `complexity` into the dataset, defined as above. `complexity_score` will contain a `float` in $\mathbb{R}$ representing the degree of complexity of a recipe, and `complexity` will be a `str` object containing either the values of `high` or `low`, which reflects the complexity class that a recipe belongs in. To create `complexity_score`, we will first standardize the `minutes`, `n_steps`, `n_ingredients` columns and then use the Principal Components Analysis (PCA) dimensionality reduction technique with $1$ component to generate a number that captures the greatest amount of variability across the three columns. Then, to create `complexity`, we define `high` complexity as being above the median `complexity_score`, and `low` complexity as being below the median `complexity_score`.

9. To finish the data cleaning process, we will only keep the most relevant columns for our analysis, that being `n_ingredients`, `n_steps`, `minutes`, `avg_rating`, `rating`, `calories`, `total fat`, `saturated fat`, `protein`, `sugar`, `sodium`, `carbohydrates`, `complexity`, `complexity_score`.

Following data cleaning, we have our resultant DataFrame, of which the first few rows are shown below.

|    | n_ingredients | n_steps | minutes | avg_rating | rating | calories | total fat | saturated fat | protein | sugar | sodium | carbohydrates | complexity | complexity_score |
|---:|-------------:|--------:|--------:|-----------:|-------:|---------:|----------:|--------------:|--------:|------:|-------:|--------------:|:----------|-----------------:|
|  0 |           9  |      10 |      40 |          4 |      4 |    138.4 |        10 |            19 |       3 |    50 |      3 |             6 | high      |       -0.0153538 |
|  1 |          11  |      12 |      45 |          5 |      5 |    595.1 |        46 |            51 |      13 |   211 |     22 |            26 | high      |        0.574059  |
|  2 |           9  |       6 |      40 |          5 |      5 |    194.8 |        20 |            36 |      22 |     6 |     32 |             3 | low       |       -0.454439  |
|  3 |           9  |       6 |      40 |          5 |      5 |    194.8 |        20 |            36 |      22 |     6 |     32 |             3 | low       |       -0.454439  |
|  4 |           9  |       6 |      40 |          5 |      5 |    194.8 |        20 |            36 |      22 |     6 |     32 |             3 | low       |       -0.454439  |


### Univariate Analysis
Beginning our analysis, we will first begin by analyzing univariate distributions of certain variables.

<iframe
  src="assets/rating_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This bar chart displays the distribution of recipe ratings, with the x-axis representing average ratings and the y-axis showing the frequency of occurrences. The trend indicates a major class imbalance with the majority of recipes receiving high ratings, with a significant concentration at a rating of 5, while the other ratings are relatively rare. This suggests that users tend to rate recipes positively, possibly due to self-selection bias where only well-received or popular recipes get rated frequently.

<iframe
  src="assets/complexity_dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Bivariate Analysis
We continue our analysis by analyzing bivariate distributions of relevant variables.

<iframe
  src="assets/rating_vs_complexity.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

<iframe
  src="assets/rating_complexity.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>