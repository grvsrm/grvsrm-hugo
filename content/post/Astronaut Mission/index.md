Predicting Astronauts Mission Duration using Tidymodels in R
================
Gaurav Sharma
17/07/2020

### Load the data

``` r
ttload <- tidytuesdayR::tt_load(x = "2020-07-14")
```

    ## --- Compiling #TidyTuesday Information for 2020-07-14 ----

    ## --- There is 1 file available ---

    ## --- Starting Download ---

    ## 
    ##  Downloading file 1 of 1: `astronauts.csv`

    ## --- Download complete ---

``` r
astranauts <- ttload$astronauts
```

### Some EDA

Chang-Diaz, Franklin R. and Ross, Jerry L. have done the most number of
missions in the history. Both have been to space 7 times. Both belong to
US. Following table sows top 10 astronauts with most number of trips

``` r
astranauts %>% 
    select(name, nationality, sex, hours_mission, total_hrs_sum) %>% 
    count(nationality, name, sex, sort = T) %>% 
    head(10) %>% 
    knitr::kable()
```

| nationality    | name                     | sex  | n |
| :------------- | :----------------------- | :--- | -: |
| U.S.           | Chang-Diaz, Franklin R.  | male | 7 |
| U.S.           | Ross, Jerry L.           | male | 7 |
| U.S.           | Brown, Curtis L., Jr.    | male | 6 |
| U.S.           | Foale, C. Michael        | male | 6 |
| U.S.           | Musgrave, Franklin Story | male | 6 |
| U.S.           | Wetherbee, James D.      | male | 6 |
| U.S.           | Young, John W.           | male | 6 |
| U.S.S.R/Russia | Krikalev, Sergei         | male | 6 |
| U.S.S.R/Russia | Malenchenko, Yuri        | male | 6 |
| U.S.           | Blaha, John E.           | male | 5 |

There are 40 countries that have been to space at least once. US tops
the lists with 854 missions, Russia has 273 mission so far. Following
table sows top 10 countries with most number of space missions.

``` r
astranauts %>% 
    select(nationality) %>% 
    count(nationality, sort = T) %>% 
    head(10) %>% 
    knitr::kable()
```

| nationality    |   n |
| :------------- | --: |
| U.S.           | 854 |
| U.S.S.R/Russia | 273 |
| Japan          |  20 |
| Canada         |  18 |
| France         |  18 |
| Germany        |  16 |
| China          |  14 |
| Italy          |  13 |
| U.K./U.S.      |   6 |
| Australia      |   4 |

Let’s plot a visualization to see how duration of space mission has
change over the decades.

``` r
astranauts %>% 
  mutate(year_of_mission = 10*(year_of_mission %/% 10)) %>% 
  mutate(year_of_mission = factor(year_of_mission)) %>% 
  select(year_of_mission, hours_mission) %>% 
  ggplot(aes(x = year_of_mission, y = hours_mission, color = year_of_mission)) +
  geom_boxplot(show.legend = F) +
  scale_y_log10() +
  ggtitle("The average number of hours spent in space missions have increased significantly after 2010",) +
  labs(x = NULL,
       y = "Duration of Mission in Hours ")
```

    ## Warning: Transformation introduced infinite values in continuous y-axis

    ## Warning: Removed 6 rows containing non-finite values (stat_boxplot).

![](index_files/figure-gfm/unnamed-chunk-4-1.png)<!-- --> Interestingly,
the duration of missions has increased over the decades and their is a
significant jump in median duration after 2010, which generates a
hypothesis here that year of mission can be a good predictor of duration
of space mission. We will confirm this hypothesis later, let’s move
further.

Similarly, We can plot a similar visualization for the number of space
mission in each decade. It will be interesting to see.

``` r
astranauts %>% 
  mutate(year_of_mission = 10*(year_of_mission %/% 10)) %>% 
  mutate(year_of_mission = factor(year_of_mission)) %>% 
  count(year_of_mission) %>% 
  ggplot(aes(x = year_of_mission, y = n, fill = year_of_mission)) +
  geom_col(show.legend = F, alpha = 0.6) +
  ggtitle("The number of space missions see a decreased trend after 1990",) +
  labs(x = NULL,
       y = "Number of Space Missions ") +
  coord_flip()
```

![](index_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

#### Lets do some more analysis to understand what can be a good predictor to predict mission duration

``` r
astranauts %>%
  select(
    sex,
    military_civilian,
    mission_number,
    total_number_of_missions,
    occupation,
    year_of_mission,
    field21,
    hours_mission
  ) %>%
  mutate(year_of_mission = 10 * (year_of_mission %/% 10)) %>% 
  mutate(year_of_mission = factor(year_of_mission)) %>% 
  mutate_if(is.character, factor) %>% 
  mutate(field21 = as.factor(field21)) %>% 
  GGally::ggpairs()
```

    ## Registered S3 method overwritten by 'GGally':
    ##   method from   
    ##   +.gg   ggplot2

    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
    ## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.

![](index_files/figure-gfm/unnamed-chunk-6-1.png)<!-- --> It seems that

  - Sex or military/civilian are not very strong predictors of mission
    duration.
  - Mission number and total number of mission show weak correlation
    with the outcome, hence they also don’t seem to be good predictors.
  - Occupation, year of mission and field 21 seem to be good predictors
    of the outcome.

These arre some good hypothesis to test, We will test them while
building the model.

#### Let’s prepare the dataset to build the model

``` r
astronauts_df <- astranauts %>% 
  select(name, mission_title, hours_mission,
         sex, military_civilian, occupation, year_of_mission, in_orbit, mission_number, total_number_of_missions, field21) %>% 
  filter(hours_mission>0) %>% 
  mutate(hours_mission = log(hours_mission)) %>% 
  mutate(in_orbit = case_when(str_detect(in_orbit, "^Salyut") ~ "Salyut",
                              str_detect(in_orbit, "^STS") ~ "STS",
                              TRUE ~ in_orbit)) %>% 
  mutate(occupation = case_when(str_detect(occupation, "^flight") ~ "Flight engineer",
                                str_detect(occupation, "tourist") ~ "Space tourist",
                                str_detect(occupation, "pilot") ~ "Pilot",
                                TRUE ~ occupation)) %>% 
  mutate(occupation = str_to_title(occupation)) %>% 
  na.omit()
```

#### Lets build the model

We will create splits, make a recipe and then build the model Splits

``` r
astro_split <- astronauts_df %>% 
  initial_split(strata = hours_mission)
astro_train <- training(astro_split)
astro_test <- testing(astro_split)
```

Recipe

``` r
astro_recipe <- recipe(hours_mission~., data = astro_train) %>% 
  update_role(name, mission_title, new_role = "ID") %>% 
  step_other(in_orbit, occupation, threshold = 0.005) %>% 
  themis::step_upsample(sex) %>% 
  step_dummy(all_nominal(), -has_role("ID"))
```

    ## Registered S3 methods overwritten by 'themis':
    ##   method               from   
    ##   bake.step_downsample recipes
    ##   bake.step_upsample   recipes
    ##   prep.step_downsample recipes
    ##   prep.step_upsample   recipes
    ##   tidy.step_downsample recipes
    ##   tidy.step_upsample   recipes

``` r
#install.packages("baguette")
library(baguette)

tree_spec <- bag_tree() %>% 
  set_engine(engine = "rpart", times = 25) %>% 
  set_mode(mode = "regression")


mars_spec <- bag_mars() %>% 
  set_engine(engine = "earth", times = 25) %>% 
  set_mode(mode = "regression")


astro_wf <- workflow() %>% 
  add_recipe(recipe = astro_recipe)


tree_fit <- astro_wf %>% 
  add_model(tree_spec) %>% 
  fit(astro_train)

mars_fit <- astro_wf %>% 
  add_model(mars_spec) %>% 
  fit(astro_train)
```

Let’s check our predictions

``` r
test_rs <- astro_test %>% 
  bind_cols(predict(tree_fit, astro_test)) %>% 
  rename(.pred_tree = .pred) %>% 
  bind_cols(predict(mars_fit, astro_test)) %>% 
  rename(.pred_mars = .pred)
```

Let’s compare the results using some visualizations

``` r
test_rs %>% 
  metrics(hours_mission, .pred_tree)
```

    ## # A tibble: 3 x 3
    ##   .metric .estimator .estimate
    ##   <chr>   <chr>          <dbl>
    ## 1 rmse    standard       0.666
    ## 2 rsq     standard       0.760
    ## 3 mae     standard       0.367

``` r
test_rs %>% 
  metrics(hours_mission, .pred_mars)
```

    ## # A tibble: 3 x 3
    ##   .metric .estimator .estimate
    ##   <chr>   <chr>          <dbl>
    ## 1 rmse    standard       0.667
    ## 2 rsq     standard       0.764
    ## 3 mae     standard       0.403
