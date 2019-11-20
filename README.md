<!-- README.md is generated from README.Rmd. Please edit that file -->
evabic <a href='https://abichat.github.io/evabic'><img src='man/figures/logo.png' align="right" height="139" /></a>
===================================================================================================================

<!-- badges: start -->
[![license](https://img.shields.io/badge/license-GPL--3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0.en.html)
[![lifecycle](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
[![packageversion](https://img.shields.io/badge/package%20version-0.0.0.9004-orange.svg)](commits/master)
[![Travis build
status](https://travis-ci.org/abichat/evabic.svg?branch=master)](https://travis-ci.org/abichat/evabic)
[![AppVeyor build
status](https://ci.appveyor.com/api/projects/status/github/abichat/evabic?branch=master&svg=true)](https://ci.appveyor.com/project/abichat/evabic)
[![Codecov test
coverage](https://codecov.io/gh/abichat/evabic/branch/master/graph/badge.svg)](https://codecov.io/gh/abichat/evabic?branch=master)
[![Codacy
Badge](https://api.codacy.com/project/badge/Grade/c32dcc4c1c3f40a5950e1c10ea6dfb18)](https://www.codacy.com/app/abichat/evabic?utm_source=github.com&utm_medium=referral&utm_content=abichat/evabic&utm_campaign=Badge_Grade)
[![Last-changedate](https://img.shields.io/badge/last%20change-2019--11--20-yellowgreen.svg)](/commits/master)
[![Documentation](https://img.shields.io/badge/documentation-pkgdown-E91E63.svg)](https://abichat.github.io/evabic/)
<!-- badges: end -->

**evabic** aims to **eva**luate **bi**nary **c**lassifiers by specifying
what is detected as true and what is actually true. It has no
dependancies.

Installation
------------

You can install the development version from
[GitHub](https://github.com/) with:

    # install.packages("remotes")
    remotes::install_github("abichat/evabic")

Measures
--------

**evabic** provides handy functions to computes 15 different measures.
Each function begins with `ebc_*`.

Available measures includes True Positive Rate (Sensitivity or Recall),
True Negative Rate (Specificity), Positive Predictive Value (Precision),
False Discovery Rate, Accuracy, F1…

    evabic::ebc_allmeasures
    #>  [1] "TP"   "FP"   "FN"   "TN"   "TPR"  "TNR"  "PPV"  "NPV"  "FNR"  "FPR"  "FDR"  "FOR"  "ACC"  "BACC"
    #> [15] "F1"

All measures are computed from confusion matrix:

<center>
<img src="man/figures/confusionmatrix.png" width="50%"/>
</center>
Example
-------

Let’s use **evabic** on a toy example.

    library(evabic)

Consider three variables `X1`, `X2` and `X3`, `Y` a variable predicted
by this three variables, and 4 more conditionally independent variables
`X4` to `X7`.

    set.seed(42)
    X1 <- rnorm(50)
    X2 <- rnorm(50)
    X3 <- rnorm(50)
    predictors <- paste0("X", 1:3)

    df_lm <- data.frame(X1 = X1, X2 = X2, X3 = X3, 
                        X4 = X1 + X2 + X3 + rnorm(50, sd = 0.5),
                        X5 = X1 + 3 * X3 + rnorm(50, sd = 0.5),
                        X6 = X2 - 2 * X3 + rnorm(50, sd = 0.5),
                        X7 = X1 - X2 + rnorm(50, sd = 2),
                        Y  = X1 - X2 + 3 * X3 + rnorm(50))

We use a linear regression to detect the actual predictors (do not
select significant variables like this at home, it’s a bad way to do
so).

    model <- lm(Y ~ ., data = df_lm)
    summary(model)
    #> 
    #> Call:
    #> lm(formula = Y ~ ., data = df_lm)
    #> 
    #> Residuals:
    #>      Min       1Q   Median       3Q      Max 
    #> -1.66504 -0.65784 -0.05977  0.51720  2.14833 
    #> 
    #> Coefficients:
    #>             Estimate Std. Error t value Pr(>|t|)   
    #> (Intercept)  0.13537    0.14528   0.932  0.35678   
    #> X1           1.35385    0.44929   3.013  0.00437 **
    #> X2          -0.64382    0.46520  -1.384  0.17368   
    #> X3           3.67893    1.18759   3.098  0.00347 **
    #> X4          -0.22998    0.33164  -0.693  0.49183   
    #> X5          -0.17073    0.30744  -0.555  0.58161   
    #> X6          -0.04023    0.28381  -0.142  0.88795   
    #> X7           0.07055    0.09245   0.763  0.44966   
    #> ---
    #> Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    #> 
    #> Residual standard error: 1.005 on 42 degrees of freedom
    #> Multiple R-squared:  0.929,  Adjusted R-squared:  0.9171 
    #> F-statistic: 78.48 on 7 and 42 DF,  p-value: < 2.2e-16
    pvalues <- summary(model)$coefficients[-1, 4]
    pvalues
    #>          X1          X2          X3          X4          X5          X6          X7 
    #> 0.004366456 0.173677616 0.003469737 0.491828466 0.581608670 0.887948400 0.449664443
    detected_var <- names(pvalues[pvalues < 0.05])
    detected_var
    #> [1] "X1" "X3"

Singles measures are available with `ebc_*()` functions.

    ebc_TPR(detected = detected_var, true = predictors)
    #> [1] 0.6666667
    ebc_ACC(detected = detected_var, true = predictors, m = 7) # the total size of the set is 7
    #> [1] 0.8571429

You can also asks for several measures in a single row summary format
with `ebc_tidy()`.

    ebc_tidy(detected = detected_var, true = predictors, m = 7, 
             measures = c("TPR", "TNR", "FDR", "ACC", "BACC", "F1"))
    #>         TPR TNR FDR       ACC      BACC  F1
    #> 1 0.6666667   1   0 0.8571429 0.8333333 0.8

Note that **evabic** also supports named logicals for `detected` and
`true` arguments.

    pvalues < 0.05
    #>    X1    X2    X3    X4    X5    X6    X7 
    #>  TRUE FALSE  TRUE FALSE FALSE FALSE FALSE
    ebc_tidy(detected = pvalues < 0.05, true = predictors, m = 7, 
             measures = c("TPR", "TNR", "FDR", "ACC", "BACC", "F1"))
    #>         TPR TNR FDR       ACC      BACC  F1
    #> 1 0.6666667   1   0 0.8571429 0.8333333 0.8

Finally, you can ask one row of measures per threshold if you provide
the vector of p-values (by default, detected elements are on the left of
the threshold).

    ebc_tidy_by_threshold(detection_values = pvalues, true = predictors, m = 7, 
                          measures = c("TPR", "TNR", "FDR", "ACC", "BACC", "F1"), 
                          direction = "leq", sup_threshold = 0)
    #>     threshold       TPR  TNR       FDR       ACC      BACC        F1
    #> 1 0.000000000 0.0000000 1.00 0.0000000 0.5714286 0.5000000 0.0000000
    #> 2 0.003469737 0.3333333 1.00 0.0000000 0.7142857 0.6666667 0.5000000
    #> 3 0.004366456 0.6666667 1.00 0.0000000 0.8571429 0.8333333 0.8000000
    #> 4 0.173677616 1.0000000 1.00 0.0000000 1.0000000 1.0000000 1.0000000
    #> 5 0.449664443 1.0000000 0.75 0.2500000 0.8571429 0.8750000 0.8571429
    #> 6 0.491828466 1.0000000 0.50 0.4000000 0.7142857 0.7500000 0.7500000
    #> 7 0.581608670 1.0000000 0.25 0.5000000 0.5714286 0.6250000 0.6666667
    #> 8 0.887948400 1.0000000 0.00 0.5714286 0.4285714 0.5000000 0.6000000
