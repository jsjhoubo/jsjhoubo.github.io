# A/B Test Hypothesis Testing and Statistical Power: A First-Principles Derivation

A derivation note that preserves the continuity of thought

## Writing Principle

This is not a lecture reverse-engineered from "standard formulas," but a note organized in the order **"problem statement → natural construction → hitting a wall → clarification → convergence."** Many results ($SE=\sqrt{\sigma^2/n}$, $\text{power}=\Phi(\delta/SE - z_{\alpha/2})$, the sample-size formula) are **derived** in the text, not memorized. Several "forks" from today's back-and-forth (is $\alpha$ equal to $P(H_0)$? why must $H_0$ be an equality? the scale/units of $Z$) are deliberately preserved, because ambiguity is exactly what comes out of these places — truly understanding a concept often means understanding clearly what it is **not**.

---

## Table of Contents

1. Starting point: what is an A/B test actually asking
2. Starting from the mean difference: CLT and normal approximation
3. A key distinction: variance of the data vs. variance of the mean (deriving $\sigma^2/n$)
4. Constructing the test statistic $Z$ and the standard error $SE$
5. Fork one: why must $H_0$ be an equality, not $\mu_0 \neq \mu_1$
6. Fork two: $\alpha$ is a conditional probability, not $P(H_0\text{ is true})$
7. Where the critical value 1.96 comes from: forward derivation vs. "reverse-engineering"
8. The soul of frequentism: $\alpha$ is a long-run frequency, not the probability of a single trial
9. p-hacking: how a single-trial guarantee gets broken by repetition
10. Forward derivation of power: where does $Z$ shift under $H_1$
11. How to track the shift: treat $SE$ as a measured value and subtract it directly from $Z$
12. Solving for sample size $n$ given power
13. Grounding in the click-through-rate (Bernoulli) case
14. Frequentist vs. Bayesian: not depth, but paradigm
15. Interview soundbites summary
16. What this derivation actually trained

---

## 1. Starting point: what is an A/B test actually asking

We have a control arm and a test arm, and we measure some metric (click-through rate, time-per-user, etc.). The core question is:

> Is there **really a difference** between the two groups, or is the difference I observed just random noise?

Samples are drawn at random, so even if the two groups are truly identical, the observed sample metrics will almost never be exactly equal. The question therefore becomes: **how large must the observed difference be before it cannot be explained by "pure noise"?**

Start with the most natural handle: compare the two groups' **means**. Let the control's true mean be $\mu_0$ and the test's be $\mu_1$. We want to decide whether $\mu_0$ and $\mu_1$ are equal.

---

## 2. Starting from the mean difference: CLT and normal approximation

The distribution of a single sample $x_i$ can be anything (a click is 0/1, dwell time is heavy-tailed). But what we want is the **sample mean** $\bar{x} = \frac{1}{n}\sum x_i$.

By the **Central Limit Theorem (CLT)**: as long as $n$ is large enough, the sample mean $\bar{x}$ is approximately normal, whatever the original distribution. So:

- $\bar{x}_0 \approx N(\mu_0,\ \text{Var}(\bar{x}_0))$
- $\bar{x}_1 \approx N(\mu_1,\ \text{Var}(\bar{x}_1))$

The **difference** of two independent normals is still normal, with **variances adding**:

$$\bar{x}_0 - \bar{x}_1 \approx N\big(\mu_0-\mu_1,\ \text{Var}(\bar{x}_0)+\text{Var}(\bar{x}_1)\big)$$

> ⚠️ Note **variances** add, not standard deviations. This is the first easy mistake — later when building the standard error, the denominator is "sum of variances, then square-root," not "standard deviations added directly."

---

## 3. A key distinction: variance of the data vs. variance of the mean

This section is the bedrock of the whole note, and the layer most easily confused. **"Variance of a single data point" and "variance of the sample mean" differ by a factor of $n$.**

- **Single-point variance** $\sigma^2$: the $\sigma^2$ of the original distribution $N(\mu,\sigma^2)$, estimated by the sample variance $\hat\sigma^2 = \frac{1}{n}\sum(x_i-\bar x)^2$. Meaning: "how spread out the data itself is."
- **Variance of the mean** $\text{Var}(\bar x)$: meaning "how imprecise the mean I estimated from the sample is."

Deriving $\text{Var}(\bar x)$ from first principles:

$$\text{Var}(\bar x) = \text{Var}\!\left(\frac{\sum_i x_i}{n}\right) = \frac{1}{n^2}\sum_i \text{Var}(x_i) = \frac{1}{n^2}\cdot n\sigma^2 = \frac{\sigma^2}{n}$$

Two steps are key: $\frac{1}{n^2}$ pulled out of the square of a constant; $\sum \text{Var}(x_i) = n\sigma^2$ ($n$ independent identically-varianced terms added).

$$\boxed{\text{Var}(\bar x) = \frac{\sigma^2}{n}}$$

**Intuition**: more samples make the mean more stable, hence divide by $n$. This $n$ is the core knob of power later — it is the mathematical origin of "collecting more data increases the test's sensitivity."

> **Easy mistake (preserved)**: a natural but wrong version writes the variance of the mean difference as $\frac{\sum(x_{0i}-\bar x_0)^2}{n_0} + \frac{\sum(x_{1i}-\bar x_1)^2}{n_1}$. Unpacking it, $\frac{\sum(x_i-\bar x)^2}{n}$ is exactly the **estimate of the single-point variance** $\hat\sigma^2$, not the variance of the mean — the variance of the mean still needs one more division by $n$, i.e. $\frac{\hat\sigma^2}{n} = \frac{\sum(x_i-\bar x)^2}{n^2}$. **The denominator should be $n^2$, not $n$.** Missing that layer of $n$ means treating "variance of the data" as "variance of the mean."

---

## 4. Constructing the test statistic $Z$ and the standard error $SE$

Putting the above together, the variance of the mean difference:

$$\text{Var}(\bar x_0 - \bar x_1) = \frac{\sigma_0^2}{n_0} + \frac{\sigma_1^2}{n_1}$$

Define the **Standard Error (SE)** as its standard deviation:

$$SE = \sqrt{\frac{\sigma_0^2}{n_0} + \frac{\sigma_1^2}{n_1}}$$

Then construct the **standardized statistic**:

$$Z = \frac{\bar x_0 - \bar x_1}{SE} = \frac{\bar x_0 - \bar x_1}{\sqrt{\sigma_0^2/n_0 + \sigma_1^2/n_1}}$$

**When $H_0$ ($\mu_0=\mu_1$) is true, $Z \sim N(0,1)$.**

> $Z$ is **dimensionless**: numerator and denominator both carry the metric's units, which cancel. This matters in Section 11 — much confusion comes from forgetting that $Z$ has already been divided by $SE$ and is dimensionless.

**Hypothesis-testing framework:**
- $H_0: \mu_0 = \mu_1$ (no difference)
- $H_1: \mu_0 \neq \mu_1$ (a difference exists)
- two-sided $\alpha=0.05$, critical value $z_{\alpha/2}=1.96$
- decision: $|Z| > 1.96$ → reject $H_0$

---

## 5. Fork one: why must $H_0$ be an equality, not $\mu_0 \neq \mu_1$

A very natural question: since we want to prove "there is a difference," why not set $H_0$ directly to $\mu_0 \neq \mu_1$?

**Because $H_0$ must be an assumption from which you can compute a definite distribution for the statistic, and $\neq$ cannot.**

- $\mu_0 = \mu_1$ (equality) → a **single definite point** (difference = 0) → $Z$ has a **unique definite distribution** $N(0,1)$ → you can set the critical value and control $\alpha$.
- $\mu_0 \neq \mu_1$ (inequality) → **infinitely many possibilities** (the difference could be 0.1, 5, −3…, each a different distribution) → $Z$ has **no unique distribution** → the critical value cannot be set.

So $H_0$ being an equality is not convention, it's a **mechanistic necessity**: only "a single point" yields a definite distribution to test against. "$\neq$" is a composite hypothesis (infinitely many cases), unfit to be $H_0$, and is left to $H_1$.

**One level deeper**: this also explains why $H_0$ is always "no effect / no difference" — it is the **specific, falsifiable, distribution-computable** assumption. You always compute probabilities inside the definite world of "assume no difference," then see how extreme the data is.

### The deep link to power: $H_0$ being an equality and "power requires assuming a specific shift" are the same principle

This is the layer most worth spelling out — **why $H_0$ is an equality, and why computing power in Section 10 requires the researcher to actively assume a specific shift, are two sides of the same principle.**

The principle is one sentence: **to compute a distribution and thus a probability, you must specify a "specific shift amount" (i.e. where the distribution is centered).**

- **$H_0$ takes $\mu_0 = \mu_1$**: this specifies "shift $=0$", a **definite point**. Centered at 0, distribution is the definite $N(0,1)$, so $\alpha$ can be computed.
- **"$\mu_0 \neq \mu_1$" cannot be $H_0$**: because it **does not specify how much the shift is** — the difference could be $0.1\sigma$, $3\sigma$, $-5\sigma$…, each a different center and different distribution. **"Not knowing how much it shifts" means "cannot compute a definite probability."** Infinitely many possible centers — which one do you compute against?
- **Computing power is exactly filling in this step**: the researcher **picks one specific shift** $\delta$ out of "$\neq$" (e.g. "assume the true difference $=$ some multiple of $\sigma$"), nailing a definite point onto the vague "there is a difference." With that specific $\delta$, the $H_1$ distribution is centered at $\delta/SE$, the distribution is definite, and power can be computed.

**So all three are coherent:**

| Assumption | Specified shift | Distribution | What can be computed |
|---|---|---|---|
| $H_0$ | $0$ (definite) | $N(0,1)$ | $\alpha$ |
| "$\neq$" (composite) | unspecified (infinitely many) | no unique distribution | **cannot compute** |
| pick a specific $\delta$ in $H_1$ | $\delta/SE$ (definite) | $N(\delta/SE,1)$ | power |

**Key conclusion**: **there is no "power over all of $\neq$", only "power for a specific $\delta$."** This is why power is always stated relative to some effect size — you must first answer "how large a true difference do I want to detect" before power is even meaningful. And "specifically assume how much shift" depends on the business and the problem (what is the smallest meaningful lift to detect), set by the researcher, not handed down.

> Aside: wanting to "compute a probability for 'there is a difference' directly," wanting $H_0$ to be $\neq$ — this is really asking for the Bayesian "compute the posterior probability of $H_1$ directly." Frequentism, to avoid introducing a prior, must nail every probability-computable hypothesis to a specific point. See Section 14.

---

## 6. Fork two: $\alpha$ is a conditional probability, not $P(H_0\text{ is true})$

This is the step frequentism gets reversed most often.

**$\alpha$ is not "the probability that $H_0$ is true."** In frequentism, "$H_0$ is true" is a **fixed fact about the world** (whether $\mu_0$ actually equals $\mu_1$); it **has no probability** — parameters are fixed, not random. So $P(H_0\text{ is true})$ simply does not exist in frequentism.

The correct definition of $\alpha$ is a **conditional probability**:

$$\alpha = P(\text{reject } H_0 \mid H_0 \text{ is true})$$

Read as: "**given that $H_0$ is true**, the probability that you wrongly reject it." The condition is "$H_0$ true"; the random thing is "your decision (because the sample is random)."

**"Parameters fixed, statistic random"** is the bedrock of frequentism:
- $\mu_0=\mu_1$ (parameters equal) — **not a random event, no probability**; it is the content/condition of the hypothesis "$H_0$."
- $|Z|>1.96$ (statistic falls in the rejection region) — **is a random event, has a probability**, because $Z$ depends on the randomly drawn sample.

$\alpha$ is the probability of **the latter conditioned on the former**.

### Line-by-line replay: is $\alpha$ equal to $P(\mu_0=\mu_1)$ / $P(H_0)$?

This exchange is worth preserving in full, because it is the core hurdle of frequentism.

**Attempt**: "$\alpha$ is the probability that $H_0$ is true, $P(\mu_0=\mu_1)$; below $\alpha$ reject."

**Correction 1**: $\alpha$ is **not** $P(H_0\text{ is true})$. In frequentism $\mu_0,\mu_1$ are **fixed true parameters**; "$\mu_0=\mu_1$" either holds or not — it **has no probability**, like $P(2+2=4)$ is not a meaningful probability. Assigning probability to the hypothesis itself is Bayesian (needs a prior); frequentism does not.

**Follow-up (natural)**: "But when back-solving for the critical value 1.96, aren't you treating $\alpha$ as a probability, and assuming $H_0$ true?"

**Reconciliation**: **Yes — this time it's correct.** Back-solving for the critical value does treat $\alpha$ as the probability of that tail, **under the premise that $H_0$ is true**, to solve for 1.96. $\alpha$ is indeed a probability. But see clearly: the two statements do not contradict; the difference is **condition vs. event**:

- ✅ "When $H_0$ is true, the probability of $|Z|>1.96$ is $\alpha$" → $\alpha$ is a **conditional probability** $P(\text{reject}\mid H_0\text{ true})$. Correct.
- ❌ "$\alpha$ is the probability that $H_0$ is true" → reading $\alpha$ as $P(H_0\text{ true})$. Wrong.

Both say "$\alpha$ is a probability," but the former is "the probability **conditioned on** $H_0$ being true," the latter is "the probability of the **event** that $H_0$ is true." **$\alpha$ is the former, not the latter.** The intuition to "write $P(\mu_0=\mu_1)$" wasn't wrong in direction ($\alpha$ is indeed related to "$\mu_0=\mu_1$"), but it mistook the "condition" for the "random event being measured": $\mu_0=\mu_1$ is the **premise world**; the random event being measured is "wrongly rejecting."

**Correction 2 (aside)**: what you compare against $\alpha$ is the **p-value**, not the statistic $Z$. "p-value $<\alpha$ → reject"; if using $Z$, compare against the critical value **1.96**. And statistically one always says "do not reject $H_0$," never "accept $H_0$" — non-significant $\neq$ proof of no difference (it may just be insufficient power, see Section 10).

**Nail it with the 2×2 table** (rows = truth, columns = your decision):

| | Truth: $H_0$ true (no effect) | Truth: $H_1$ true (effect) |
|---|---|---|
| **Decide reject** | Type I error $=\alpha$ | Correct $=$ **power** |
| **Decide don't reject** | Correct $=1-\alpha$ | Type II error $=\beta$ |

$\alpha$ is the top-left cell ($H_0$ true yet rejected); power is the top-right cell ($H_1$ true and rejected).

> **On the term "false positive"**: $\alpha$'s standard name is the false positive rate, because it $=$ "$H_0$ true yet reported positive (rejected)." Industry tools all call it that. If it feels odd, "the probability of wrongly rejecting when $H_0$ is true" is fully equivalent — understand the event, the name doesn't matter. This note mostly just says "reject / don't reject $H_0$."

---

## 7. Where the critical value 1.96 comes from: forward derivation vs. "reverse-engineering"

**Setting the critical value** is a step with no data yet, no decision yet. It is:

Under the hypothetical that $H_0$ is true, $Z\sim N(0,1)$, require "the probability of falling in the rejection region $=\alpha$":

$$P(|Z| > \varphi \mid H_0) = \alpha \implies P(Z>\varphi) = \alpha/2 = 0.025 \implies \varphi = 1.96$$

**This step treats $\alpha$ as a probability and, under the premise $H_0$ true, back-solves the critical value $\varphi=1.96$.** This $\alpha$ is "the wrongly-reject probability allowed in the $H_0$ world"; you use it to cut 1.96 out of the standard normal.

> **A subtle but correct point**: here "treating $\alpha$ as a probability" is correct — $\alpha$ is indeed a (conditional) probability, condition being "$H_0$ true." This **does not contradict** Section 6's "$\alpha$ is not $P(H_0\text{ true})$": $\alpha$ is "the probability **conditioned on** $H_0$ true," not "the probability of the **event** that $H_0$ is true." Condition vs. event, that's the difference.

**Making the decision** is a separate step: with data, compute a specific $Z$ (say 2.3), compare to 1.96, $|2.3|>1.96$ → reject. **This step is deterministic for a single dataset — no probability.**

**Key clarification**: the critical value 1.96 is **forward-fixed** by $\alpha$ (under $H_0$); it is fixed and does not change with $H_1$ or power. There is only one decision rule ($|Z|>1.96$ → reject); when running the experiment you **do not know** whether the truth is $H_0$ or $H_1$, so you cannot swap the boundary just because you "assume $H_1$ holds" — otherwise $\alpha$ goes out of control. **What is reverse-solved is the sample size $n$ (Section 12), not the critical value.**

---

## 8. The soul of frequentism: $\alpha$ is a long-run frequency, not the probability of a single trial

A genuine confusion: for **a single specific dataset**, $Z$ is determined and the decision is determined, so saying "the probability of rejecting" sounds odd.

That confusion is well-placed; it touches the essence of frequentist probability:

- **For a single specific dataset**: $Z$ is fixed, the decision is fixed, **there is no "probability of rejecting."**
- **For the hypothetical population "repeat the experiment infinitely many times"**: when $H_0$ is true, running this test over and over, a fraction $\alpha$ of experiments will, by chance, compute $|Z|>1.96$ and wrongly reject. **This is the correct reading of $\alpha = P(\text{reject}\mid H_0)$.**

> $\alpha=0.05$ means "when $H_0$ is true, over long-run repetition of this test procedure, 5% of experiments will wrongly reject" — **not** "this one experiment has a 5% chance of rejecting." **Probability belongs to 'the procedure's behavior under repetition,' not to 'a single result.'** This is the core distinction between frequentism and a single decision, and the bedrock of the next section on p-hacking.

---

## 9. p-hacking: how a single-trial guarantee gets broken by repetition

**Single test**: when $H_0$ is true, the wrongly-reject probability $=\alpha=0.05$. This is the "one time" guarantee.

**But if you run $m$ independent tests** (look at data $m$ times / test $m$ metrics / slice $m$ subgroups / try $m$ variants), and declare "found significance" as long as **any one** rejects:

$$P(\text{at least one wrong rejection}) = 1 - (1-\alpha)^m$$

Derivation: each not-wrongly-rejecting has probability $1-\alpha$; all $m$ not wrongly rejecting is $(1-\alpha)^m$; at least one wrong rejection is $1-(1-\alpha)^m$.

Plugging in numbers:

| $m$ | $1-(1-0.05)^m$ |
|---|---|
| 1 | 0.05 |
| 5 | 0.23 |
| 10 | 0.40 |
| 20 | **0.64** |
| 100 | 0.994 |

> **Easy mistake (preserved)**: one might write "the probability that all $m$ are significant $=\alpha^m$." $\alpha^m$ is "**all** wrongly reject" (tiny, $0.05^m$), which is **backwards** — p-hacking cares about "**at least one** wrong rejection," which **rises toward 1** with $m$, whereas $\alpha^m$ **falls toward 0**. The p-hacker only needs "at least one" to hit, and drops the rest.

**The essence of p-hacking**: turning "a single procedure's 5% wrong-reject rate," through repetition, into "a high overall probability of wrongly rejecting." Test 20 metrics and, even if the truth is all-no-effect, there's a 64% chance of catching at least one "significant" result. **You didn't discover an effect; you exhausted that 5% luck quota.**

**Common forms**: multi-metric fishing; peeking (looking at data repeatedly before hitting the sample size, stopping the moment it's significant, equivalent to multiple tests); subgroup slicing; picking the winning variant among many.

**Corrections**:
- **Bonferroni**: lower the threshold from $\alpha$ to $\alpha/m$ (conservative but simple).
- **Benjamini-Hochberg (FDR)**: control the "false discovery proportion" rather than "at least one," common for large-scale testing.
- **Dedicated fix for peeking**: sequential testing / alpha spending (mSPRT, group sequential) — allow continuous monitoring while controlling total $\alpha$. Standard in industrial A/B platforms.

**Practical discipline**: **pre-register the primary metric and sample size**, avoiding post-hoc fishing. This is the first principle of anti-p-hacking.

---

## 10. Forward derivation of power: where does $Z$ shift under $H_1$

**Power $=P(\text{reject } H_0 \mid H_1 \text{ is true})$** — fully symmetric with $\alpha$, only the condition changes from "$H_0$ true" to "$H_1$ true." It is the top-right cell: truth has an effect, and you correctly reject.

**Forward derivation, from the researcher's view:**

$H_1$ true means the true difference $\delta = \mu_1 - \mu_0 \neq 0$. Then the true mean of $\bar x_0 - \bar x_1$ is $\delta$, not 0. So the center of the standardized statistic $Z = \frac{\bar x_0-\bar x_1}{SE}$ **shifts**:

$$E[Z \mid H_1] = \frac{E[\bar x_0-\bar x_1]}{SE} = \frac{\delta}{SE}$$

**This $\delta/SE$ is the "non-centrality parameter," and it is everything about power.** It is "how far" the $H_1$ distribution shifts relative to the $H_0$ distribution, and it is the **signal-to-noise ratio** (effect ÷ noise).

So under $H_1$, $Z \sim N(\delta/SE,\ 1)$ (center moved to $\delta/SE$, shape unchanged). The rejection region is still the fixed $|Z|>1.96$.

**Power = probability that the shifted distribution falls in the rejection region** (take the main side, ignore the small far tail):

$$\text{Power} = P\!\left(Z > 1.96 \mid Z\sim N(\tfrac{\delta}{SE},1)\right) = P\!\left(N(0,1) > 1.96 - \tfrac{\delta}{SE}\right) = \Phi\!\left(\frac{\delta}{SE} - z_{\alpha/2}\right)$$

**Read this formula (all the intuition is here)**: power depends on $\frac{\delta}{SE} - z_{\alpha/2}$.

- larger $\delta$ (effect) → larger shift → higher power ✓
- smaller $SE$ (larger $n$, smaller $\sigma$) → larger $\delta/SE$ → higher power ✓
- smaller $z_{\alpha/2}$ (larger $\alpha$) → higher power ✓ (the $\alpha$–power trade-off)

The four knobs (effect size, sample size, variance, $\alpha$) all fall out of the single formula $\Phi(\delta/SE - z_{\alpha/2})$.

**A concrete number**: if we set the true difference $\delta = 2\cdot SE$ (i.e. signal-to-noise $\delta/SE = 2$), then $Z\sim N(2,1)$:

$$\text{Power} = P(Z>1.96 \mid N(2,1)) = \Phi(2 - 1.96) = \Phi(0.04) \approx 0.52$$

**Shifting 2 SEs gives only about 52% power** — short of the common 0.8/0.9. For power=0.9 you need $\delta/SE = 1.96 + 1.28 = 3.24$, i.e. shift 3.24 SEs. This also answers "what should $\delta$ (or $\mu$) be": it is an input the researcher decides jointly from "how large an effect to detect + how much power I want."

---

## 11. How to track the shift: treat $SE$ as a measured value and subtract it directly from $Z$

This section nails down the one place in the power derivation that genuinely trips people up repeatedly — "how far does $Z$'s center shift, and what exactly do you subtract when standardizing." The conclusion is actually clean; state it as the **main notation** first, then add one line on the common source of self-contradiction.

### Main notation: $SE$ is just a measured value, and $Z$ subtracts that number

**Ignore units and dimensions; treat $SE = \sqrt{\sigma_0^2/n_0 + \sigma_1^2/n_1}$ as a computed specific value (a measured value).** Say it comes out to $=4$.

The researcher assumes the true difference shifts $Z$'s center by $k$ SEs (say $k=2$, i.e. shift by the value $2SE = 8$). Then standardizing just means **subtracting that measured value directly**:

$$Z_1 = Z - k\cdot SE \sim N(0,1)$$

The rejection region uses the same scale: $|Z| > 1.96\cdot SE$.

**That's it — there is no "divide by $SE$ again" operation.** As long as $Z$ and $SE$ use the **same scale** throughout (both the measured value on the original metric), $Z_1 = Z - k\cdot SE$ is completely natural. Power is then:

$$\text{Power} = P(Z > 1.96\cdot SE \mid Z \text{ centered at }k\cdot SE) = \Phi(k - 1.96)$$

$k=2$ → $\Phi(0.04)\approx 0.52$. Note that power depends only on $k$ (how many SEs it shifted); the specific value 4 of $SE$ cancels itself out — confirming that "power depends only on the signal-to-noise ratio."

**In one line**: $SE$ is just a number, $Z$ subtracts that number (or its multiple), everything on one scale, no extra division needed.

### The one self-contradiction to avoid: switching scales midway

The reason the above sometimes gets confusing is that **the same $Z$ can have two definitions, and once you switch midway without correspondingly changing "what you subtract," they clash**:

- If your $Z = \bar x_0 - \bar x_1$ (**the mean difference itself**, same scale as $SE$) → subtract $k\cdot SE$, rejection region $1.96\,SE$. ✅ This is the main notation above.
- If your $Z = \dfrac{\bar x_0 - \bar x_1}{SE}$ (**already divided by $SE$**, dimensionless) → then its center's shift is $k$ (a pure number), subtract $k$, rejection region $1.96$.

**Both are correct, each internally consistent, power is $\Phi(k-1.96)$ in both.** Error arises in only one situation: **using the "already divided by $SE$" $Z$, but subtracting $k\cdot SE$ (a quantity carrying $SE$)** — a quantity that already divided out $SE$, then subtracting a quantity containing $SE$, so the scales don't match. The reverse (using the mean-difference $Z$ but subtracting the pure number $k$) is the same mistake.

> **Remember in one line**: what $Z$ subtracts must be on the same scale as $Z$. If $Z$ is the mean difference, subtract $k\cdot SE$; if $Z$ is already divided by $SE$, subtract $k$. **Pick one within a derivation, don't switch midway.** The main notation ($SE$ as a measured value, $Z-k\cdot SE$) is the least error-prone, because it lives entirely on the single scale of "the original measured value."

---

## 12. Solving for sample size $n$ given power

The real use in practice is the **reverse**: fix the desired power and $\alpha$, and solve for the required sample size. Usually assume **$n_0 = n_1 = n$ (50/50 split)**.

Requiring power $=1-\beta$, the shift must satisfy:

$$\frac{\delta}{SE} = z_{\alpha/2} + z_\beta$$

where $z_{\alpha/2}=1.96$ ($\alpha=0.05$ two-sided) and $z_\beta = \Phi^{-1}(\text{power})$ (power$=0.8 \Rightarrow 0.84$; power$=0.9 \Rightarrow 1.28$).

Substitute $SE = \sqrt{\dfrac{\sigma^2}{n}+\dfrac{\sigma^2}{n}} = \sqrt{\dfrac{2\sigma^2}{n}}$ (equal sample, equal variance):

$$\frac{\delta}{\sqrt{2\sigma^2/n}} = z_{\alpha/2}+z_\beta \implies \frac{\delta^2 n}{2\sigma^2} = (z_{\alpha/2}+z_\beta)^2$$

$$\boxed{n = \frac{2\sigma^2\,(z_{\alpha/2}+z_\beta)^2}{\delta^2}}\quad\text{(per arm)}$$

**Read it (core memory)**:

- $n \propto \sigma^2$: larger variance needs more samples → **reducing variance (CUPED, stratification, covariate adjustment) = saving samples = boosting power**.
- $n \propto 1/\delta^2$: **the smaller the effect you want to detect, the sample size grows as the square — halving $\delta$ needs 4× the $n$.**
- smaller $\alpha$, higher required power → larger $z$ → larger $n$.
- constants: at $\alpha=0.05$, power$=0.8$ gives $(1.96+0.84)^2\approx 7.85$, so $n\approx 16\sigma^2/\delta^2$ (per arm); power$=0.9$ gives $(1.96+1.28)^2\approx 10.5$, so $n\approx 21\sigma^2/\delta^2$.

---

## 13. Grounding in the click-through-rate (Bernoulli) case

When the metric is click / no-click, each user is a Bernoulli variable: $X_i \sim \text{Bernoulli}(p)$, where $p$ is the true click rate ($=$ mean $\mu$).

**Bernoulli variance is not a free parameter — it's determined by $p$**:

$$\text{Var}(X_i) = p(1-p)$$

So substitute $\mu \to p$, $\sigma^2 \to p(1-p)$ in the general formulas:

$$SE = \sqrt{\frac{p_0(1-p_0)}{n_0} + \frac{p_1(1-p_1)}{n_1}}, \qquad Z = \frac{\hat p_0 - \hat p_1}{SE}$$

Sample size:

$$n = \frac{2\,p(1-p)\,(z_{\alpha/2}+z_\beta)^2}{\delta^2}$$

where $\delta$ is the desired **absolute lift** in click rate (e.g. from 10% to 10.5%, $\delta=0.005$).

> **A classic point**: whether to use the **pooled proportion** when computing $SE$. Under $H_0$, $p_0=p_1$, so tests often estimate the common $p$ by $\hat p = \frac{\text{total clicks}}{n_0+n_1}$, with $SE=\sqrt{\hat p(1-\hat p)(1/n_0+1/n_1)}$. But for deriving power / sample size, use each arm's own $p$. Which to use in which context is a favorite interview detail.

---

## 14. Frequentist vs. Bayesian: not depth, but paradigm

Bayesian is not the "advanced version" of frequentist — it's a **parallel road**:

- **Frequentist** (this whole note): the condition is always "given $H_0/H_1$ is true," compute probabilities of the statistic. **Does not assign probability to hypotheses.** Answers: "when $H_0$ is true, the probability of seeing data this extreme" (p-value).
- **Bayesian**: compute $P(H_1 \text{ true}\mid \text{data})$, $P(p_1 > p_0 \mid \text{data})$ directly — **assign a posterior probability to the hypothesis itself**, needs a prior. Answers: "after seeing the data, the probability that $H_1$ is true."

> The intuition in Sections 5–6 to "compute a probability for 'there is a difference' directly, make $H_0$ be $\neq$, write $P(H_0)$" — illegal in frequentism, but exactly the legal core of Bayesian. Not right vs. wrong, but two paradigms.

**Both are used in industrial A/B**: frequentist is the mainstream default (plus sequential); the Bayesian version ("$P(\text{test is better})=95\%$") is used on some platforms, with the advantage of directly saying "95% probability test is better" and being naturally more peeking-friendly.

---

## 15. Interview soundbites summary (compressing the derivation to a few sayable lines)

**What power is**:
> "Power is the probability of correctly rejecting $H_0$ when there's a true effect $\delta$, equal to $1-\beta$. Given $n$ it's $\Phi(\delta/SE - z_{\alpha/2})$. It's determined by four quantities: effect size, sample size, variance, and $\alpha$."

**How to use it in practice**:
> "Before launch I use it in reverse to solve for sample size $n = 2\sigma^2(z_{\alpha/2}+z_\beta)^2/\delta^2$, fixing power=0.8 and $\alpha$=0.05. The key is $n$ is inversely proportional to $\delta^2$ — the smaller the lift I want to detect, the faster the sample size grows; and reducing variance (CUPED) directly saves samples. This avoids underpowered experiments where a real effect can't be detected."

**Interpreting a negative result**:
> "A non-significant experiment must distinguish 'truly no effect' from 'underpowered, couldn't detect it.' In an underpowered experiment, non-significant $\neq$ no effect."

**p-hacking**:
> "A single $\alpha=0.05$ only guarantees the single-trial wrong-reject rate; $m$ trials give at least one wrong rejection $=1-(1-\alpha)^m$, which is 64% for 20 metrics. So I pre-register the primary metric and sample size, use Bonferroni/FDR for multiple comparisons, and sequential testing for peeking."

**Bayesian** (bonus):
> "There's also Bayesian A/B; the difference is assigning a posterior to the hypothesis, being able to say $P(\text{better})$ directly, and being more peeking-friendly."

---

## 16. What this derivation actually trained

Not "memorized the power formula," but re-walking a path of constructing statistical inference from first principles — transferable:

1. **Distinguish 'variance of the mean' from 'variance of the data'** (they differ by an $n$) — the bedrock of all standard errors.
2. **$H_0$ being an equality is a mechanistic necessity**, not convention: only "a single point" has a definite distribution.
3. **$\alpha$ is a conditional probability** (condition = $H_0$ true), not $P(H_0)$; parameters fixed, statistic random.
4. **Frequentist probability is a long-run frequency**, belonging to "the procedure," not to "a single result" — this directly yields p-hacking.
5. **Power is the symmetric other half**: $Z$'s center shifts from 0 to $\delta/SE$, and the probability of falling in the fixed rejection region.
6. **Scale consistency**: treat $SE$ as a measured value, $Z$ subtracts that number (or its multiple), one scale throughout and you won't go wrong. What $Z$ subtracts must be on the same scale as $Z$ — switching scales midway is the most common source of self-contradiction.
7. **Solving for $n$**: $n \propto \sigma^2/\delta^2$, halve the effect and quadruple the samples; reducing variance is saving samples.

---

**Compressed to one sentence**: the whole A/B test is "assume no difference ($H_0$ takes a single point) → by CLT the mean difference is approximately normal with standard error $\sqrt{\sigma^2/n}$ → standardize into $Z$ → in the $H_0$ world set the critical value 1.96 (controlling the long-run wrong-reject rate $\alpha$) → in the $H_1$ world compute the probability that $Z$, shifted by $\delta/SE$, falls in the rejection region (power) → reverse-solve the sample size." Every step grows out of the single starting point of "parameters fixed, sample random + wanting to control two kinds of error."
