---
title: "Today I Learned - Linear Regression"
excerpt:
tags:
  - mathematics
  - statistics
  - today i learned
header:
  overlay_image: /assets/images/cool-backgrounds/cool-background6.png
  caption: 'Photo credit: [coolbackgrounds.io](https://coolbackgrounds.io/)'
mathjax: true
last_modified_at: 2018-06-02
---

_Linear regression_ is one of the most powerful and versatile workhorses of
statistics, and yet somehow I've never really managed to understand it in class.
The presentation always seemed very canned, each topic coming out like a
sardine: packed so close together, but always slipping from your hands whenever
you pick them up.

I think that what makes linear regression so difficult to learn is that it is a
topic that has been studied _ad nauseam_. Mathematicians, scientists,
bioinformaticians, statisticians, social scientists, psychologists... So many
disciplines have their own understanding of linear regression, which means that
the exact same idea has multiple names, notations and formalisms. Just as an
example, what mathematicians call **Tikhonov regularization**, statisticians call
**ridge regression**, machine learning experts call **weight decay**, and other
people simply call **linear regularization**.

With so many different possible ways to introduce linear regression, it's easy
to get lost and drown. So what I've done is take the time to really dig into the
machinery, and explain how all of this linear regression stuff hangs together,
without mentioning any discipline-specific names. This post will hopefully be
helpful for people who have had some exposure to linear regression before, and
some fuzzy recollection of what it might be, but really wants to see how
everything fits together.

There's going to be a fair amount of math (enough to properly explain the gist
of linear regression), but I'm really not emphasizing proofs here, in favor of
explaining the various flavors of linear regression.

## So Uh, What is Linear Regression?

So what is linear regression?

The basic idea is this: we have some number that we're interested in. This
number could be the price of a stock, the number of stars a restaurant has on
Yelp... Let's denote this _number-that-we-are-interested-in_ by the letter
$$y$$. Occasionally, we may have multiple observations for $$y$$ (e.g. we
monitored the price of the stock over many days, or we surveyed many restaurants
in a neighborhood). In this case, we stack these values of $$y$$ and consider
them as a single vector: $$\textbf{y}$$. To be explicit, if we have $$n$$
observations of $$y$$, then $$\textbf{y}$$ will be an $$n$$-dimensional vector.

We also have some other numbers that we think are related to $$y$$. More
explicitly, we have some other numbers that we suspect _tell us something_ about
$$y$$. For example (in each of the above scenarios), they could be how the stock
market is doing, or the average price of the food at this restaurant. Let us
denote these _numbers-that-tell-us-something-about-y_ by the letter $$x$$.
So if we have $$p$$ such numbers, we'd call them $$x_1, x_2, ..., x_p$$. Again,
we occasionally have multiple observations: in which case, we arrange the $$x$$
values into an $$n \times p$$ matrix which we call $$X$$; similarly, we stack the
$$\beta$$s into a $$p$$-dimensional vectors, $$\textbf{\beta}$$. Note that
$$\alpha$$ remains common throughout all observations.

If we have this setup, linear regression simply tells us that $$y$$ is a
weighted sum of the $$x$$s, plus some constant term. Easier to show you.

$$ y = \alpha + \beta_1 x_1 + \beta_2 x_2 + ... + \beta_p x_p + \epsilon $$

where the $$\alpha$$ and $$\beta$$s are all scalars to be determined, and the
$$\epsilon$$ is an error term (a.k.a. the **residuals**).

If we consider $$n$$ different observations, we can write the equation much more
succinctly by simply prepending a column of 1s to the $$\textbf{X}$$ matrix and
prepending an extra element to the $$\textbf{\beta}$$ vector. Then the equation can
be written as:

$$ \textbf{y} = \textbf{X} \textbf{\beta} + \textbf{\epsilon} $$

That's it. The hard part (and the whole zoo of different kinds of linear
regressions) now comes from two questions:

1. What can we assume, and more importantly, what _can't_ we assume about $$X$$ and $$y$$?
2. Given $$X$$ and $$y$$, how exactly do we find $$\alpha$$ and $$\beta$$?

## The Small-Brain Solution: Ordinary Least Squares

<img style="float: middle" src="http://i1.kym-cdn.com/photos/images/facebook/001/232/375/3fb.jpg">

This section is mostly just a re-packaging of what you could find in any
introductory statistics book, just in fewer words.

Instead of futzing around with whether or not we have multiple observations,
let's just assume we have $$n$$ observations: we can always set $$ n = 1 $$ if
that's the case. So,

- Let $$\textbf{y}$$ $$\textbf{\alpha}$$ and $$\beta$$ be $$p$$-dimensional vectors
- Let $$\textbf{X}$$ be an $$n \times p$$ matrix

The simplest, small-brain way of getting our parameter $$\textbf{\beta}$$ is by
minimizing the sum of squares:

$$\textbf{\hat{\beta}} = argmin |\textbf{y} - \textbf{X}\textbf{\beta}|^2 $$

Our estimate for $$\textbf{\beta}$$ then has a miraculous closed-form solution given
by:

$$ \textbf{\hat{\beta}} = (\textbf{X}^T \textbf{X})^{-1} \textbf{X} \textbf{y} $$

This solution is so (in)famous that it been blessed with a fairly universal
name, but cursed with the unimpressive name _ordinary least squares_ (a.k.a.
OLS).

If you have a bit of mathematical statistics under your belt, it's worth noting
that the least squares estimate for $$\textbf{\beta}$$ has a load of nice
statistical properties. It has a simple closed form solution, where the
trickiest thing is a matrix inversion: hardly asking for a computational
miracle. If we can assume that $$\epsilon$$ is zero-mean Gaussian, the least
squares estimate is the maximum likelihood estimate. Even better, if the errors
are uncorrelated and homoskedastic, then the least squares estimate is the best
linear unbiased estimator. _Basically, this is very nice._ If most of that flew
over your head, don't worry - in fact, forget I said anything at all.

## Why the Small-Brain Solution Sucks

[There are a ton of
reasons.](http://www.clockbackward.com/2009/06/18/ordinary-least-squares-linear-regression-flaws-problems-and-pitfalls/)
Here, I'll just highlight a few.

1. The least-squares estimate of $$\textbf{\beta}$$ is very susceptible to outliers
2. Assumption of homoskedasticity
3. If we have collinearity
3. If we have too many features

Points 1 and 2 are specific to the method of ordinary least squares, while 3 and
4 are just suckish things about linear regression in general.

### Outliers

The OLS estimate for $$\textbf{\beta}$$ is famously susceptible to outliers. As an
example, consider the third dataset in [Anscombe's
quartet](https://en.wikipedia.org/wiki/Anscombe%27s_quartet). That is, the data
is almost a perfect line, but the $$n$$th data point is a clear outlier. That
single data point pulls the entire regression line closer to it, which means it
fits the rest of the data worse, in order to accommodate that single outlier.

<img style="float: middle" src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/ec/Anscombe%27s_quartet_3.svg/990px-Anscombe%27s_quartet_3.svg.png">

### Heteroskedasticity and Correlated Residuals

Baked into the OLS estimate is an implicit assumption that the $$\epsilon$$s all
have the same variance. That is, the amount of noise in our data is independent
of what region of our feature space we're in. However, this is usually not a
great assumption. For example, harking back to our stock price and Yelp rating
examples, this assumption states that the price of a stock fluctuates just as
much in the hour before lunch as it does in the last 5 minutes before market
close, or that Michelin-starred restaurants have as much variation in their Yelp
ratings as do local coffee shops.

Even worse: not only can the residuals have different variances, but they may
also even be correlated! There's no reason why this can't be the case. Going
back to the stock price example, we know that high-volatility regimes introduce
much higher noise in the price of a stock, and volatility regimes tend to stay
fairly constant over time (notwithstanding structural breaks), which means that
the level of volatility (i.e. noise, or residual) suffers very high
autocorrelation.

The long and short of this is that some points in our training data are more
likely to be impaired by noise and/or correlation than others, which means that
some points in our training set are more reliable/valuable than others. We don’t
want to ignore the less reliable points completely, but they should count less
in our computation of $$\textbf{\beta}$$ than points that come from regions of
space with less noise, or not impaired as much by correlation.

### Collinearity

Collinearity (or multi-collinearity) is just a fancy way of saying that our
features are correlated. In the worst case, suppose that two of our columns in
the $$\textbf{X}$$ matrix are identical: that is, we have repeated data. Then, bad
things happen: the matrix $$\textbf{X}^T \textbf{X}$$ no longer has full rank (or at
least, becomes
[ill-conditioned](https://en.wikipedia.org/wiki/Condition_number)), which means
the actual inversion becomes an extremely sensitive operation and is liable to
give you nonsensically large or small regression coefficients, which will impact
model performance.

### Too Many Features

Having more data may be a good thing, but more specifically, having more
_observations_ is a good thing. Having more _features_ might not be a great
thing. In the extreme case, if you have more features than observations, (i.e.
$$ n < p $$), then the OLS estimate of $$\textbf{\beta}$$ generally fails to be
unique. In fact, as you add more and more independent features to your model,
you will find that model performance will begin to degrade long before you reach
this point where $$ n < p $$.

## Expanding-Brain Solutions and Practical Considerations

<img style="float: middle" src="http://i1.kym-cdn.com/entries/icons/facebook/000/022/266/brain.jpg">

Here I'll discuss some add-ons and plugins you can use to upgrade your Ordinary
Least Squares Linear Regression™ to cope with the four problems I described
above.

### Heteroskedasticity and Correlated Residuals

To cope with different levels of noise, we can turn to *generalized least
squares* (a.k.a. GLS), which is basically a better version of ordinary least
squares. A little bit of math jargon lets us explain GLS very concisely: instead
of minimizing the _Euclidean norm_ of the residuals, we minimize its
_Mahalanobis norm_: in this way, we take into account the second-moment
structure of the residuals, and allows us to put more weight on the data points
on more valuable data points (i.e. those not impaired by noise or correlation).

Mathematically, the OLS estimate is given by

$$\textbf{\hat{\beta}} = argmin |\textbf{y} - \textbf{X}\textbf{\beta}|^2 $$

whereas the GLS estimate is given by

$$\textbf{\hat{\beta}} = argmin (\textbf{y} - \textbf{X}\textbf{\beta})^T \textbf{\Sigma} (\textbf{y} - \textbf{X}\textbf{\beta})$$

where $$\textbf{\Sigma}$$ is the _known_ covariance matrix of the residuals.

Now, the GLS estimator enjoys a lot of statistical properties: it is unbiased,
consistent, efficient, and asymptotically normal. _Basically, this is very
**very** nice._

In practice though, since $$\Sigma$$ is usually not known, approximate methods
(such as [weighted least
squares](https://en.wikipedia.org/wiki/Least_squares#Weighted_least_squares), or
[feasible generalized least
squares](https://en.wikipedia.org/wiki/Generalized_least_squares#Feasible_generalized_least_squares))
which attempt to estimate the optimal weight for each training point, are used.
One thing that I found interesting while researching this was that these
methods, while they attempt to approximate something better than OLS, may end up
performing _worse_ than OLS! In other words (and more precisely), it's true that
these approximate estimators are _asymptotically_ more efficient, for small or
medium data sets, they can end up being _less_ efficient than OLS. This is why
some authors prefer to just use OLS and find _some other way_ to estimate the
variance of the estimator (where this _some other way_ is, of course, robust to
heteroskedasticity or correlation).

### Outliers

Recall that OLS minimizes the sum of squares:

$$\textbf{\beta} = argmin |\textbf{y} - \textbf{X}\textbf{\beta}|^2 $$

A _regularized estimation_ scheme adds a penalty term on the size of the coefficients:

$$\textbf{\beta} = argmin |\textbf{y} - \textbf{X}\textbf{\beta}|^2 + P(\textbf{\beta}) $$

where $$P$$ is some function of $$\textbf{\beta}$$. Common choices for $$P$$ are:

- $$P(\textbf{\beta}) = ||\textbf{\beta}||_1$$ (i.e. the $$l_1$$ norm)

- $$P(\textbf{\beta}) = ||\textbf{\beta}||_2$$ (i.e. the $$l_2$$ norm)

- $$P(\textbf{\beta}) = a ||\textbf{\beta}||_1 + (1-a) ||\textbf{\beta}||_2$$
  (i.e. interpolating between the $$l_1$$ and $$l_2$$ norms)

While regularizes has empirically been found to be more resilient against
outliers, it comes at a cost: the regression coefficients lose their nice
interpretation of "effect of increasing this regressor by one unit".
Indeed, regularization can be thought of as telling the machine: "I don't care
about interpreting regression coefficients, so long as I get a reasonable fit
that is resilient to overfitting". For this reason, regularization is usually
used for prediction problems, and not for inference.

An alternative solution would be to apply some pre-processing to our data: for
example, some anomaly detection on our data points could remove outliers from
the consideration of our linear regression. However, this method also comes with
its own problems - what if it removes the wrong points? It has the potential to
really mess up our model if it did.

The main takeaway, then, is that _outliers just suck_.

### Collinearity

Collinearity a problem that comes and goes - sometimes it's there, othertimes
not, and it's better to always check and correct for it than it is to risk
having it there.

There are many ways to [detect
multicollinearity](https://en.wikipedia.org/wiki/Multicollinearity#Detection_of_multicollinearity),
many ways to [remedy
it](https://en.wikipedia.org/wiki/Multicollinearity#Remedies_for_multicollinearity)
and [many consequences if you
don't](https://en.wikipedia.org/wiki/Multicollinearity#Consequences_of_multicollinearity).
The Wikipedia page is pretty good at outlining all of those, so I'll just defer
to them.

An alternative that Wikipedia doesn't mention is principal components regression
(PCR), which is literally just principal components analysis followed by
ordinary least squares. As you can imagine, by throwing away some of the
lower-variance components, you can usually remove some of the collinearity.
However, this comes at the cost of interpretability: there is no easy way to
intuit the meaning of a principal component.

A more sophisticated approach would be a close cousin of PCR: [partial least
squares
regression](https://en.wikipedia.org/wiki/Partial_least_squares_regression).
It's a bit more mathematically involved, and I definitely don't have the time to
do it full justice here. Google!

### Too Many Features

Having too many features to choose from sounds like the first-world problem of
data science, but it opens up the whole world of high-dimensional statistics and
feature selection. There are a lot of techniques that are at your disposal to
winnow down the number of features here, but the one that is most related to
linear regression is [least angle
regression](https://en.wikipedia.org/wiki/Least-angle_regression) (a.k.a. LAR or
LARS). It's an iterative process that determines the regression coefficients
according to which features are most correlated with the target, and increases
(or decreases) these regression coefficients until some other feature looks like
it has more explanatory power (i.e. more correlated with the target). Like so
many other concepts in this post, I can't properly do LAR justice in such a
short space, but hopefully the idea was made apparent.