---
categories:  
- ""    #the front matter should be like the one found in, e.g., blog2.md. It cannot be like the normal Rmd we used
- ""
date: "2023-06-18"
description: Predicting Bedechel Test Results using Various ML Models # the title that will show up once someone gets to this page
draft: false
image: escalator.jpg # save picture in \static\img\blogs. Acceptable formats= jpg, jpeg, or png . Your iPhone pics wont work

keywords: ""
slug: Machine_Learning # slug is the shorthand URL address... no spaces plz
title: Predicting Bedechel Test Results
  
---

```r
knitr::opts_chunk$set(
  message = FALSE, 
  warning = FALSE, 
  tidy=FALSE,     # display code as typed
  size="small")   # slightly smaller font for code
options(digits = 3)

# default figure size
knitr::opts_chunk$set(
  fig.width=6.75, 
  fig.height=6.75,
  fig.align = "center"
)
```





# The Bechdel Test

https://fivethirtyeight.com/features/the-dollar-and-cents-case-against-hollywoods-exclusion-of-women/

The [Bechdel test](https://bechdeltest.com) is a way to assess how women are depicted in Hollywood movies.  In order for a movie to pass the test:

1. It has to have at least two [named] women in it
2. Who talk to each other
3. About something besides a man


```r
bechdel <- read_csv(here::here("data", "bechdel.csv")) %>% 
  mutate(test = factor(test)) 
glimpse(bechdel)
```

```
## Rows: 1,394
## Columns: 10
## $ year          <dbl> 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 2013, 20…
## $ title         <chr> "12 Years a Slave", "2 Guns", "42", "47 Ronin", "A Good …
## $ test          <fct> Fail, Fail, Fail, Fail, Fail, Pass, Pass, Fail, Pass, Pa…
## $ budget_2013   <dbl> 2.00, 6.10, 4.00, 22.50, 9.20, 1.20, 1.30, 13.00, 4.00, …
## $ domgross_2013 <dbl> 5.311, 7.561, 9.502, 3.836, 6.735, 1.532, 1.801, 6.052, …
## $ intgross_2013 <dbl> 15.861, 13.249, 9.502, 14.580, 30.425, 8.732, 1.801, 24.…
## $ rated         <chr> "R", "R", "PG-13", "PG-13", "R", "R", "PG-13", "PG-13", …
## $ metascore     <dbl> 97, 55, 62, 29, 28, 55, 48, 33, 90, 58, 52, 78, 83, 53, …
## $ imdb_rating   <dbl> 8.3, 6.8, 7.6, 6.6, 5.4, 7.8, 5.7, 5.0, 7.5, 7.4, 6.2, 7…
## $ genre         <chr> "Biography", "Action", "Biography", "Action", "Action", …
```

```r
head(bechdel)
```

```
## # A tibble: 6 × 10
##    year title      test  budget_2013 domgross_2013 intgross_2013 rated metascore
##   <dbl> <chr>      <fct>       <dbl>         <dbl>         <dbl> <chr>     <dbl>
## 1  2013 12 Years … Fail          2            5.31         15.9  R            97
## 2  2013 2 Guns     Fail          6.1          7.56         13.2  R            55
## 3  2013 42         Fail          4            9.50          9.50 PG-13        62
## 4  2013 47 Ronin   Fail         22.5          3.84         14.6  PG-13        29
## 5  2013 A Good Da… Fail          9.2          6.73         30.4  R            28
## 6  2013 About Time Pass          1.2          1.53          8.73 R            55
## # ℹ 2 more variables: imdb_rating <dbl>, genre <chr>
```

How many films fail/pass the test, both as a number and as a %?


```r
#Count how many films pass/fail the test

pass_fail <- bechdel %>% 
  #filter out any NA's in the test column, exclude these films form the stats
  filter(!is.na(test)) %>% 
  #select the test,count, percent columns
  select(test) %>% 
  #mutate to add a count column
  mutate(count = n(),
         percent= count/sum(count)) %>%
  #group by test & summarise
  group_by(test) %>% 
  summarise(count = sum(count),
            percent=sum(percent))

#view the results
pass_fail
```

```
## # A tibble: 2 × 3
##   test    count percent
##   <fct>   <int>   <dbl>
## 1 Fail  1076168   0.554
## 2 Pass   867068   0.446
```


## Movie scores

```r
ggplot(data = bechdel, aes(
  x = metascore,
  y = imdb_rating,
  colour = test
)) +
  geom_point(alpha = .3, size = 3) +
  scale_colour_manual(values = c("tomato", "olivedrab")) +
  labs(
    x = "Metacritic score",
    y = "IMDB rating",
    colour = "Bechdel test"
  ) +
 theme_light()
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-4-1.png" width="648" style="display: block; margin: auto;" />


# Split the data

```r
# **Split the data**

set.seed(123)

data_split <- initial_split(bechdel, # updated data
                           prop = 0.8, 
                           strata = test)

bechdel_train <- training(data_split) 
bechdel_test <- testing(data_split)
```

Check the counts and % (proportions) of the `test` variable in each set.

```r
#calculate stats for training set
train_stats <- bechdel_train %>% 
  #filter out any NA's in the test column, exclude these films form the stats
  filter(!is.na(test)) %>% 
  #select the test,count, percent columns
  select(test) %>% 
  #mutate to add a count column
  mutate(count = n(),
         percent= count/sum(count)) %>%
  #group by test & summarise
  group_by(test) %>% 
  summarise(count = sum(count),
            percent=sum(percent))

#calculate stats for test set
test_stats <- bechdel_test %>% 
  #filter out any NA's in the test column, exclude these films form the stats
  filter(!is.na(test)) %>% 
  #select the test,count, percent columns
  select(test) %>% 
  #mutate to add a count column
  mutate(count = n(),
         percent= count/sum(count)) %>%
  #group by test & summarise
  group_by(test) %>% 
  summarise(count = sum(count),
            percent=sum(percent))

#view the results
test_stats
```

```
## # A tibble: 2 × 3
##   test  count percent
##   <fct> <int>   <dbl>
## 1 Fail  43400   0.554
## 2 Pass  35000   0.446
```

```r
train_stats
```

```
## # A tibble: 2 × 3
##   test   count percent
##   <fct>  <int>   <dbl>
## 1 Fail  687338   0.554
## 2 Pass  553658   0.446
```

```r
#the percentage pass/fail is ~the same for the train and testing data set.
```

## Feature exploration



```r
bechdel %>% 
  select(test, budget_2013, domgross_2013, intgross_2013, imdb_rating, metascore) %>% 

    pivot_longer(cols = 2:6,
               names_to = "feature",
               values_to = "value") %>% 
  ggplot()+
  aes(x=test, y = value, fill = test)+
  coord_flip()+
  geom_boxplot()+
  facet_wrap(~feature, scales = "free")+
  theme_bw()+
  theme(legend.position = "none")+
  labs(x=NULL,y = NULL)
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-7-1.png" width="648" style="display: block; margin: auto;" />

## Scatterplot - Correlation Matrix


```r
bechdel %>% 
  select(test, budget_2013, domgross_2013, intgross_2013, imdb_rating, metascore)%>% 
  ggpairs(aes(colour=test), alpha=0.2)+
  theme_bw()
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-8-1.png" width="648" style="display: block; margin: auto;" />
Explaining the correlation matrix:
[colour references, fail=red and blue=pass.]

From the output we can see that films that failed the bechdel test has higher mean rating (imdb/meta score) and slightly higher domestic/international box office gross figures. The IMDB ratings for both pass/fail movies are normally distributed about the mean, which is similar for the meta score. The scatter plot also shows that IMDB rating and meta score are highly correlated (backed up by the ~0.7 correlation coefficient).
The strongest positive correlation appears to be between international and domestic boxoffice gross (almost 1!).

budget, domestic, and international gross all show significant kurtosis, with extreme right handed skewness visible (implying the majority of the films are towards the lower end of those variables).
1 very significant outliers appear to be visible, one for budget (extremely high value within the fail category).
Overall the pass/fail bedechel test categories appear to have very similar characteristics, with minor differences in the fail category leading to marginally higher mean box office and budget figures.

## Categorical variables



```r
bechdel %>% 
  group_by(genre, test) %>%
  summarise(n = n()) %>% 
  mutate(prop = n/sum(n))
```

```
## # A tibble: 24 × 4
## # Groups:   genre [14]
##    genre     test      n  prop
##    <chr>     <fct> <int> <dbl>
##  1 Action    Fail    260 0.707
##  2 Action    Pass    108 0.293
##  3 Adventure Fail     52 0.559
##  4 Adventure Pass     41 0.441
##  5 Animation Fail     63 0.677
##  6 Animation Pass     30 0.323
##  7 Biography Fail     36 0.554
##  8 Biography Pass     29 0.446
##  9 Comedy    Fail    138 0.427
## 10 Comedy    Pass    185 0.573
## # ℹ 14 more rows
```

```r
bechdel %>% 
  group_by(rated, test) %>%
  summarise(n = n()) %>% 
  mutate(prop = n/sum(n))
```

```
## # A tibble: 10 × 4
## # Groups:   rated [5]
##    rated test      n  prop
##    <chr> <fct> <int> <dbl>
##  1 G     Fail     16 0.615
##  2 G     Pass     10 0.385
##  3 NC-17 Fail      5 0.833
##  4 NC-17 Pass      1 0.167
##  5 PG    Fail    115 0.561
##  6 PG    Pass     90 0.439
##  7 PG-13 Fail    283 0.529
##  8 PG-13 Pass    252 0.471
##  9 R     Fail    353 0.568
## 10 R     Pass    269 0.432
```
genre summary:

Action has the highest failure rate c.70% of the genres with >10 movies.For example, Documentary has a 100% failure rate but only a sample size of 3 which is too small to draw conclusions from.
Horror on the other hand has the highest pass %, with 67% of all Horror films passing the bedechel test.
The generes with the largest number of films (Action, Comedy, drama, Horror) all show significant differences from the overall pass/fail %'s, highlighting significant differencess in the bedechel test outcomes based on the genre selected.

Ratings summary:

PG,PG-13, and R rated films all showcase similar behaviour to the overall sample in terms of the pass/fail %'s.
However, G & NC-17 exhibited markedly different results, with much higher % rates of failure than the average. Again this showcases that the pass/failure rate is influenced by the rating for those particular cases.

# Train first models. `test ~ metascore + imdb_rating`


```r
lr_mod <- logistic_reg() %>% 
  set_engine(engine = "glm") %>% 
  set_mode("classification")

lr_mod
```

```
## Logistic Regression Model Specification (classification)
## 
## Computational engine: glm
```

```r
tree_mod <- decision_tree() %>% 
  set_engine(engine = "C5.0") %>% 
  set_mode("classification")

tree_mod 
```

```
## Decision Tree Model Specification (classification)
## 
## Computational engine: C5.0
```


```r
lr_fit <- lr_mod %>% # parsnip model
  fit(test ~ metascore + imdb_rating, # a formula
    data = bechdel_train # dataframe
  )

tree_fit <- tree_mod %>% # parsnip model
  fit(test ~ metascore + imdb_rating, # a formula
    data = bechdel_train # dataframe
  )
```

## Logistic regression


```r
lr_fit %>%
  broom::tidy()
```

```
## # A tibble: 3 × 5
##   term        estimate std.error statistic  p.value
##   <chr>          <dbl>     <dbl>     <dbl>    <dbl>
## 1 (Intercept)   2.80     0.494        5.68 1.35e- 8
## 2 metascore     0.0207   0.00536      3.86 1.13e- 4
## 3 imdb_rating  -0.625    0.100       -6.24 4.36e-10
```

```r
lr_preds <- lr_fit %>%
  augment(new_data = bechdel_train) %>%
  mutate(.pred_match = if_else(test == .pred_class, 1, 0))
```

### Confusion matrix


```r
lr_preds %>% 
  conf_mat(truth = test, estimate = .pred_class) %>% 
  autoplot(type = "heatmap")
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-13-1.png" width="648" style="display: block; margin: auto;" />


## Decision Tree

```r
tree_preds <- tree_fit %>%
  augment(new_data = bechdel) %>%
  mutate(.pred_match = if_else(test == .pred_class, 1, 0)) 
```


```r
tree_preds %>% 
  conf_mat(truth = test, estimate = .pred_class) %>% 
  autoplot(type = "heatmap")
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-15-1.png" width="648" style="display: block; margin: auto;" />

## Draw the decision tree


```r
draw_tree <- 
    rpart::rpart(
        test ~ metascore + imdb_rating,
        data = bechdel_train, # uses data that contains both birth weight and `low`
        control = rpart::rpart.control(maxdepth = 5, cp = 0, minsplit = 10)
    ) %>% 
    partykit::as.party()
plot(draw_tree)
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-16-1.png" width="648" style="display: block; margin: auto;" />

# Cross Validation

Run the code below. What does it return?
The number of folds used for the model (i.e- how the model has been segmented, allows for corss validation with the training data split into equally sized folds)


```r
set.seed(123)
bechdel_folds <- vfold_cv(data = bechdel_train, 
                          v = 2, 
                          strata = test)
bechdel_folds
```

```
## #  2-fold cross-validation using stratification 
## # A tibble: 2 × 2
##   splits            id   
##   <list>            <chr>
## 1 <split [556/558]> Fold1
## 2 <split [558/556]> Fold2
```

## `fit_resamples()`

Trains and tests a resampled model.


```r
lr_fit <- lr_mod %>%
  fit_resamples(
    test ~ metascore + imdb_rating,
    resamples = bechdel_folds
  )


tree_fit <- tree_mod %>%
  fit_resamples(
    test ~ metascore + imdb_rating,
    resamples = bechdel_folds
  )
```


## `collect_metrics()`

Unnest the metrics column from a tidymodels `fit_resamples()`

```r
collect_metrics(lr_fit)
```

```
## # A tibble: 2 × 6
##   .metric  .estimator  mean     n std_err .config             
##   <chr>    <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy binary     0.574     2  0.0169 Preprocessor1_Model1
## 2 roc_auc  binary     0.602     2  0.0148 Preprocessor1_Model1
```

```r
collect_metrics(tree_fit)
```

```
## # A tibble: 2 × 6
##   .metric  .estimator  mean     n std_err .config             
##   <chr>    <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy binary     0.551     2 0.00260 Preprocessor1_Model1
## 2 roc_auc  binary     0.510     2 0.0103  Preprocessor1_Model1
```



```r
tree_preds <- tree_mod %>% 
  fit_resamples(
    test ~ metascore + imdb_rating, 
    resamples = bechdel_folds,
    control = control_resamples(save_pred = TRUE) #<<
  )

# What does the data for ROC look like?
tree_preds %>% 
  collect_predictions() %>% 
  roc_curve(truth = test, .pred_Fail)  
```

```
## # A tibble: 5 × 3
##   .threshold specificity sensitivity
##        <dbl>       <dbl>       <dbl>
## 1   -Inf           0           1    
## 2      0.391       0           1    
## 3      0.554       0.131       0.890
## 4      0.607       0.632       0.389
## 5    Inf           1           0
```

```r
# Draw the ROC
tree_preds %>% 
  collect_predictions() %>% 
  roc_curve(truth = test, .pred_Fail) %>% 
  autoplot()
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-20-1.png" width="648" style="display: block; margin: auto;" />


# Build a better training set with `recipes`

## Preprocessing options

- Encode categorical predictors
- Center and scale variables
- Handle class imbalance
- Impute missing data
- Perform dimensionality reduction 
- ... ...

## To build a recipe

1. Start the `recipe()`
1. Define the variables involved
1. Describe **prep**rocessing [step-by-step]

## Collapse Some Categorical Levels

Do we have any `genre` with few observations?  Assign genres that have less than 3% to a new category 'Other'


<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-21-1.png" width="648" style="display: block; margin: auto;" />



```r
movie_rec <-
  recipe(test ~ .,
         data = bechdel_train) %>%
  
  # Genres with less than 5% will be in a catewgory 'Other'
    step_other(genre, threshold = .03) 
```
  

## Before recipe


```
## # A tibble: 14 × 2
##    genre           n
##    <chr>       <int>
##  1 Action        293
##  2 Comedy        254
##  3 Drama         213
##  4 Adventure      75
##  5 Animation      72
##  6 Crime          68
##  7 Horror         68
##  8 Biography      50
##  9 Mystery         7
## 10 Fantasy         5
## 11 Sci-Fi          3
## 12 Thriller        3
## 13 Documentary     2
## 14 Musical         1
```


## After recipe


```r
movie_rec %>% 
  prep() %>% 
  bake(new_data = bechdel_train) %>% 
  count(genre, sort = TRUE)
```

```
## # A tibble: 9 × 2
##   genre         n
##   <fct>     <int>
## 1 Action      293
## 2 Comedy      254
## 3 Drama       213
## 4 Adventure    75
## 5 Animation    72
## 6 Crime        68
## 7 Horror       68
## 8 Biography    50
## 9 other        21
```

## `step_dummy()`

Converts nominal data into numeric dummy variables


```r
movie_rec <- recipe(test ~ ., data = bechdel) %>%
  step_other(genre, threshold = .03) %>% 
  step_dummy(all_nominal_predictors()) 

movie_rec 
```

## `step_novel()`

Adds a catch-all level to a factor for any new values not encountered in model training, which lets R intelligently predict new levels in the test set.


```r
movie_rec <- recipe(test ~ ., data = bechdel) %>%
  step_other(genre, threshold = .03) %>% 
  step_novel(all_nominal_predictors) %>% # Use *before* `step_dummy()` so new level is dummified
  step_dummy(all_nominal_predictors()) 
```


## `step_zv()`

Intelligently handles zero variance variables (variables that contain only a single value)


```r
movie_rec <- recipe(test ~ ., data = bechdel) %>%
  step_other(genre, threshold = .03) %>% 
  step_novel(all_nominal(), -all_outcomes()) %>% # Use *before* `step_dummy()` so new level is dummified
  step_dummy(all_nominal(), -all_outcomes()) %>% 
  step_zv(all_numeric(), -all_outcomes()) 
```


## `step_normalize()`

Centers then scales numeric variable (mean = 0, sd = 1)


```r
movie_rec <- recipe(test ~ ., data = bechdel) %>%
  step_other(genre, threshold = .03) %>% 
  step_novel(all_nominal(), -all_outcomes()) %>% # Use *before* `step_dummy()` so new level is dummified
  step_dummy(all_nominal(), -all_outcomes()) %>% 
  step_zv(all_numeric(), -all_outcomes())  %>% 
  step_normalize(all_numeric()) 
```


## `step_corr()`

Removes highly correlated variables


```r
movie_rec <- recipe(test ~ ., data = bechdel) %>%
  step_other(genre, threshold = .03) %>% 
  step_novel(all_nominal(), -all_outcomes()) %>% # Use *before* `step_dummy()` so new level is dummified
  step_dummy(all_nominal(), -all_outcomes()) %>% 
  step_zv(all_numeric(), -all_outcomes())  %>% 
  step_normalize(all_numeric()) %>% 
  step_corr(all_predictors(), threshold = 0.75, method = "spearman") 

movie_rec
```


# Define different models to fit


```r
## Model Building

# 1. Pick a `model type`
# 2. set the `engine`
# 3. Set the `mode`: regression or classification

# Logistic regression
log_spec <-  logistic_reg() %>%  # model type
  set_engine(engine = "glm") %>%  # model engine
  set_mode("classification") # model mode

# Show your model specification
log_spec
```

```
## Logistic Regression Model Specification (classification)
## 
## Computational engine: glm
```

```r
# Decision Tree
tree_spec <- decision_tree() %>%
  set_engine(engine = "C5.0") %>%
  set_mode("classification")

tree_spec
```

```
## Decision Tree Model Specification (classification)
## 
## Computational engine: C5.0
```

```r
# Random Forest
library(ranger)

rf_spec <- 
  rand_forest() %>% 
  set_engine("ranger", importance = "impurity") %>% 
  set_mode("classification")


# Boosted tree (XGBoost)
library(xgboost)

xgb_spec <- 
  boost_tree() %>% 
  set_engine("xgboost") %>% 
  set_mode("classification") 

# K-nearest neighbour (k-NN)
knn_spec <- 
  nearest_neighbor(neighbors = 4) %>% # we can adjust the number of neighbors 
  set_engine("kknn") %>% 
  set_mode("classification") 
```


# Bundle recipe and model with `workflows`



```r
log_wflow <- # new workflow object
 workflow() %>% # use workflow function
 add_recipe(movie_rec) %>%   # use the new recipe
 add_model(log_spec)   # add your model spec

# show object
log_wflow
```

```
## ══ Workflow ════════════════════════════════════════════════════════════════════
## Preprocessor: Recipe
## Model: logistic_reg()
## 
## ── Preprocessor ────────────────────────────────────────────────────────────────
## 6 Recipe Steps
## 
## • step_other()
## • step_novel()
## • step_dummy()
## • step_zv()
## • step_normalize()
## • step_corr()
## 
## ── Model ───────────────────────────────────────────────────────────────────────
## Logistic Regression Model Specification (classification)
## 
## Computational engine: glm
```

```r
## A few more workflows

tree_wflow <-
 workflow() %>%
 add_recipe(movie_rec) %>% 
 add_model(tree_spec) 

rf_wflow <-
 workflow() %>%
 add_recipe(movie_rec) %>% 
 add_model(rf_spec) 

xgb_wflow <-
 workflow() %>%
 add_recipe(movie_rec) %>% 
 add_model(xgb_spec)

knn_wflow <-
 workflow() %>%
 add_recipe(movie_rec) %>% 
 add_model(knn_spec)
```

## Model Comparison


```r
#utilising the workflows to evaluate the models
#check which model has the best predictor

log_res <- log_wflow %>% 
  fit_resamples(
    resamples = bechdel_folds, 
    metrics = metric_set(
      recall, precision, f_meas, accuracy,
      kap, roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)) 

# Show average performance over all folds (note that we use log_res):
log_res %>%  collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 6
##   .metric   .estimator   mean     n std_err .config             
##   <chr>     <chr>       <dbl> <int>   <dbl> <chr>               
## 1 accuracy  binary      0.422     2 0.00255 Preprocessor1_Model1
## 2 f_meas    binary      0.384     2 0.0396  Preprocessor1_Model1
## 3 kap       binary     -0.129     2 0.0204  Preprocessor1_Model1
## 4 precision binary      0.469     2 0.00184 Preprocessor1_Model1
## 5 recall    binary      0.329     2 0.0562  Preprocessor1_Model1
## 6 roc_auc   binary      0.416     2 0.0111  Preprocessor1_Model1
## 7 sens      binary      0.329     2 0.0562  Preprocessor1_Model1
## 8 spec      binary      0.537     2 0.0755  Preprocessor1_Model1
```

```r
# Show performance for every single fold:
log_res %>%  collect_metrics(summarize = FALSE)
```

```
## # A tibble: 16 × 5
##    id    .metric   .estimator .estimate .config             
##    <chr> <chr>     <chr>          <dbl> <chr>               
##  1 Fold1 recall    binary         0.385 Preprocessor1_Model1
##  2 Fold1 precision binary         0.470 Preprocessor1_Model1
##  3 Fold1 f_meas    binary         0.423 Preprocessor1_Model1
##  4 Fold1 accuracy  binary         0.419 Preprocessor1_Model1
##  5 Fold1 kap       binary        -0.150 Preprocessor1_Model1
##  6 Fold1 sens      binary         0.385 Preprocessor1_Model1
##  7 Fold1 spec      binary         0.462 Preprocessor1_Model1
##  8 Fold1 roc_auc   binary         0.405 Preprocessor1_Model1
##  9 Fold2 recall    binary         0.273 Preprocessor1_Model1
## 10 Fold2 precision binary         0.467 Preprocessor1_Model1
## 11 Fold2 f_meas    binary         0.344 Preprocessor1_Model1
## 12 Fold2 accuracy  binary         0.424 Preprocessor1_Model1
## 13 Fold2 kap       binary        -0.109 Preprocessor1_Model1
## 14 Fold2 sens      binary         0.273 Preprocessor1_Model1
## 15 Fold2 spec      binary         0.613 Preprocessor1_Model1
## 16 Fold2 roc_auc   binary         0.427 Preprocessor1_Model1
```

```r
## `collect_predictions()` and get confusion matrix{.smaller}

log_pred <- log_res %>% collect_predictions()

log_pred %>%  conf_mat(test, .pred_class) 
```

```
##           Truth
## Prediction Fail Pass
##       Fail  203  230
##       Pass  414  267
```

```r
log_pred %>% 
  conf_mat(test, .pred_class) %>% 
  autoplot(type = "mosaic") +
  geom_label(aes(
      x = (xmax + xmin) / 2, 
      y = (ymax + ymin) / 2, 
      label = c("TP", "FN", "FP", "TN")))
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-1.png" width="648" style="display: block; margin: auto;" />

```r
log_pred %>% 
  conf_mat(test, .pred_class) %>% 
  autoplot(type = "heatmap")
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-2.png" width="648" style="display: block; margin: auto;" />

```r
## ROC Curve

log_pred %>% 
  group_by(id) %>% # id contains our folds
  roc_curve(test, .pred_Pass) %>% 
  autoplot()
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-3.png" width="648" style="display: block; margin: auto;" />

```r
## Decision Tree results

tree_res <-
  tree_wflow %>% 
  fit_resamples(
    resamples = bechdel_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)
    ) 

tree_res %>%  collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 6
##   .metric   .estimator  mean     n std_err .config             
##   <chr>     <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy  binary     0.581     2  0.0250 Preprocessor1_Model1
## 2 f_meas    binary     0.636     2  0.0392 Preprocessor1_Model1
## 3 kap       binary     0.143     2  0.0405 Preprocessor1_Model1
## 4 precision binary     0.610     2  0.0103 Preprocessor1_Model1
## 5 recall    binary     0.668     2  0.0735 Preprocessor1_Model1
## 6 roc_auc   binary     0.558     2  0.0138 Preprocessor1_Model1
## 7 sens      binary     0.668     2  0.0735 Preprocessor1_Model1
## 8 spec      binary     0.473     2  0.0352 Preprocessor1_Model1
```

```r
## Random Forest

rf_res <-
  rf_wflow %>% 
  fit_resamples(
    resamples = bechdel_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)
    ) 

rf_res %>%  collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 6
##   .metric   .estimator  mean     n std_err .config             
##   <chr>     <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy  binary     0.633     2 0.00156 Preprocessor1_Model1
## 2 f_meas    binary     0.692     2 0.00133 Preprocessor1_Model1
## 3 kap       binary     0.243     2 0.00495 Preprocessor1_Model1
## 4 precision binary     0.646     2 0.00346 Preprocessor1_Model1
## 5 recall    binary     0.746     2 0.00769 Preprocessor1_Model1
## 6 roc_auc   binary     0.647     2 0.0267  Preprocessor1_Model1
## 7 sens      binary     0.746     2 0.00769 Preprocessor1_Model1
## 8 spec      binary     0.493     2 0.0131  Preprocessor1_Model1
```

```r
## Boosted tree - XGBoost

xgb_res <- 
  xgb_wflow %>% 
  fit_resamples(
    resamples = bechdel_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)
    ) 

xgb_res %>% collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 6
##   .metric   .estimator  mean     n std_err .config             
##   <chr>     <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy  binary     0.598     2  0.0366 Preprocessor1_Model1
## 2 f_meas    binary     0.647     2  0.0282 Preprocessor1_Model1
## 3 kap       binary     0.180     2  0.0769 Preprocessor1_Model1
## 4 precision binary     0.630     2  0.0341 Preprocessor1_Model1
## 5 recall    binary     0.664     2  0.0216 Preprocessor1_Model1
## 6 roc_auc   binary     0.612     2  0.0256 Preprocessor1_Model1
## 7 sens      binary     0.664     2  0.0216 Preprocessor1_Model1
## 8 spec      binary     0.515     2  0.0553 Preprocessor1_Model1
```

```r
## K-nearest neighbour

knn_res <- 
  knn_wflow %>% 
  fit_resamples(
    resamples = bechdel_folds, 
    metrics = metric_set(
      recall, precision, f_meas, 
      accuracy, kap,
      roc_auc, sens, spec),
    control = control_resamples(save_pred = TRUE)
    ) 

knn_res %>% collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 6
##   .metric   .estimator     mean     n  std_err .config             
##   <chr>     <chr>         <dbl> <int>    <dbl> <chr>               
## 1 accuracy  binary      0.552       2 0.00170  Preprocessor1_Model1
## 2 f_meas    binary      0.711       2 0.00141  Preprocessor1_Model1
## 3 kap       binary     -0.00359     2 0.00359  Preprocessor1_Model1
## 4 precision binary      0.553       2 0.000708 Preprocessor1_Model1
## 5 recall    binary      0.997       2 0.00325  Preprocessor1_Model1
## 6 roc_auc   binary      0.577       2 0.0151   Preprocessor1_Model1
## 7 sens      binary      0.997       2 0.00325  Preprocessor1_Model1
## 8 spec      binary      0           2 0        Preprocessor1_Model1
```

```r
## Model Comparison

log_metrics <- 
  log_res %>% 
  collect_metrics(summarise = TRUE) %>%
  # add the name of the model to every row
  mutate(model = "Logistic Regression") 

tree_metrics <- 
  tree_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Decision Tree")

rf_metrics <- 
  rf_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Random Forest")

xgb_metrics <- 
  xgb_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "XGBoost")

knn_metrics <- 
  knn_res %>% 
  collect_metrics(summarise = TRUE) %>%
  mutate(model = "Knn")

# create dataframe with all models
model_compare <- bind_rows(log_metrics,
                           tree_metrics,
                           rf_metrics,
                           xgb_metrics,
                           knn_metrics) 

#Pivot wider to create barplot
  model_comp <- model_compare %>% 
  select(model, .metric, mean, std_err) %>% 
  pivot_wider(names_from = .metric, values_from = c(mean, std_err)) 

# show mean are under the curve (ROC-AUC) for every model
model_comp %>% 
  arrange(mean_roc_auc) %>% 
  mutate(model = fct_reorder(model, mean_roc_auc)) %>% # order results
  ggplot(aes(model, mean_roc_auc, fill=model)) +
  geom_col() +
  coord_flip() +
  scale_fill_brewer(palette = "Blues") +
   geom_text(
     size = 3,
     aes(label = round(mean_roc_auc, 2), 
         y = mean_roc_auc + 0.08),
     vjust = 1
  )+
  theme_light()+
  theme(legend.position = "none")+
  labs(y = NULL)
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-4.png" width="648" style="display: block; margin: auto;" />

```r
## `last_fit()` on test set

# - `last_fit()`  fits a model to the whole training data and evaluates it on the test set. 
# - provide the workflow object of the best model as well as the data split object (not the training data). 
 
last_fit_xgb <- last_fit(xgb_wflow, 
                        split = data_split,
                        metrics = metric_set(
                          accuracy, f_meas, kap, precision,
                          recall, roc_auc, sens, spec))

last_fit_xgb %>% collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 4
##   .metric   .estimator .estimate .config             
##   <chr>     <chr>          <dbl> <chr>               
## 1 accuracy  binary         0.579 Preprocessor1_Model1
## 2 f_meas    binary         0.642 Preprocessor1_Model1
## 3 kap       binary         0.134 Preprocessor1_Model1
## 4 precision binary         0.606 Preprocessor1_Model1
## 5 recall    binary         0.684 Preprocessor1_Model1
## 6 sens      binary         0.684 Preprocessor1_Model1
## 7 spec      binary         0.448 Preprocessor1_Model1
## 8 roc_auc   binary         0.598 Preprocessor1_Model1
```

```r
#Compare to training
xgb_res %>% collect_metrics(summarize = TRUE)
```

```
## # A tibble: 8 × 6
##   .metric   .estimator  mean     n std_err .config             
##   <chr>     <chr>      <dbl> <int>   <dbl> <chr>               
## 1 accuracy  binary     0.598     2  0.0366 Preprocessor1_Model1
## 2 f_meas    binary     0.647     2  0.0282 Preprocessor1_Model1
## 3 kap       binary     0.180     2  0.0769 Preprocessor1_Model1
## 4 precision binary     0.630     2  0.0341 Preprocessor1_Model1
## 5 recall    binary     0.664     2  0.0216 Preprocessor1_Model1
## 6 roc_auc   binary     0.612     2  0.0256 Preprocessor1_Model1
## 7 sens      binary     0.664     2  0.0216 Preprocessor1_Model1
## 8 spec      binary     0.515     2  0.0553 Preprocessor1_Model1
```

```r
## Variable importance using `{vip}` package

library(vip)

last_fit_xgb %>% 
  pluck(".workflow", 1) %>%   
  pull_workflow_fit() %>% 
  vip(num_features = 10) +
  theme_light()
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-5.png" width="648" style="display: block; margin: auto;" />

```r
## Final Confusion Matrix

last_fit_xgb %>%
  collect_predictions() %>% 
  conf_mat(test, .pred_class) %>% 
  autoplot(type = "heatmap")
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-6.png" width="648" style="display: block; margin: auto;" />

```r
## Final ROC curve
last_fit_xgb %>% 
  collect_predictions() %>% 
  roc_curve(test, .pred_Pass) %>% 
  autoplot()
```

<img src="/blogs/Machine_Learning_files/figure-html/unnamed-chunk-32-7.png" width="648" style="display: block; margin: auto;" />

Random Forest appears to be the best model (with 2 Folds)



