# Binary GLMs

Binary GLMs come from trying to model outcomes that can take only two
values. Some examples include: survival or not at the end of a study,
winning versus losing of a team and success versus failure of a treatment or
product. Often these outcomes are called Bernoulli outcomes, from the
Bernoulli distribution named after the famous probabilist and mathematician.

If we happen to have several exchangeable binary outcomes for the same level
of covariate values, then that is *binomial* data and we can aggregate the 0's and
1's into the count of 1's. As an example, imagine if we sprayed insect pests
with 4 different pesticides and counted whether they died or not. Then for
each spray, we could summarize the data with the count of dead and total number
that were sprayed and treat the data as binomial rather than Bernoulli.


## Example Baltimore Ravens win/loss

The Baltimore Ravens are an American Football team in
the US's National Football League.[^f2] The data contains the wins
and losses of the Ravens by the number of points that they scored. (In American football, the
goal is to score more points than your opponent.) It should be clear
that there would be a positive relationship between the number of
points scored and the probability of winning that particular game.

[^f2]: Baltimore is the home of Johns Hopkins University where your author works. I got this data set
from Jeff Leek, the instructor of three of the Data Science Specialization
courses.

Let's load the data and use `head` to look at the first few rows.
{lang=r,line-numbers=off}
~~~
> download.file("https://dl.dropboxusercontent.com/u/7710864/data/ravensData.rda"
              , destfile="./data/ravensData.rda",method="curl")
> load("./data/ravensData.rda")
> head(ravensData)
  ravenWinNum ravenWin ravenScore opponentScore
1           1        W         24             9
2           1        W         38            35
3           1        W         28            13
4           1        W         34            31
5           1        W         44            13
6           0        L         23            24
~~~


A linear regression model would look something like this:

{$$} Y_i = \beta_0 + \beta_1 X_i + e_i {/$$}

Where,
{$$}Y_i{/$$} is a binary indicator of whether or
not the Ravens won game {$$}i{/$$}(1 for a win, 0 for a loss).
{$$}X_i{/$$} is the number of points that they scored for that game.
and {$$}\epsilon_i{/$$} is the residual error term.

Under this model then {$$}\beta_0{/$$} is the expected value of
{$$}Y_i{/$$} given a 0 point game. For a 0/1 variable, the expected
value is the probability, so the intercept is the probability that
the Ravens win with 0 points scored. Then {$$}\beta_1{/$$} is the
increase in probability of a win for each addiational point.

At this point in the book, I hope that fitting and interpreting
this model would be second nature.

{lang=r,line-numbers=off}
~~~
> lmRavens <- lm(ravensData$ravenWinNum ~ ravensData$ravenScore)
> summary(lmRavens)$coef
                      Estimate Std. Error t value Pr(>|t|)
(Intercept)             0.2850   0.256643   1.111  0.28135
ravensData$ravenScore   0.0159   0.009059   1.755  0.09625
~~~

There's numerous problems with this model. First, if the Ravens
score more than 63 points in a game, we estimate a {$$}0.0159 \times
63 > 1{/$$} increase in the probability of them winning. This is
an impossibility, since a probability can't be greater than 1.
(63 is an unusual, but not impossible, score in American football.)

Perhaps less galling, but still an annoying aspect of the model, is that
if the error is assumed to be Gaussian, then our model allows {$$}Y_i{/$$}
to be anything from minus infinity to plus infinity, when we know our
data can be only be 0 or 1. If we assume that our errors are discrete
to force this, we assume a very strange distribution on the errors.

There also aren't any transformations to make things better. Any
isomorphic transformation of our outcome
is still just going to have two values with the same set of problems.

The key insight was to transform the probability of a 1 (win in our
  example) rather than the data itself. Which transformation is most
useful? It turns out that it involves the log of the odds, called
the logit.

## Odds
You've heard of odds before, most likely from discussions of gambling.
First note, odds are a fraction, not a percentage. So, when someone says
"The odds are fifty percent", they are mistaking probability and odds.
They likely mean "The probability is fifty percent.", or equivalently
"The odds are one.", or "The odds are one to one", or "The odds are
fifty [divided by] fifty." The odds statements are all the same since:
1, 1 / 1 and 50 / 50 are all the same number.

If {$$}p{/$$} is a probability, the odds are defined as
{$$}o = p/(1-p){/$$}. Note that we can go backwards as
{$$}p = o / (1 + o){/$$}. Thus, if someone says the odds are 1 to 1,
they're saying that the odds are 1 and thus {$$}p = 1 / (1 + 1) = 0.5{/$$}.
Conversely, if someone says that the probability that something occurrs
is 50%, then they mean that {$$}p=0.5{/$$} so that the odds are
{$$}o = p / (1 - p) = 0.5 / (1 - 0.5) = {/$$}.

The odds are famously derived using a standard fair game setting.
Imagine that you are playing a game where you flip a coin with success probability
{$$}p{/$$} with the following rules:

* If it comes up heads, you win {$$}X{/$$} dollars.
* If it comes up tails, you lose {$$}Y{/$$}.

What should we set {$$}X{/$$} and {$$}Y{/$$} for the game to be fair?
Fair means the expected earnings for either player is 0. That is:

{$$}E[earnings]= X p - Y (1 - p) = 0{/$$}

This implies {$$}\frac{Y}{X} = \frac{p}{1 - p} = o{/$$}.
Consider setting {$$}X=1{/$$}, then {$$} Y = o{/$$}.
Thus, the odds can be interpreted as
"How much should you be willing to pay for a {$$}p{/$$} probability
of winning a dollar?"
If {$$}p > 0.5{/$$} you have to pay more if you lose than you get if you win.
If {$$}p < 0.5{/$$} you have to pay less if you lose than you get if you win.

So, imagine that I go to a horse race and the odds that a horse loses are
50 to 1. They usually specify in terms of losing at horse tracks, so this
would be said to be 50 to 1 "against" where the against is implied and
not stated on the boards. The odds of the horse winning are
then 1/50. Thus, for a fair bet if I were to bet on the horse winning,
they should pay me 50 dollars if he wins and should pay 1 dollar if he
loses. (Or any agreed upon multiple, such as
100 dollars if he wins and 2 dollars if he loses.) The implied probability
that the horse loses is {$$}50 / (1 + 50){/$$}.  

It's interesting  
to note that the house sets the odds (hence the implied probability)
only by the bets coming in. They take a small fee for every bet win or lose
(the rake). So, by setting the odds dynamically as the bets roll in,
they can guarantee that they make money (risk free) via the rake. Thus
the phrase "the house always wins" applies literally. Even more interesting
is that by the wisdom of the crowd, the final probabilities tend to match the percentage
of times that event happens. That is, 50 to 1 horses tend to win about
1 out of 51 times, even though that 50 to 1 was set by a random collection
of mostly uninformed bettors. This is why even your sports
junkie friend with seemingly endless sports knowledge
can't make a killing betting on sports; the crowd is just too smart as a group
even though all of the individuals know less.

Finally, and then I'll get back on track, many of the machine learning
algorithms work on the principle of the wisdom of crowds:
many small dumb models can make really
good models. This is often called ensemble learning, where a lot of independent
weak classifiers are combined to make a strong one. Random forests and boosting
come to mind as examples.

Probabilities are between 0 and 1. Odds are between 0 and infinity. So, it
makes sense to model the log of the odds (the logit), since it goes from
minus infinity to plus infinity. The logit is defined as

{$$}
g = \mathrm{logit}(p) = \log(p / 1 - p)
{/$$}

We can go backwards from the logit to the probability with the so-called
expit

{$$}
\mathrm{expit}(g) = e^g / (1 + e^g) = p.
{/$$}

## Modeling the odds

Let's come up with notation for modeling the odds. Recall that
{$$}Y_i{/$$} was our outcome, a 0 or 1 indicator of whether or not
the Raven's won game {$$}i{/$$}.

Let

{$$}
p_i = \mathrm{P}(Y_i = 1 ~|~ X_i = x_i, \beta_0, \beta_1)
{/$$}

be the probability of a win for number of points {$$}x_i{/$$}.
Then the odds that they win is {$$}p_i / (1 - p_i){/$$} and the
log odds is {$$}\log{p_i / (1 - p_i)} = \mathrm{logit}(p_i){/$$}.

Logistic regression puts the model on the log of the odds (logit)
scale. In other words,

{$$}
\mathrm{logit}(p_i) = \beta_0 + \beta_1 x_i
{/$$}

Or, equivalently, we could just say

{$$}
P(Y_i = 1 ~|~ X_i = x_i, \beta_0,\beta_1)
= p_i = \frac{ \exp^{\beta_0 + \beta_1 x_i}}{1 + \exp(\beta_0 + \beta_1 x_i)}
{/$$}


<!--
## Interpreting Logistic Regression

$$ \log\left(\frac{\rm{Pr}(RW_i | RS_i, b_0, b_1 )}{1-\rm{Pr}(RW_i | RS_i, b_0, b_1)}\right) = b_0 + b_1 RS_i $$


$b_0$ - Log odds of a Ravens win if they score zero points

$b_1$ - Log odds ratio of win probability for each point scored (compared to zero points)

$\exp(b_1)$ - Odds ratio of win probability for each point scored (compared to zero points)

---

---
## Visualizing fitting logistic regression curves
```
x <- seq(-10, 10, length = 1000)
manipulate(
    plot(x, exp(beta0 + beta1 * x) / (1 + exp(beta0 + beta1 * x)),
         type = "l", lwd = 3, frame = FALSE),
    beta1 = slider(-2, 2, step = .1, initial = 2),
    beta0 = slider(-2, 2, step = .1, initial = 0)
    )
```

---

## Ravens logistic regression


```r
logRegRavens <- glm(ravensData$ravenWinNum ~ ravensData$ravenScore,family="binomial")
summary(logRegRavens)
```

```

Call:
glm(formula = ravensData$ravenWinNum ~ ravensData$ravenScore,
    family = "binomial")

Deviance Residuals:
   Min      1Q  Median      3Q     Max  
-1.758  -1.100   0.530   0.806   1.495  

Coefficients:
                      Estimate Std. Error z value Pr(>|z|)
(Intercept)            -1.6800     1.5541   -1.08     0.28
ravensData$ravenScore   0.1066     0.0667    1.60     0.11

(Dispersion parameter for binomial family taken to be 1)

    Null deviance: 24.435  on 19  degrees of freedom
Residual deviance: 20.895  on 18  degrees of freedom
AIC: 24.89

Number of Fisher Scoring iterations: 5
```



---

## Ravens fitted values


```r
plot(ravensData$ravenScore,logRegRavens$fitted,pch=19,col="blue",xlab="Score",ylab="Prob Ravens Win")
```

<div class="rimage center"><img src="fig/unnamed-chunk-1.png" title="plot of chunk unnamed-chunk-1" alt="plot of chunk unnamed-chunk-1" class="plot" /></div>



---

## Odds ratios and confidence intervals


```r
exp(logRegRavens$coeff)
```

```
          (Intercept) ravensData$ravenScore
               0.1864                1.1125
```

```r
exp(confint(logRegRavens))
```

```
                         2.5 % 97.5 %
(Intercept)           0.005675  3.106
ravensData$ravenScore 0.996230  1.303
```



---

## ANOVA for logistic regression


```r
anova(logRegRavens,test="Chisq")
```

```
Analysis of Deviance Table

Model: binomial, link: logit

Response: ravensData$ravenWinNum

Terms added sequentially (first to last)

                      Df Deviance Resid. Df Resid. Dev Pr(>Chi)  
NULL                                     19       24.4
ravensData$ravenScore  1     3.54        18       20.9     0.06 .
---
Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
```




---

## Interpreting Odds Ratios

* Not probabilities
* Odds ratio of 1 = no difference in odds
* Log odds ratio of 0 = no difference in odds
* Odds ratio < 0.5 or > 2 commonly a "moderate effect"
* Relative risk $\frac{\rm{Pr}(RW_i | RS_i = 10)}{\rm{Pr}(RW_i | RS_i = 0)}$ often easier to interpret, harder to estimate
* For small probabilities RR $\approx$ OR but __they are not the same__!

[Wikipedia on Odds Ratio](http://en.wikipedia.org/wiki/Odds_ratio)


---

## Further resources

* [Wikipedia on Logistic Regression](http://en.wikipedia.org/wiki/Logistic_regression)
* [Logistic regression and glms in R](http://data.princeton.edu/R/glms.html)
* Brian Caffo's lecture notes on: [Simpson's paradox](http://ocw.jhsph.edu/courses/MethodsInBiostatisticsII/PDFs/lecture23.pdf), [Case-control studies](http://ocw.jhsph.edu/courses/MethodsInBiostatisticsII/PDFs/lecture24.pdf)
* [Open Intro Chapter on Logistic Regression](http://www.openintro.org/stat/down/oiStat2_08.pdf)
-->
