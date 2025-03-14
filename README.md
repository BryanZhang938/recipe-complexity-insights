# Unraveling Culinary Complexity: A Data-Driven Analysis of Recipe Complexity and User Satisfaction

## Bryan Zhang

*March 14th, 2025*

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

Before analyzing our data, we must first clean and prepare the dataset. We performed the following cleaning steps:

1. **Merge Datasets:**  
   We will combine the two datasets, `raw_interactions` and `raw_recipes`. To accomplish this, we performed a left merge on `raw_recipes` to ensure that all recipes were preserved. We will now refer to this merged dataset as `recipes`, which contains **234,429 rows** and **18 columns**.

2. **Clean the Ratings Column:**  
   Upon inspection, the `ratings` column in `recipes` contains the values: {0, 1, 2, 3, 4, 5}. Since ratings are defined in the range [1, 5], we replaced any value of 0 with `np.nan` to eliminate any biases within the `ratings` distribution.

3. **Compute Average Rating:**  
   We added an `avg_rating` column that represents the average rating given by users for each recipe, providing a comprehensive measure of favorability.

4. **Validate List Columns:**  
   We converted the columns `steps`, `ingredients`, `nutrition`, and `tags` to the proper `list` type for greater accessibility and data consistency.

5. **Split Nutritional Information:**  
   We split the list entries of `nutrition` into separate columns representing each nutritional component.

6. **Validate Data Types:**  
   We converted `recipe_id` and `user_id` to integers to maintain data consistency.

7. **Validate Count Columns:**  
   The columns `n_ingredients` and `n_steps` indicate the number of elements in the `ingredients` and `steps` lists, respectively. We verified that these counts accurately reflect the length of their corresponding lists.

8. **Create Complexity Metrics:**  
   To analyze recipe complexity, we introduced two new columns:  
   - **`complexity_score`:** A `float` representing the degree of complexity. We computed this by standardizing the `minutes`, `n_steps`, and `n_ingredients` columns and applying Principal Component Analysis (PCA) with 1 component.  
   - **`complexity`:** A string that classifies recipes as either `high` or `low` complexity. Recipes with a `complexity_score` above the median are labeled as `high`, while those below are labeled as `low`.

9. **Select Relevant Columns:**  
   Finally, we retained only the most relevant columns for analysis:
   `n_ingredients`, `n_steps`, `minutes`, `avg_rating`, `rating`, `calories`, `total fat`, `saturated fat`, `protein`, `sugar`, `sodium`, `carbohydrates`, `complexity`, and `complexity_score`.

After cleaning, the first five rows of our resulting DataFrame are shown below:

|    | n_ingredients | n_steps | minutes | avg_rating | rating | calories | total fat | saturated fat | protein | sugar | sodium | carbohydrates | complexity | complexity_score |
|---:|-------------:|--------:|--------:|-----------:|-------:|---------:|----------:|--------------:|--------:|------:|-------:|--------------:|:----------|-----------------:|
|  0 |           9  |      10 |      40 |          4 |      4 |    138.4 |        10 |            19 |       3 |    50 |      3 |             6 | high      |       -0.01535   |
|  1 |          11  |      12 |      45 |          5 |      5 |    595.1 |        46 |            51 |      13 |   211 |     22 |            26 | high      |        0.57406   |
|  2 |           9  |       6 |      40 |          5 |      5 |    194.8 |        20 |            36 |      22 |     6 |     32 |             3 | low       |       -0.45444   |
|  3 |           9  |       6 |      40 |          5 |      5 |    194.8 |        20 |            36 |      22 |     6 |     32 |             3 | low       |       -0.45444   |
|  4 |           9  |       6 |      40 |          5 |      5 |    194.8 |        20 |            36 |      22 |     6 |     32 |             3 | low       |       -0.45444   |

---

### Univariate Analysis

We begin our analysis by exploring the univariate distributions of key variables.

<iframe  
  src="assets/rating_dist.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>

*Figure: Distribution of Recipe Ratings*  
This bar graph displays the distribution of recipe ratings. The x-axis represents the average ratings, and the y-axis indicates the frequency of occurrences. The chart reveals a significant class imbalance, with most recipes receiving high ratings while lower ratings are relatively rare. This pattern may reflect a self-selection bias, where only well-received or popular recipes are rated frequently.

<iframe  
  src="assets/complexity_dist.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>

*Figure: Distribution of Recipe Complexity Scores*  
This histogram illustrates the distribution of recipe complexity scores. Most recipes have scores between -2 and 2, with a right-skewed distribution indicating that the majority of recipes have relatively low-complexity, while fewer recipes exhibit high-complexity.

---

### Bivariate Analysis

Next, we examine the bivariate distributions between pairs of variables.

<iframe  
  src="assets/rating_vs_complexity.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>

*Figure: Heatmap of Complexity Scores vs. Ratings*  
This heatmap visualizes the relationship between recipe complexity scores on the x-axis and ratings on the y-axis, with color intensity representing density. The highest density is observed for recipes with complexity scores near 0 and ratings around 5, suggesting that simpler recipes are more frequently rated and receive higher ratings. More complex recipes are less common, as indicated by lower density across rating levels.

---

### Interesting Aggregates

Below is a pivot table summarizing key metrics by recipe complexity:

| Complexity | Average Minutes | Average Ingredients | Average Steps | Average Rating |
|:----------:|---------------:|--------------------:|-------------:|---------------:|
| **High**   | 119.48         | 11.73               | 13.75        | 4.6764         |
| **Low**    | 94.10          | 6.41                | 6.28         | 4.6833         |

This table highlights key differences between high and low-complexity recipes:
- **Preparation Time:** High-complexity recipes take longer to cook compared to low-complexity recipes.
- **Ingredients and Steps:** High-complexity recipes use more ingredients and require more steps.
- **Ratings:** Regardless of the recipe complexity, the average ratings are nearly identical.

These findings suggest that while high-complexity recipes require more effort in terms of time, ingredients, and steps, they do not necessarily receive higher ratings. We will later perform a permutation test to further investigate this observation.

## Assessment of Missingness

The only columns in `recipes` that have a significant amount of missing values are `rating`, `review`, and `date`. In this section, we will explore the missingness in `recipes`.

### NMAR Analysis
We claim that the missingness in the `review` column is likely to be Not Missing At Random (NMAR). This is likely due to the fact that users who feel negative about a recipe may be less likely to leave a review. Users who had a bad experience with the recipe may be less likely to leave a rreview because they may not feel that it is worth their time or they prefer to not engage with the product after a negative experience. On the other hand, users who had a good experience may be more inclined to leave a review as to recommend to other people or to create more traction on the particular recipe.

### Misingness Dependency
We now continue by examining the missingness of the `rating` column in `recipes` by testing its dependency on the `complexity` column. To achieve this, we will be performing a permutation test. So, we propose the following Null and Alternate Hypotheses and the Test Statistic and the Significance Level.

**Null Hypothesis**: The missingness of ratings does not depend on the complexity score of a recipe.

**Alternate Hypothesis**: The missingness of ratings depends on the complexity score of a recipe.

**Test Statistic**: The absolute difference of the complexity score means between the recipes without missing ratings and the recipes with missing ratings.

**Significance Level**: 0.05

After performing the permutation test, we yield the following empirical distribution of the test statistics.

<iframe  
  src="assets/missingness1.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>

<iframe  
  src="assets/missingness2.html"  
  width="800"  
  height="600"  
  frameborder="0"  
></iframe>