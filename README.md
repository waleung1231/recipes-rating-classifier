# Recipes Ratings RandomForestClassifier
By Wan-Rong (Emma) Leung and Camille Sicat

## Introduction
Our dataset is focused on recipes. Based on this data, we want to answer the question: What factors determine if a recipe is given a high rating? We want to answer this question so that when we search online recipe sites we can look at small details and determine if a recipe is good to follow. 

The recipes dataset contains 234429 rows and the columns 'name', 'id', 'minutes', 'contributor_id', 'submitted', 'tags', 'nutrition', 'n_steps', 'steps', 'description', 'ingredients', 'n_ingredients', 'user_id', 'date', 'rating', and 'review'. 
![Our dataset columns and descriptions](assets/column_explanations.png)

### Data Cleaning and Exploratory Data Analysis
We started off with spliting all the values in nuitrition into new seperate columns (calories, total fat, sugar, sodium, protien, saturated fat, carbohydrate). By splitting them into seperate columns we were able to see if different segmants of nuitrition may have affects to the ratings. E.g. if higher calories food can lead to higher ratings. 

In the 'rating' column, we changed all values with rating of 0 to np.NaN. This is because if a review contained no rating, it was a missing rating, and thus further analysis must be done to determine why it is missing.

After this step, we created an 'average_rating' column, where each recipe has an average_rating based on the mean of all ratings given by reviewers. 

For our univariate analysis, we decided to look at the distributin of the 'ratings' column. This is especially relevant because our model is predicting rating so we want to see what the distribution is like before we start training. The plot is heavily skewed left, towards 5-star ratings. 
## Distribution of Rating
<iframe
  src="assets/ratings_dist_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

For our bivariate analysis we wanted to examine the relationship between number of ingredients ('n_ingredients') and number of steps ('n_steps'). 
## Number of Ingredients vs. Number of Steps
<iframe
  src="assets/ingredients_steps_scatter.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

As you can see above, there does not appear to be a relationship, as there is a large variation in number of steps in recipes with similar number of ingredients. 

For our aggregation, we wanted to see the distribution of 'calories (#)', 'n_ingredients', and 'n_steps', so we created the pivot table below with the aggregation method 'mean'. 
## Pivot Table of Recipe Data Means
<iframe
  src="assets/recipe_means.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

While mean is not the most illustrative measure of center, in the pivot table on average recipes with higher ratings have slightly more ingredients and slightly lower number of steps. That being said, because the data is relatively similar, it is unlikely that rating is influenced solely by these factors. More exploration is needed. 

### Assessment of Missingness
We have 4 columns with relevant proportions of missing data: 'description', 'review', 'rating', and 'average_rating'. Of these 4 columns, we believe that 'description' and 'review' are NMAR because users chose not to submit them because they didn't think they were relevant. 

Meanwhile, we believe that 'rating' or 'average_rating' is MAR. Below is the plot of our permutation test to see if 'average_rating' is MAR conditioned on 'n_steps'.
## 'average_rating' Conditioned on 'n_steps'
<iframe
  src="assets/avg_rating_steps_perm.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

For this plot, our p-value was 0.0, meaning the results are statistically significant. This also applies for our permutation test to see if 'average_rating' is MAR conditioned on 'n_ingredients'. 
## 'average_rating' conditioned on 'n_ingredients'
<iframe
  src="assets/avg_ratings_ingredients_perm_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>.

Finally, we ran a permutation test to see if 'average_rating' was MAR conditioned on another numerical column, 'sodium (PDV)'. In the permutation test below you can see that our results are not statistically significant. 
## 'average_rating' conditioned on 'sodium (PDV)'
<iframe
  src="assets/precision_difference_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Since 'average_rating' is MAR conditioned on 'n_steps' and 'n_ingredients', and 'average_rating' is based on 'rating', there is a high chance 'rating' is heavily related to 'n_steps' and 'n_ingredients'. We will use this information in our model. 

### Hypothesis Testing
For our hypothesis test, we wanted to answer the question: Do recipes with a higher number of ingredients have higher average ratings? 

We checked if recipes with a higher number of ingredients have the same average_ratings as recipes with a lower number of ingredients.

**Null hypothesis**: The distribution of average ratings for recipes with a high number of ingredients is the same as the distribution of average ratings for recipes with a low number of ingredients.
**Alternate hypothesis**: The distribution of average ratings for recipes with a high number of ingredients is different from the distribution of average ratings for recipes with a low number of ingredients.

For our test statistic we used the K-S statistic with a p-value cutoff of 0.05. 

Our test gave us a K-S statistic of 0.181 and a p-value of 1.47e-16. Based on these results, we reject our null hypothesis that the distribution of ratings for recipes with a high number of ingredients is the same as the distribution of ratings for recipes with a lower number of ingredients. This means 'n_ingredients' is significant for determining a recipe's rating, a fact we will use in our model. 

### Framing a Prediction Problem
For our prediction problem, we want to predict the rating of a recipe using a classification model. Both 'rating' (and 'average_rating') are heavily correlated with the numerical columns 'n_ingredients' and 'n_steps', meaning a relationship can be determined. Our classifier will be a multiclass classification model taking in these rows such as n_ingredients and n_steps to predict whether a recipe is 1, 2, 3, 4, or 5 stars. We will evaluate the effectiveness of our model using accuracy because it tells us whether or not the model was successful at predicting the correct category. We chose accuracy over the F-1 score because it is easier to interpet in the context of our question - that is, if a recipe has been classified as having a correct rating. 

At the time of classification, we can assume 'n_ingredients' and 'n_steps' would be known, so it is okay to use these as variables. 

### Baseline Model
For our baseline model we will use a DecisionTreeClassifier to classify our recipes data as having a rating of 1, 2, 3, 4, or 5 stars. In this baseline model, we are using the 2 quantative columns, 'n_steps' and 'n_ingriedients', to predict whether what rating a recipe will receive. To prevent outliers from skewing the data, we decided to standardize these columns with StandardScaler(). 

Our train/test split was the default 75%/25%. On training data across all 5 categories our model had an mean accuracy of ~77.4%. On testing data, across all 5 categories it had a mean accuracy of ~77.1%. 

We also tried a RandomForestClassifier, but the results were roughly the same as the DecisionTreeClassifier, so we used the DecisionTreeClassifier as our final baseline model. Because the training/testing accuracy scores of our DecisionTreeClassifier model are very similar, we believe we didn't overfit the data. Therefore, the high accuracy means our baseline model is good. That being said, we believe it can be improved by testing categorical columns and seeing if they improve our model's accuracy. 

### Final Model
To improve our model accuracy, we decided to introduce new variables. First, we engineered 2 new features: 'baking' and 'top three tags'. For 'baking', we thought that whether or not a recipe needed the extra prep time (as indicated by an oven) was an indicator of rating. For 'top three tags', the top 3 tags used on our recipes data were '60-minutes-or-less', '30-minutes-or-less', and '15-minutes-or-less'. If a recipe has popular tags, it means it was viewed by more people, meaning in our eyes it was more likely to have more ratings and thus more data to correlate. Then, we decided to use 'calories (#)', because oftentimes people will consider recipe quality through how many calories it introduces in the meal. We also decided not to scale 'n_steps' and 'n_ingredients' out of fear that scaling them last time reduced the quality of analysis.

For 'baking', we examined the nominal column 'steps', which have no inherent order as they are lists of strings. We assumed if the 'steps' column contained the word 'preheat' then the recipe involved an oven and thus 'baking'. We encoded this value using 1 to indicate 'baking' and 0 to indicate otherwise.

For 'tags', the top 3 tags '60-minutes-or-less', '30-minutes-or-less', and '15-minutes-or-less' have an inherent ordering, making this feature ordinal. To encode whether or not a reicpe fell into one of these three categories, we made a 2-dimensional matrix with each column representing a tag, with a 4th column for 'none of the above'. A 0 in one column meant the recipe did not fall under this category, and a 1 meant the recipe fell under the category. Note that we dropped the 'none of the above' column in our model analysis and tuning to avoid multicollinearity. 

'calories (#)' is a continuous quantiative column. For our model, we examined health data and determined that a recipe is considered high calorie if it contains over 600 calories, and low calorie otherwise. In the preprocessing step we used a Binarizer to encode this data. 

As stated earlier, we left 'n_steps' and 'n_ingredients' alone as they were numerical columns whose values were significant on their own to the problem at hand. 

To start the hyperparameter tuning process, we preprocessed the data to encode our given features. To better avoid overfitting given the increased complexity of our model, we decided to use a RandomForestClassifier with a 75%/25% train/test split. Our hyperparameters were 'max_depth' with values \[2,3,7\], 'min_samples_split' with values \[2, 5, 10\], and 'criterion' of \['gini', 'entropy'\]. We used GridSearchCV with five-fold cross-validation to determine that our best paramters were a criterion of 'gini', a 'max_depth' of 7, and a 'min_samples_split' of 10. 

Using these hyperparameters, we created a RandomForestClassifier. On training data the model had a mean accuracy across all 5 categories of ~98.3%. On testing data it had a mean accuracy across all 5 categories of ~98.2%. Based on these dramatic improvement in accuracy across all 5 categories, we can see this RandomForestClassifier's performance is superior to our previous DecisionTreeClassifier. (To illustrate, see the confusion matrix below.)
[Insert confusion matrix here]

While we did use a RandomForestClassifier to avoid overfitting, it is posssible that it overfit to the training/test set. However, that cannot be tested without using more data. 

### Fairness Analysis
For our fairness analysis we chose whether or not the recipe require baking as our evaluation metric group. Whether or not a recipe require preheat indicates if this recipe requires baking. The evalutation matrix we chose is the Precision Parity, this is because our data would not be valid for accuracy parity since our predicting column (rating) is an imbalanced dataset, where we have significantly more 5 star ratings. Therefore, using accuracy parity might not lead to fair outcomes. 

Our Null and Alternative Hypothesis:

Null Hypothesis: Our model is fair. There is no difference in precision between recipe that require pre-heat and recipe that do not require preheat. And any observed difference is due to random chance. 

Alternative Hypothesis: Our model is not fair.There is a significant difference between the precision between recipe that require pre-heat and recipe that do not require preheat. And any observed difference is not due to chance. 

The test statistic we chose is the different in precision score across recipes that requires pre-heat and those that do not require pre-heat.
And the significant level we chose for our data is 0.05. And the resulting p-value that we got for our data is 0.253. Thus, this indicated that we fail to reject the null hypothesis, and that our data is fair. In addition, any observed differences could be due to random variation rather than a systematic difference in how well the model performs for individuals with and without preheat. All in all, we can conclude that under the case of precision parity our model performs similarly across the two groups. 

## Precision Difference Plot
<iframe
  src="assets/precision_difference_plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
