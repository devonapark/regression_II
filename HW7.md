HW7
================
Devon Park
2026-04-06

#### Question 1:

##### Fit a Poisson regression model with hospital length of stay as the outcome and procedure, gender, age, and type as covariates (model 1). Write down the model. Is overdispersion a potential problem for this Poisson model?

    ## 
    ## Call:
    ## glm(formula = los ~ procedure_grp + gender_grp + age + type_grp, 
    ##     family = poisson, data = hwdata3)
    ## 
    ## Coefficients:
    ##                           Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)               0.554710   0.150124   3.695  0.00022 ***
    ## procedure_grpCABG         1.121862   0.018971  59.137  < 2e-16 ***
    ## gender_grpMale           -0.102885   0.018197  -5.654 1.57e-08 ***
    ## age                       0.010251   0.002086   4.913 8.96e-07 ***
    ## type_grpEmergency/Urgent  0.189919   0.016844  11.275  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for poisson family taken to be 1)
    ## 
    ##     Null deviance: 7541.9  on 1958  degrees of freedom
    ## Residual deviance: 3471.0  on 1954  degrees of freedom
    ## AIC: 10457
    ## 
    ## Number of Fisher Scoring iterations: 5

General Model Form:
$log{E(Y \mid procedure, gender, age, type)} = \beta_0 + \beta_1x_1 + \beta_2x_2 + \beta_3x_3 + beta_4x_4$

Y = length of hospital stay in days $x_1$ = procedure where 1 = CABG, 0
= PTCA $x_2$ = gender where 1 = male, 0 = female $x_3$ = age in years
$x_4$ = type where 1 = emergency/urgent, 0 = elective

Model with Coefficients: General Model Form:
$log{E(Y)} = 0.555 + 1.122x_1 - 0.103x_2 + 0.010x_3 + 0.190x_4$

To check overdispersion we can yse quasipoisson method:

    ## 
    ## Call:
    ## glm(formula = los ~ procedure_grp + gender_grp + age + type_grp, 
    ##     family = quasipoisson, data = hwdata3)
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error t value Pr(>|t|)    
    ## (Intercept)               0.55471    0.22379   2.479 0.013270 *  
    ## procedure_grpCABG         1.12186    0.02828  39.671  < 2e-16 ***
    ## gender_grpMale           -0.10289    0.02713  -3.793 0.000153 ***
    ## age                       0.01025    0.00311   3.296 0.000999 ***
    ## type_grpEmergency/Urgent  0.18992    0.02511   7.564 5.99e-14 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for quasipoisson family taken to be 2.222171)
    ## 
    ##     Null deviance: 7541.9  on 1958  degrees of freedom
    ## Residual deviance: 3471.0  on 1954  degrees of freedom
    ## AIC: NA
    ## 
    ## Number of Fisher Scoring iterations: 5

The dsipersion parameter is calculated as 2.22. As this value is greater
than 1, this indicates overdispersion where `Var(Y) ~ 2.22*E(Y)`As
poisson assumes that variance and mean are equal, this parameter
suggests that the actual variance is more than 2x larger what model 1
assumes (and therefore the standard errors of model 1 are too small). We
should address overdispersion.

#### Question 2:

##### Refit model 1 with the scale parameter being equal to Pearson chi-square divided by residual DF. Estimate the length of stay rate ratio between patients undergoing CABG and PTCA procedures. Provide the 95% confidence interval and interpret

When I ran `quasi_model` above, it already recalculated using the scale
parameter being equal to `Pearson chi-square/ DF`. (Estimates stay the
same and the standard errors are recalculated).

    ## Waiting for profiling to be done...

    ##                                 RR     2.5 %    97.5 %
    ## (Intercept)              1.7414359 1.1229778 2.6999862
    ## procedure_grpCABG        3.0705661 2.9056701 3.2463267
    ## gender_grpMale           0.9022309 0.8556642 0.9516665
    ## age                      1.0103034 1.0041606 1.0164780
    ## type_grpEmergency/Urgent 1.2091514 1.1510733 1.2701389

To estimate the los rate ratio between patients undergoing CABG and PTCA
procedures, we look at the procedures covariate. We get the following
values: $beta (procedure) = 1.122$ $Rate Ratio = 3.071$
$95% Confidence Interval = 2.906,3.246$

Interpretation: The estimated los for patients undergoing CABG is 3.071
times that of patients undergoing the PTCA procedure after adjusting for
age, gender, and admission type. We are 95% confident that the true risk
ratio exists between 2.906 and 3.246. At alpha = 0.05, these values are
statistically significant as the p-value of the coefficient is
\<\<0.001.

#### Question 3:

##### Use the fitted model in part (2), calculate the expected days of hospital stay for male patients aged 68 who underwent CABG procedure and stayed in an elective type

My attempt to do this via code:

    ##       1 
    ## 9.68651

The output is 9.69 days.

We can also do this manually by plugging into our model 1.

$log{E(Y)} = 0.555 + 1.122(1) − 0.103(1) + 0.010(68) + 0.190(0)$
$log{E(Y)} = 0.555 + 1.122 − 0.103 + 0.680 + 0 = 2.254$
$exp(2.254) = 9.687 days$

Woohoo I get the same results!

Interpretation: A 68-year-old male patient who has an elective CABG
procedure is expected to stay in the hospital for approximately 9.69
days.

#### Question 4:

##### Refit model 1 using negative binomial regression. Provide a formal test to decide whether a negative binomial model is needed for this data than a Poisson regression model.

    ## 
    ## Call:
    ## glm.nb(formula = los ~ procedure_grp + gender_grp + age + type_grp, 
    ##     data = hwdata3, init.theta = 9.292017288, link = log)
    ## 
    ## Coefficients:
    ##                          Estimate Std. Error z value Pr(>|z|)    
    ## (Intercept)               0.54270    0.20759   2.614 0.008942 ** 
    ## procedure_grpCABG         1.13067    0.02425  46.633  < 2e-16 ***
    ## gender_grpMale           -0.10723    0.02528  -4.242 2.22e-05 ***
    ## age                       0.01020    0.00289   3.529 0.000418 ***
    ## type_grpEmergency/Urgent  0.21810    0.02338   9.328  < 2e-16 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## (Dispersion parameter for Negative Binomial(9.292) family taken to be 1)
    ## 
    ##     Null deviance: 4231.5  on 1958  degrees of freedom
    ## Residual deviance: 1886.3  on 1954  degrees of freedom
    ## AIC: 9927.3
    ## 
    ## Number of Fisher Scoring iterations: 1
    ## 
    ## 
    ##               Theta:  9.292 
    ##           Std. Err.:  0.668 
    ## 
    ##  2 x log-likelihood:  -9915.253

From `nb_model`, theta (1/$\alpha$) is 9.292 with a standard error of
0.668.

H0: alpha equals zero and poisson model is good HA: alpha does not equal
zero, so negative binomial model is needed Test statistic:
z=$\theta/SE(\theta)=9.292/0.668=13.91$ When `z=13.91`, p-value\<0.001.

Interpretation: at $\alpha = 0.05$, we reject the null hypothesis. There
is statistically significant evidence that the dispersion parameter is
different from zero. THis means that the negative binomial model is more
appropriate than the Poisson model for this data.

#### Question 5:

##### Use the negative binomial model to estimate the length of stay rate ratio between patients undergoing CABG and PTCA procedures and provide 95% confidence interval. Is the conclusion different from the Poisson model in part (2)?

    ## Waiting for profiling to be done...

    ##                                 RR     2.5 %    97.5 %
    ## (Intercept)              1.7206492 1.1473241 2.5797906
    ## procedure_grpCABG        3.0977181 2.9538518 3.2490671
    ## gender_grpMale           0.8983166 0.8548482 0.9439975
    ## age                      1.0102517 1.0045649 1.0159711
    ## type_grpEmergency/Urgent 1.2437176 1.1878470 1.3022563

From the earlier code for `nb_model`, the estimate for procedure
($\beta(procedure)$) is 1.131. The rate ratio between patients
undergoing CABG and PTCA procedures is 3.098. The 95% confidence
interval is (2.95, 3.249).

Interpretation: Based on the negative binomial model, the estimated
length of hospital stay for patients undergoing CABG is 3.098 times that
of patients undergoing the PTCA procedure, after adjusting for gender,
age, and admission type. We are 95% confident that the true rate ratio
lies between 2.954 and 3.249. This result is statistically significant
at the pre-specified alpha of 0.05 as the p-value of the estimator is \<
0.001.

After comparing both the poisson model and the negative binomial model
we see that the estimated rate ratios and their confidence intervals are
extremely similar. So, while our conclusion doesnt change significantly
after using negative binomial it is still the more appropriate model
because we found overdispersion earlier.
