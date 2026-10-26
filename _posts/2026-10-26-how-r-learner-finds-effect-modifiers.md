---
layout: post
blog_section: technical
title: "Why subtracting predictions reveals treatment-effect modifiers: an interactive R-learner"
date: 2026-10-26
tags: [causal-inference, R-learner, treatment-effect-heterogeneity, statistics, visualization]
author: Xiang Meng
description: "Follow the same simulated patients from potential outcomes through noisy observed outcomes, marginal prediction, residualization, and the R-learner interaction regression."
---

[← Technical posts]({{ "/blog/" | relative_url }}#technical)

*When does a treatment help one kind of patient more than another? A visual explanation of residualization, from the clinical intuition to the mathematics.*

## Part I — The nontechnical story

Suppose we want to know whether a patient's characteristic, **X**, changes how much they benefit from treatment. It might represent a biomarker level, a measure of disease severity, or another feature available before treatment.

Imagine a world where we could see *two expected outcomes* for each patient profile: one under treatment and one without it. The gap between those curves would tell us the treatment effect. If the gap gets bigger or smaller as X changes, **X is an effect modifier**.

In real data, however, each patient receives only one of the two treatment options, and their observed outcome is noisy. So how can we recover that changing gap?

The R-learner of [Nie and Wager (2021)](https://doi.org/10.1093/biomet/asaa076) uses an elegant idea: **remove what we can predict before asking what treatment changes.** First, we predict outcomes from X alone, temporarily ignoring treatment assignment. Next, we subtract that prediction from each patient's observed outcome. We also subtract the probability of treatment from the treatment indicator. The remaining association can reveal the treatment effect.

In this interactive illustration, X takes three representative values, **−1, 0, and +1**. Treatment is randomized with 50% probability, and the true treatment effect is deliberately set to zero at X = 0. We use continuous outcomes so that the regression geometry is easy to see.

**Explore the six steps below.** Click *Next stage* or *Play story* to watch the same patient dots move. Change **β₁** to control how strongly treatment effects vary across patients, or change the noise to see what makes the relationship harder to estimate.

<div style="margin: 1.4rem 0;">
  <iframe title="Interactive R-learner visual derivation" src="{{ '/assets/interactive/r-learner/index.html' | relative_url }}" width="100%" height="880" loading="lazy" style="border:0;border-radius:12px;background:#0c1727;display:block;" allowfullscreen></iframe>
  <p style="font-size:0.92em;margin-top:0.5rem;"><a href="{{ '/assets/interactive/r-learner/index.html' | relative_url }}" target="_blank" rel="noopener">Open the visualization in a full browser window</a> (recommended on narrow screens).</p>
</div>

### What happened to the dots?

In the first two stages, the curves show the two potential conditional means, and each dot is a patient whose *single observed outcome* includes random noise. Stage 3 fits a purple line, **m(X)**, using patient characteristics without treatment assignment. In stage 4, each dot moves vertically as we subtract that prediction: we have removed predictable baseline outcome variation, but the treatment-related signal remains.

In stage 5, the horizontal axis changes to **observed treatment minus predicted treatment probability**. At the default slider settings, the gold, blue, and pink patient groups display different slopes: treatment is harmful at X = −1, neutral at X = 0, and beneficial at X = +1 in this illustrative outcome convention, where a larger outcome is better. (For adverse-event outcomes, the clinical direction reverses.)

Finally, multiply that horizontal axis by X. The three groups now line up around **one common slope**. This slope measures **how much the treatment effect changes for a one-unit change in X**. That is the effect-modification coefficient.

> **Main intuition:** After removing outcome variation explained by X and treatment allocation explained by X, what remains lets us learn *how treatment response changes with X* rather than separately estimating two complicated outcome curves.

The shapes are a teaching device, not real patients or a claim that a particular biomarker modifies a real treatment effect. The common-slope picture uses a specially simplified model in which the treatment-effect intercept is zero.

---

## Part II — The technical derivation

### 1. Start with conditional means, not an additive modeling assumption

For a binary treatment A in {0, 1}, write

```text
μ₀(X) = E[Y | X, A=0]
μ₁(X) = E[Y | X, A=1]
τ(X)  = μ₁(X) − μ₀(X)
```

Regardless of the functional complexity of the conditional means, the binary-treatment decomposition is exact:

```text
E[Y | X,A] = μ₀(X) + A τ(X).
```

In a randomized experiment, τ(X) can be interpreted causally. In observational data, that interpretation additionally requires consistency, conditional exchangeability, and overlap, with appropriate definitions of treatment and outcome.

Let the treatment propensity and pooled conditional mean be

```text
e(X) = P(A=1 | X)
m(X) = E[Y | X] = μ₀(X) + e(X)τ(X).
```

With `ε = Y − E[Y | X,A]`, this gives the identity

```text
Y − m(X) = [A − e(X)] τ(X) + ε,
E[ε | X,A] = 0.
```

**The residual from regressing Y on X is not just noise**. It contains a centered treatment signal. The R-learner is designed to extract that signal.

### 2. Why the first regression deliberately ignores treatment

Fitting m(X) from Y and X alone averages over the treatment patterns at each X. That is intentional. We are not treating m(X) as the untreated outcome model: in general, it differs from both μ₀(X) and μ₁(X).

In the interactive simulation, treatment is randomized at probability 0.5, so m(X) is the midpoint of the two expected-outcome curves. The figure fits m̂(X) from pooled **observed** outcomes: it does not peek at unobserved counterfactual outcomes. To make the visual story transparent, it uses within-X sample means and a quadratic interpolation. For real analyses, nuisance models may be learned flexibly and **cross-fitted** on separate folds.

### 3. Why multiplying by X reveals effect modification

Now impose a *working model on the treatment effect*, not on the whole outcome surface:

```text
τ(X) = β₀ + β₁X.
```

The R-learner second-stage equation becomes

```text
Y − m(X) = β₀[A−e(X)] + β₁X[A−e(X)] + ε.
```

The simulation makes **β₀ = 0** and **β₁ = 0.80**, so that

```text
R = Y−m(X)          (outcome residual)
Z = A−e(X)          (treatment residual)
W = X[A−e(X)] = XZ  (interaction regressor)

R = β₁W + ε.
```

At a fixed X, plotting R versus Z gives a line with slope τ(X) = β₁X. For X = −1, 0, +1, these slopes are −0.80, 0, +0.80. Replacing the horizontal coordinate Z with W = XZ collapses the three slopes onto one. The teal line is **fitted through the origin** using the actual simulated residuals, not drawn by hand.

The point estimate used by the teal line is

```text
β̂₁ = Σᵢ Wᵢ [Yᵢ − m̂(Xᵢ)] / Σᵢ Wᵢ².
```

At the initial settings, the true β₁ is **0.80** and the fitted estimate is approximately **0.816**; the difference comes from random outcome noise and finite-sample fitting. The horizontal jitter seen in the dots is only for display and never enters this regression.

### 4. What changes if β₀ is not zero?

In a general application, β₀ should not be fixed to zero. The correct second stage includes both centered treatment and its interaction:

```r
R <- Y - m_hat   # preferably out-of-fold predictions
Z <- A - e_hat
fit <- lm(R ~ 0 + Z + I(Z * X))
coef(fit)
```

There is **no free regression intercept**, but the coefficient on Z estimates β₀. The coefficient on `I(Z * X)` estimates the treatment-effect modifier β₁. Merely regressing R on XZ alone would generally confound the β₁ term with β₀Z.

With estimated e(X) and m(X), estimation errors matter. Orthogonalization reduces first-order sensitivity to nuisance-estimation error; it does **not** make every flexible model automatically unbiased. Cross-fitting, overlap, nuisance convergence, correct targeting, and suitable regularity conditions are important for inference.

### 5. How this relates to binary outcomes and DINA

For a binary outcome, the R-learner's linear-outcome visualization is an intuition rather than the actual DINA fitting procedure. DINA (the difference in natural parameters), introduced by [Gao and Hastie (2025)](https://doi.org/10.1093/biomtc/ujaf162), models the **natural-parameter contrast**, the conditional log-odds ratio, and fits a Bernoulli logistic loss with an offset and a variance-adjusted treatment-centering term. It does **not** regress observed outcome log-odds residuals by least squares.

In the sparse extension used for variable screening, one can write the treatment log-odds ratio as `f(X)=α+Xβ` and penalize the modification coefficients. A nonzero coefficient nominates a *candidate* log-OR modifier; it does not confirm causal effect modification or directly produce a risk-difference estimate.

*To be written: an intuitive explanation of DINA and how it extends the residualization idea illustrated here.*

### References

- Nie, X., and Wager, S. (2021). [Quasi-oracle estimation of heterogeneous treatment effects](https://doi.org/10.1093/biomet/asaa076). *Biometrika*, 108(2), 299–319.
- Gao, Z., and Hastie, T. (2025). [Estimating heterogeneous treatment effects for general responses](https://doi.org/10.1093/biomtc/ujaf162). *Biometrics*, 81(4), ujaf162.

---

*Illustration notes:* All outcomes are simulated; X takes three exact values (small horizontal jitter is added for readability); A is randomized with balanced assignment at each X; the displayed curves and simulated errors are generated from a known model. Changing β₁ or noise recomputes observed outcomes, the fitted marginal model, the residuals, and the final estimated slope consistently.

*Authorship note:* This blog post was generated by ChatGPT and supervised by me (Xiang Meng) through a few iterations.
