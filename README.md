# 🍳 Investigating Recipe Ratings from Cooking Time and Recipe Features

## Overview

This data science project, conducted at UC San Diego, focuses on **predicting the average rating of a recipe** using recipe-level characteristics from the Food.com dataset. In particular, this project studies whether **cooking time** helps predict recipe ratings and whether that relationship remains meaningful after accounting for other factors such as recipe complexity, nutritional content, and recipe category.

---

## Introduction

Food is an important part of everyday life, and cooking is both a practical task and an enjoyable activity for many people. However, recipes vary widely in the time, effort, and ingredients they require. Some users may prefer recipes that are fast and convenient, while others may be more willing to spend extra time on recipes they expect to be more satisfying. Because of this, recipe ratings may depend not only on cooking time, but also on a broader set of recipe features.

In this project, **I investigate whether a recipe’s average rating can be predicted using information available in the recipe metadata**. My main variable of interest is `minutes`, the total preparation or cooking time of the recipe, but I also consider additional predictors such as the number of ingredients, number of steps, nutritional content, and selected recipe tags. This allows me to study whether cooking time remains useful for prediction after controlling for recipe complexity and other relevant characteristics.

---

## Datasets Description

This project uses two datasets from [Food.com](https://www.food.com/):

- `RAW_recipes`: contains recipe-level information  
- `interactions`: contains user ratings and reviews  

These datasets were originally collected for recommender system research in the paper *Generating Personalized Recipes from Historical User Preferences* by Majumder et al.

### 📘 RAW_recipes

This dataset contains **83,782 rows**, where each row corresponds to a unique recipe.

| Column | Description |
|--------|------------|
| `name` | Recipe name |
| `id` | Recipe ID |
| `minutes` | Minutes required to prepare the recipe |
| `contributor_id` | User ID of the recipe contributor |
| `submitted` | Date the recipe was submitted |
| `tags` | Food.com tags describing the recipe |
| `nutrition` | Nutrition info: [calories, fat, sugar, sodium, protein, saturated fat, carbohydrates] |
| `n_steps` | Number of preparation steps |
| `steps` | Recipe instructions |
| `description` | Recipe description |
| `ingredients` | List of ingredients |
| `n_ingredients` | Number of ingredients |

---

### 📗 interactions

This dataset contains **731,927 rows**, where each row represents a user interaction with a recipe.

| Column | Description |
|--------|------------|
| `user_id` | User ID |
| `recipe_id` | Recipe ID |
| `date` | Date of interaction |
| `rating` | User rating |
| `review` | User review text |

---

Given the datasets, investigating whether **average recipe ratings can be predicted using cooking time and other recipe characteristics**. To facilitate this analysis, I constructed a clean, recipe-level dataset by combining information from both the recipes and interactions data.
First, I processed the `interactions` dataset to compute a new variable, `avg_rating`, which represents the average rating for each recipe. Ratings of `0` were treated as missing values, since they do not represent valid user ratings. I then grouped by `recipe_id`, calculated the mean rating, and merged this information into the recipes dataset using the `id` column. Next, I extracted and engineered several features to support my analysis. From the recipes dataset, I used `minutes` as our main variable of interest, representing the total cooking or preparation time. I also included measures of recipe complexity such as `n_steps` and `n_ingredients`. In addition, I expanded the `nutrition` column into separate variables including `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates`, allowing us to control for nutritional differences across recipes. From the `tags` column, I created indicator variables such as `is_dessert`, `is_quick`, `is_healthy`, and `is_vegetarian` to capture recipe context. Finally, I extracted `submitted_year` from the `submitted` column to account for temporal trends. The most relevant variables for our prediction task are `minutes`, `avg_rating`, `n_steps`, `n_ingredients`, the nutrition variables, and the engineered tag indicators. These features allow us to evaluate whether cooking time remains a meaningful predictor of ratings after accounting for recipe complexity, nutritional content, and contextual factors. By structuring the data in this way, I create a realistic modeling setup that allows us to compare simple and more complex predictive models, and to assess the true contribution of cooking time to recipe ratings.
And seeking answer to my research question intend to: 
- Help users choose recipes more effectively  
- Help recipe creators better understand audience preferences  
- Improve recommendation systems on recipe platforms  
provide insight into how time investment, complexity, and recipe features relate to user satisfaction.
 :contentReference[oaicite:0]{index=0}

## Variables Used

### Response Variable
- `avg_rating`: average rating per recipe  

### Main Predictor
- `minutes`: cooking/preparation time  

### Control Variables
- `n_steps`: number of steps  
- `n_ingredients`: number of ingredients  
- Nutrition variables (calories, fat, sugar, sodium, protein, etc.)  
- Tag-based indicators (e.g., dessert, quick, healthy)  

## Data Cleaning and Exploratory Data Analysis

In this section, I cleaned the merged recipe dataset and explored the key variables used in the project. Since the goal is to predict `avg_rating` using cooking time and other recipe-level characteristics, the EDA focuses on the response variable `avg_rating`, the main predictor `minutes`, and additional control variables related to recipe complexity.

### Data Cleaning

Before conducting analysis, I cleaned and transformed the dataset to ensure that all variables used in the project were valid, interpretable, and appropriate for modeling.

First, I addressed the `rating` column in the `interactions` dataset. In this dataset, ratings of `0` do not represent actual user ratings but instead indicate missing values. Therefore, I replaced all instances of `rating = 0` with `NaN` to avoid biasing the calculation of average ratings.

Next, I constructed a recipe-level response variable, `avg_rating`, by grouping the interactions dataset by `recipe_id` and computing the mean of all non-missing ratings for each recipe. This produced a single average rating per recipe, which I then merged into the `RAW_recipes` dataset using the recipe identifiers (`id` and `recipe_id`). This step transformed the data from user-level interactions to a recipe-level dataset suitable for prediction.

I then cleaned and engineered several features from the recipes dataset. The `nutrition` column, which stores values as a list, was parsed into separate numeric columns representing calories, total fat, sugar, sodium, protein, saturated fat, and carbohydrates. This allowed nutritional information to be used directly as model features.

Additionally, I converted the `submitted` column into a datetime format and extracted the year of submission as a new variable, `submitted_year`, to capture potential temporal trends in ratings.

To incorporate categorical information, I created indicator variables based on the `tags` column. For example, I constructed binary variables such as `is_dessert` and `is_quick` by checking whether those keywords appeared in the tag list. These features help capture differences in recipe type that may influence ratings.

I also addressed issues in the `minutes` variable. Since cooking time must be positive, I treated all nonpositive values as missing. I observed that the distribution of cooking time was highly right-skewed, with some extreme outliers. To make this variable more suitable for analysis and modeling, I created a log-transformed version, `log_minutes`, which compresses large values and improves interpretability.

Finally, I created a categorical version of cooking time by grouping recipes into bins (`0–15`, `16–30`, `31–60`, `61–120`, and `120+` minutes). This transformation is useful for aggregate comparisons and helps identify broader trends across different cooking-time groups.

After these cleaning and feature engineering steps, I restricted the dataset to rows with non-missing values in key variables such as `minutes`, `avg_rating`, `n_steps`, and `n_ingredients`. The resulting cleaned dataset is used for all subsequent exploratory analysis and modeling. Resulting dataset: **81,172 observations and 31 columns**.

---

### Cleaned Dataset Preview

| name                                   | minutes | log_minutes | avg_rating | n_steps | n_ingredients | cook_time_bin |
|----------------------------------------|--------|-------------|-----------|--------|--------------|--------------|
| 1 brownies in the world best ever      | 40.0   | 3.71        | 4.0       | 10     | 9            | 31-60        |
| 1 in canada chocolate chip cookies     | 45.0   | 3.83        | 5.0       | 12     | 11           | 31-60        |
| 412 broccoli casserole                 | 40.0   | 3.71        | 5.0       | 6      | 9            | 31-60        |
| millionaire pound cake                 | 120.0  | 4.80        | 5.0       | 7      | 7            | 61-120       |
| 2000 meatloaf                          | 90.0   | 4.51        | 5.0       | 17     | 13           | 61-120       |


### Summary Statistics

| Statistic | minutes | avg_rating | n_steps | n_ingredients |
|----------|--------|-----------|--------|--------------|
| count    | 81,172 | 81,172    | 81,172 | 81,172       |
| mean     | 111.0  | 4.63      | 10.06  | 9.21         |
| std      | 4020   | 0.64      | 6.33   | 3.82         |
| 50%      | 35.0   | 5.0       | 9.0    | 9.0          |
| 75%      | 60.0   | 5.0       | 13.0   | 12.0         |
| 95%      | 250.0  | —         | —      | —            |
| 99%      | 724.0  | —         | —      | —            |
| max      | 1,050,000 | —      | —      | —            |


### Missing Values

| Column          | Missing Count | Missing Proportion |
|----------------|--------------|--------------------|
| minutes        | 0            | 0.0                |
| avg_rating     | 0            | 0.0                |
| n_steps        | 0            | 0.0                |
| n_ingredients  | 0            | 0.0                |
| calories       | 0            | 0.0                |
| sugar          | 0            | 0.0                |


The summary statistics reveal that cooking time (`minutes`) has a mean of approximately 111 minutes but a much lower median (35 minutes), indicating a highly right-skewed distribution with extreme outliers. The maximum value exceeds 1,000,000 minutes, confirming the presence of unrealistic values.

The average rating (`avg_rating`) is around 4.63 and is concentrated near the upper end of the rating scale. This suggests that most recipes receive positive ratings, resulting in limited variability in the response variable.

---

### Univariate Analysis

To better understand the distribution of cooking time, I plotted a histogram after restricting the data to recipes with cooking times of 300 minutes or less.

<iframe src="assets/minutes_distribution.html" width="800" height="600" frameborder="0"></iframe>

The distribution is strongly right-skewed, with most recipes taking between 0 and 60 minutes and progressively fewer recipes requiring longer cooking times. The need to filter extreme values highlights the presence of outliers, and the skewed shape motivates the use of a log transformation for modeling.

I also examined the distribution of the response variable, `avg_rating`.

<iframe src="assets/avg_rating_distribution.html" width="800" height="600" frameborder="0"></iframe>

The distribution of `avg_rating` is heavily concentrated near 4–5, indicating that most recipes receive high ratings. This limited spread suggests that distinguishing between recipes based on rating alone may be challenging.

---

### Bivariate Analysis

To explore the relationship between cooking time and rating, I plotted `log_minutes` against `avg_rating`.

<iframe src="assets/log_minutes_vs_avg_rating.html" width="800" height="600" frameborder="0"></iframe>

The plot shows a very weak relationship between cooking time and average rating. Although there is a slight negative trend, the points are widely dispersed, indicating that cooking time alone does not strongly predict recipe ratings.

I also compared rating distributions across cooking-time bins.

<iframe src="assets/cook_time_bin_boxplot.html" width="800" height="600" frameborder="0"></iframe>

The boxplot shows that ratings are consistently high across all cooking-time groups, with substantial overlap between categories. This further suggests that cooking time is not a strong standalone predictor.

---

### Interesting Aggregates

To summarize patterns across groups, I computed aggregate statistics by cooking-time bin.

To summarize broader patterns, I grouped recipes by cooking-time bin and computed aggregate statistics such as average rating, average number of ingredients, and average number of steps.

<iframe
  src="assets/mean_rating_by_cook_time_bin.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

This bar chart makes it easier to compare mean ratings across cooking-time groups. The differences between groups are fairly small, which reinforces the conclusion from the earlier plots: cooking time alone is not likely to strongly predict recipe ratings. Instead, it will be important to include other relevant recipe-level variables, such as number of ingredients, number of steps, and nutritional characteristics, in the final model.

### EDA Summary

Overall, the EDA suggests that cooking time has only a modest relationship with recipe rating. Cooking time is highly right-skewed, while average ratings are concentrated near higher values. Although grouped comparisons show some variation, the differences are relatively small. These results motivate building a prediction model that uses cooking time together with additional features rather than relying on cooking time alone.

## Assessment of Missingness

Looking at the cleaned dataset, there are still three columns contain missing values:

- `avg_rating`: ~3.11% missing  
- `recipe_id_count`: ~3.11% missing  
- `description`: ~0.084% missing  

Since `avg_rating` is the response variable for this project, understanding its missingness is especially important.

---

### MNAR Analysis

A plausible **MNAR** column in this dataset is `description`. Because `description` is written by the recipe contributor, whether it is missing may depend on unobserved factors such as how much effort the contributor wants to put into the recipe, how confident they feel about it, or whether they believe additional explanation is necessary. These factors are not fully captured in the dataset.

Additional data that could help explain this missingness (and potentially make it MAR) would include contributor-level information such as experience level, engagement history, or whether Food.com prompted users to include a description during submission.

---

### Missingness Dependency (Permutation Tests)

I conducted permutation tests to determine whether the missingness of `avg_rating` depends on other observed variables.

- **Null hypothesis:** Missingness of `avg_rating` does not depend on column X  
- **Alternative hypothesis:** Missingness of `avg_rating` does depend on column X  
- **Test statistic:** Absolute difference in mean of column X between missing vs observed groups  

---

### Column Where Missingness Depends on It: `n_steps`

- Observed test statistic: **1.4930**  
- p-value: **0.0000**

<iframe src="assets/missingness_n_steps_distribution.html" width="800" height="600" frameborder="0"></iframe>

The distribution of `n_steps` differs noticeably between recipes with missing and observed `avg_rating`. Recipes with missing ratings tend to have a slightly different spread of number of steps.

<iframe src="assets/missingness_n_steps_null.html" width="800" height="600" frameborder="0"></iframe>

Because the p-value is less than 0.05, we reject the null hypothesis. This provides strong evidence that the missingness of `avg_rating` **depends on `n_steps`**.

---

### Column Where Missingness Does Not Depend on It: `is_quick`

- Observed test statistic: **0.0022**  
- p-value: **0.4390**

<iframe src="assets/missingness_is_quick_distribution.html" width="800" height="600" frameborder="0"></iframe>

The distributions of `is_quick` are nearly identical between missing and observed groups, indicating little to no relationship.

<iframe src="assets/missingness_is_quick_null.html" width="800" height="600" frameborder="0"></iframe>

Because the p-value is greater than 0.05, we fail to reject the null hypothesis. There is no strong evidence that the missingness of `avg_rating` depends on `is_quick`.

Overall, the missingness of `avg_rating` is **not completely random**. The permutation tests show that it depends on at least one observed variable (`n_steps`), but not on others (`is_quick`). 

This suggests that the missingness mechanism is likely **MAR rather than MCAR**, meaning that whether a recipe receives ratings is related to observable characteristics such as recipe complexity. This is important for later modeling, as it implies that missing ratings are systematically associated with certain types of recipes rather than occurring purely at random.

---
### Hypothesis Testing

To investigate whether cooking time is associated with recipe ratings, I compared recipes with cooking times **below the median (35 minutes)** to those with cooking times **at or above the median**.

### Hypotheses

- **Null hypothesis (H₀):** Recipes with shorter cooking times and recipes with longer cooking times come from the **same distribution of average ratings**. Any observed difference is due to random chance.

- **Alternative hypothesis (H₁):** Recipes with shorter cooking times and recipes with longer cooking times come from **different distributions of average ratings**.

Test Statistic: I used the **absolute difference in mean average rating (`avg_rating`)** between the two groups:
Significance Level: I used a significance level of **0.05**.

I chose a **permutation test** because I am comparing two observed groups and want to determine whether the difference in their average ratings could plausibly occur by chance. 

Under the null hypothesis, cooking-time group membership should not matter, so randomly shuffling the ratings simulates what differences would look like if there were truly no relationship between cooking time and ratings. This approach avoids relying on strong parametric assumptions.

Test results: 
- **Observed test statistic:** 0.0350  
- **Number of permutations:** 2000  
- **p-value:** 0.0000  

The observed difference in mean ratings is relatively small in magnitude, but when compared to the permutation distribution, it is **extremely unlikely to occur by chance**.
Therefore, at the **0.05 significance level**, I **reject the null hypothesis**. This provides strong statistical evidence that **shorter-cook and longer-cook recipes do not come from the same distribution of average ratings**. Although the difference in average ratings is small (around 0.035), the large sample size makes this difference statistically significant.

### Interpretation

This result suggests that cooking time is **associated** with recipe ratings. However, this should **not be interpreted as a causal relationship**. 

There may be other factors (such as recipe complexity, ingredients, or cuisine type) that influence both cooking time and ratings.

<iframe src="assets/step4_permutation_test.html" width="800" height="600" frameborder="0"></iframe>
---

### Framing a Prediction Problem
- Prediction Problem: In this project, I aim to predict the average rating (`avg_rating`) of a recipe, making this a regression problem since the response variable is continuous and ranges from 1 to 5. I chose to model the rating as a continuous outcome rather than converting it into categories, as this preserves more information and allows for more precise predictions.

- Response Variable: The response variable is `avg_rating`, which represents the average user rating of a recipe. This is a meaningful target because it captures overall user satisfaction. From earlier exploratory data analysis, variables such as cooking time, number of ingredients, number of steps, and nutritional content showed relationships with ratings, suggesting they may be useful predictors.

- Evaluation Metrics:
To evaluate model performance, I use Root Mean Squared Error (RMSE) as the primary metric. RMSE is appropriate because it penalizes larger prediction errors more heavily, which is important when predicting values on a bounded scale like ratings (1–5).

In addition, I report:
- Mean Absolute Error (MAE) for an interpretable average prediction error  
- R² (coefficient of determination) to measure how much variance in ratings is explained by the model  

Using multiple metrics provides a more complete understanding of model performance.


To ensure a realistic prediction setting, I restrict the model to use only features that are available at the time a recipe is created, before any users have rated it. This prevents data leakage and ensures the model reflects a real-world use case.

The features used include:
- Cooking time (`minutes`)
- Number of steps (`n_steps`)
- Number of ingredients (`n_ingredients`)
- Nutritional attributes (e.g., calories, sugar, fat)
- Recipe tags (e.g., `is_vegetarian`, `is_healthy`)
- Submission metadata (e.g., `submitted_year`)

I explicitly exclude variables such as number of ratings, reviews, or any post-publication feedback, since these would not be known at prediction time.

This prediction framework allows us to estimate how well a recipe will be received based solely on its intrinsic characteristics, without relying on user feedback. Such a model could be useful for surfacing promising recipes early or assisting users in selecting high-quality recipes before ratings are available.

---

## Baseline Model

For my baseline model, I used a **linear regression model** and split the dataset into training and test sets to evaluate performance on unseen data. The goal of this model is to predict a recipe’s **average rating (`avg_rating`)**.

The features used in this model are: `minutes` (a quantitative variable representing cooking/preparation time ), `n_ingredients` (a quantitative variable representing recipe complexity), and `is_dessert` (a nominal variable (0, 1) indicating whether the recipe is a dessert)
To prepare the data for modeling, I applied the following transformations:
- For `minutes` and `n_ingredients`, I filled missing values using the median and standardized the variables using `StandardScaler`
- For `is_dessert`, I applied one-hot encoding and dropped one category to avoid redundancy  

The metric I used to evaluate this model is **RMSE (Root Mean Squared Error)**, along with MAE and R² for additional interpretation.
The performance of the model on the test set is:
- **RMSE:** 0.6354  
- **MAE:** 0.4650  
- **R²:** 0.0013  

For comparison, a **dummy model** that always predicts the average rating achieved nearly identical performance:
- **RMSE:** 0.6400  
- **MAE:** 0.4700  
- **R²:** 0.0000  

These results show that the baseline model performs **only slightly better than predicting the mean rating**. The very low R² value indicates that the model explains almost none of the variation in recipe ratings. This suggests that the features used (`minutes`, `n_ingredients`, and `is_dessert`) are **not strong predictors** of recipe ratings on their own. While they capture basic characteristics of recipes, they do not reflect more complex factors that influence user preferences. Overall, this baseline model is **not very strong**, but it provides a useful starting point. In the next step, we will build a more advanced model using additional features to improve predictive performance.
