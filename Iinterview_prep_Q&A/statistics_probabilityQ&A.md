# Statistics & Probability — Interview Q&A

---

## 1. What is the difference between descriptive and inferential statistics?

**Descriptive statistics** summarize and describe features of a dataset (mean, median, mode, standard deviation, histograms). They answer "what happened."

**Inferential statistics** use a sample to make conclusions about a larger population. They answer "what can we predict." Techniques include hypothesis testing, confidence intervals, and regression analysis.

**Example:** If you compute the average salary of 500 surveyed employees → descriptive. If you use that sample to estimate the average salary of all 100,000 employees in the company → inferential.

---

## 2. Explain Bayes' Theorem and give a practical example.

Bayes' Theorem describes the probability of an event based on prior knowledge of conditions related to the event:

$$P(A|B) = \frac{P(B|A) \cdot P(A)}{P(B)}$$

- **P(A|B)** = Posterior probability (what we want)
- **P(B|A)** = Likelihood
- **P(A)** = Prior probability
- **P(B)** = Evidence (marginal likelihood)

**Spam filter example:**
- P(Spam) = 0.3 (30% of emails are spam)
- P("free" | Spam) = 0.8 (80% of spam emails contain "free")
- P("free") = 0.35 (35% of all emails contain "free")

$$P(\text{Spam} | \text{"free"}) = \frac{0.8 \times 0.3}{0.35} = 0.686$$

So if an email contains "free," there's a 68.6% chance it's spam.

---

## 3. What is the Central Limit Theorem (CLT)?

The CLT states that the **distribution of sample means** approaches a **normal distribution** as the sample size increases, regardless of the population's original distribution (as long as the population has finite variance).

**Key implications:**
- Works for sample sizes ≥ 30 (rule of thumb)
- The mean of sample means = population mean (μ)
- Standard error = σ / √n
- This is why we can use z-tests and t-tests even when the data isn't normally distributed

**Why it matters:** It's the foundation for confidence intervals and hypothesis testing.

---

## 4. Explain Type I and Type II errors.

| | Null Hypothesis True | Null Hypothesis False |
|---|---|---|
| **Reject H₀** | Type I Error (α) — False Positive | ✅ Correct (Power) |
| **Fail to Reject H₀** | ✅ Correct | Type II Error (β) — False Negative |

- **Type I Error (α):** Rejecting a true null hypothesis. "Crying wolf." Example: Saying a drug works when it doesn't.
- **Type II Error (β):** Failing to reject a false null hypothesis. "Missing the signal." Example: Saying a drug doesn't work when it actually does.
- **Power = 1 - β:** Probability of correctly detecting a real effect.

**Trade-off:** Reducing α (stricter threshold) increases β, and vice versa. You can reduce both by increasing sample size.

---

## 5. What is a p-value? What does p = 0.03 mean?

The **p-value** is the probability of observing results as extreme as (or more extreme than) the observed results, **assuming the null hypothesis is true**.

If p = 0.03:
- There's a 3% chance of seeing this result (or more extreme) if the null hypothesis were true
- At significance level α = 0.05, we reject H₀ (since 0.03 < 0.05)
- At α = 0.01, we fail to reject H₀ (since 0.03 > 0.01)

**Common misconception:** p-value is NOT the probability that H₀ is true. It's the probability of the data given H₀.

---

## 6. What is the difference between a confidence interval and a prediction interval?

**Confidence Interval (CI):** Estimates the range where the **population parameter** (e.g., mean) lies. Narrows with more data.
- "We are 95% confident the true average height is between 170 cm and 175 cm."

**Prediction Interval (PI):** Estimates the range where a **single new observation** will fall. Always wider than CI because it accounts for both parameter uncertainty and individual variation.
- "We are 95% confident the next person's height will be between 155 cm and 190 cm."

---

## 7. Explain the difference between Z-test and T-test.

| Feature | Z-test | T-test |
|---------|--------|--------|
| Population variance | Known | Unknown (estimated from sample) |
| Sample size | Large (n ≥ 30) | Small (n < 30) preferred |
| Distribution | Standard normal | Student's t-distribution |
| Tails | Heavier in t-distribution | Normal |

As sample size grows, the t-distribution converges to the normal distribution, so for large n they give similar results.

**Types of t-tests:**
- **One-sample:** Compare sample mean to a known value
- **Two-sample (independent):** Compare means of two groups
- **Paired:** Compare means of same group at two times

---

## 8. What is A/B testing? Walk through the steps.

A/B testing is a **randomized controlled experiment** to compare two variants (A = control, B = treatment).

**Steps:**
1. **Define hypothesis:** H₀: No difference. H₁: Treatment is better.
2. **Choose metric:** CTR, conversion rate, revenue, etc.
3. **Calculate sample size:** Based on desired power (0.8), significance level (0.05), and minimum detectable effect (MDE).
4. **Randomize:** Split users into control and treatment groups.
5. **Run experiment:** Collect data for sufficient duration.
6. **Analyze:** Compute p-value using appropriate test (z-test for proportions, t-test for means).
7. **Decide:** If p < α, reject H₀ and ship the change.

**Pitfalls:**
- Peeking at results early inflates false positive rate
- Simpson's paradox — segment your analysis
- Network effects — ensure independence between groups
- Novelty/primacy effects — run long enough

---

## 9. What is the Law of Large Numbers?

As the number of trials (sample size) increases, the **sample mean converges to the population mean**.

**Weak LLN:** Convergence in probability.
**Strong LLN:** Almost sure convergence.

**Practical example:** Flip a fair coin 10 times — you might get 70% heads. Flip it 10,000 times — you'll get very close to 50%.

**Difference from CLT:** LLN says the average converges to the true mean. CLT says the distribution of that average becomes normal.

---

## 10. What are common probability distributions and when do you use each?

| Distribution | Type | Use Case |
|---|---|---|
| **Bernoulli** | Discrete | Single yes/no trial (click or not) |
| **Binomial** | Discrete | Number of successes in n trials (10 coin flips) |
| **Poisson** | Discrete | Count of events in fixed time (requests/sec, emails/day) |
| **Uniform** | Continuous | Equal probability in a range (random number generator) |
| **Normal (Gaussian)** | Continuous | Natural phenomena, errors, heights (CLT makes this ubiquitous) |
| **Exponential** | Continuous | Time between events (time between server crashes) |
| **Log-Normal** | Continuous | Positive skewed data (income, stock prices) |
| **Beta** | Continuous | Probabilities / proportions (Bayesian prior for CTR) |

---

## 11. What is correlation vs causation?

**Correlation** measures the linear relationship between two variables (-1 to +1). It does NOT imply one causes the other.

**Causation** means one variable directly affects the other.

**Example:** Ice cream sales and drowning deaths are correlated (both increase in summer). But ice cream doesn't cause drowning — the confounding variable is temperature/season.

**How to establish causation:**
- Randomized controlled experiments (A/B tests)
- Natural experiments
- Instrumental variables
- Causal inference frameworks (do-calculus, DAGs)

---

## 12. What is the difference between Pearson and Spearman correlation?

| Feature | Pearson (r) | Spearman (ρ) |
|---------|------------|---------------|
| Measures | Linear relationship | Monotonic relationship |
| Data type | Continuous, normally distributed | Ordinal or continuous |
| Sensitive to | Outliers (yes) | Outliers (less) |
| Range | -1 to +1 | -1 to +1 |

**Use Pearson** when the relationship is linear and data is normally distributed.
**Use Spearman** when the relationship is monotonic but not necessarily linear, or with ordinal data (rankings).

---

## 13. What is maximum likelihood estimation (MLE)?

MLE finds the parameter values that **maximize the likelihood** of observing the given data.

**Steps:**
1. Write the likelihood function: L(θ) = P(data | θ)
2. Take the log → log-likelihood (easier to work with)
3. Take derivative, set to zero, solve for θ

**Example (coin flip):** Observed 7 heads in 10 flips.  
L(p) = C(10,7) × p⁷ × (1-p)³  
MLE gives p̂ = 7/10 = 0.7

**MLE vs MAP:** MLE uses only data. MAP (Maximum A Posteriori) includes a prior distribution → acts as regularization.

---

## 14. Explain the bias-variance tradeoff in the context of statistics.

**Bias:** Error from wrong assumptions. High bias = underfitting (model too simple).

**Variance:** Error from sensitivity to small fluctuations in training data. High variance = overfitting (model too complex).

$$\text{Expected Error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible Noise}$$

**Statistical context:**
- An estimator with zero bias is called "unbiased" (e.g., sample mean for population mean)
- An estimator with low variance is "efficient"
- Sometimes a slightly biased estimator with much lower variance gives better overall performance (e.g., ridge regression)

---

## 15. What is the Chi-Square test and when do you use it?

The **Chi-Square (χ²) test** checks whether there is a significant association between categorical variables.

**Types:**
1. **Goodness of fit:** Does observed data match an expected distribution? (Is this die fair?)
2. **Test of independence:** Are two categorical variables independent? (Is gender independent of product preference?)

**Formula:**
$$\chi^2 = \sum \frac{(O_i - E_i)^2}{E_i}$$

Where O = observed frequency, E = expected frequency.

**Assumptions:** Sufficiently large sample (expected frequency ≥ 5 in each cell), independent observations.

---

## 16. What is the difference between parametric and non-parametric tests?

| Feature | Parametric | Non-parametric |
|---------|-----------|----------------|
| Assumption | Data follows a known distribution (usually normal) | No distributional assumption |
| Data type | Continuous (interval/ratio) | Any (ordinal, nominal, continuous) |
| Power | Higher (when assumptions met) | Lower |
| Examples | t-test, ANOVA, Pearson r | Mann-Whitney U, Wilcoxon, Kruskal-Wallis, Spearman ρ |

**Rule of thumb:** Use parametric when assumptions are reasonably met (more powerful). Use non-parametric when data is skewed, ordinal, or has small sample sizes.

---

## 17. What is the difference between population and sample? Why do we use n-1 (Bessel's correction)?

**Population:** The entire group you want to study (all users).
**Sample:** A subset of the population (surveyed users).

**Bessel's correction (n-1):** When calculating sample variance, we divide by (n-1) instead of n because:
- Using n underestimates the true population variance (biased estimator)
- The sample mean is calculated from the same data, reducing degrees of freedom by 1
- Dividing by (n-1) gives an unbiased estimate of population variance

---

## 18. What is the ANOVA test?

**ANOVA (Analysis of Variance)** tests whether the means of **three or more groups** are significantly different.

- **One-way ANOVA:** One independent variable (e.g., do 3 diets produce different weight loss?)
- **Two-way ANOVA:** Two independent variables (e.g., diet AND exercise on weight loss)

**How it works:**
- Compares variance between groups vs variance within groups
- F-statistic = (Between-group variance) / (Within-group variance)
- High F → groups are significantly different

**Assumptions:** Normal distribution within groups, equal variances (homoscedasticity), independent observations.

**Post-hoc:** If ANOVA is significant, use Tukey's HSD or Bonferroni correction to find which specific groups differ.

---

## 19. What is conditional probability? How does it differ from joint probability?

**Joint Probability P(A ∩ B):** Probability that both A and B occur.

**Conditional Probability P(A|B):** Probability of A occurring given that B has already occurred.

$$P(A|B) = \frac{P(A \cap B)}{P(B)}$$

**Example:** Deck of cards.
- P(King AND Heart) = 1/52 (joint)
- P(King | Heart) = 1/13 (conditional — given it's a heart, what's the chance it's a king?)

**Independent events:** If A and B are independent, P(A|B) = P(A), and P(A ∩ B) = P(A) × P(B).

---

## 20. What is the expected value and how is it used in decision-making?

**Expected Value (E[X])** is the long-run average outcome of a random variable:

$$E[X] = \sum x_i \cdot P(x_i) \quad \text{(discrete)}$$
$$E[X] = \int x \cdot f(x) \, dx \quad \text{(continuous)}$$

**Decision-making example:** Should we launch feature B?
- P(success) = 0.6, revenue if success = $1M
- P(failure) = 0.4, loss if failure = $500K

E[Value] = (0.6 × $1M) + (0.4 × -$500K) = $600K - $200K = **$400K**

Positive expected value → launch the feature.

**Limitations:** Doesn't account for risk tolerance or variance. A risk-averse company might pass on a positive EV bet with huge downside.
