# Adjustment

Adjustment,
is the idea of putting regressors into a linear model
to investigate the role of a third variable on the relationship
between another two. Since it is often the case that
a third variable can distort, or confound if you will,
the relationship between two others.

<!--

As an example, consider looking at lung cancer rates and breath mint
usage. For the sake of completeness, imagine if you were looking
at forced expiratory volume (a measure of lung function) and breath
mint usage.  If you found a statistically significant regression relationship, it
wouldn't be wise to rush off to the newspapers with the headline
"Breath mint usage causes shortness of breath!", for a variety of reasons.
First off, even if the association is sound, you don't know that it's
causal. But, more importantly in this case, the likely culprit
is smoking habits. Smoking rates are likely related to both breath mint
usage rates and lung function. How would you defend your finding
against the accusation that it's just variability in smoking habits?

If your finding held up among non-smokers and smokers analyzed
separately, then you might have something. In other words, people
wouldn't even begin to believe this finding unless it held up
while holding smoking status constant. That is the idea of
adding a regression variable into a model as adjustment. The
coefficient of interest is interpreted as the effect of the
predictor on the response, holding the adjustment variable
constant.

In this chapter, we'll use simulation to
investigate how adding a regressor into a model addresses
the idea of adjustment.


## Experiment 1

Let's first generate some data. Consider the model

{$$}Y_i = \beta_0 + \beta_1 X + \tau T  + \epsilon_i{/$$}

We're interested in the relationship between our binary treatment,
{$$}T{/$$}, and
{$$}Y{/$$}. However, we're concerned that the relationship
may depend on the continuous variable, {$$}X{/$$}.

Let's simulate some data.

{lang=r,line-numbers=off}
~~~
n <- 100; t <- rep(c(0, 1), c(n/2, n/2)); x <- c(runif(n/2), runif(n/2));
beta0 <- 0; beta1 <- 2; tau <- 1; sigma <- .2
y <- beta0 + x * beta1 + t * tau + rnorm(n, sd = sigma)
~~~

Let's plot the data. Below I give the code for the first plot, the rest omitted,
though you can see the course git repository for the rest of the code.

{lang=r,line-numbers=off,title="Simulation 1"}
~~~
plot(x, y, type = "n", frame = FALSE)
abline(lm(y ~ x), lwd = 2)
abline(h = mean(y[1 : (n/2)]), lwd = 3)
abline(h = mean(y[(n/2 + 1) : n]), lwd = 3)
fit <- lm(y ~ x + t)
abline(coef(fit)[1], coef(fit)[2], lwd = 3)
abline(coef(fit)[1] + coef(fit)[3], coef(fit)[2], lwd = 3)
points(x[1 : (n/2)], y[1 : (n/2)], pch = 21, col = "black", bg = "lightblue", cex = 2)
points(x[(n/2 + 1) : n], y[(n/2 + 1) : n], pch = 21, col = "black", bg = "salmon", cex = 2)
~~~

![Simulation 1.](images/adjustment1.png)

Looking at this plot notices that the X variable is
unrelated to treatment/group status (color). In addition, the X variable
is clearly linearly related to Y, but the intercept
of this relationship depends on group status. The treatment variable is also
related to Y; especially look at the horizontal lines
which connect the group means onto the Y axis.

* The X variable is unrelated to group status
* The X variable is related to Y, but the intercept depends
  on group status.
* The group variable is related to Y.
  * The relationship between group status and Y is constant depending on X.
  * The relationship between group and Y disregarding X is about the same as holding X constant

<!--
---
## Simulation 2
```{r, fig.height=5, fig.width=5, echo = FALSE, results='hide'}
n <- 100; t <- rep(c(0, 1), c(n/2, n/2)); x <- c(runif(n/2), 1.5 + runif(n/2));
beta0 <- 0; beta1 <- 2; tau <- 0; sigma <- .2
y <- beta0 + x * beta1 + t * tau + rnorm(n, sd = sigma)
plot(x, y, type = "n", frame = FALSE)
abline(lm(y ~ x), lwd = 2)
abline(h = mean(y[1 : (n/2)]), lwd = 3)
abline(h = mean(y[(n/2 + 1) : n]), lwd = 3)
fit <- lm(y ~ x + t)
abline(coef(fit)[1], coef(fit)[2], lwd = 3)
abline(coef(fit)[1] + coef(fit)[3], coef(fit)[2], lwd = 3)
points(x[1 : (n/2)], y[1 : (n/2)], pch = 21, col = "black", bg = "lightblue", cex = 2)
points(x[(n/2 + 1) : n], y[(n/2 + 1) : n], pch = 21, col = "black", bg = "salmon", cex = 2)
```


---
## Discussion
### Some things to note in this simulation
* The X variable is highly related to group status
* The X variable is related to Y, the intercept
  doesn't depend on the group variable.
  * The X variable remains related to Y holding group status constant
* The group variable is marginally related to Y disregarding X.
* The model would estimate no adjusted effect due to group.
  * There isn't any data to inform the relationship between
    group and Y.
  * This conclusion is entirely based on the model.

---
## Simulation 3
```{r, fig.height=5, fig.width=5, echo = FALSE, results='hide'}
n <- 100; t <- rep(c(0, 1), c(n/2, n/2)); x <- c(runif(n/2), .9 + runif(n/2));
beta0 <- 0; beta1 <- 2; tau <- -1; sigma <- .2
y <- beta0 + x * beta1 + t * tau + rnorm(n, sd = sigma)
plot(x, y, type = "n", frame = FALSE)
abline(lm(y ~ x), lwd = 2)
abline(h = mean(y[1 : (n/2)]), lwd = 3)
abline(h = mean(y[(n/2 + 1) : n]), lwd = 3)
fit <- lm(y ~ x + t)
abline(coef(fit)[1], coef(fit)[2], lwd = 3)
abline(coef(fit)[1] + coef(fit)[3], coef(fit)[2], lwd = 3)
points(x[1 : (n/2)], y[1 : (n/2)], pch = 21, col = "black", bg = "lightblue", cex = 2)
points(x[(n/2 + 1) : n], y[(n/2 + 1) : n], pch = 21, col = "black", bg = "salmon", cex = 2)
```

---
## Discussion
### Some things to note in this simulation
* Marginal association has red group higher than blue.
* Adjusted relationship has blue group higher than red.
* Group status related to X.
* There is some direct evidence for comparing red and blue
holding X fixed.



---
## Simulation 4
```{r, fig.height=5, fig.width=5, echo = FALSE, results='hide'}
n <- 100; t <- rep(c(0, 1), c(n/2, n/2)); x <- c(.5 + runif(n/2), runif(n/2));
beta0 <- 0; beta1 <- 2; tau <- 1; sigma <- .2
y <- beta0 + x * beta1 + t * tau + rnorm(n, sd = sigma)
plot(x, y, type = "n", frame = FALSE)
abline(lm(y ~ x), lwd = 2)
abline(h = mean(y[1 : (n/2)]), lwd = 3)
abline(h = mean(y[(n/2 + 1) : n]), lwd = 3)
fit <- lm(y ~ x + t)
abline(coef(fit)[1], coef(fit)[2], lwd = 3)
abline(coef(fit)[1] + coef(fit)[3], coef(fit)[2], lwd = 3)
points(x[1 : (n/2)], y[1 : (n/2)], pch = 21, col = "black", bg = "lightblue", cex = 2)
points(x[(n/2 + 1) : n], y[(n/2 + 1) : n], pch = 21, col = "black", bg = "salmon", cex = 2)
```

---
## Discussion
### Some things to note in this simulation
* No marginal association between group status and Y.
* Strong adjusted relationship.
* Group status not related to X.
* There is lots of direct evidence for comparing red and blue
holding X fixed.

---
## Simulation 5
```{r, fig.height=5, fig.width=5, echo = FALSE, results='hide'}
n <- 100; t <- rep(c(0, 1), c(n/2, n/2)); x <- c(runif(n/2, -1, 1), runif(n/2, -1, 1));
beta0 <- 0; beta1 <- 2; tau <- 0; tau1 <- -4; sigma <- .2
y <- beta0 + x * beta1 + t * tau + t * x * tau1 + rnorm(n, sd = sigma)
plot(x, y, type = "n", frame = FALSE)
abline(lm(y ~ x), lwd = 2)
abline(h = mean(y[1 : (n/2)]), lwd = 3)
abline(h = mean(y[(n/2 + 1) : n]), lwd = 3)
fit <- lm(y ~ x + t + I(x * t))
abline(coef(fit)[1], coef(fit)[2], lwd = 3)
abline(coef(fit)[1] + coef(fit)[3], coef(fit)[2] + coef(fit)[4], lwd = 3)
points(x[1 : (n/2)], y[1 : (n/2)], pch = 21, col = "black", bg = "lightblue", cex = 2)
points(x[(n/2 + 1) : n], y[(n/2 + 1) : n], pch = 21, col = "black", bg = "salmon", cex = 2)
```

---
## Discussion
### Some things to note from this simulation
* There is no such thing as a group effect here.
  * The impact of group reverses itself depending on X.
  * Both intercept and slope depends on group.
* Group status and X unrelated.
  * There's lots of information about group effects holding X fixed.

---
### Simulation 6
```{r, fig.height=5, fig.width=5, echo = FALSE, results='hide'}
p <- 1
n <- 100; x2 <- runif(n); x1 <- p * runif(n) - (1 - p) * x2
beta0 <- 0; beta1 <- 1; tau <- 4 ; sigma <- .01
y <- beta0 + x1 * beta1 + tau * x2 + rnorm(n, sd = sigma)
plot(x1, y, type = "n", frame = FALSE)
abline(lm(y ~ x1), lwd = 2)
co.pal <- heat.colors(n)
points(x1, y, pch = 21, col = "black", bg = co.pal[round((n - 1) * x2 + 1)], cex = 2)
```

---
### Do this to investigate the bivariate relationship
```
library(rgl)
plot3d(x1, x2, y)
```

---
### Residual relationship
```{r, fig.height=5, fig.width=5, echo = FALSE, results='hide'}
plot(resid(lm(x1 ~ x2)), resid(lm(y ~ x2)), frame = FALSE, col = "black", bg = "lightblue", pch = 21, cex = 2)
abline(lm(I(resid(lm(x1 ~ x2))) ~ I(resid(lm(y ~ x2)))), lwd = 2)
```


---
## Discussion
### Some things to note from this simulation

* X1 unrelated to X2
* X2 strongly related to Y
* Adjusted relationship between X1 and Y largely unchanged
  by considering X2.
  * Almost no residual variability after accounting for X2.

---
## Some final thoughts
* Modeling multivariate relationships is difficult.
* Play around with simulations to see how the
  inclusion or exclustion of another variable can
  change analyses.
* The results of these analyses deal with the
impact of variables on associations.
  * Ascertaining mechanisms or cause are difficult subjects
    to be added on top of difficulty in understanding multivariate associations.

-->
