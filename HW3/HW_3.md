HW_3
================
Devon Park
2026-02-15

``` r
library(survival)
library(survminer)
```

    ## Loading required package: ggpubr

    ## 
    ## Attaching package: 'survminer'

    ## The following object is masked from 'package:survival':
    ## 
    ##     myeloma

``` r
library(broom)
library(knitr)
library(tidyverse)
```

## Question 1

The Framingham Heart Study has collected data on cardiovascular risk
factors and long-term follow-up for nearly 5,000 residents of
Framingham, Massachusetts. These data were used as an example in Section
7.5 of Dupont (2008).

An investigator wishes to study the association between baseline age and
the risk of coronary heart disease (CHD) among 4,699 participants using
a Cox proportional hazards regression model.

#### Question 1A

1)  Specify the Cox proportional hazards model with baseline age
    included as a continuous covariate. Using SAS/R, estimate the
    hazards ratio and its 95% confidence interval.

``` r
framingham = read.csv("/Users/devonpark/Desktop/MPH - R Homebase/Applied Regression II/regression_II/HW3/framingham.csv")

view(framingham)
head(framingham)
```

    ##   sex sbp dbp scl chdfate followup age  bmi month   id
    ## 1   1 120  80 267       1       18  55 25.0     8 2642
    ## 2   1 130  78 192       1       35  53 28.4    12 4627
    ## 3   1 144  90 207       1      109  61 25.1     8 2568
    ## 4   1  92  66 231       1      147  48 26.2    11 4192
    ## 5   1 162  98 271       1      169  39 28.4    11 3977
    ## 6   2 212 118 182       1      199  61 33.3     2  659

Fit Cox proportional Hazard Model

``` r
#CHUNK 1
cox_age = coxph(
  Surv(followup, chdfate) ~ age,
  data = framingham
  #,ties = "efron"
  )

# coxph --> fit a cox proportional hazard model

# Surv --> create a special object of class "Surv" that behaves somewhat like a matrix but is specifically designed for survival analysis.
    #where:
    #followup = the survival time variable (The amount of time each individual was followed until event occurred or indv. was censored)
    #chdfate = the event indicator variable (tells us if the event occurred)

# ~ --> before ~ is the survival outcome and after ~ are the predictors
# so the model tells us "hazard depends on baseline age (continuous)"

# data = .... --> tells R where to find the data (aka the variables followup, chdfate, age)

# ties = "efron" --> is the DEFAULT in coxph (so if I had not included it, I would have gotten the same answer)
    #what ties means: a tie occurrs when multiple individuals have the same event time
    #Because Partial likelihood forumla assumes that events occur one at a time, when events occur at the same time, we need an approximation. 
    #"efron" is the recommend method of approximation (could also use "breslow" or "exact")



summary(cox_age)
```

    ## Call:
    ## coxph(formula = Surv(followup, chdfate) ~ age, data = framingham)
    ## 
    ##   n= 4699, number of events= 1473 
    ## 
    ##         coef exp(coef) se(coef)     z Pr(>|z|)    
    ## age 0.057004  1.058660 0.003166 18.01   <2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##     exp(coef) exp(-coef) lower .95 upper .95
    ## age     1.059     0.9446     1.052     1.065
    ## 
    ## Concordance= 0.644  (se = 0.007 )
    ## Likelihood ratio test= 321.2  on 1 df,   p=<2e-16
    ## Wald test            = 324.3  on 1 df,   p=<2e-16
    ## Score (logrank) test = 336.9  on 1 df,   p=<2e-16

. exp(coef) exp(-coef) lower .95 upper .95 age 1.059 0.9446 1.052 1.065

``` r
#CHUNK 2: Tidy output of cox model 

cox_age_results = 
  coxph(
    Surv(followup, chdfate) ~ age, 
    data = framingham) |>
  tidy(exponentiate = TRUE, conf.int = TRUE)

#

cox_age_results |>
  knitr::kable(
    digits = 3,
    col.names = c("Variable", "Hazard Ratio", "Std Error", "Z statistic", "p-value", "Lower 95% CI", "Upper 95% CI"),
    caption = "Hazard ratio for baseline age from Cox proportional hazards model"
  )
```

| Variable | Hazard Ratio | Std Error | Z statistic | p-value | Lower 95% CI | Upper 95% CI |
|:---------|-------------:|----------:|------------:|--------:|-------------:|-------------:|
| age      |        1.059 |     0.003 |      18.007 |       0 |        1.052 |        1.065 |

Hazard ratio for baseline age from Cox proportional hazards model

INTERPRETATION:

Every one-year increase in baseline age is associated with approximately
a 5.9% increase in the hazard of coronary heart disease (HR = 1.059, 95%
CI: 1.052–1.065).We are 95% confident that each additional year of
baseline age increases the hazard of CHD by between 5.2% and 6.5%. Since
the 95% confidence interval does not include 1 and the p-value is is
\<0.05, baseline age is a statistically significant predictor of CHD.

#### Question 1B

Specify the Cox proportional hazards model with age categorized into
quartiles. Using SAS/R, obtain the hazard ratios for comparisons among
the age quartiles. In addition, show how these pairwise hazard ratios
are calculated manually from the fitted model.

USED VERSION 2! – SCROLL DOWN !!!

``` r
# Version 1
# Create quartiles within my original dataset

framingham = framingham |>
  mutate(age_q = ntile(age, 4) |>                    
  factor(labels = c("Q1","Q2","Q3","Q4"))
  ) |>
  mutate(age_q = relevel(age_q, ref = "Q1"))

# BIG PICTURE: Creating a new variable called age_q that groups age into quartiles (Q1–Q4), and sets Q1 as the reference group.
# mutate --> create a new variable called age_q that will take the values 1, 2, 3, or 4.
# ntile --> splits up age into 4 equally sized groups. (divides based on rank order, not explicitly calculated on cutpoints).
    # sorts ages from smallest to largest and then assignes group numbers in increasing order
# factor(labels...) --> converts numerical quartiles into labeled categories. (1 becomes Q1, 2 becomes 2, and so on (because of factor))
    #maps labels based on numeric values. Factor does NOT use dataset order
# mutate...relevel... --> sets the reference group (needed as the baseline group against which Cox regression can comparein )
```

``` r
#Using quartiles from version 1 (method that utilises ntile)

cox_q = framingham |>
  coxph(Surv(followup, chdfate) ~ age_q, data = _)
```

Output:

``` r
summary(cox_q)
```

    ## Call:
    ## coxph(formula = Surv(followup, chdfate) ~ age_q, data = framingham)
    ## 
    ##   n= 4699, number of events= 1473 
    ## 
    ##            coef exp(coef) se(coef)      z Pr(>|z|)    
    ## age_qQ2 0.28922   1.33539  0.08210  3.523 0.000427 ***
    ## age_qQ3 0.77810   2.17733  0.07816  9.955  < 2e-16 ***
    ## age_qQ4 1.08250   2.95204  0.07775 13.922  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##         exp(coef) exp(-coef) lower .95 upper .95
    ## age_qQ2     1.335     0.7488     1.137     1.569
    ## age_qQ3     2.177     0.4593     1.868     2.538
    ## age_qQ4     2.952     0.3387     2.535     3.438
    ## 
    ## Concordance= 0.624  (se = 0.007 )
    ## Likelihood ratio test= 245.6  on 3 df,   p=<2e-16
    ## Wald test            = 240.9  on 3 df,   p=<2e-16
    ## Score (logrank) test = 255.7  on 3 df,   p=<2e-16

Compared to individuals in the lowest age quartile (Q1): Individuals in
Q2 have 1.335 times the hazard of CHD (1.137, 1.569) Individuals in Q3
have 2.177 times the hazard of CHD (1.868, 2.538) Individuals in Q4 have
2.952 times the hazard of CHD (2.535, 3.438)

This shows a strong increasing hazard across age quartiles.

``` r
# VERSION 2: 
# If i wanted to actually calculate the quartiles

q = framingham |>
  pull(age) |>
  quantile(probs = c(0.25, 0.50, 0.75), na.rm = TRUE)

# output will be 25% X_1 50% X_2 75% X_3 (aka the age values that define the quartile boundaries)

framingham = framingham |>
  mutate(
    age_q_v2 = cut(
      age,
      breaks = c(-Inf, q[1], q[2], q[3], Inf),
      labels = c("Q1","Q2","Q3","Q4"),
      include.lowest = TRUE
    ) |> 
      relevel(age_q_v2, ref = "Q1")
    )
  
# cut --> assign quartile groups for variable age
    #creating intervals such as:Q1: (-Inf, q1], Q2: (q1, q2], Q3: (q2, q3], Q4: (q3, Inf]
# include.lowest = TRUE --> ensures the smallest observed value is included in the first interval (aka make sure that Q1:[lowest age, q1] and Q4: (q3, highest age])
```

``` r
#Q1B VERSION 2: OUTPUT CHUNK 1
#Using quartiles from version 2 (method that utilises cut and quartiles)

cox_q_v2 = framingham |>
  coxph(Surv(followup, chdfate) ~ age_q_v2, data = _)

summary(cox_q_v2)
```

    ## Call:
    ## coxph(formula = Surv(followup, chdfate) ~ age_q_v2, data = framingham)
    ## 
    ##   n= 4699, number of events= 1473 
    ## 
    ##               coef exp(coef) se(coef)      z Pr(>|z|)    
    ## age_q_v2Q2 0.33443   1.39715  0.08169  4.094 4.24e-05 ***
    ## age_q_v2Q3 0.78605   2.19471  0.07518 10.455  < 2e-16 ***
    ## age_q_v2Q4 1.20430   3.33441  0.07580 15.888  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ##            exp(coef) exp(-coef) lower .95 upper .95
    ## age_q_v2Q2     1.397     0.7157     1.190     1.640
    ## age_q_v2Q3     2.195     0.4556     1.894     2.543
    ## age_q_v2Q4     3.334     0.2999     2.874     3.868
    ## 
    ## Concordance= 0.634  (se = 0.007 )
    ## Likelihood ratio test= 292.4  on 3 df,   p=<2e-16
    ## Wald test            = 290.8  on 3 df,   p=<2e-16
    ## Score (logrank) test = 312.6  on 3 df,   p=<2e-16

``` r
cox_q_v2 |>
  tidy(exponentiate = TRUE, conf.int = TRUE) |>
  kable(
    digits = 3,
    col.names = c("Comparison", "Hazard Ratio", "Std Error", "Z", "p-value", "Lower 95% CI", "Upper 95% CI"),
    caption = "Hazard ratios for CHD by age quartile (Q1 reference)"
  )
```

| Comparison | Hazard Ratio | Std Error |      Z | p-value | Lower 95% CI | Upper 95% CI |
|:-----------|-------------:|----------:|-------:|--------:|-------------:|-------------:|
| age_q_v2Q2 |        1.397 |     0.082 |  4.094 |       0 |        1.190 |        1.640 |
| age_q_v2Q3 |        2.195 |     0.075 | 10.455 |       0 |        1.894 |        2.543 |
| age_q_v2Q4 |        3.334 |     0.076 | 15.888 |       0 |        2.874 |        3.868 |

Hazard ratios for CHD by age quartile (Q1 reference)
