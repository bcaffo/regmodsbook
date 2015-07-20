# Ordinary least squares
[Watch this video before beginning](https://www.youtube.com/watch?v=LapyH7MG3Q4&list=PLpl-gQkQivXjqHAJd2t-J_One_fYE55tC&index=6)

**Ordinary least squares** (OLS) is the workhorse of statistics. It gives
a way of taking complicated outcomes and explaining behavior (such as trends)
using linearity. The simplest application of OLS is fitting a line.


## General least squares for linear equations
Consider again the parent and child height data from Galton.

![Plot of parent and child heights.](images/freqGalton.png)

Let's try fitting the best line.
Let {$$}Y_i{/$$} be the {$$}i^{th}{/$$} child's height and
{$$}X_i{/$$} be the {$$}i^{th}{/$$} (average over the pair of) parental heights.
Consider finding the best line of the form

{$$} \mbox{Child Height} = \beta_0 + \mbox{Parent Height} \beta_1, {/$$}

Let's try using least squares by minimizing the following equation
over {$$}\beta_0{/$$} and {$$}\beta_1{/$$}:

{$$}
\sum_{i=1}^n \{Y_i - (\beta_0 + \beta_1 X_i)\}^2,
{/$$}

Minimizing this equation will minimize the sum of the squared distances
between the fitted line at the pareNnts heights ({$$} \beta_1 X_i{/$$})
and the observed child heights ({$$}Y_i{/$$}).

The result actually has a closed form. Specifically, the least squares
of the line:

{$$}Y = \beta_0 + \beta_1 X,{/$$}

through the data pairs {$$}(X_i, Y_i){/$$} with {$$}Y_i{/$$} as the outcome obtains
the line {$$}Y = \hat \beta_0 + \hat \beta_1 X{/$$}
where:

{$$}\hat \beta_1 = Cor(Y, X) \frac{Sd(Y)}{Sd(X)} ~~~ \hat \beta_0 = \bar Y - \hat \beta_1 \bar X{/$$}

At this point, a couple of notes are in order.
First, the slope, {$$}\hat \beta_1{/$$}, has the units of {$$}Y / X{/$$}. Secondly,
the intercept, {$$}\hat \beta_0{/$$}, has the units of {$$}Y{/$$}.

The line passes through the point {$$}(\bar X, \bar Y){/$$}. If you center your
Xs and Ys first, then the line will pass through the origin. Moreover,
the slope is the same one you would get if you centered the data,
{$$}(X_i - \bar X, Y_i - \bar Y){/$$}, and either fit a linear regression
or regression through the origin.

To elaborate, regression through the origin, assuming that {$$}\beta_0 = 0{/$$},
yields the following solution to the least squares criteria:

{$$}
\hat \beta_1 = \frac{\sum_{i=1}^n X_i Y_i}{\sum_{i=1}^n X_i^2},
{/$$}

This is exactly the correlation time the
ratio in the standard deviations if
the both the Xs and Ys have been centered first. (Try it out using R to
verify this!)

It is interesting to think about what happens when you reverse the role of {$$}X{/$$}
and {$$}Y{/$$}. Specifically, the slope of the regression line with {$$}X{/$$}
as the outcome and {$$}Y{/$$} as the predictor is {$$}Cor(Y, X) Sd(X)/ Sd(Y){/$$}.

If you normalized the data,
{$$}\{ \frac{X_i - \bar X}{Sd(X)}, \frac{Y_i - \bar Y}{Sd(Y)}\}{/$$},
the slope is simply the correlation, {$$}Cor(Y, X){/$$}, regardless of which variable
is treated as the outcome.


## Revisiting Galton's data
[Watch this video before beginning](https://www.youtube.com/watch?v=O7cDyrjWBBc&index=7&list=PLpl-gQkQivXjqHAJd2t-J_One_fYE55tC)

Let's double check our calculations using R

{title="Fitting Galton's data using linear regression.", line-numbers=off,lang=r}
~~~
> y <- galton$child
> x <- galton$parent
> beta1 <- cor(y, x) *  sd(y) / sd(x)
> beta0 <- mean(y) - beta1 * mean(x)
> rbind(c(beta0, beta1), coef(lm(y ~ x)))
     (Intercept)      x
[1,]       23.94 0.6463
[2,]       23.94 0.6463
~~~

We can see that the result of `lm` is identical to hard coding the fit ourselves.
Let's Reversing the outcome/predictor relationship.

{line-numbers=off,lang=r}
~~~
> beta1 <- cor(y, x) *  sd(x) / sd(y)
> beta0 <- mean(x) - beta1 * mean(y)
> rbind(c(beta0, beta1), coef(lm(x ~ y)))
     (Intercept)      y
[1,]       46.14 0.3256
[2,]       46.14 0.3256
~~~

Now let's show that
regression through the origin yields an equivalent slope if you center the data first

{line-numbers=off,lang=r}
~~~
> yc <- y - mean(y)
> xc <- x - mean(x)
> beta1 <- sum(yc * xc) / sum(xc ^ 2)
c(beta1, coef(lm(y ~ x))[2])
            x
0.6463 0.6463
~~~

Now let's show that normalizing variables results in the slope being the correlation.

{line-numbers=off,lang=r}
~~~
> yn <- (y - mean(y))/sd(y)
> xn <- (x - mean(x))/sd(x)
> c(cor(y, x), cor(yn, xn), coef(lm(yn ~ xn))[2])
                  xn
0.4588 0.4588 0.4588
~~~

The image below plots the data again, the best fitting line and standard
error bars for the fit. We'll work up to that point later. But, understanding
that our fitted line is estimated with error is an important concept.
You can find the code for the plot [here](https://github.com/bcaffo/courses/blob/master/07_RegressionModels/01_03_ols/index.Rmd).

![Image of the data, the fitted line and error bars.](images/galton4.png)

## Showing the OLS result
If you would like to see a proof of why the ordinary least squares result works out to be the way that it is:
[watch this video](https://www.youtube.com/watch?v=COVQX8WZVA8&index=8&list=PLpl-gQkQivXjqHAJd2t-J_One_fYE55tC).
