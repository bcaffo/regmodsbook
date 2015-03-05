# Statistical linear regression models

Up to this point, we've only considered estimation. Estimation is useful,
but we also need to know how to extend our estimates to a population.
This is the process of statistical inference. Our approach to statistical
inference will be through a statistical model. At the bare minimum, we
need a few distributional assumptions on the errors. However, we'll focus on
full model assumptions under Gaussianity.


## Basic regression model with additive Gaussian errors.
Consider developing a probabilistic model for linear regression. Our
starting point will assume a systematic component via a line and then
independent and identically distributed Gaussian errors. We can write
the model out as:

{$$}
Y_i = \beta_0 + \beta_1 X_i + \epsilon_{i}
{/$$}

Here the {$$}\epsilon_{i}{/$$} are assumed to be independent and
identically distributed as
{$$}N(0, \sigma^2){/$$}. Under this model,

{$$}E[Y_i ~|~ X_i = x_i] = \mu_i = \beta_0 + \beta_1 x_i{/$$}

and

{$$}Var(Y_i ~|~ X_i = x_i) = \sigma^2.{/$$}

This model implies
 that the {$$}Y_i{/$$} are independent and normally
distributed with means {$$}\beta_0 + \beta_1 x_i{/$$} and variance
{$$}\sigma^2{/$$}. We could write this more compactly as

{$$}
Y_i ~|~ X_i = x_i \sim N(\beta_0 + \beta_1 x_i, \sigma^2).
{/$$}


While this specification of the model is a perhaps better for advanced
purposes, specifying the model as linear with additive error terms is
generally more useful. With that specification, we can hypothesize and
discuss the nature of the errors. In fact, we'll even cover ways to estimate
them to investigate our model assumption.

Remember that our least squares estimates of
{$$}\beta_0{/$$} and {$$}\beta_1{/$$} are:

{$$}\hat \beta_1 = Cor(Y, X) \frac{Sd(Y)}{Sd(X)} ~~~\mbox{and}~~~ \hat \beta_0 = \bar Y - \hat \beta_1 \bar X.{/$$}


It is convenient that under our Gaussian additive error model
that the maximum likelihood estimates of
{$$}\beta_0{/$$} and {$$}\beta_1{/$$} are the least squares estimates.

## Interpreting regression coefficients, the intercept

Our model allows us to attach statistical interpretations to our parameters.
Let's start with the intercept; {$$}\beta_0{/$$} represents
the expected value of the response when the predictor is 0. We can show this
as:

{$$}
E[Y | X = 0] =  \beta_0 + \beta_1 \times 0 = \beta_0.
{/$$}

Note, the intercept isn't always of interest. For example,
when {$$}X=0{/$$} is impossible or far outside of the range of data.
Take as a specific instance, when X is blood pressure, no one is interested
in studying blood pressure's impact on anything for values near 0.

There is a way to make your intercept more interpretable.
Consider that:

{$$}
Y_i = \beta_0 + \beta_1 X_i + \epsilon_i
= \beta_0 + a \beta_1 + \beta_1 (X_i - a) + \epsilon_i
= \tilde \beta_0 + \beta_1 (X_i - a) + \epsilon_i.
{/$$}

Therefore, shifting your {$$}X{/$$} values by value {$$}a{/$$}
changes the intercept, but not the slope.
Often {$$}a{/$$} is set to {$$}\bar X{/$$}, so that the intercept is
interpreted as the expected response at the average {$$}X{/$$} value.

## Interpreting regression coefficients, the slope
Now that we understand how to interpret the intercept, let's try interpreting
the slope. Our slope, {$$}\beta_1{/$$},
is the expected change in response for a 1 unit change in the predictor.
We can show that as follows:

{$$}
E[Y ~|~ X = x+1] - E[Y ~|~ X = x] =
\beta_0 + \beta_1 (x + 1) - (\beta_0 + \beta_1 x ) = \beta_1
{/$$}

Notice that the interpretation of {$$}\beta_1{/$$} is tied to the
units of the X variable. Let's consider the impact of changing the units.



{$$}
Y_i = \beta_0 + \beta_1 X_i + \epsilon_i
= \beta_0 + \frac{\beta_1}{a} (X_i a) + \epsilon_i
= \beta_0 + \tilde \beta_1 (X_i a) + \epsilon_i
{/$$}


Therefore, multiplication of {$$}X{/$$} by a factor {$$}a{/$$}
results in dividing the coefficient by a factor of {$$}a{/$$}.
<!--

As an example, suppose that {$$}X{/$$} is height in meters (m) and {$$}Y{/$$}
is weight in kilograms (kg). Then {$$}\beta_1{/$$} is kg/m.
Converting {$$}X{$$} to centimeters implies multiplying {$$}X{/$$} by 100 cm/m.
To get {$$}\beta_1{/$$} in the right units if we had fit the model in meters,
we have to divide by 100 cm/m. Or, we can write out the notation as:


{$$}
X m \times \frac{100cm}{m} = (100 X) cm
~~\mbox{and}~~
\beta_1 \frac{kg}{m} \times\frac{1 m}{100cm} =
\left(\frac{\beta_1}{100}\right)\frac{kg}{cm}
{/$$}

<!--
## Using regression coeficients for prediction
* If we would like to guess the outcome at a particular
  value of the predictor, say $X$, the regression model guesses
  $$
  \hat \beta_0 + \hat \beta_1 X
  $$


---
## Example
### `diamond` data set from `UsingR`
Data is diamond prices (Singapore dollars) and diamond weight
in carats (standard measure of diamond mass, 0.2 $g$). To get the data use `library(UsingR); data(diamond)`


---
## Plot of the data
<div class="rimage center"><img src="fig/unnamed-chunk-1.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" class="plot" /></div>


---
## Fitting the linear regression model

```r
fit <- lm(price ~ carat, data = diamond)
coef(fit)
```

```
(Intercept)       carat
     -259.6      3721.0
```


* We estimate an expected 3721.02 (SIN) dollar increase in price for every carat increase in mass of diamond.
* The intercept -259.63 is the expected price
  of a 0 carat diamond.

---
## Getting a more interpretable intercept

```r
fit2 <- lm(price ~ I(carat - mean(carat)), data = diamond)
coef(fit2)
```

```
           (Intercept) I(carat - mean(carat))
                 500.1                 3721.0
```


Thus $500.1 is the expected price for
the average sized diamond of the data (0.2042 carats).

---
## Changing scale
* A one carat increase in a diamond is pretty big, what about
  changing units to 1/10th of a carat?
* We can just do this by just dividing the coeficient by 10.
  * We expect  a 372.102 (SIN) dollar   change in price for every 1/10th of a carat increase in mass of diamond.
* Showing that it's the same if we rescale the Xs and refit

```r
fit3 <- lm(price ~ I(carat * 10), data = diamond)
coef(fit3)
```

```
  (Intercept) I(carat * 10)
       -259.6         372.1
```


---
## Predicting the price of a diamond

```r
newx <- c(0.16, 0.27, 0.34)
coef(fit)[1] + coef(fit)[2] * newx
```

```
[1]  335.7  745.1 1005.5
```

```r
predict(fit, newdata = data.frame(carat = newx))
```

```
     1      2      3
 335.7  745.1 1005.5
```


---
Predicted values at the observed Xs (red)
and at the new Xs (lines)
<div class="rimage center"><img src="fig/unnamed-chunk-6.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" class="plot" /></div>
-->
