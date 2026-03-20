# 🍳 DSC 80 Project 04: Predicting Recipe Ratings from Cooking Time and Recipe Features

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

## Hypothesis

Cooking time alone will have limited predictive power, but prediction performance will improve when recipe complexity, nutritional variables, and recipe category information are included. Additionally, the relationship between cooking time and rating is expected to be nonlinear, where extremely short or extremely long recipes may receive lower ratings compared to moderately time-intensive recipes.

---

## Data Processing

To prepare the data for modeling, a recipe-level response variable called `avg_rating` is constructed to represent the average rating of each recipe.

Steps:

1. Treat `rating = 0` as missing values  
2. Group the interactions dataset by `recipe_id`  
3. Compute the mean rating for each recipe  
4. Merge these averages with the recipes dataset using recipe IDs  

---

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

These variables help control for recipe complexity, nutritional profile, and recipe type so that the effect of cooking time can be evaluated more meaningfully.


## Why This Matters

Understanding how recipe characteristics influence ratings can:

- Help users choose recipes more effectively  
- Help recipe creators better understand audience preferences  
- Improve recommendation systems on recipe platforms  

This project provides insight into how time investment, complexity, and recipe features relate to user satisfaction.
