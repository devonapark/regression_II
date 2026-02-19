HW_4
================
Devon Park
2026-02-19

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

# Question 1A

We continue to use the hwdata1.csv as in Homework 1 to study risk
factors for the breast cancer recurrence. Three Cox proportional hazards
regression models were fit

### Prep Data:

``` r
hwdata1 = read_csv("hwdata1-1.csv") |> 
  mutate(
    event = censrec, 
    #Event indicator: 1 = recurrence, 0 = censored (based on codebook)
    
    hormone_grp = factor(hormone, 
                         levels = c(1, 2), 
                         labels = c("Hormone: Yes", "Hormone: No")) |>
                          #Hormone therapy groups (codebook: 1=Yes, 2=No)
  
        relevel(ref = "Hormone: No"),
        #Making sure that "No Hormone" is the reference group
  
    meno_status = factor(menopause,
                      levels = c(1, 2),
                      labels = c("Menopausal: Yes", "Menopausal: No")) |> 
      relevel(ref = "Menopausal: No")
      #Women who are not yet experiencing menopause are the reference group
  )
```

    ## Rows: 686 Columns: 13
    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr  (2): diagdate, recdate
    ## dbl (11): id, age, menopause, hormone, size, grade, nodes, prog_recp, estrg_...
    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

### Model 1: Covariate: Hormone Therapy Group

General Model: $\lambda(t|hormone) = \lambda_0e^{\beta_1*hormone}$

The Cox proportional hazards with the covariate of horomone therapy
group is as follows (beta rounded to three decimal places):
$\lambda(t|hormone) = \lambda_0e^{0.364*hormone}$

``` r
model_1_results = hwdata1 |> coxph(Surv(rectime, event) ~ hormone_grp, 
                      data = _) |> 
          tidy(exponentiate = TRUE, conf.int = TRUE)

#Clean Variable Names in Output
model_1_results = model_1_results |>
  mutate(
    term = case_when(
      term == "hormone_grpHormone: Yes" ~ "Hormone: Yes",
      TRUE ~ term
    )
  )


#Cleaned Up Output
model_1_results |>
  knitr::kable(
    digits = 3,
    col.names = c("Variable", "Hazard Ratio", "Std Error", "Z statistic", "p-value", "Lower 95% CI", "Upper 95% CI"),
    caption = ""
  )
```

| Variable | Hazard Ratio | Std Error | Z statistic | p-value | Lower 95% CI | Upper 95% CI |
|:---|---:|---:|---:|---:|---:|---:|
| Hormone: Yes | 1.439 | 0.125 | 2.911 | 0.004 | 1.126 | 1.839 |

### Model 2: Covariates: Hormone therapy groups, Age, Menopause status, Tumor size, and Number of nodes

General Model:
$\lambda(t|X) = \lambda_0e^{\beta_1*hormone + \beta_2*age + \beta_3*menopausal + \beta_4*size + \beta_5*nodes}$

The Cox proportional hazards with the above covariates is as follows
(betas rounded to three decimal places):
$\lambda(t|X) = \lambda_0e^{0.381*hormone -0.014*age -0.353*menopausal + 0.008*size + 0.052*nodes}$

``` r
model_2_results = hwdata1 |> coxph(Surv(rectime, event) 
                           ~ hormone_grp + age + meno_status + size + nodes,
                           data = _) |> 
          tidy(exponentiate = TRUE, conf.int = TRUE)


#Clean Variable Names in Output
model_2_results = model_2_results |>
  mutate(
    term = case_when(
      term == "hormone_grpHormone: Yes" ~ "Hormone: Yes",
      term == "meno_statusMenopausal: Yes" ~ "Menopausal: Yes",
      term == "nodes" ~ "Nodes involved",
      term == "age" ~ "Age (years)",
      term == "size" ~ "Tumor size (mm)",
      TRUE ~ term
    )
  )


#Cleaned Up Output
model_2_results |>
  knitr::kable(
    digits = 3,
    col.names = c("Variable", "Hazard Ratio", "Std Error", "Z statistic", "p-value", "Lower 95% CI", "Upper 95% CI"),
    caption = ""
  )
```

| Variable | Hazard Ratio | Std Error | Z statistic | p-value | Lower 95% CI | Upper 95% CI |
|:---|---:|---:|---:|---:|---:|---:|
| Hormone: Yes | 1.464 | 0.128 | 2.970 | 0.003 | 1.138 | 1.882 |
| Age (years) | 0.987 | 0.009 | -1.506 | 0.132 | 0.969 | 1.004 |
| Menopausal: Yes | 0.702 | 0.179 | -1.977 | 0.048 | 0.495 | 0.997 |
| Tumor size (mm) | 1.008 | 0.004 | 2.013 | 0.044 | 1.000 | 1.016 |
| Nodes involved | 1.053 | 0.007 | 7.109 | 0.000 | 1.038 | 1.069 |

### Model 3: Covariates: Hormone therapy groups, Age, Menopause status, Tumor size, Number of nodes, and the Interaction between hormone therapy groups and number of nodes

General Model:
$\lambda(t|X) = \lambda_0e^{\beta_1*hormone + \beta_2*age + \beta_3*menopausal + \beta_4*size + \beta_5*nodes + \beta_6*nodes*hormone}$

The Cox proportional hazards model with interaction between hormone
therapy and nodes is as follows (betas rounded to three decimal places):
$\lambda(t|X) = \lambda_0e^{0.630*hormone  -0.015*age  -0.393*menopausal + 0.007*size + 0.083*nodes -0.038*nodes*hormone}$

``` r
model_3_results = hwdata1 |> coxph(Surv(rectime, event) 
                           ~ hormone_grp*nodes + age + meno_status + size,
                           data = _)|> 
          tidy(exponentiate = TRUE, conf.int = TRUE)

model_3_results = model_3_results |>
  mutate(
    term = case_when(
      term == "hormone_grpHormone: Yes" ~ "Hormone: Yes",
      term == "hormone_grpHormone: Yes:nodes" ~ "Hormone: Yes × nodes",
      term == "meno_statusMenopausal: Yes" ~ "Menopausal: Yes",
      term == "nodes" ~ "Nodes involved",
      term == "age" ~ "Age (years)",
      term == "size" ~ "Tumor size (mm)",
      TRUE ~ term
    )
  )

#Cleaned Up Output
model_3_results |>
  knitr::kable(
    digits = 3,
    col.names = c("Variable", "Hazard Ratio", "Std Error", "Z statistic", "p-value", "Lower 95% CI", "Upper 95% CI"),
    caption = ""
  )
```

| Variable | Hazard Ratio | Std Error | Z statistic | p-value | Lower 95% CI | Upper 95% CI |
|:---|---:|---:|---:|---:|---:|---:|
| Hormone: Yes | 1.878 | 0.167 | 3.766 | 0.000 | 1.353 | 2.607 |
| Nodes involved | 1.086 | 0.014 | 5.991 | 0.000 | 1.057 | 1.116 |
| Age (years) | 0.985 | 0.009 | -1.674 | 0.094 | 0.968 | 1.003 |
| Menopausal: Yes | 0.675 | 0.179 | -2.195 | 0.028 | 0.475 | 0.959 |
| Tumor size (mm) | 1.007 | 0.004 | 1.666 | 0.096 | 0.999 | 1.014 |
| Hormone: Yes × nodes | 0.963 | 0.015 | -2.480 | 0.013 | 0.934 | 0.992 |

``` r
#hormone_grp*nodes --> includes the individual terms as well as the interaction term
```

Betas of all the models

``` r
#Beta for Model 1
model_1_betas = hwdata1 |> coxph(Surv(rectime, event) ~ hormone_grp, 
                      data = _) 
model_1_betas
```

    ## Call:
    ## coxph(formula = Surv(rectime, event) ~ hormone_grp, data = hwdata1)
    ## 
    ##                          coef exp(coef) se(coef)     z      p
    ## hormone_grpHormone: Yes 0.364     1.439    0.125 2.911 0.0036
    ## 
    ## Likelihood ratio test=8.82  on 1 df, p=0.002977
    ## n= 686, number of events= 299

``` r
#Betas for Model 2
model_2_betas = hwdata1 |> coxph(Surv(rectime, event) 
                           ~ hormone_grp + age + meno_status + size + nodes,
                           data = _)
model_2_betas
```

    ## Call:
    ## coxph(formula = Surv(rectime, event) ~ hormone_grp + age + meno_status + 
    ##     size + nodes, data = hwdata1)
    ## 
    ##                                 coef exp(coef)  se(coef)      z        p
    ## hormone_grpHormone: Yes     0.380887  1.463582  0.128253  2.970  0.00298
    ## age                        -0.013537  0.986554  0.008990 -1.506  0.13213
    ## meno_statusMenopausal: Yes -0.353350  0.702332  0.178732 -1.977  0.04804
    ## size                        0.007841  1.007872  0.003895  2.013  0.04409
    ## nodes                       0.052096  1.053476  0.007328  7.109 1.17e-12
    ## 
    ## Likelihood ratio test=65.9  on 5 df, p=7.293e-13
    ## n= 686, number of events= 299

``` r
#Betas for Model 3

model_3_betas = hwdata1 |> coxph(Surv(rectime, event) 
                           ~ hormone_grp*nodes + age + meno_status + size,
                           data = _)|> 
  tidy(exponentiate = FALSE, conf.int = FALSE)


model_3_betas = model_3_betas |>
  mutate(
    term = case_when(
      term == "hormone_grpHormone: Yes" ~ "Hormone: Yes",
      term == "hormone_grpHormone: Yes:nodes" ~ "Hormone: Yes × nodes",
      term == "meno_statusMenopausal: Yes" ~ "Menopausal: Yes",
      term == "nodes" ~ "Nodes involved",
      term == "age" ~ "Age (years)",
      term == "size" ~ "Tumor size (mm)",
      TRUE ~ term
    )
  ) |> 
  knitr::kable(
    digits = 3,
    col.names = c("Variable", "Betas", "Std Error", "Z statistic", "p-value"),
    caption = ""
  )
model_3_betas
```

| Variable             |  Betas | Std Error | Z statistic | p-value |
|:---------------------|-------:|----------:|------------:|--------:|
| Hormone: Yes         |  0.630 |     0.167 |       3.766 |   0.000 |
| Nodes involved       |  0.083 |     0.014 |       5.991 |   0.000 |
| Age (years)          | -0.015 |     0.009 |      -1.674 |   0.094 |
| Menopausal: Yes      | -0.393 |     0.179 |      -2.195 |   0.028 |
| Tumor size (mm)      |  0.007 |     0.004 |       1.666 |   0.096 |
| Hormone: Yes × nodes | -0.038 |     0.015 |      -2.480 |   0.013 |
