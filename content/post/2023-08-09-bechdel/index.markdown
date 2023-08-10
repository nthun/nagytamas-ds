---
title: "Passing the Bechdel test in movies"
author: "Tamas Nagy"
date: "2021-05-26"
description: "I used the Bechdel test dataset from the TidyTuesday project to predict if a movie passed the test or not, based on the predicted gender of the writers."
output:
  html_document:
    code_download: true
    df_print: paged
    toc: yes
    toc_depth: 3
    toc_float:
      collapsed: yes
      smooth_scroll: yes
  html_notebook:
    theme: spacelab
editor_options:
  chunk_output_type: console
---




# What is this

This is a tutorial document to use R for creating an analysis report. It includes the following elements:

- Data transformation.
- Data visualization.
- Using the gender package to infer the gender based on first name.
- Using, interpreting, and evaluating binomial logistic regression results.

# Aim

We will use the Bechdel test dataset from the TidyTuesday project to predict if a movie passed the test or not, based on the predicted gender of the writers. Dataset and info available  from here: https://github.com/rfordatascience/tidytuesday/blob/master/data/2021/2021-03-09/readme.md
This dataset is collected by https://fivethirtyeight.com/ 

## What is the Bechdel test?
According to Wikipedia:

> The Bechdel test (/ˈbɛkdəl/ BEK-dəl), also known as the Bechdel–Wallace test, is a measure of the representation of women in fiction. It asks whether a work features at least two women who talk to each other about something other than a man. 




There are two datasets in the project. The first one contains 8839 Bechdel ratings, movie title, and release year. 



The second dataset contains more information about the movies, but contains movies only from 1970 to 2013. Even though we could harvest the movie data from imdb for the first dataset, we will settle with using the second dataset, containing 1794 movies. Interested readers can improve this analysis by extending the analysis to more movies using the imdb API, or a related R package.

Our aim is to predict if a movie is passing the Bechdel test. Our predictor is the proportion of women in the writer team, the year of release, and the runtime. The latter will be used as a control variable, I assume that longer movies might have more chance to introduce more complex female characters.

## Preprocessing the data

### Identifying the gender of writers

The dataset contains the names of the writers for each movie. Using this, we first get a list of first names for each movie. Then we will use the `{gender}` package to assume the gender associated with each unique name.

The gender package assumes the gender, based on the first name. As different names are used for both males and females in different times, it also requires a year as an input. Then it checks the proportion of females vs. males that had that name in the specific year, and assumes a gender.

In my analysis, I will make an arbitrary decision to use 1970 as the year for the classification (a sensitivity analysis could be added to see if this introduces much error, I guess not).



```r
# First get all unique first names from writer teams
writer_names <- 
    raw_movies %>% 
    separate_rows(writer, sep = ", ") %>% 
    transmute(writer_name = str_match(writer, pattern = "(^\\w+) .*")[,2])

# Use the gender package to assume a gender in 1970
name_gender <- 
    gender(unique(writer_names$writer_name), year = 1970) %>% 
    select(writer_name = name, 
           gender)
```

Then I calculate the proportion of females in each writer team for each movie. 


```r
# Calculate the proportion of female writers for each movie
female_writers <- 
    raw_movies %>% 
    separate_rows(writer, sep = ", ") %>%
    mutate(writer_name = str_match(writer, pattern = "(^\\w+) .*")[,2]) %>% 
    left_join(name_gender, by = "writer_name") %>% 
    group_by(imdb_id) %>% 
    summarise(female_writer = mean(gender == "female", na.rm = TRUE)) %>% 
    drop_na()
```

Finally, I create the final analysis dataset, where I only keep relevant variables, recode the outcome to numeric (0/1), create a decade variable instead of year, and convert the runtime in hours.


```r
movies <-
    raw_movies %>% 
    transmute(imdb_id,
              title,
              bechdel_pass = recode(binary, "FAIL" = 0L, "PASS" = 1L),
              decade = (year %/% 10)*10,
              runtime = parse_number(runtime)/60,
              writer) %>% 
    left_join(female_writers, by = "imdb_id") %>% 
    drop_na(female_writer)
```

## Creating a model

I will use the the proportion of female writers as a main predictor, but as control variables, I will also add the decade of release and the runtime in minutes. A binomial logistic model will be fit to predict the passing on the Bechdel test. 


```r
bechdel_model <-
    glm(bechdel_pass ~ female_writer + decade + runtime, 
        data = movies, 
        family = "binomial")

# This is the standard regression output
summary(bechdel_model)
```

```
## 
## Call:
## glm(formula = bechdel_pass ~ female_writer + decade + runtime, 
##     family = "binomial", data = movies)
## 
## Coefficients:
##                 Estimate Std. Error z value Pr(>|z|)    
## (Intercept)   -28.091190  11.059813  -2.540  0.01109 *  
## female_writer   2.107092   0.215895   9.760  < 2e-16 ***
## decade          0.014191   0.005525   2.568  0.01022 *  
## runtime        -0.433736   0.161521  -2.685  0.00725 ** 
## ---
## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
## 
## (Dispersion parameter for binomial family taken to be 1)
## 
##     Null deviance: 2118.3  on 1545  degrees of freedom
## Residual deviance: 1980.2  on 1542  degrees of freedom
##   (1 observation deleted due to missingness)
## AIC: 1988.2
## 
## Number of Fisher Scoring iterations: 4
```

```r
# This is the tidy approach, that converts log likelihoods to odds ratios, and adds confidence intervals.
tidy(bechdel_model, conf.int = TRUE, exponentiate = TRUE)
```

```
## # A tibble: 4 × 7
##   term          estimate std.error statistic  p.value conf.low conf.high
##   <chr>            <dbl>     <dbl>     <dbl>    <dbl>    <dbl>     <dbl>
## 1 (Intercept)   6.31e-13  11.1         -2.54 1.11e- 2 2.08e-22   0.00145
## 2 female_writer 8.22e+ 0   0.216        9.76 1.68e-22 5.44e+ 0  12.7    
## 3 decade        1.01e+ 0   0.00553      2.57 1.02e- 2 1.00e+ 0   1.03   
## 4 runtime       6.48e- 1   0.162       -2.69 7.25e- 3 4.71e- 1   0.887
```

```r
# This is for conveniently making an APA compatible summary table. We also use robust statndard errors to account for the heteroscedasticity.

tab_model(bechdel_model, 
          show.aic = TRUE,
          show.loglik = TRUE, 
          string.ci = "95% CI", 
          show.stat = TRUE,
          robust = TRUE,
          dv.labels = "Passing the Bechdel test",
          pred.labels = c("(Intercept)",
                          "% female in writer team", 
                          "Decade of release", 
                          "Runtime (h)"))
```

<table style="border-collapse:collapse; border:none;">
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="4" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">Passing the Bechdel test</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Odds Ratios</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">95% CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Statistic</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">(Intercept)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00&nbsp;&ndash;&nbsp;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;2.58</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.010</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">% female in writer team</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">8.22</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">5.44&nbsp;&ndash;&nbsp;12.70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">9.94</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Decade of release</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">1.00&nbsp;&ndash;&nbsp;1.03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">2.60</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.009</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Runtime (h)</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.47&nbsp;&ndash;&nbsp;0.89</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;2.71</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.007</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="4">1546</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> Tjur</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="4">0.086</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">AIC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="4">1988.156</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">log-Likelihood</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="4">-990.078</td>
</tr>

</table>

The results reveal that all predictors are significant. This means that the chance of passing the Bechdel test is 8.22-fold if the entire writer team is female, compared to an all male writer team. This holds even if we account for the effects of the release decade (newer movies pass the test more often), and runtime (longer movies are more likely to fail the test, wtf?).

Assumption checks are available from the `{performance}` package, that is part of the `{easystats}` ecosystem. I won't really do much about the assumption checks now, other than showing the output.


```r
check_model(bechdel_model)
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-7-1.png" width="672" />

## Evaluate model performance

In general, 44% of the investigated movies passed the Bechdel test. Full male teams are twice as likely to fail the test, while full female writer teams are approximately 4 times more likely to pass the test. Thus there is an 8-fold difference between all male and all female writer teams.


```r
# Create a prediction dataset with all possible predictor values
newdata <-
    crossing(female_writer = seq(0,1,.025),
             decade = seq(1970, 2010, 10),
             runtime = unique(movies$runtime))

# Interpolate the predicted values, and create a
augment(bechdel_model, newdata = newdata, type.predict = "link") %>%
    group_by(female_writer) %>% 
    mutate( .fitted = exp(.fitted),
            avg_fitted = mean(.fitted, na.rm = TRUE)) %>% 
    ungroup() %>% 
    ggplot() +
    aes(x = female_writer, y = .fitted) +
    geom_hline(yintercept = 1, color = "red", lty = "dashed") +
    geom_point(alpha = .01) +
    geom_line(aes(y = avg_fitted), color = "blue", size = 2, alpha = .5) +
    scale_x_continuous(labels = scales::percent_format()) +
    labs(x = "% of female writers for the movie",
         y = "Odds ratio of passing the Bechdel test",
         title = "Proportion of female writers vs. passing the Bechdel test in movies",
         subtitle = "When all writers are female, passing the Bechdel test is 8 times more likely compared to an all male writer team.")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-8-1.png" width="672" />

### ROC curve

Finally, we will create a ROC curve and calculate its AUC to see how efficiently our model can predict the outcome, based on the predictors.


```r
mod_pred <-
    augment(bechdel_model, type.predict = "response") %>% 
    transmute(truth = as.factor(bechdel_pass), 
              estimate = .fitted)
```


The AUC for the model is ok, but not great: 64%.


```r
roc_bechdel <- roc_curve(mod_pred, truth, estimate, event_level = "second")
    
autoplot(roc_bechdel) +
    scale_x_continuous(labels = percent_format()) +
    scale_y_continuous(labels = percent_format()) +
    labs(title = "ROC curve for the model")
```

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-10-1.png" width="672" />

# Improvement ideas

- It would be possible to create a more comprehensive proportion of females variable, taking into account the director(s) along with the writers. 
- Gender classification could be improved by tinkering with the parameters of the gender package.
- Other types of models could be more useful, e.g. random forest.
- The words from the description of the movie could be added as predictors.
