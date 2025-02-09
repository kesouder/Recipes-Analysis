# Happy Recipes
### Calories in Recipes -- Analysis
Final Project for DSC80 at UCSD - 2024

https://kesouder.github.io/Recipes-Analysis/

# Introduction
The data set used for the final project is from the food.com website, which include recipes, its ingredients, stats, and user input (reviews and ratings). This project explores the question of what types of recipes have more or less calories.

This question is particularly interesting since it relates to user preferences and trends (like keto or veganism). This can better help the food industry, whether that be websites, apps etc, better understand their consumers and suggest better recipe suggestions. Like most online websites that track patterns and suggest things consumers want, it’s not different with food and food recipes.
The number of rows in the data set I’ll be working on is 234429 with relevant column names being:
	```
{
‘name’: The recipe name
‘id’: Unique recipe id number
‘nutrition‘: Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] where PDV stands for “percentage of daily value”
‘rating_avg’: The average rating for each recipe
}
```

# Data Cleaning and Exploratory Analysis
The first thing I did was merge the ‘recipes’ and ‘interactions’ data frames together based on the unique recipe ID, so there would be rating data corresponding to each recipe. This is necessary for computing any kind of analysis on the recipe ratings in general and referring back to the recipes themselves.

Replacing the zeros with NaN for the ‘rating’ column is so then any calculations done with ‘rating’ column values (for example the mean) doesn’t affect the overall statistic. In the context of the recipes data, a rating of zero could mean there was no rating left at all, except a review. One specific example of this is from the food.com website where a user wrote “it is easy.”, but left no rating. A recipe being easy is not necessarily a bad thing and one could guess that the rating would be at least or higher, but instead, it’s safer to input NaN into the dataset. 

After this step, I calculated the average rating based on each unique recipe, and then added these values into the merged data frame, corresponding to the correct recipe ID. 
<iframe
  src="assets/dist_of_cal.html"
  width="800"
  height="430"
  frameborder="0"
></iframe>
The graph above is the distribution of calories for each recipe where the highest percentage of recipes (~38%) has a range of 100-300 (exclusive) calories. Most of the recipes are within 100-700 calories, which can range from serving size. Serving size is most likely the cause for the calorie outliers.

<iframe
  src="assets2/cal_vs_fat.html"
  width="800"
  height="430"
  frameborder="0"
></iframe>
The scatter plot above is the relationship between the Percent of Daily Fat and number of calories for each recipe(represented by each data point). From this graph, there is a positive relationship between the two variables, i.e. as the PDV of fat increases, typically the number of calories also increases. However, there are other confounding factors such as specific ingredients and other nutritional facts.

The table below shows essentially what we can infer about what we know about food. Looking at the PDV of fat and PDV of sugar, as these variables increase, the number calories increases and is thus ranked about healthy or unhealthy. This is ultimately just interesting to take into account.
<iframe
  src="assets2/healthy_stats.html"
  width="800"
  height="150"
  frameborder="0"
></iframe>

# Assessment of Missingness
Of the dataset I’m working on, there are three columns with missing data. These columns are 'name', 'description', and 'average_rating'. Of these three columns, I believe that the ‘average_rating’ column could be classified as NMAR (depends on the actual missing value), since the rating could be 0, meaning the recipe hasn’t been made. For data cleaning, I made all ratings of 0 to be np.NaN. 
I could look at the ‘name’ or ‘description’ columns to see if the ‘average_rating’ column’s missing data is dependent on these columns. This would mean better descriptions and recipe names would attract more attention and thus more reviews. 

<iframe
  src="assets/Name Length vs Average Rating.html"
  width="800"
  height="430"
  frameborder="0"
></iframe>
This graph is the distribution of ‘name_length’ when ‘average_rating’ is missing (red) and the distribution of ‘name_length’ when ‘average_rating’ is not missing (green). As shown above, the distributions are extremely different, as is the number of recipes that are missing versus not missing in regards to the length of the recipe name. My previous hypothesis that longer recipe titles result in non-missing ratings is found to be false. 

# Hypothesis Testing
My null hypothesis: There is no significant relationship between the number of steps in the recipe and the average rating for recipes.
My alternative hypothesis: There is a significant relationship between the number of steps in the recipe and the average rating for recipes.
For my test statistic, I chose to use the correlation coefficient r, since I’m testing to see if there is a relationship between the two variables. My significance level is 0.05, and I found the p-value to be 0.15. This p-value is significantly greater than my cut-off, meaning I fail to reject my null hypothesis.

# Framing A Prediction Model
I want to predict the number of calories of a recipe using regression analysis. The response variable I’ll be using is the ‘cals’ column derived from the ‘nutrition’ column in the original 'recipes' data set. I chose this variable because it’s the exact variable I’m testing for, calories per recipe. I will be using the Root Mean Squared Error (RMSE) and R^2 since I want to minimize the prediction of calories and it’ll be easy to interpret the results of calories.

# Baseline Model
My model will use the ‘minutes’ and ‘n_ingredients’ column of the data frame to predict the ‘cals’ variable. The features I’m using in my model are ‘n_ingredients’ which is the number of ingredients for each recipe, and ‘minutes’ which is the amount of time it takes to prepare the recipe. Both variables are quantitative. I’ve transformed the ‘minutes’ column using the log function because I wanted to make more accurate predictions with minutes and the number of calories in a recipe. I’ve transformed the ‘n_ingredients’ column with the PolynomialFeature() method because I figured there was not a linear relationship between calories and the number of ingredients.
The performance of this model is not very thorough or accurate. I believe this model is not good because it only used two variables that were both quantitative. Since I’m trying to predict the number of calories per recipe, I should also consider other values from the 'nutrition' column, not just how many ingredients and the recipe’s cooking time.


# Final Model
For my final model, I took what I learned from my baseline model and added more features relating to calories of a recipe. I added the ‘fat_pdv’ and ‘sugar_pdv’ columns to my features to predict the number of calories. These are better features since they are part of the nutritional values I’m trying to predict. Also, adding these features help generalize the data better, and using the GridSearchCV helped with preventing underfitting. These features helped with the underlying data and relationship since any relationship to calories isn’t linear. 

I chose to use a Decision Tree Regressor, and with this, I then used GridSearchCV to find the best hyperparameters for my model. The best hyperparameters were a Decision Tree max depth 10 and minimum sample split 2.
In my ColumnTransformer classifier object, I used a Function Transformer to apply a log function to the ‘minutes’ column. I used PolynomialFeatures() for the ‘n_ingredients’ column. I used StandardScalar() for 'n_ingredients', 'sugar_pdv', and 'fat_pdv' columns since the relationship is not linear, this will change the model's error. Instead of using LinearRegression(), I used the DecisionTreeRegressor() with a max depth of 10, since it was the best choice for this hyperparameter and the default split value of 2 to refit my model to get a more accurate .
This model has improved significantly from the baseline model. The final model included a more indepth engineering model such as the DecisionTreeRegressor() classifier as well as the hyperparameters that yielded the best R2. The baseline models R2 was extremely low, 0.04 at the maximum, versus the final models R2, being ~0.75. 

# Fairness Analysis
For my two groups, I chose to use the ‘n_steps’ column to determine if data belonged to the long group or the short group, using a Binarizer threshold of 10, based on the ‘n_steps’ mean, mode, and median. Using the ‘n_steps’ column, it uses a column that doesn’t directly affect the prediction of calories. 
My evaluation metric was the difference in RMSE of the two groups.
Null Hypothesis: My final model is fair. The RMSE difference is the same between the healthy and unhealthy groups. Any observed difference is due to random chance.
Alternative Hypothesis: My final model is not fair. The RMSE difference is significantly different between the healthy and unhealthy groups.
My hypothesis was tested with a test statistic of 0.05. My resulting p-value was 0.006, meaning the difference between RMSE was significant and thus I reject the null hypothesis. My model is not fair since there is a significant difference between the two groups and the number of steps for the recipes, meaning the model's prediction of calories is not fair for different recipes.
<iframe
  src="assets2/dist_fairness2.html"
  width="800"
  height="430"
  frameborder="0"
></iframe>
