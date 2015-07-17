# Count data

Many data take the form of unbounded count
data. For example, consider the number of calls
to a call center or the number of flu cases
in an area or the number of hits to a web site.

In some of these cases the counts are clearly
bounded. However, modeling the counts as unbounded
is often done when the upper limit is not known
or very large relative to the number of events.

If the upper bound is known, the techniques we're
discussing can be used to model the proportion or
rate. The
starting point for most count analysis is the
the Poisson distribution.


## Poisson distribution

The Poisson distribution is the goto distribution for modeling
counts and rates. We'll define a rate as a count per unit of time.
For example your heart rate is often expressed in beats per minute.
So, we might look at web hits per day, or disease cases per
exposure time (incidence rates). Also, though not exactly a rate,
we can treat proportions as if rates when {$$}n{/$$} is large
and the success probability is small.

We would write that a random variable is Poisson,
{$$}X \sim Poisson(t\lambda){/$$}, if its density function is:

{$$}
P(X = x) = \frac{(t\lambda)^x e^{-t\lambda}}{x!}
{/$$}

where {$$}x = 0, 1, \ldots{/$$}. The
mean of the Poisson is {$$}E[X] = t\lambda{/$$}, thus {$$}E[X / t] = \lambda{/$$}.
The variance of the Poisson is {$$}Var(X) = t\lambda{/$$}.
The Poisson tends to a normal as {$$}t\lambda{/$$} gets large and
approximates a binomial with large {$$}n{/$$} and small {$$}p{/$$}
where we would think of {$$}t\lambda{/$$} as {$$}n p {/$$}.

Here are some plots of the Poisson density to illustrate
how it closely approximates a normal.

{lang=r,line-numbers=off}
~~~
par(mfrow = c(1, 3))
plot(0 : 10, dpois(0 : 10, lambda = 2), type = "h", frame = FALSE)
plot(0 : 20, dpois(0 : 20, lambda = 10), type = "h", frame = FALSE)
plot(0 : 200, dpois(0 : 200, lambda = 100), type = "h", frame = FALSE)
~~~

![Poisson densities as the mean increases.](figures/simPois.png)


<!--

## Poisson distribution
### Sort of, showing that the mean and variance are equal

```r
x <- 0 : 10000; lambda = 3
mu <- sum(x * dpois(x, lambda = lambda))
sigmasq <- sum((x - mu)^2 * dpois(x, lambda = lambda))
c(mu, sigmasq)
```

```
[1] 3 3
```


---

## Example: Leek Group Website Traffic
* Consider the daily counts to Jeff Leek's web site

[http://biostat.jhsph.edu/~jleek/](http://biostat.jhsph.edu/~jleek/)

* Since the unit of time is always one day, set $t = 1$ and then
the Poisson mean is interpretted as web hits per day. (If we set $t = 24$, it would
be web hits per hour).

---

## Website data


```r
download.file("https://dl.dropboxusercontent.com/u/7710864/data/gaData.rda",destfile="./data/gaData.rda",method="curl")
load("./data/gaData.rda")
gaData$julian <- julian(gaData$date)
head(gaData)
```

```
        date visits simplystats julian
1 2011-01-01      0           0  14975
2 2011-01-02      0           0  14976
3 2011-01-03      0           0  14977
4 2011-01-04      0           0  14978
5 2011-01-05      0           0  14979
6 2011-01-06      0           0  14980
```


[http://skardhamar.github.com/rga/](http://skardhamar.github.com/rga/)


---

## Plot data


```r
plot(gaData$julian,gaData$visits,pch=19,col="darkgrey",xlab="Julian",ylab="Visits")
```

<div class="rimage center"><img src="fig/unnamed-chunk-2.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" class="plot" /></div>



---

## Linear regression

$$ NH_i = b_0 + b_1 JD_i + e_i $$

$NH_i$ - number of hits to the website

$JD_i$ - day of the year (Julian day)

$b_0$ - number of hits on Julian day 0 (1970-01-01)

$b_1$ - increase in number of hits per unit day

$e_i$ - variation due to everything we didn't measure


---

## Linear regression line


```r
plot(gaData$julian,gaData$visits,pch=19,col="darkgrey",xlab="Julian",ylab="Visits")
lm1 <- lm(gaData$visits ~ gaData$julian)
abline(lm1,col="red",lwd=3)
```

<div class="rimage center"><img src="fig/linReg.png" title="plot of chunk linReg" alt="plot of chunk linReg" class="plot" /></div>

---

## Aside, taking the log of the outcome
- Taking the natural log of the outcome has a specific interpretation.
- Consider the model

$$ \log(NH_i) = b_0 + b_1 JD_i + e_i $$

$NH_i$ - number of hits to the website

$JD_i$ - day of the year (Julian day)

$b_0$ - log number of hits on Julian day 0 (1970-01-01)

$b_1$ - increase in log number of hits per unit day

$e_i$ - variation due to everything we didn't measure

---
## Exponentiating coefficients
- $e^{E[\log(Y)]}$ geometric mean of $Y$.
    - With no covariates, this is estimated by $e^{\frac{1}{n}\sum_{i=1}^n \log(y_i)} = (\prod_{i=1}^n y_i)^{1/n}$
- When you take the natural log of outcomes and fit a regression model, your exponentiated coefficients
estimate things about geometric means.
- $e^{\beta_0}$ estimated geometric mean hits on day 0
- $e^{\beta_1}$ estimated relative increase or decrease in geometric mean hits per day
- There's a problem with logs with you have zero counts, adding a constant works

```r
round(exp(coef(lm(I(log(gaData$visits + 1)) ~ gaData$julian))), 5)
```

```
  (Intercept) gaData$julian
        0.000         1.002
```


---

## Linear vs. Poisson regression

__Linear__

$$ NH_i = b_0 + b_1 JD_i + e_i $$

or

$$ E[NH_i | JD_i, b_0, b_1] = b_0 + b_1 JD_i$$

__Poisson/log-linear__

$$ \log\left(E[NH_i | JD_i, b_0, b_1]\right) = b_0 + b_1 JD_i $$

or

$$ E[NH_i | JD_i, b_0, b_1] = \exp\left(b_0 + b_1 JD_i\right) $$


---

## Multiplicative differences

<br><br>
$$ E[NH_i | JD_i, b_0, b_1] = \exp\left(b_0 + b_1 JD_i\right) $$

<br><br>

$$ E[NH_i | JD_i, b_0, b_1] = \exp\left(b_0 \right)\exp\left(b_1 JD_i\right) $$

<br><br>

If $JD_i$ is increased by one unit, $E[NH_i | JD_i, b_0, b_1]$ is multiplied by $\exp\left(b_1\right)$

---

## Poisson regression in R


```r
plot(gaData$julian,gaData$visits,pch=19,col="darkgrey",xlab="Julian",ylab="Visits")
glm1 <- glm(gaData$visits ~ gaData$julian,family="poisson")
abline(lm1,col="red",lwd=3); lines(gaData$julian,glm1$fitted,col="blue",lwd=3)
```

<div class="rimage center"><img src="fig/poisReg.png" title="plot of chunk poisReg" alt="plot of chunk poisReg" class="plot" /></div>



---

## Mean-variance relationship?


```r
plot(glm1$fitted,glm1$residuals,pch=19,col="grey",ylab="Residuals",xlab="Fitted")
```

<div class="rimage center"><img src="fig/unnamed-chunk-4.png" title="plot of chunk unnamed-chunk-4" alt="plot of chunk unnamed-chunk-4" class="plot" /></div>


---

## Model agnostic standard errors


```r
library(sandwich)
confint.agnostic <- function (object, parm, level = 0.95, ...)
{
    cf <- coef(object); pnames <- names(cf)
    if (missing(parm))
        parm <- pnames
    else if (is.numeric(parm))
        parm <- pnames[parm]
    a <- (1 - level)/2; a <- c(a, 1 - a)
    pct <- stats:::format.perc(a, 3)
    fac <- qnorm(a)
    ci <- array(NA, dim = c(length(parm), 2L), dimnames = list(parm,
                                                               pct))
    ses <- sqrt(diag(sandwich::vcovHC(object)))[parm]
    ci[] <- cf[parm] + ses %o% fac
    ci
}
```

[http://stackoverflow.com/questions/3817182/vcovhc-and-confidence-interval](http://stackoverflow.com/questions/3817182/vcovhc-and-confidence-interval)

---

## Estimating confidence intervals


```r
confint(glm1)
```

```
                  2.5 %     97.5 %
(Intercept)   -34.34658 -31.159716
gaData$julian   0.00219   0.002396
```

```r
confint.agnostic(glm1)
```

```
                   2.5 %     97.5 %
(Intercept)   -36.362675 -29.136997
gaData$julian   0.002058   0.002528
```



---

## Rates


<br><br>


$$ E[NHSS_i | JD_i, b_0, b_1]/NH_i = \exp\left(b_0 + b_1 JD_i\right) $$

<br><br>

$$ \log\left(E[NHSS_i | JD_i, b_0, b_1]\right) - \log(NH_i)  =  b_0 + b_1 JD_i $$

<br><br>

$$ \log\left(E[NHSS_i | JD_i, b_0, b_1]\right) = \log(NH_i) + b_0 + b_1 JD_i $$

---

## Fitting rates in R


```r
glm2 <- glm(gaData$simplystats ~ julian(gaData$date),offset=log(visits+1),
            family="poisson",data=gaData)
plot(julian(gaData$date),glm2$fitted,col="blue",pch=19,xlab="Date",ylab="Fitted Counts")
points(julian(gaData$date),glm1$fitted,col="red",pch=19)
```

<div class="rimage center"><img src="fig/ratesFit.png" title="plot of chunk ratesFit" alt="plot of chunk ratesFit" class="plot" /></div>


---

## Fitting rates in R


```r
glm2 <- glm(gaData$simplystats ~ julian(gaData$date),offset=log(visits+1),
            family="poisson",data=gaData)
plot(julian(gaData$date),gaData$simplystats/(gaData$visits+1),col="grey",xlab="Date",
     ylab="Fitted Rates",pch=19)
lines(julian(gaData$date),glm2$fitted/(gaData$visits+1),col="blue",lwd=3)
```

<div class="rimage center"><img src="fig/unnamed-chunk-6.png" title="plot of chunk unnamed-chunk-6" alt="plot of chunk unnamed-chunk-6" class="plot" /></div>


---

## More information

* [Log-linear models and multiway tables](http://ww2.coastal.edu/kingw/statistics/R-tutorials/loglin.html)
* [Wikipedia on Poisson regression](http://en.wikipedia.org/wiki/Poisson_regression), [Wikipedia on overdispersion](http://en.wikipedia.org/wiki/Overdispersion)
* [Regression models for count data in R](http://cran.r-project.org/web/packages/pscl/vignettes/countreg.pdf)
* [pscl package](http://cran.r-project.org/web/packages/pscl/index.html) - the function _zeroinfl_ fits zero inflated models.

-->
