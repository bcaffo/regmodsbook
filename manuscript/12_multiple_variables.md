# Multiple variables and model selection

This chapter represents a challenging question: "How do we chose
what to variables to include in a regression model?". Sadly, no
single easy answer exists and the most reasonable answer would be "It depends.".
These concepts bleed into ideas of machine learning, which is largely focused
on high dimensional variable selection and weighting.
In this chapter we cover some of the basics and, most importantly, the
consequences of over- and under-fitting a model.

## Multivariable regression
In our Coursera Data Science Specialization, we have an entire class on prediction and machine learning.
So, in this class, our focus will be on modeling. That is, our primary concern is winding up with an
interpretable model, with interpretable coefficients. This is a very different process than
if we only care about prediction or machine learning.
Prediction has a different set of criteria, needs for interpretability and standards for generalizability.
In modeling, our interest lies in parsimonious, interpretable representations of the data that enhance our
understanding of the phenomena under study.

Like nearly all aspects of statistics, good modeling decisions are context dependent. Consider
a good model for prediction, versus one for studying mechanisms, versus one for trying to establish causal effects.
There are, however, some principles to help you guide your way.

*Parsimony* is a core concept in model selection. The idea of parsimony is to keep your models as simple
as possible (but no simpler). This builds on the idea of [Occam's razor](https://en.wikipedia.org/wiki/Occam's_razor),
in that all else being equal simpler explanations are better than complex ones. Simpler models are easier
to interpret and are less finicky. Complex models often have issues with fitting and, especially, overfitting.
([To see a counterargument, consider Andrew Gelman's blog.](http://andrewgelman.com/2004/12/10/against_parsimo/).)

Another principle that I find useful for looking at statistical models is to consider them as lenses through
which to look at your data. (I attribute this quote to great statistician
Scott Zeger.) Under this philosophy, what's the right model - whatever one connects the data to a true, parsimonious statement about what you're
studying. Unwin and authors have formalized these ideas more into something they call [exploratory model analysis](http://www.sciencedirect.com/science/article/pii/S016794730200292X)
I like this, as it turns our focus away from trying to get a single, best, true model and instead focuses on.
This is useful, since there are uncountable ways that a model can be wrong.
In this lecture, we'll focus on variable inclusion and exclusion.


## The Rumsfeldian triplet

Before we begin, I'd like to give a quote from Donal Rumsfeld, the controversial Secretary of Defense of the US during
the start of the Afghanistan the second Iraq wars. He gave this quote regarding weapons of mass destruction
([read more about it here](https://en.wikipedia.org/wiki/There_are_known_knowns)):

"There are known knowns. These are things we know that we know. There are known unknowns. That is to say, there are things that we know we don't know. But there are also unknown unknowns. There are things we don't know we don't know." - Donald Rumsfeld

This quote, widely derided for its intended purpose, is quite insightful in the unintended
context of regression model selection. Specifically, in our context
"Known Knowns" are regressors that we know we should check to include in the model and have.
The "Known Unknowns" are regressors that we would like to include in the model, but don't have.
The "Unknown Unknowns" are regressors that we don't even know about that we should have included in the model.

In this chapter, we'll talk about Known Knowns; variables that are potentially of interest in our model that we have.
Known Unknowns and Unknown Unknowns (especially) are more challenging to deal with. A central method for dealing with
Unknown Unknowns is randomization. If you'd like to compare a treatment to a control, or perform an A/B test of two
advertising strategies, randomization will help insure that your treatment is balanced across levels of the Unknown
Unknowns with high probability. (Of course, being unobserved, you can never know whether or not the randomization was
effective.)

For Known Unknowns, those variables we wish we had collected but are aware about, there are several strategies.
For example, a proxy variable might be of use. As an example, we had some brain volumetric measurements via MRIs
and really wished we had done the processing to get intra-cranial volume (head size). The need for this variable
was because we didn't want to compare brain volumetric measurements and conclude that bigger people with bigger
heads have more brain mass. This would be a useless conclusion, for example whales have bigger brains than dolphins,
but that doesn't tell you much about whales or dolphins. More interesting would be if whales who were exposed to toxic chemicals had lower
brain volume relative to their intra-cranial volume than whales who weren't exposed. In our case, (we were studying humans),
we used height, gender and other anthropomorphic measurements to get a good guess of intra-cranial volume.

For the rest of the lecture, let's discuss the known knowns and what their unnecessary inclusion and exclusion implies
in our analysis.

## General rules
Here we state a couple of general rules regarding model selection for our known knowns.

* Omitting variables results in bias in the coefficients of interest - unless the regressors are uncorrelated with the omitted ones.

I want to reiterate this point: if the omitted variable is uncorrelated with the included variables, its omission has no impact
on estimation. It might explain some residual variation, thus it could have an impact on inference.
As previously mentioned, this lack of impact of uncorrelated variables is why we randomize treatments;
randomization attempts to disassociate our treatment indicator with variables that we don't have to put in the model. Formal
theories of inference can be designed around the use of randomization.  However,
in a practical sense, if there's too many unobserved confounding variables, even randomization won't help you, since
with high probability one will stay correlated with the treatment.

In most cases we won't have randomization. So, to avoid bias, why don't we throw everything into the regression model?
The following rule prevents us from doing that:

* Including variables that we shouldn't have increases standard errors of the regression variables.

Actually, including any new variables increases the actual (not estimated) standard errors of other regressors.
So we don't want to idly throw variables into the model. In addition the model must tend toward perfect
fit as the number of non-redundant regressors approaches the sample size.  Our {$$}R^2{/$$}
increases monotonically as more regressors are included, even unrelated white noise.


## {$$}R^2{/$$} goes up as you put regressors in the model

Let's try a simulation. In this simulation, no regression relationship exists. We simulate data and {$$}p{/$$} regressors as random normals.
The plot is of the {$$}R^2{/$$}.

{lang=r,line-numbers=off}
~~~
n <- 100
plot(c(1, n), 0 : 1, type = "n", frame = FALSE, xlab = "p", ylab = "R^2")
y <- rnorm(n); x <- NULL; r <- NULL
for (i in 1 : n){
   x <- cbind(x, rnorm(n))
   r <- c(r, summary(lm(y ~ x))$r.squared)
}
lines(1 : n, r, lwd = 3)
abline(h = 1)
~~~

![Plot of {$$}R^2{/$$} by {$$}n{/$$} as more regressors are included. No actual regression ](images/model1.png)

Notice that the {$$}R^2{/$$} goes up, monotonically, as the number of regressors
is increased.


## Variance inflation

Now let's use simulation to demonstrate variation inflation. In this case,
we're going to simulate three regressors, x1, x2 and x3. We then repeatedly
generate data from a model where y only depends on x1. We fit three models,
`y ~ x1`, `y ~ x1 + x2`, and `y ~ x1 + x2 + x3`. We do this over and over
again and look at the standard deviation of the x1 coefficient.

{lang=r, line-numbers=off}
~~~
> n <- 100; nosim <- 1000
> x1 <- rnorm(n); x2 <- rnorm(n); x3 <- rnorm(n);
> betas <- sapply(1 : nosim, function(i){
  y <- x1 + rnorm(n, sd = .3)
  c(coef(lm(y ~ x1))[2],
    coef(lm(y ~ x1 + x2))[2],
    coef(lm(y ~ x1 + x2 + x3))[2])
  })
> round(apply(betas, 1, sd), 5)

     x1      x1      x1
0.02839 0.02872 0.02884
~~~


---
---
---
## Variance inflation

```r
n <- 100; nosim <- 1000
x1 <- rnorm(n); x2 <- x1/sqrt(2) + rnorm(n) /sqrt(2)
x3 <- x1 * 0.95 + rnorm(n) * sqrt(1 - 0.95^2);
betas <- sapply(1 : nosim, function(i){
  y <- x1 + rnorm(n, sd = .3)
  c(coef(lm(y ~ x1))[2],
    coef(lm(y ~ x1 + x2))[2],
    coef(lm(y ~ x1 + x2 + x3))[2])
})
round(apply(betas, 1, sd), 5)
```

```
     x1      x1      x1
0.03131 0.04270 0.09653
```

## Variance inflation factors
* Notice variance inflation was much worse when we included a variable that
was highly related to `x1`.
* We don't know $\sigma$, so we can only estimate the increase in the actual standard error of the coefficients for including a regressor.
* However, $\sigma$ drops out of the relative standard errors. If one sequentially adds variables, one can check the variance (or sd) inflation for including each one.
* When the other regressors are actually orthogonal to the regressor of interest, then there is no variance inflation.
* The variance inflation factor (VIF) is the increase in the variance for the ith regressor compared to the ideal setting where it is orthogonal to the other regressors.
  * (The square root of the VIF is the increase in the sd ...)
* Remember, variance inflation is only part of the picture. We want to include certain variables, even if they dramatically inflate our variance.

---
## Revisting our previous simulation

```r
##doesn't depend on which y you use,
y <- x1 + rnorm(n, sd = .3)
a <- summary(lm(y ~ x1))$cov.unscaled[2,2]
c(summary(lm(y ~ x1 + x2))$cov.unscaled[2,2],
  summary(lm(y~ x1 + x2 + x3))$cov.unscaled[2,2]) / a
```

```
[1] 1.895 9.948
```

```r
temp <- apply(betas, 1, var); temp[2 : 3] / temp[1]
```

```
   x1    x1
1.860 9.506
```

---
## Swiss data

```r
data(swiss);
fit1 <- lm(Fertility ~ Agriculture, data = swiss)
a <- summary(fit1)$cov.unscaled[2,2]
fit2 <- update(fit, Fertility ~ Agriculture + Examination)
fit3 <- update(fit, Fertility ~ Agriculture + Examination + Education)
  c(summary(fit2)$cov.unscaled[2,2],
    summary(fit3)$cov.unscaled[2,2]) / a
```

```
[1] 1.892 2.089
```


---
## Swiss data VIFs,

```r
library(car)
fit <- lm(Fertility ~ . , data = swiss)
vif(fit)
```

```
     Agriculture      Examination        Education         Catholic Infant.Mortality
           2.284            3.675            2.775            1.937            1.108
```

```r
sqrt(vif(fit)) #I prefer sd
```

```
     Agriculture      Examination        Education         Catholic Infant.Mortality
           1.511            1.917            1.666            1.392            1.052
```


---
## What about residual variance estimation?
* Assuming that the model is linear with additive iid errors (with finite variance), we can mathematically describe the impact of omitting necessary variables or including unnecessary ones.
  * If we underfit the model, the variance estimate is biased.
  * If we correctly or overfit the model, including all necessary covariates and/or unnecessary covariates, the variance estimate is unbiased.
    * However, the variance of the variance is larger if we include unnecessary variables.

---
## Covariate model selection
* Automated covariate selection is a difficult topic. It depends heavily on how rich of a covariate space one wants to explore.
  * The space of models explodes quickly as you add interactions and polynomial terms.
* In the prediction class, we'll cover many modern methods for traversing large model spaces for the purposes of prediction.
* Principal components or factor analytic models on covariates are often useful for reducing complex covariate spaces.
* Good design can often eliminate the need for complex model searches at analyses; though often control over the design is limited.
* If the models of interest are nested and without lots of parameters differentiating them, it's fairly uncontroversial to use nested likelihood ratio tests. (Example to follow.)
* My favorite approach is as follows. Given a coefficient that I'm interested in, I like to use covariate adjustment and multiple models to probe that effect to evaluate it for robustness and to see what other covariates knock it out.  This isn't a terribly systematic approach, but it tends to teach you a lot about the the data as you get your hands dirty.

---
## How to do nested model testing in R

```r
fit1 <- lm(Fertility ~ Agriculture, data = swiss)
fit3 <- update(fit, Fertility ~ Agriculture + Examination + Education)
fit5 <- update(fit, Fertility ~ Agriculture + Examination + Education + Catholic + Infant.Mortality)
anova(fit1, fit3, fit5)
```

```
Analysis of Variance Table

Model 1: Fertility ~ Agriculture
Model 2: Fertility ~ Agriculture + Examination + Education
Model 3: Fertility ~ Agriculture + Examination + Education + Catholic +
    Infant.Mortality
  Res.Df  RSS Df Sum of Sq    F  Pr(>F)
1     45 6283
2     43 3181  2      3102 30.2 8.6e-09 ***
3     41 2105  2      1076 10.5 0.00021 ***
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```
-->
