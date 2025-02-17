
<!-- README.md is generated from README.Rmd. Please edit that file -->

A cross estimation procedure is used to estimate average treatment
effects and confidence intervals for randomized experiments. The data is
split into $K$ folds, the estimates in each fold are averaged to give
$\hat{\tau}$, and an estimate of the variance of $\hat{\tau}$ is
computed. For each fold $k$ and treatments $w = 0, 1$, suppose that
$\bar{Y}_w^{(k)}, \bar{X}_w^{(k)}$ are the means in that fold. Also,
suppose that $\hat{\beta}_w^{(-k)}$ are regression adjustments
calculated on the other folds based on demeaned data. Then, the average
treatment effect estimate is computed with the formula

$$ \hat{\tau}^{(k)} = \bar{Y}_1^{(k)} - \bar{X}_1^{(k)}\hat{\beta}^{(1,-k)} + \bar{X}^{(k)}\hat{\beta}^{(1,-k)} - (\bar{Y}_0^{(k)} - \bar{X}_0^{(k)}\hat{\beta}^{(0,-k)} + \bar{X}^{(k)}\hat{\beta}^{(0,-k)}) $$

$$ \hat{\tau} = \sum_{k=1}^K\hat{\tau}^{(k)}n^{(k)}/n $$

The variance of this estimate is computed with the formula

$$ \text{Var}(\hat{\tau}^{(k)}) = \sum_{w = 0,1}\text{Var}(Y_w^{(k)}) / n_w^{(k)} - 2\text{Cov}(Y_w^{(k)}, X_w^{(k)}\bar{\hat{\beta}}^{(-k)}) / n_w^{(k)} + \text{Var}(X_w^{(k)}\bar{\hat{\beta}}^{(-k)}) / n_w^{(k)} $$

$$ \text{Var}(\hat{\tau})=\sum_{k=1}^K\text{Var}(\hat{\tau}^{(k)})(n^{(k)}/n)^2 $$

The covariances and variances above are substituted with estimated
values in the data to obtain an estimate of the variance of
$\hat{\tau}$. Confidence intervals are also provided under a Gaussian
assumption, which holds asymptotically under certain conditions.

The glmnet package is used for the estimation of regression adjustments.
The regularization parameter is chosen by cross validation, which can be
done on the same $k$ folds as used for cross estimation above. A
simulated example is below:

``` r
# simulation with Gaussian covariates based on Figure 1 in reference paper
library(crossEstimation)
set.seed(30)
n <- 200
p <- 500
xmean <- 1
xsigma <- 1
sigma <- .1
# set average treatment effect equal to one
ymean0 <- 4
ymean1 <- 3
# set no heterogeneous treatment effects
theta0 <- c(1, rep(0, p-1))
theta1 <- c(1, rep(0, p-1))
tau <- ymean1 - ymean0 + sum(xmean * theta1) - sum(xmean * theta0)
# run 100 times and calculate coverage
cover <- 0
for (i in 1:100) {
  x <- matrix(rnorm(n * p, xmean, xsigma), n, p)
  T <- (runif(n) < 0.2)
  mu <- (ymean1 + x %*% theta1) * T + (ymean0 + x %*% theta0) * (1 - T)
  epsC <- rnorm(n, 0, sigma)
  epsT <- rnorm(n, 0, sigma)
  eps <- epsT * T + epsC * (1 - T)
  yobs <- mu + eps
  res <- ate.glmnet(x, yobs, T, alpha = 1, nfolds = 10, method = "joint", lambda.choice = "lambda.min")
  cover <- cover + (res$conf.int[1] < tau & tau < res$conf.int[2])
}
cover
#> [1] 90
```
