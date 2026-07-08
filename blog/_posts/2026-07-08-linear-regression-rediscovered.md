---
#layout: post
title: "Linear Regression, Rediscovered: From a Line on Springs to a Bayesian Belief"
description: >
  A ground-up walk through the mathematics of linear regression — least squares, variance partitioning, significance testing, maximum likelihood, and Bayesian regression — built from James V. Stone's "Linear Regression with Python" and reframed through the lens of a working data scientist.
author: author1
comments: true
---


Linear regression is usually the first model anyone learns, and often the last one anyone really *thinks* about. It's easy to fit a line with one call to `sklearn` and move on. But every time I've had to explain *why* a fitted line should be trusted to myself — I've realised how much of the intuition underneath gets skipped.

I recently went through James V. Stone's *Linear Regression with Python*, and rebuilt my notes from scratch as a single coherent story rather than a stack of disconnected formulas. This post is that story, kept at full derivation depth: how a straight line gets fitted, how we decide whether to trust it, and how the whole framework extends into weighted regression, nonlinear regression, and Bayesian regression.

---

## 1. The Model Is Just a Line, and the Noise Is Everything Left Over

$$\hat{y}_i = b_1 x_i + b_0$$

- $$x_i$$: independent variable, known exactly (say, a person's salary)
- $$\hat{y}_i$$: predicted value of the dependent variable (say, predicted height)
- $$y_i$$: the actual observed value
- $$b_1$$: slope — how steeply $$\hat y$$ changes per unit change in $$x$$
- $$b_0$$: intercept — the value of $$\hat y$$ where the line crosses $$x=0$$

The gap between prediction and observation is **noise**:

$$\eta_i = y_i - \hat{y}_i$$

**Why is $$x$$ "known exactly" but $$y$$ noisy?** Because regression is normally used in a setting where the experimenter *controls* or precisely records $$x$$ (the regressor), while $$y$$ is the result of a physical measurement, which always carries some error. That asymmetry is why the line minimizes *vertical* distances only — all the uncertainty is assumed to live in $$y$$.

**The slope, derived from first principles:**

$$\Delta x = x_2 - x_1, \qquad \Delta \hat y = \hat y_2 - \hat y_1 = b_1(x_2-x_1) \;\Rightarrow\; b_1 = \frac{\Delta \hat y}{\Delta x}$$

Nothing more than rise over run — the intuitive gradient of the line.

> **Correlation ≠ Causation.** The fact that a straight line fits $$x$$ and $$y$$ well tells you they *co-vary* — not that $$x$$ causes $$y$$. A classic trap: two variables can move together only because a third, unmeasured variable drives both. Regression is silent about mechanism; it only describes association.

---

## 2. Finding the Best Fitting Line

### 2.1 What "best" means

We want the line that minimizes the **sum of squared vertical errors**:

$$E = \sum_{i=1}^n \big(y_i - (b_1 x_i + b_0)\big)^2$$

The $$(b_1, b_0)$$ that minimizes $$E$$ is the **least squares estimate (LSE)**. This particular flavor — one regressor, squared vertical distance, every point treated as equally reliable — is **simple linear regression**, also commonly called **Ordinary Least Squares (OLS)**, where "ordinary" signals that no per-point weighting is applied (more on that in Section 8).

Why squared error, and not absolute error? Squaring (a) always gives a positive penalty, (b) is differentiable everywhere, which we need for the calculus-based solution below, and (c) penalizes large errors disproportionately more than small ones — which turns out to correspond to a very natural statistical assumption of Gaussian noise (Section 6).

**Mean Squared Error** is just the per-point average of $$E$$:

$$\bar E = \frac{E}{n} \quad \text{(sometimes } n-1 \text{ is used instead — see the degrees-of-freedom discussion in Section 4)}$$

### 2.2 A physical picture: a line suspended on springs

<img src="/assets/blog/2026-07-08/spring_analogy.png" alt="A line suspended on springs, pulled by data points" />

Imagine the line is a rigid rod, and every data point pulls on it vertically through a spring, where each spring's stretch is $$\eta_i$$ and its pull is proportional to that stretch. The rod settles wherever the net pulling force balances out — and that resting position is exactly the least-squares line. This is a nice physical intuition for *why* minimizing squared error feels like "averaging out" every data point's pull simultaneously, rather than fitting any single point exactly.

If some points are more trustworthy than others, give their springs more **stiffness** — a stiffer spring pulls harder and has more influence on where the line settles. That's the entire intuition behind **weighted linear regression** (Section 8): unreliable points get soft springs, reliable points get stiff springs.

### 2.3 Three ways to actually find $$(b_1, b_0)$$

| Method | Idea | Cost |
|---|---|---|
| **Exhaustive search** | Try many $$(b_1, b_0)$$ pairs, compute $$E$$ for each, keep the smallest | Simple but wasteful; doesn't scale |
| **Gradient descent** | Walk downhill on the "error surface" $$E(b_1, b_0)$$ step by step | General-purpose; needed when no closed form exists (most nonlinear models, neural nets) |
| **Calculus (normal equations)** | Solve $$\partial E/\partial b_1 = 0$$ and $$\partial E/\partial b_0 = 0$$ directly | Exact, one-shot, because $$E$$ is a quadratic bowl with a single minimum |

**Gradient descent intuition:** picture $$E$$ as the height of a landscape, and each point on the ground plane as one candidate $$(b_1, b_0)$$. Getting to the valley floor means repeatedly asking "which direction is steepest downhill from here?" and taking a small step that way. The gradient — the vector of partial derivatives — *is* that steepest-descent direction. Because $$E$$ for linear regression is a smooth quadratic bowl with one global minimum and no local traps, this process is guaranteed to converge. In fact, for linear regression we don't even need to take the small steps: calculus lets us jump straight to the bottom.

**The normal equations** — set both partial derivatives to zero. (New notation: $$\bar x = \frac1n\sum_i x_i$$ and $$\bar y = \frac1n\sum_i y_i$$ are simply the sample means of $$x$$ and $$y$$ — every bar accent below means "average of.")

$$\frac{\partial E}{\partial b_0} = -2\sum_i\big(y_i - (b_1x_i+b_0)\big) = 0 \;\Rightarrow\; b_1\bar x + b_0 = \bar y \quad \text{(i)}$$

$$\frac{\partial E}{\partial b_1} = -2\sum_i x_i\big(y_i - (b_1x_i+b_0)\big) = 0 \;\Rightarrow\; b_1\sum_i x_i^2 + b_0 \sum_i x_i = \sum_i x_i y_i \quad \text{(ii)}$$

Two linear equations, two unknowns — solve them simultaneously and something genuinely elegant falls out, which is where Section 3 picks up.

> **Note:** regressing $$y$$ on $$x$$ gives a *different* line than regressing $$x$$ on $$y$$, because the two problems minimize different quantities — vertical distances versus horizontal distances. "Best fitting line" is only meaningful once you've said which variable is being predicted.

---

## 3. The Slope Is Just Covariance in Disguise

- **Variance**: mean squared deviation from the mean, $$\mathrm{var}(y) = \frac1n\sum_i(y_i-\bar y)^2$$. Measures spread. Scaling $$y$$ by $$k$$ scales variance by $$k^2$$; shifting $$y$$ by a constant leaves variance unchanged.
- **Standard deviation**: $$\sqrt{\mathrm{var}(y)}$$ — same units as $$y$$ itself, which is why it's often more interpretable than variance.
- **Covariance**: $$\mathrm{cov}(x,y) = \frac1n\sum_i (x_i-\bar x)(y_i - \bar y)$$ — measures whether $$x$$ and $$y$$ move together (positive), apart (negative), or independently (near zero). Also scale-sensitive: $$\mathrm{cov}(kx, ky) = k^2\,\mathrm{cov}(x,y)$$.
- **Correlation** $$r$$ — covariance, but normalized so magnitude no longer matters. This is the **Pearson correlation coefficient** specifically (there are other correlation measures, like Spearman's, but "correlation" here always means Pearson):

$$r = \frac{\mathrm{cov}(x,y)}{s_x s_y}, \qquad -1 \le r \le 1$$

### Deriving $$b_1 = \mathrm{cov}(x,y)/\mathrm{var}(x)$$ from the normal equations

This is one of my favorite derivations in the whole book, because it turns an abstract optimization problem into a one-sentence intuition. Start from normal equation (i): $$b_1\bar x + b_0 = \bar y \;\Rightarrow\; b_0 = \bar y - b_1\bar x$$.

Substitute this into normal equation (ii), $$b_1\sum_i x_i^2 + b_0\sum_i x_i = \sum_i x_iy_i$$:

$$b_1\sum_i x_i^2 + (\bar y - b_1\bar x)\sum_i x_i = \sum_i x_iy_i$$

Since $$\sum_i x_i = n\bar x$$, the term $$(\bar y - b_1\bar x)\sum_i x_i$$ becomes $$n\bar x\bar y - b_1 n\bar x^2$$. Rearranging to collect all $$b_1$$ terms on one side:

$$b_1\Big(\sum_i x_i^2 - n\bar x^2\Big) = \sum_i x_iy_i - n\bar x\bar y \;\;\Rightarrow\;\; b_1 = \frac{\sum_i x_iy_i - n\bar x\bar y}{\sum_i x_i^2 - n\bar x^2}$$

Now recognize both sides as disguised covariance and variance. Expanding $$\mathrm{cov}(x,y) = \frac1n\sum_i(x_i-\bar x)(y_i-\bar y)$$ term by term gives $$\mathrm{cov}(x,y) = \frac1n\sum_i x_iy_i - \bar x\bar y$$, i.e. $$n\cdot\mathrm{cov}(x,y) = \sum_i x_iy_i - n\bar x\bar y$$ — exactly the numerator above. The identical trick on $$\mathrm{var}(x) = \frac1n\sum_i(x_i-\bar x)^2$$ gives $$n\cdot\mathrm{var}(x) = \sum_i x_i^2 - n\bar x^2$$ — the denominator. The two factors of $$n$$ cancel, leaving:

$$b_1 = \frac{\mathrm{cov}(x,y)}{\mathrm{var}(x)}$$

**The slope is just how much $$x$$ and $$y$$ move together, rescaled by how much $$x$$ moves on its own.** And it connects directly back to correlation: if you standardize both variables (divide each by its own standard deviation), the slope and the correlation become the exact same number, $$b_1 = r$$. Correlation is "slope on a unit-free scale."

---

## 4. How Good Is the Fit, Really? Partitioning the Variance

### 4.1 Signal vs. noise

Each observation's deviation from the overall mean $$\bar y$$ splits cleanly into a part the model explains and a part it doesn't:

$$\underbrace{y_i - \bar y}_{\text{total deviation}} = \underbrace{(\hat y_i - \bar y)}_{\text{signal }\psi_i} + \underbrace{(y_i - \hat y_i)}_{\text{noise }\eta_i}$$

Squaring and summing gives the variance decomposition every regression textbook leans on:

$$SS_T = SS_{Exp} + SS_{Noise}$$

| Symbol | Meaning | Other common names |
|---|---|---|
| $$SS_T$$ | total sum of squares, $$\sum(y_i-\bar y)^2$$ | SST |
| $$SS_{Exp}$$ | explained by the model, $$\sum(\hat y_i - \bar y)^2$$ | SSR ("regression sum of squares") |
| $$SS_{Noise}$$ | left over, unexplained, $$\sum(y_i-\hat y_i)^2$$ | SSE, RSS ("residual sum of squares") |

*(Watch out — SSR/SSE naming conventions are genuinely inconsistent across textbooks and software. Always check which convention a given source is using.)*

### 4.2 $$r^2$$ — the coefficient of determination, and why it's the same $$r^2$$ as correlation

$$r^2 = \frac{SS_{Exp}}{SS_T} = 1 - \frac{SS_{Noise}}{SS_T}$$

Equivalently, $$r^2 = \mathrm{var}(\hat y)/\mathrm{var}(y)$$ — the proportion of total variability in $$y$$ that the line accounts for.

Here's the part that isn't always spelled out, and it's worth the two lines of algebra. Since $$\hat y_i = b_1x_i + b_0$$ is a linear rescaling of $$x_i$$, variance scales predictably under a linear transform ($$\mathrm{var}(aX+c) = a^2\mathrm{var}(X)$$, the shift $$c$$ dropping out entirely):

$$\mathrm{var}(\hat y) = \mathrm{var}(b_1 x + b_0) = b_1^2\,\mathrm{var}(x)$$

So $$\mathrm{var}(\hat y)/\mathrm{var}(y) = b_1^2\,\mathrm{var}(x)/\mathrm{var}(y)$$. Now substitute the key bridge from Section 3, $$b_1 = \mathrm{cov}(x,y)/\mathrm{var}(x)$$:

$$b_1^2\,\mathrm{var}(x) = \frac{\mathrm{cov}(x,y)^2}{\mathrm{var}(x)^2}\cdot\mathrm{var}(x) = \frac{\mathrm{cov}(x,y)^2}{\mathrm{var}(x)}$$

$$\Rightarrow\;\; \frac{\mathrm{var}(\hat y)}{\mathrm{var}(y)} = \frac{\mathrm{cov}(x,y)^2}{\mathrm{var}(x)\,\mathrm{var}(y)} = \left(\frac{\mathrm{cov}(x,y)}{s_x s_y}\right)^{\!2} = r^2$$

which is exactly the squared Pearson correlation from Section 3. "Proportion of variance explained" — a purely variance-partitioning idea — and "squared correlation" — a purely association-strength idea — turn out to be *guaranteed* to be the same number. Not a coincidence: a direct algebraic consequence of $$\hat y$$ being a linear function of $$x$$.

### 4.3 Dropping the intercept (a simplifying trick)

If we **center** the data — subtract $$\bar x$$ from every $$x_i$$ and $$\bar y$$ from every $$y_i$$ — the best-fit line is forced through the origin, so $$b_0 = 0$$ without loss of generality, and $$\bar{\hat y}=0$$ too. This doesn't change the slope, but it strips a lot of unnecessary algebra out of later derivations, including the significance test coming up next.

### 4.4 Degrees of freedom

If a model has $$p$$ free parameters, only $$n-p$$ of the $$n$$ data points are "free to vary" once the model is fit — the rest are pinned down by the constraints the parameters impose. For simple linear regression, $$p=2$$ (slope and intercept), so $$\nu = n-2$$.

This matters because the *sample* variance $$\mathrm{var}(y) = \frac1n\sum(y_i-\bar y)^2$$ systematically **underestimates** the true population variance $$\sigma_y^2$$ — dividing by $$n$$ instead of the degrees of freedom "spends" one data point just estimating $$\bar y$$ itself. The unbiased estimator divides by $$\nu = n-1$$ instead. The same logic — spend one degree of freedom per estimated parameter — is why regression residual variance is typically divided by $$n-2$$, not $$n$$.

**A worked numerical example**, because formulas alone never quite land for me: take five points, $$x = 1,2,3,4,5$$ and $$y = 2,4,5,4,5$$. So $$\bar x = 3$$, $$\bar y = 4$$, and the deviations $$(x_i-\bar x)$$ are $$-2,-1,0,1,2$$ while $$(y_i-\bar y)$$ are $$-2,0,1,0,1$$.

$$\mathrm{cov}(x,y) = \frac{(-2)(-2)+(-1)(0)+(0)(1)+(1)(0)+(2)(1)}{5} = \frac{6}{5} = 1.2, \qquad \mathrm{var}(x) = \frac{4+1+0+1+4}{5} = 2$$

$$b_1 = \frac{1.2}{2} = 0.6, \qquad b_0 = \bar y - b_1\bar x = 4 - 0.6(3) = 2.2$$

So the fitted line is $$\hat y = 0.6x + 2.2$$, giving predictions $$2.8, 3.4, 4.0, 4.6, 5.2$$ and residuals $$-0.8, 0.6, 1.0, -0.6, -0.2$$.

$$SS_T = \sum(y_i-\bar y)^2 = 4+0+1+0+1 = 6, \qquad SS_{Noise} = \sum \eta_i^2 = 0.64+0.36+1.00+0.36+0.04 = 2.4$$

$$SS_{Exp} = 6 - 2.4 = 3.6 \;\Rightarrow\; r^2 = \frac{3.6}{6} = 0.6 \;\Rightarrow\; r = 0.775 \;\text{(positive, since } b_1>0\text{)}$$

Cross-checking against the correlation formula directly: $$\mathrm{var}(y) = 1.2 \Rightarrow s_y = 1.095$$, $$s_x = \sqrt2 = 1.414$$, so $$r = 1.2/(1.414\times1.095) = 0.775$$ — matches exactly, confirming that both routes to $$r^2$$ agree, just as Section 4.2's derivation promised.

---

## 5. Statistical Significance

### 5.1 Warm-up: significance of a single mean

Think of your dataset as one sample drawn from an infinite population. If you repeated the sampling many times, the sample means $$\bar y_1, \bar y_2, \dots$$ would cluster around the true population mean $$\mu$$ (the Law of Large Numbers), and — thanks to the **Central Limit Theorem** — that cluster of sample means is approximately Gaussian, *regardless of the shape of the underlying population*. This is what licenses everything that follows.

- **z-score**: $$z = (\bar y - \mu)/\sigma_{\bar y}$$, where $$\sigma_{\bar y} = \sigma_y/\sqrt n$$ is the standard error of the mean.
- 95% of a standard normal distribution's mass lies within $$\pm 1.96$$ — this is where the familiar $$p < 0.05$$ threshold comes from.
- **z-test**: valid if the population standard deviation $$\sigma_y$$ is genuinely known (rare in practice).
- **t-test**: the realistic case — $$\sigma_y$$ is unknown, so we substitute its unbiased estimate $$\hat\sigma_y$$. That substitution adds extra uncertainty, so the resulting distribution isn't quite Gaussian — it's a **t-distribution**, with heavier tails that approach Gaussian as $$\nu = n-1$$ grows.
- **One-tailed vs two-tailed**: if a variable is physically constrained (say, it can't be negative), only one direction of the alternative hypothesis makes sense, which changes the critical value used.
- **Significance ≠ importance.** A tiny, practically meaningless true effect ($$\mu \approx 0.001$$) can still produce a "highly significant" p-value if $$n$$ is large enough. Statistical significance answers *"is this probably not zero?"* — not *"does this matter?"*

**Why is noise assumed Gaussian in the first place?** Real-world noise is usually the sum of many small, independent contributing factors. The Central Limit Theorem guarantees that any sum, or weighted mean, of many independent random quantities tends toward a Gaussian distribution, almost regardless of each individual contributor's own distribution. That's the real justification — not an arbitrary convenience assumption.

### 5.2 Significance of the regression itself

Two separate questions matter: does the model as a whole explain a real, non-chance proportion of variance, and is this specific slope meaningfully different from zero?

**Testing the slope** — null hypothesis $$H_0: b_1 = 0$$, a horizontal line, no real relationship:

$$t_{b_1} = \frac{b_1 - 0}{\hat\sigma_{b_1}}, \qquad \hat\sigma_{b_1} \approx \frac{s_\eta}{s_x}$$

Since $$s_x$$ is fixed ($$x$$ is known exactly), the uncertainty in the estimated slope is driven entirely by how noisy the residuals are. This can equivalently be expressed purely in terms of the correlation:

$$t_{b_1} = \frac{r\sqrt{n-2}}{\sqrt{1-r^2}}$$

which is a genuinely satisfying result: **the significance of a regression slope is fully determined by how strong the correlation is and how much data you have.** More data or a stronger correlation pushes $$t$$ up and the p-value down.

Running this on the worked example from Section 4.4: with $$\nu = n-2 = 3$$,

$$t_{b_1} = \frac{0.775\sqrt3}{\sqrt{0.4}} = \frac{1.342}{0.632} \approx 2.12$$

The critical two-tailed t-value at $$p=0.05$$ with $$\nu=3$$ is about $$3.18$$, so this particular slope is **not** statistically significant — a nice, honest illustration that small samples make it hard to distinguish a real effect from noise, even when $$r^2=0.6$$ looks respectable on its own.

**Testing the overall fit — the F-test:**

$$F = \frac{\text{proportion of variance explained}}{\text{proportion of variance unexplained}} = \frac{r^2}{1-r^2} \quad \text{(scaled by degrees of freedom in practice)}$$

For simple linear regression with one predictor, the t-test on the slope and the F-test on the model are mathematically equivalent ($$F = t^2$$) — the F-test only earns its keep once you have multiple regressors and want to test them jointly (Section 7).

---

## 6. Maximum Likelihood Estimation — Why Least Squares "Just Works"

Instead of asking "which line minimizes squared error," MLE asks a philosophically different question: **which parameter values make the data I actually observed most probable?**

Assume the noise on each point is Gaussian, mean zero, with its own variance $$\sigma_i^2$$:

$$p(\eta_i) = \frac{1}{\sqrt{2\pi\sigma_i^2}} \, e^{-\eta_i^2/2\sigma_i^2}$$

<img src="/assets/blog/2026-07-08/mle_noise.png" alt="Each data point as a draw from its own Gaussian distribution around the line" />

Picture a little bell curve straddling the regression line at every $$x_i$$ — wide where $$\sigma_i$$ is large (unreliable point), narrow where $$\sigma_i$$ is small (reliable point). The observed $$y_i$$ is a draw from that local bell curve. Because $$b_1$$, $$b_0$$, and $$x_i$$ are fixed given the model, $$p(\eta_i) = p(y_i)$$: the probability of observing a value $$y_i$$ that contains noise $$\eta_i$$ varies as a Gaussian function of that noise.

Assuming each point's noise is independent of the others, the joint probability — the **likelihood** — of the whole dataset is the product of each point's individual probability:

$$p(y_1,\dots,y_n \mid b_1,b_0) = \prod_i p(y_i \mid b_1, b_0)$$

Taking logs turns that product into a sum, which is both numerically nicer and preserves the location of the maximum since the log is monotonic:

$$\log p(y\mid b_1,b_0) = \sum_i \log\frac{1}{\sigma_i\sqrt{2\pi}} \;-\; \frac12\sum_i \frac{(y_i - (b_1x_i+b_0))^2}{\sigma_i^2}$$

**The punchline:** maximizing this log-likelihood is *the same optimization problem* as minimizing a variance-weighted sum of squared errors. So:

- **If every point is equally reliable** ($$\sigma_i$$ constant for all $$i$$) → MLE and ordinary LSE give *identical* answers.
- **If reliability differs across points** → MLE naturally reduces to weighted least squares (Section 8), because each squared error term is automatically discounted by its own noise variance.

This is the deep reason least squares isn't an arbitrary convenience — it's the maximum-likelihood solution under the very common, CLT-justified assumption of Gaussian noise.

---

## 7. Multivariate Regression — From Line to Plane (and Beyond)

$$y_i = b_1 x_{i1} + b_2 x_{i2} + \dots + b_0 + \eta_i$$

Same idea as simple regression, one dimension higher: instead of fitting a line to 2-D data, we fit a **plane**, or hyperplane for 3+ regressors, minimizing the same sum of squared vertical distances. Each coefficient $$b_j$$ is the partial slope in the direction of its own regressor: $$b_j = \partial \hat y/\partial x_j$$.

**Vector-matrix formulation** — the notational upgrade that lets everything generalize cleanly beyond two regressors:

$$\mathbf b = (b_1, b_2, b_0), \quad \mathbf x_i = (x_{i1}, x_{i2}, 1), \quad \hat y_i = \mathbf x_i \mathbf b$$

$$E = (\mathbf y - X\mathbf b)^T(\mathbf y - X\mathbf b)$$

Differentiating and setting the gradient vector to zero:

$$\nabla E = 2\big[(X^TX)\mathbf b - X^T\mathbf y\big] = \mathbf 0 \;\;\Rightarrow\;\; \mathbf b = (X^TX)^{-1}X^T\mathbf y$$

And plugging back in gives the elegant **hat matrix** form:

$$\hat{\mathbf y} = X(X^TX)^{-1}X^T\mathbf y = H\mathbf y$$

$$H$$ is called the "hat matrix" because it puts the hat on $$y$$ — a single linear operator mapping raw observations directly onto their fitted values.

### 7.1 Standardized coefficients

Raw coefficients $$b_1$$ vs $$b_2$$ **cannot** be compared directly to judge which regressor matters more — halving all values of $$x_1$$ doubles $$b_1$$ without $$x_1$$ becoming any less important. The fix: standardize every variable to have standard deviation 1 first ($$x_1' = x_1/s_{x_1}$$, etc.), then compare the resulting coefficients on equal footing.

### 7.2 Multicollinearity

Multivariate regression implicitly assumes each regressor's contribution can simply be *added* to the others'. If two regressors are themselves correlated, this double-counts their shared contribution, and the individual coefficient estimates become unstable and hard to interpret — even though the model as a whole may fit fine. Multicollinearity is the name for this problem, and it's why you should always check correlations among your predictors before trusting individual coefficient interpretations.

### 7.3 How many regressors? — the Extra Sum-of-Squares method

Recall $$SS_{Exp} = \sum_i(\hat y_i - \bar y)^2$$ — the sum of squares explained by whichever model produced $$\hat y_i$$. Adding more regressors mechanically increases $$SS_{Exp}$$ — you can never explain *less* variance with more predictors — so $$SS_{Exp}$$ alone can't tell you whether a new regressor is worth adding. Instead, compare a **full model** against a **reduced model** with one regressor removed:

$$SS_{Exp}(b_2 \mid \text{reduced}) = SS_{Exp}(\text{full}) - SS_{Exp}(\text{reduced})$$

This "extra" explained variance is converted into an F-statistic and tested against $$H_0: b_2 = 0$$ — removing $$x_2$$ doesn't hurt the fit. This is the workhorse tool for deciding whether a given predictor earns its place in the model, and it comes back below for polynomial terms too.

---

## 8. Weighted Linear Regression

**Ordinary Least Squares (OLS)** — the plain LSE fitting from Section 2, where every point contributes to $$E$$ with the same weight — implicitly assumes every data point is equally trustworthy. **Weighted regression** relaxes that: each point's error is discounted by its own noise variance $$\sigma_i^2$$, which — as shown in Section 6 — falls directly out of the MLE derivation once $$\sigma_i$$ is allowed to vary:

$$E = \sum_i \frac{\big(y_i - (b_1x_i+b_0)\big)^2}{\sigma_i^2}$$

Minimizing this gives the **weighted least squares estimate (WLSE)**. Points with small $$\sigma_i$$ — reliable, low-noise measurements — pull the line toward them much more strongly than points with large $$\sigma_i$$.

<img src="/assets/blog/2026-07-08/wls_vs_ols.png" alt="Comparison of OLS versus weighted least squares fitting" />

A 2-D scatter with a few points marked as high confidence (short error bars) and a few as low confidence (long error bars). The OLS line treats them all equally; the WLS line visibly bends toward the tight, low-variance points and is comparatively indifferent to the noisy, outlier-prone ones.

---

## 9. Nonlinear Regression

Needed when the underlying physical process is known, or suspected, not to be a straight line — exponential decay or growth, for instance, $$\hat y = b_0 e^{b_1 x}$$. The general form is $$\hat y_i = f(x_i, \mathbf b)$$.

### 9.1 Polynomial regression is secretly linear

$$\hat y_i = b_0 + b_1 x_i + b_2 x_i^2$$

Subtle but important: even though the *curve* is nonlinear in $$x$$, the model is **linear in its parameters** ($$b_0, b_1, b_2$$) — so this is really just multivariate regression in disguise, treating $$x_i^2$$ as a second regressor.

**Danger of overfitting:** a first-order polynomial, a straight line, will almost never have zero error; a high-enough-order polynomial can be made to pass through *every* data point exactly, driving $$SS_{Noise}$$ to almost zero. That means $$SS_{Noise}$$ alone is **not** a valid measure of model quality — a perfect-looking fit might just be memorizing noise. This is exactly why the Extra Sum-of-Squares method from Section 7.3 exists: it tests whether adding the $$x^2$$ term earns a statistically real improvement in fit, rather than an inevitable mechanical one.

**Multicollinearity caveat:** $$x$$ and $$x^2$$ are inherently correlated, so — as in Section 7.2 — individual coefficient significance becomes hard to interpret cleanly, even though the overall model fit can still be assessed.

### 9.2 True nonlinear regression

If the function truly can't be reduced to a polynomial:

- **Linearizable case**: transform it into something linear. For example $$\hat y = e^{b_1 x} \Rightarrow \log \hat y = b_1 x$$ — fit a straight line to $$\log y$$ vs. $$x$$ instead.
- **Non-linearizable case**, two options remain: approximate the function locally with a **Taylor/Maclaurin expansion**, e.g. $$e^{b_1 x} \approx 1 + b_1x + \frac{b_1^2x^2}{2!} + \frac{b_1^3x^3}{3!}$$; or fall back to **brute-force search or gradient descent** — exhaustive search if there are very few parameters, gradient descent once the parameter space gets large. This is, not coincidentally, the same core algorithm underlying modern neural network training.

---

## 10. A Different Worldview: Bayesian Regression

Everything from Section 2 through Section 9 is **frequentist**: we find one "best" pair $$(b_1, b_0)$$ — a single point estimate — and then separately ask how confident we should be in it, via t-tests, p-values, standard errors. **Bayesian regression** flips this entirely: instead of estimating one best line, it treats $$b_1$$ and $$b_0$$ themselves as *random variables* with their own probability distributions, and the goal becomes computing the full distribution of plausible lines given the data.

$$p(b_1, b_0 \mid y) \;\propto\; p(y \mid b_1, b_0)\; p(b_1, b_0)$$

This is Bayes' rule applied to the regression parameters:

- $$p(y \mid b_1, b_0)$$ — the **likelihood**, exactly the same term MLE maximizes in Section 6. It says how probable the observed data is, for any given choice of $$b_1, b_0$$.
- $$p(b_1, b_0)$$ — the **prior**: what we believed about plausible slopes and intercepts *before* seeing this data.
- $$p(b_1, b_0 \mid y)$$ — the **posterior**: the updated belief *after* seeing the data — a full probability distribution, not a single number.

### How the difference from MLE actually plays out

MLE asks: which single $$(b_1, b_0)$$ makes the observed data most probable? It maximizes $$p(y\mid b_1, b_0)$$ alone, and the prior never enters the calculation. Bayesian regression asks: given what I believed beforehand, and what the data now shows, what is the full range of $$(b_1, b_0)$$ values I should now believe in, and how strongly? It works with the whole posterior distribution, not just its peak.

Here's the connection that ties this whole post together: if you deliberately choose a **flat, uninformative prior** — treating every possible $$(b_1,b_0)$$ as equally plausible beforehand — the posterior becomes proportional to the likelihood alone, and the peak of the Bayesian posterior collapses *exactly* onto the MLE/OLS point estimate. **MLE is the special case of Bayesian inference where the prior contributes no information.**

### How the prior actually gets injected

The prior is a genuine design choice, not something derived from the data at hand:

- **Weak/uninformative prior** — a very wide Gaussian centered at zero. Says "I have almost no strong opinion about the slope, but implausibly huge slopes are a bit less likely than modest ones." With enough data, the likelihood dominates and this barely matters.
- **Informative prior** — "from past studies, I expect the slope to be around 2.0, plausibly anywhere from 1.5 to 2.5." This is exactly how domain knowledge — a previous experiment, a physical constraint, expert judgment — gets formally folded into the model, *before* a single new data point is observed.

Mechanically, combining a Gaussian likelihood with a Gaussian prior (the common textbook case) yields a Gaussian posterior — the prior behaves like a small number of extra "pseudo-observations" pulling the estimate toward the prior's mean, with an effect that shrinks as real data accumulates. **A prior behaves like data you already had before the experiment started.**

### Two genuinely different worldviews

| | Frequentist | Bayesian |
|---|---|---|
| What is $$b_1$$? | A fixed, unknown true number | A random variable with its own distribution |
| What you compute | A single point estimate + a p-value/confidence interval | A full posterior distribution |
| "95% interval" means | If you repeated the sampling process many times, 95% of such intervals would contain the true $$b_1$$ | There is a 95% probability the true $$b_1$$ lies in this specific interval, given this data and this prior |
| Role of prior belief | None — only this dataset's likelihood counts | Explicit and required — must be stated up front |
| When data is scarce | Estimates can be unstable, wide confidence intervals | Prior can stabilize estimates, at the cost of a subjective assumption |
| When data is abundant | — | Converges toward the frequentist MLE answer as the prior's influence washes out |

Neither view is "more correct" — they answer subtly different questions. The frequentist p-value tells you about the *procedure's* long-run reliability; the Bayesian posterior tells you directly about *this specific* estimate's plausibility, at the cost of requiring you to commit to a prior up front.

---

## Tying It All Together

<img src="/assets/blog/2026-07-08/entity_map.png" alt="Concept map connecting all major ideas in linear regression" />

Every piece above connects: the model produces parameters via a fitting method, those parameters get judged through variance partitioning and significance testing, and the same core idea extends outward into multivariate, weighted, nonlinear, and Bayesian regression.

And because "is this model actually any good" is a question I ask myself on almost every project, here's the practical checklist I now run through every time I fit one:

<img src="/assets/blog/2026-07-08/assessment_flow.png" alt="Flow chart for assessing whether a fitted regression model is useful" />

---

## Closing Thought

None of this is new mathematics — Stone's book, and the century of statistics behind it, has all of this worked out precisely. What changed for me going through it slowly was realizing how much of "just fit a line" actually rests on a chain of assumptions — Gaussian noise, equal reliability, linearity — that are worth checking explicitly rather than taking for granted, especially once a model moves from a notebook into something a stakeholder will actually act on.

If nothing else, I'd recommend this exercise to anyone who uses regression regularly: pick the model you use most often, and try to derive it from scratch. The gaps in your own explanation are usually exactly where the interesting intuition is hiding.