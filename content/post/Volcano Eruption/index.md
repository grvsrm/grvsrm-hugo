---
title: "A Classification Model on Volcanic Eruptions in R"
subtitle: "An example of Multinomial Classification using Random Forest and Tidymodels in R"
summary: "An example of Multinomial Classification using Random Forest and Tidymodels in R"
tags: [rstats,tidymodels,random forest, classification]
date: 30-06-2020
authors: ["admin"]
share: false
image:
  placement: 1
  caption: " Image by Julius Silver from Pixabay "
  preview_only: false
---

NOTE: These days I am following [Julia Silge](https://juliasilge.com/) for learning tidymodels framework better. This post is inspired from what I learned there. 

I love doing data science in R. It is so easy to follow along when you work in R. In this post, we will talk about a volcano eruption dataset available [here](https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-05-12/volcano.csv). This dataset talks about various types of volcanic eruptions that take place around the world. We are interested in understanding that by having demographic information present in data, Is it possible to correctly identify the type of the volcanic eruption. It is a multiclass or multinomial classification probelm and we will use random forest algorithm to train the model. Let's get started..!

Let's load the data and see what we have here.
``` r
volcano_raw <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-05-12/volcano.csv")
volcano_raw %>% 
    head() %>% 
    knitr::kable()
```

| volcano\_number | volcano\_name   | primary\_volcano\_type | last\_eruption\_year | country       | region                         | subregion                             | latitude | longitude | elevation | tectonic\_settings                            | evidence\_category | major\_rock\_1               | major\_rock\_2               | major\_rock\_3          | major\_rock\_4               | major\_rock\_5 | minor\_rock\_1        | minor\_rock\_2                   | minor\_rock\_3               | minor\_rock\_4 | minor\_rock\_5 | population\_within\_5\_km | population\_within\_10\_km | population\_within\_30\_km | population\_within\_100\_km |
| --------------: | :-------------- | :--------------------- | :------------------- | :------------ | :----------------------------- | :------------------------------------ | -------: | --------: | --------: | :-------------------------------------------- | :----------------- | :--------------------------- | :--------------------------- | :---------------------- | :--------------------------- | :------------- | :-------------------- | :------------------------------- | :--------------------------- | :------------- | :------------- | ------------------------: | -------------------------: | -------------------------: | --------------------------: |
|          283001 | Abu             | Shield(s)              | \-6850               | Japan         | Japan, Taiwan, Marianas        | Honshu                                |   34.500 |   131.600 |       641 | Subduction zone / Continental crust (\>25 km) | Eruption Dated     | Andesite / Basaltic Andesite | Basalt / Picro-Basalt        | Dacite                  |                              |                |                       |                                  |                              |                |                |                      3597 |                       9594 |                     117805 |                     4071152 |
|          355096 | Acamarachi      | Stratovolcano          | Unknown              | Chile         | South America                  | Northern Chile, Bolivia and Argentina | \-23.292 |  \-67.618 |      6023 | Subduction zone / Continental crust (\>25 km) | Evidence Credible  | Dacite                       | Andesite / Basaltic Andesite |                         |                              |                |                       |                                  |                              |                |                |                         0 |                          7 |                        294 |                        9092 |
|          342080 | Acatenango      | Stratovolcano(es)      | 1972                 | Guatemala     | México and Central America     | Guatemala                             |   14.501 |  \-90.876 |      3976 | Subduction zone / Continental crust (\>25 km) | Eruption Observed  | Andesite / Basaltic Andesite | Dacite                       |                         |                              |                | Basalt / Picro-Basalt |                                  |                              |                |                |                      4329 |                      60730 |                    1042836 |                     7634778 |
|          213004 | Acigol-Nevsehir | Caldera                | \-2080               | Turkey        | Mediterranean and Western Asia | Turkey                                |   38.537 |    34.621 |      1683 | Intraplate / Continental crust (\>25 km)      | Eruption Dated     | Rhyolite                     | Dacite                       | Basalt / Picro-Basalt   | Andesite / Basaltic Andesite |                |                       |                                  |                              |                |                |                    127863 |                     127863 |                     218469 |                     2253483 |
|          321040 | Adams           | Stratovolcano          | 950                  | United States | Canada and Western USA         | USA (Washington)                      |   46.206 | \-121.490 |      3742 | Subduction zone / Continental crust (\>25 km) | Eruption Dated     | Andesite / Basaltic Andesite | Basalt / Picro-Basalt        |                         |                              |                | Dacite                |                                  |                              |                |                |                         0 |                         70 |                       4019 |                      393303 |
|          283170 | Adatarayama     | Stratovolcano(es)      | 1996                 | Japan         | Japan, Taiwan, Marianas        | Honshu                                |   37.647 |   140.281 |      1728 | Subduction zone / Continental crust (\>25 km) | Eruption Observed  | Andesite / Basaltic Andesite |                              |                         |                              |                | Dacite                | Basalt / Picro-Basalt            |                              |                |                |                       428 |                       3936 |                     717078 |                     5024654 |
|          221170 | Adwa            | Stratovolcano          | Unknown              | Ethiopia      | Africa and Red Sea             | Africa (northeastern) and Red Sea     |   10.070 |    40.840 |      1733 | Rift zone / Intermediate crust (15-25 km)     | Evidence Credible  | Rhyolite                     | Basalt / Picro-Basalt        | Trachyte / Trachydacite |                              |                |                       |                                  |                              |                |                |                       101 |                        485 |                      18645 |                     1242922 |
|          221110 | Afdera          | Stratovolcano          | Unknown              | Ethiopia      | Africa and Red Sea             | Africa (northeastern) and Red Sea     |   13.088 |    40.853 |      1250 | Rift zone / Intermediate crust (15-25 km)     | Evidence Uncertain | Rhyolite                     |                              |                         |                              |                | Basalt / Picro-Basalt | Trachybasalt / Tephrite Basanite | Andesite / Basaltic Andesite |                |                |                        51 |                       6042 |                       8611 |                      161009 |
|          284160 | Agrigan         | Stratovolcano          | 1917                 | United States | Japan, Taiwan, Marianas        | Izu, Volcano, and Mariana Islands     |   18.770 |   145.670 |       965 | Subduction zone / Crustal thickness unknown   | Eruption Observed  | Basalt / Picro-Basalt        | Andesite / Basaltic Andesite |                         |                              |                |                       |                                  |                              |                |                |                         0 |                          0 |                          0 |                           0 |
|          342100 | Agua            | Stratovolcano          | Unknown              | Guatemala     | México and Central America     | Guatemala                             |   14.465 |  \-90.743 |      3760 | Subduction zone / Continental crust (\>25 km) | Evidence Credible  | Andesite / Basaltic Andesite |                              |                         |                              |                | Dacite                |                                  |                              |                |                |                      9890 |                     114404 |                    2530449 |                     7441660 |

### Now, Let's explore the data
We will check the different type of volcanic eruptions that are given in the dataset. We see that there are 26 types of eruptions listed here.
``` r
volcano_raw %>% 
    count(primary_volcano_type, sort = T)
```

    ## # A tibble: 26 x 2
    ##    primary_volcano_type     n
    ##    <chr>                <int>
    ##  1 Stratovolcano          353
    ##  2 Stratovolcano(es)      107
    ##  3 Shield                  85
    ##  4 Volcanic field          71
    ##  5 Pyroclastic cone(s)     70
    ##  6 Caldera                 65
    ##  7 Complex                 46
    ##  8 Shield(s)               33
    ##  9 Submarine               27
    ## 10 Lava dome(s)            26
    ## # ... with 16 more rows


It shows that there are 26 different types of eruptions. Although it can be modelled to classify for all the 26 types but seeing the size of the data it makes more sense to group the eruptions in three categories. Lets take the first two categories and put rest of the data in "other" category to keep it simple.

``` r
volcano_df <- volcano_raw %>% 
    transmute(volcano_type = case_when(str_detect(primary_volcano_type, "Stratovolcano") ~ "Stratovolcano",
                                       str_detect(primary_volcano_type, "Shield") ~ "Shield",
                                       TRUE ~ "Others"),
              volcano_number, latitude, longitude, tectonic_settings, elevation, major_rock_1) %>% 
    mutate_if(is.character, factor)
```

### Lets plot some visualizations
Since we have spatial data, Lets see where these volcanos are situated on a world map. We will put these volcanos on their location and will color them according to the volcanic type. Let's see, how it looks..!

``` r
world <- map_data("world")

ggplot() +
    geom_map(data = world, map = world,
             aes(x = long, y = lat, map_id = region), color = "white", fill = "gray50", alpha = 0.2) +
    geom_point(data = volcano_df, aes(longitude, latitude, color = volcano_type), alpha = 0.5) +
    labs(x = NULL, y = NULL,
         title = "Different Type of Volcanic Eruptions around the world") +
    scale_x_continuous(labels = NULL) +
    scale_y_continuous(labels = NULL) 
```

![Volcano Map](plots/Volcano%20Map-1.png)

It looks like these volcanos create a ring of fire around the pacific ocean. Also, they are spreaded all over the world but we can see most of them in coastal areas.

### Let's prepare the data
Rather than creating a split we will create bootstrap resamples as we don’t have much data. Hence resampling will help in preventing the model from overfitting and will also give us a robust fit. 

``` r
volcano_boot <- volcano_df %>% 
    bootstraps()
volcano_boot
```

    ## # Bootstrap sampling 
    ## # A tibble: 25 x 2
    ##    splits            id         
    ##    <list>            <chr>      
    ##  1 <split [958/353]> Bootstrap01
    ##  2 <split [958/357]> Bootstrap02
    ##  3 <split [958/340]> Bootstrap03
    ##  4 <split [958/358]> Bootstrap04
    ##  5 <split [958/363]> Bootstrap05
    ##  6 <split [958/361]> Bootstrap06
    ##  7 <split [958/336]> Bootstrap07
    ##  8 <split [958/353]> Bootstrap08
    ##  9 <split [958/353]> Bootstrap09
    ## 10 <split [958/368]> Bootstrap10
    ## # ... with 15 more rows

### Let's pre-process the data
We will create a recipe now. We will use smote analysis to overcome class imbalance. smote() function is a part of themis package. So, remember to load themis package before creating recipe. We will also remove volcano number as it is not useful in modelling. We will group the less frequent classes and put them into a class called 'other'. We will do the same thing with major_rock_1 variable also. Then we will create dummy variable for all categorical variables. will remove zero variance variable and normalize the numerical variable 

``` r
library(themis)

volcano_rec <- recipe(volcano_type~., data = volcano_df) %>% 
    update_role(volcano_number, new_role = "id") %>% 
    step_other(tectonic_settings) %>% 
    step_other(major_rock_1) %>% 
    step_dummy(tectonic_settings, major_rock_1) %>% 
    step_zv(all_predictors()) %>% 
    step_normalize(all_predictors()) %>% 
    step_smote(volcano_type)

volcano_prep <- prep(volcano_rec)
volcano_prep
```

### Lets create a model spec
Lets create a random forest model specification. We will set engine as ranger and mode as classification. We keep number of trees equal to 1000 to be grown.

``` r
rf_spec <- rand_forest(trees = 1000) %>% 
    set_engine(engine = "ranger") %>% 
    set_mode(mode = "classification")
```

### Lets create a workflow
Workflow helps us in putting all the things together which in turn helps us in modelling faster. We are adding recipe and model specs heer to the model.

``` r
volcano_wf <- workflow() %>% 
    add_recipe(recipe = volcano_rec) %>% 
    add_model(rf_spec)
```

### Lets train the model
We will train the model now using bootstrap resamples. Verbose argument is not necessary as in but it helps us in seeing the progress while random forest algorithm converges.
 
``` r
volcano_res <- fit_resamples(volcano_wf, 
              resamples = volcano_boot,
              control = control_resamples(save_pred = T, 
                                          verbose = T))
```

### Explore results

Let’s see how the model performs on the resampled data. Although area under the curve is ~79% still model doesn't have a good accuracy. It fares slightly better than a base model.

``` r
volcano_res %>% 
    collect_metrics()
```

    ## # A tibble: 2 x 5
    ##   .metric  .estimator  mean     n std_err
    ##   <chr>    <chr>      <dbl> <int>   <dbl>
    ## 1 accuracy multiclass 0.648    25 0.00506
    ## 2 roc_auc  hand_till  0.788    25 0.00414

Lets have a look at the confusion metrics to see how model predicts different classes.

``` r
volcano_res %>% 
    collect_predictions() %>% 
    conf_mat(volcano_type, .pred_class)
```

    ##                Truth
    ## Prediction      Others Shield Stratovolcano
    ##   Others          1960    344           853
    ##   Shield           289    583           222
    ##   Stratovolcano   1236    193          3238

We will also like to see the positive predicted value which is a useful measure in multinomial classification models.

``` r
volcano_res %>% 
    collect_predictions() %>% 
    ppv(volcano_type, .pred_class)
```

    ## # A tibble: 1 x 3
    ##   .metric .estimator .estimate
    ##   <chr>   <chr>          <dbl>
    ## 1 ppv     macro          0.616

We can check the ppv across various bootstraps to see how it behaves in different data samples. It doesn't change much, which indicates that model is not too sensitive to the new data.
``` r
volcano_res %>% 
    collect_predictions() %>% 
    group_by(id) %>% 
    ppv(volcano_type, .pred_class)
```

    ## # A tibble: 25 x 4
    ##    id          .metric .estimator .estimate
    ##    <chr>       <chr>   <chr>          <dbl>
    ##  1 Bootstrap01 ppv     macro          0.610
    ##  2 Bootstrap02 ppv     macro          0.622
    ##  3 Bootstrap03 ppv     macro          0.621
    ##  4 Bootstrap04 ppv     macro          0.638
    ##  5 Bootstrap05 ppv     macro          0.616
    ##  6 Bootstrap06 ppv     macro          0.641
    ##  7 Bootstrap07 ppv     macro          0.578
    ##  8 Bootstrap08 ppv     macro          0.671
    ##  9 Bootstrap09 ppv     macro          0.672
    ## 10 Bootstrap10 ppv     macro          0.616
    ## # ... with 15 more rows

It will also be interesting to see which variable is of more importance for the model to correctly classify volcano eruptions. Let's plot a variable importance plot.

``` r
library(vip)
rf_spec %>%
    set_engine("ranger", importance = "permutation") %>%
    fit(
        volcano_type ~ .,
        data = juice(volcano_prep) %>% select(-volcano_number) %>% janitor::clean_names()
    ) %>% 
    vip(geom = "point")
```

![VIP Plot](plots/Vip%20plot-1.png)

Let's combine the predictions into a dataframe

``` r
volcano_pred <- volcano_res %>% 
    collect_predictions() %>% 
    mutate(correct = .pred_class == volcano_type) %>% 
    left_join(volcano_df %>% 
                  mutate(.row = row_number()))
```

### Let's check on the world map, how many of the valcanos can our model classify correctly.
The map shows, how it fares in terms of predictions by plotting our predictions again on a world map.

``` r
ggplot() +
    geom_map(data = world, map = world,
             aes(x = long, y = lat, map_id = region), color = "white", fill = "gray50", alpha = 0.5) +
    stat_summary_hex(data = volcano_pred, aes(longitude, latitude, z = as.integer(correct)), 
                     fun = "mean", alpha = 0.8, bins = 60) +
    labs(x = NULL, y = NULL,
         title = "On Average how accurate were our predictions across the world",
         fill = "Percent correct") +
    scale_x_continuous(labels = NULL) +
    scale_y_continuous(labels = NULL) +
    scale_fill_gradient(high = "cyan3", labels = scales::percent)
```

![Volcano Pred Plot](plots/Volcano%20Pred%20Plot-1.png)

Okay. So in this post, we used volcanic data to build a multinomial classification model to see if on the basis of demographic data and other information about a volcano, how correctly can we predict the type of the eruption. We also built a use case around multiclass classification and used some performance metrics to evaluate the model effectiveness. 
I hope this helps. Thank you so much for reading. See you again in the next post..!!
