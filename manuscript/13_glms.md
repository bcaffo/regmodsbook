# Generalized Linear Models

[Watch this video before beginning.](https://youtu.be/xEwM1nzQckY)

Generalized linear models (GLMs) were a great advance in statistical modeling. The
original manuscript with the GLM framework was from [Nelder and Wedderburn in 1972](http://www.jstor.org/stable/2344614).
in the Journal of the Royal Statistical Society. The McCullagh and Nelder book[^f1] is the famous
standard treatise on the subject.

[^f1]: McCullagh, Peter, and John A. Nelder. Generalized linear models. Vol. 37. CRC press, 1989.

Recall linear models.  Linear models are the most useful applied statistical technique. However, they are not without their limitations.
Additive response models don't make much sense if the response is discrete, or strictly positive.
Additive error models often don't make sense, for example, if the outcome has to be positive.
Transformations, such as taking a cube root of a count outcome, are often hard to interpret.  
In addition, there's value in modeling the data on the scale that it was collected.
Particularly interpretable transformations, natural logarithms in specific,
aren't applicable for negative or zero values.

The generalized linear model is family of models that includes linear models. By extending the family,
it handles many of the issues with linear models, but at the expense of some complexity and loss of some
of the mathematical tidiness. A GLM involves three components

  * An *exponential family* model for the response.
  * A systematic component via a linear predictor.
  * A link function that connects the means of the response to the linear predictor.


The three most famous cases of GLMs are: linear models, binomial and binary regression and Poisson regression.
We'll go through the GLM model specification and likelihood for all three. For linear models, we've developed them
throughout the book. The next two chapters will be devoted to binomial and Poisson regression. We'll only focus on
the most popular and useful link functions.

## Example, linear models
Let's go through an example. Assume that our response is {$$}Y_i \sim N(\mu_i, \sigma^2){/$$}. The Gaussian distribution is an exponential family distribution.
Define the **linear predictor** to be

{$$}\eta_i = \sum_{k=1}^p X_{ik} \beta_k{/$$}.

The **link function** as {$$}g{/$$} so that {$$}g(\mu) = \eta{/$$}.
For linear models {$$}g(\mu) = \mu{/$$} so that {$$}\mu_i = \eta_i{/$$}
  This yields the same likelihood model as our additive error Gaussian linear model

{$$}
Y_i = \sum_{k=1}^p X_{ik} \beta_k + \epsilon_{i}
{/$$}

where {$$}\epsilon_i \stackrel{iid}{\sim} N(0, \sigma^2){/$$}. So, we've specified our
model as a GLM above and with a more traditional linear model specification below.
Let's try an example where the GLM is more necessary.


## Example, logistic regression

Assume that our outcome is a 0, 1 variable. Let's model {$$}Y_i \sim Bernoulli(\mu_i){/$$}
so that {$$}E[Y_i] = \mu_i{/$$} where {$$}0\leq \mu_i \leq 1{/$$}.

* Linear predictor: {$$}\eta_i = \sum_{k=1}^p X_{ik} \beta_k{/$$}
* Link function {$$}g(\mu) = \eta = \log\left( \frac{\mu}{1 - \mu}\right){/$$}

In this case, {$$}g{/$$} is the (natural) log odds, referred to as the **logit**.
Note then we can invert the logit function as:

{$$}
\mu_i = \frac{\exp(\eta_i)}{1 + \exp(\eta_i)} ~~~\mbox{and}~~~
1 - \mu_i = \frac{1}{1 + \exp(\eta_i)}
{/$$}

Some people like to call this the expit function. The logit is useful as it
converts probabilities which lie in [0,1] into the whole real line, a more
natural space for the linear part of the model to live.  Notice further,
we're not transforming the outcome (Y). Instead, we'll modeling our Y
as if it were a collection of coin flips and applying the transformation
to the probability of a head.

To get the estimates we maximize the likelihood. We can write out the likelihood as:

{$$}
\prod_{i=1}^n \mu_i^{y_i} (1 - \mu_i)^{1-y_i}
= \exp\left(\sum_{i=1}^n y_i \eta_i \right)
\prod_{i=1}^n (1 + \eta_i)^{-1}
{/$$}


## Example, Poisson regression
Let's consider a problem with count data.
Assume that :

* {$$}Y_i \sim Poisson(\mu_i){/$$} so that {$$}E[Y_i] = \mu_i{/$$} where {$$}0\leq \mu_i{/$$}.
* Linear predictor {$$}\eta_i = \sum_{k=1}^p X_{ik} \beta_k{/$$}.
* Link function {$$}g(\mu) = \eta = \log(\mu){/$$}

Recall that {$$}e^x{/$$} is the inverse of {$$}\log(x){/$$} so that we have:

{$$}
\mu_i = e^{\eta_i}
{/$$}

Thus, the likelihood is:

{$$}
\prod_{i=1}^n (y_i !)^{-1} \mu_i^{y_i}e^{-\mu_i}
\propto \exp\left(\sum_{i=1}^n y_i \eta_i - \sum_{i=1}^n \mu_i\right)
{/$$}


## How estimates are obtained

For GLMs, estimates have to be obtained numerically through an iterative algorithm. The
algorithms are very well behaved, so convergence is usually not a problem unless you have
a lot of data on a boundary, such as a lot of 0 counts in binomial or Poisson data.
The standard errors are obtained also numerically, and are usually based on large sample
theory. The exact equation that gets solved is the so-called normal equations

{$$}
0=\sum_{i=1}^n \frac{(Y_i - \mu_i)}{Var(Y_i)}W_i
{/$$}

The variance differs by the model. The {$$}W_i{/$$} are derivative terms that we won't deal with.

* For the linear model {$$}Var(Y_i) = \sigma^2{/$$} (is constant).
* For Bernoulli case {$$}Var(Y_i) = \mu_i (1 - \mu_i){/$$}
* For the Poisson case {$$}Var(Y_i) = \mu_i{/$$}.

In the latter two cases, it is often relevant to have a more flexible variance model, even if it doesn't correspond to an actual likelihood.
We might make the following changes:

{$$}
0=\sum_{i=1}^n \frac{(Y_i - \mu_i)}{\phi \mu_i (1 - \mu_i ) } W_i ~~~\mbox{and}~~~
0=\sum_{i=1}^n \frac{(Y_i - \mu_i)}{\phi \mu_i} W_i
{/$$}

These are called 'quasi-likelihood' normal equations. R offers these as an option in the `glm` function
as the `quasipoisson` and `quasibinomial` options. These offer more flexible variance options than straight
Poisson and binomial models.


## Odds and ends
At this point, let's do some bookkeeping before we work through examples.

* The normal equations have to be solved iteratively. Resulting in
{$$}\hat \beta_k{/$$} and, if included, {$$}\hat \phi{/$$}.
* Predicted linear predictor responses can be obtained as {$$}\hat \eta = \sum_{k=1}^p X_k \hat \beta_k{/$$}
* Predicted mean responses as {$$}\hat \mu = g^{-1}(\hat \eta){/$$}
* Coefficients are interpreted as

{$$}
g(E[Y | X_k = x_k + 1, X_{\sim k} = x_{\sim k}]) - g(E[Y | X_k = x_k, X_{\sim k}=x_{\sim k}]) = \beta_k
{/$$}

or the change in the link function of the expected response per unit change in {$$}X_k{/$$} holding other regressors constant.

* Variations on Newon/Raphson's algorithm are used to do it.
* Asymptotics are used for inference usually (but not always).
* Many of the ideas from linear models can be brought over to GLMs.

## Exercises
1. True or false, generalized linear models transform the observed outcome. (Discuss.)
2. True or false, the interpretation of the coefficients in a GLM are on the scale of the link function. (Discuss.)
3. True or false, the generalized linear model assumes an exponential family for the outcome. (Discuss.)
4. True or false, GLM estimates are obtained by maximizing the likelihood. (Discuss.)
5. True or false, some GLM distributions impose restrictions on the relationship between the mean and the variance. (Discuss.)
