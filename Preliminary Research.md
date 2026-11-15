# Notation Table

|                |                                                          |                            |
| -------------- | -------------------------------------------------------- | -------------------------- |
| Symbol         | Meaning                                                  | Units / range              |
| $x$            | Age last birthday                                        | years, integer 0–100+      |
| $t$          | Calendar year                                            | e.g. 1961–2023             |
| $D_{x,t}$    | Deaths aged x in year t                                  | count                      |
| $E_{x,t}$    | Central exposure (person-years lived at age x in year t) | person-years               |
| $m_{x,t}$    | Central death rate, D/E                                  | per person-year            |
| $μ(x)$       | Force of mortality (instantaneous death rate)            | per year                   |
| $q(x), p(x)$ | One-year probability of death / survival                 | 0 to 1                     |
| $l(x), d(x)$ | Survivors to exact age x, deaths between x and x+1       | people out of l(0)=100,000 |
| $L(x), T(x)$ | Person-years lived in [x, x+1), and from x onwards       | person-years               |
| $e(x)$       | Remaining life expectancy at age x                       | years                      |
| $i, v$       | Annual interest rate, discount factor v = 1/(1+i)        | rate, 0 to 1               |
| $a(x)$       | EPV of a £1-a-year whole-life annuity-immediate          | £                          |

---
# Mortality Data and Rates

## Deaths vs Rates

Everything rests on one ratio: deaths divided by the time people spent at risk of dying. If $D$ and $E$ are not measured on the same basis (same ages, same year, same population), every later number inherits the error.

We take $D_{x,t}$ as a death count and mix two things:

- How risky age $x$ is
- How many people are aged $x$

Dividing by exposure will remove the population-size effect and produce the risk:

$$
m_{x, t}=\frac{D_{x, t}}{E_{x, t}^c}
$$

## The Denominator : Exposure

Central exposure, $E^c$, is the total person-years lived at age $x$ during year $t$. For instance, if you were aged $x$ at for four months, you contribute $\frac{1}{3}$

Within the ONS data, mid-year population estimates are the usual stand-in for central exposure. We are assuming here that the population at each age changes roughly linearly through the year.

Initial Exposure, $E^0$ is the number alive at the start of the interval and is relevant to another statistic we will interpret. It is important to acknowledge here that deaths by 'registration' year vs 'occurrence' year and age at death vs age at last birthday can cause mismatching.

## Statistical View of m

Let $D \sim \operatorname{Poisson}(E\cdot m))$. The maximum likelihood estimate of $m$ is then exactly $\frac{D}{E}$ with an approximate standard error of:

$$
\operatorname{SE}(\hat m_{x}) \approx \sqrt{\frac{\hat m_x}{E_x}} = \frac{\sqrt{D_x}}{E_x}
$$


We have 'noise' that can interfere with our statistics when we examine very old and very young ages.

## Crude vs Age-Specific vs Standardised Rates

The crude death rate is one number for the whole population:

$$
\mathrm{CDR}_t=\frac{\sum_x D_{x, t}}{\sum_x E_{x, t}}=\sum_x w_{x, t} m_{x, t}, \quad w_{x, t}=\frac{E_{x, t}}{\sum_y E_{y, t}}
$$

It is a weighted average of the age-specific rates, and the weights are the populations own age structure.

Because of this, the crude rate misleads. Aging populations put more weight on high $m$-ages, so the crude rate can rise even while every age-specific rate falls.

We fix this with direct age standardisation : we fix one standard population with $W^s$ and apply it to every year:

$$
\text{ASMR}_t = \sum_x w^{s}_x\, m_{x,t}
$$

## Expected Patterns

- **Shape over age.** High infant mortality, a minimum around ages 5–12, an "accident hump" for young men around 18–25, then roughly exponential growth from about age 30. On a log scale, adult mortality is close to a straight line.
- **Sex gap.** Male rates are higher at nearly every age. The ratio is largest in the accident hump and in late middle age.
- **Trend over time.** Rates fall at most ages, with an improvement rate (Section 5) of very roughly 1–3% a year. Improvements were fast in the 2000s and slowed after about 2011.
- **Shocks.** Look for 2020–21 $COVID-19$, heavy flu winters, and the "golden cohort" born around 1925–35, whose improvements were unusually fast. On a heatmap, cohort effects show up as diagonal streaks, because a cohort ages one year for every calendar year.

---
# The Period Life Tables

## $m$ and $q$

A life table follows an imaginary cohort of 100,000 newborns through one year's age-specific death rates. Every column comes from the column before it, so the only real modelling choice is how you turn $m$ into $q$. From here, we discuss the difference between $m$ and $q$ :


- $m_{x,t}$ is a _rate_: deaths per person-year lived at age $x$. Its denominator is central exposure. In principle it can exceed 1.
- $q(x)$ is a _probability_: of the people alive at exact age $x$, the share who die before $x+1$. Its denominator is the number alive at the start. It is always between 0 and 1.
- For small rates they are almost equal. They diverge at high ages, where many people die partway through the year and so contribute less than a full year of exposure.

## Deriving a conversion formula

Let $\mathrm{a}(\mathrm{x})$ be the average fraction of the year lived by those who die in $[\mathrm{x}, \mathrm{x}+1)$. Then the person-years lived in the interval are:

$$
L_x=l_{x+1}+a_x d_x=l_x-\left(1-a_x\right) d_x
$$

The central rate is deaths over person-years, $\mathrm{m}=\mathrm{d} / \mathrm{L}$. Substitute $\mathrm{d}=\mathrm{L} \cdot \mathrm{q}$ and divide through by I:

$$
m_x=\frac{q_x}{1-\left(1-a_x\right) q_x} \quad \Longleftrightarrow \quad q_x=\frac{m_x}{1+\left(1-a_x\right) m_x}
$$

The report should feature this. Setting $\mathrm{a}(\mathrm{x})=0.5$ says that deaths are spread evenly across the year of age. That is reasonable where mortality changes slowly within a year. It is poor at age 0, and slightly biased at very old ages, where mortality rises steeply within the year.

## Code Implementation

$$
\begin{aligned}
p_x &= 1 - q_x, \qquad l_0 = 100\,000, \qquad l_{x+1} = l_x\,p_x \\
d_x &= l_x - l_{x+1} = l_x\,q_x \\
L_x &= l_x - (1-a_x)\,d_x \\
T_x &= \sum_{y \ge x} L_y, \qquad \mathring{e}_x = \frac{T_x}{l_x}
\end{aligned}
$$

Checks your code should run (these make good unit tests):

- Every $\mathrm{q}(\mathrm{x})$ is in $[0,1]$ and the last one equals 1.
 - $\mathrm{I}(\mathrm{x})$ is non-increasing, and the $\mathrm{d}(\mathrm{x})$ values sum to $\mathrm{I}(0)$.
- Each $\mathrm{L}(\mathrm{x})$ sits between $\mathrm{I}(\mathrm{x}+1)$ and $\mathrm{I}(\mathrm{x})$.
- $\mathrm{e}(\mathrm{x})+\mathrm{x}$ rises with x: a person who has survived to 80 expects to die later than a new born does.

## Complete vs Curate Life Expectancy

The $e_{x}$ above is complete, counting fractions of years. The curtate version counts only whole years survived. Annuity work uses curtate-style sums:
$$
e_x=\sum_{k \geq 1} k p_x=\sum_{k \geq 1} \frac{l_{x+k}}{l_x}, \quad \dot{e}_x \approx e_x+\frac{1}{2}
$$
## Shape of Survival Curve


On a log scale $\mathrm{m}(\mathrm{x})$ is roughly a rising straight line; $\mathrm{l}(\mathrm{x})$ is the opposite shape. It stays flat near 100,000 to about age 50, then falls steeply between 70 and 95. Over the last century the curve has become more rectangular: deaths are compressed into a narrower band of old ages.

## Period vs Cohort life expectancy

Period e(0) is a summary of one year's death rates. It is not a forecast. It answers the question: "How long would a newborn live if every age kept this year's mortality forever?" A baby born today reaches 70 around 2096, by which time the rate at 70 will probably be far lower. When mortality is improving, cohort life expectancy is higher than period life expectancy. Period figures also over-react to temporary shocks: the 2020 dip did not happen to any real cohort for its whole life.

## Benchmarking ONS Figures

In the UK national life tables for 2022 to 2024, life expectancy at birth was 83.0 years for females and 79.1 years for males; at 65 it was 21.2 and 18.7 years ([ONS, released 10 Dec 2025](https://ons.gov.uk/peoplepopulationandcommunity/birthsdeathsandmarriages/lifeexpectancies/bulletins/nationallifetablesunitedkingdom/latest)). Single-year figure differ from these because:

- ONS pools three consecutive years of data, which smooths out shocks such as the pandemic and bad flu seasons.
- Your coverage (England & Wales vs UK) or top age may differ.
- Your $a(x)$ choices, especially at age 0 and at the oldest ages, may differ from the ONS method.
- The exposure estimates differ (mid-year population vs HMD exposures built from births by month).

---
# Parametric Mortality Laws

We use parametric laws to replace a large number of noisy adult death rates. This makes mortality smooth and easy to extrapolate to the oldest ages, and comparable across sexes and years.

## Force of Mortality

The force of mortality, $\mu(x)$, is the instantaneous death rate at exact age $x$. We link it to the survival by:

$$
\mu(x)=-\frac{d}{d x} \ln l(x), \quad{ }_t p_x=\exp \left(-\int_0^t \mu(x+s) d s\right)
$$

If $\mu$ is constant within each year of age, then $\mathrm{m}(\mathrm{x})=\mu$. More generally, $\mathrm{m}(\mathrm{x}) \approx \mu(\mathrm{x}+1 / 2)$. So when you fit a law to observed m , use the mid-age $\mathrm{x}+0.5$ as the regressor, or at least say that you used x.

## Gompertz : $\mu_(x) = A e ^{Bx}$

This formula uses:

- $A$ is the level: the hazard extrapolated to age $0$. It moves the whole log-curve up or down and is typically tiny.
- $B$ is the rate of ageing: the slope of the log-mortality. Each extra year of age multiplies mortality by $e^B$. The mortality doubling time is In $2 / B$. Typical B is $0.08-0.11$, so mortality doubles roughly every $7-9$ years.
``
By taking logs of both sides, we construct our ordinary straight line regression:

$$
\ln \mu(x) = \ln A + Bx \quad\Rightarrow\quad y_x = \beta_0 + \beta_1 x + \varepsilon_x,\quad \hat A = e^{\hat\beta_0},\ \hat B = \hat\beta_1
$$

We may also obtain a closed-form survival function which can be used to close the life table at high ages:

$$
{}_tp_x = \exp\!\Big(-\frac{A}{B}\,e^{Bx}\big(e^{Bt}-1\big)\Big)
$$

This produces three ways to fit, in increasing rigour:

|                                 |                        |                                                            |                                                                                                               |
| ------------------------------- | ---------------------- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Method                          | What it minimises      | MATLAB                                                     | Pros and cons                                                                                                 |
| OLS on log m                    | Σ (ln m − ln μ̂)²      | `polyfit(x, log(m), 1)` or `fitlm`                         | Simplest, and the brief asks for it. Treats a noisy age-95 rate as just as reliable as a precise age-60 rate. |
| Weighted least squares on log m | Σ D(x) (ln m − ln μ̂)² | `fitlm(..., 'Weights', D)`                                 | Weight by D because Var(ln m̂) ≈ 1/D. A cheap, large improvement.                                             |
| Poisson maximum likelihood      | −Σ [D ln(Eμ) − Eμ]     | `fitglm(x, D, 'Distribution','poisson', 'Offset', log(E))` | The statistically correct model for counts. Gives standard errors and deviance directly.                      |

The Poisson version is still log-linear in its parameters: $\ln \mathrm{E}[\mathrm{D}]=\ln \mathrm{E}+\ln \mathrm{A}+\mathrm{Bx}$. So `fitglm` fits Gompertz exactly, with In E as an offset. Doing all three and comparing them is an easy way to expand on the report.

## Choosing the age range for Gompertz

We can use Gompertz for describing adult mortality only. So we fit over roughly ages 30-90 and then show what happens outside that range:

- **Below about 30:** infant mortality and the accident hump sit far above the Gompertz line.
- **Above about 90:** observed mortality bends below the line (late-life deceleration). An exponential will over-predict deaths there and under-predict survival at very old ages. Remember that the data are also noisiest there.

## Residuals and goodness of fit

- **Log-scale residual:** $r(x) = \ln m(x) - \ln \hat{\mu}(x)$. Plot it against age. Random scatter is good; an S-shape or a U-shape means the curve has the wrong form.
- **Poisson (Pearson) residual:** $D - Ẹ\hat{\mu}(x)/ \sqrt{E\hat{\mu}(x)}$. If the model is right, these are roughly $\mathrm{N}(0,1)$, so $|\mathrm{r}|>2$ should be rare.
- $\mathrm{RSS}=\Sigma \mathrm{r}^2$ and RMSE $=\sqrt{(\mathrm{RSS} / \mathrm{n})}$. Always say which scale you used. The log scale and the raw-rate scale give very different numbers, and raw-scale RMSE is dominated by the oldest ages.
- For comparing models with different numbers of parameters, use $\mathrm{AlC}=2 \mathrm{k}-2 \mathrm{ln} \mathrm{L}$, BIC, or a likelihood-ratio test. RSS alone always favours the bigger model.

## Gompertz - Makeham : $\mu(x) = C + Ae^{Bx}$

We update our equation to have $C$ as an age independent background hazard. These are accidents and other causes unrelated to ageing. It matters at 20-45, where the Gompertz term is tiny but barely matters at 80.

Doing this makes our model no longer linear after taking logs, we must use non-linear fittings such as : `fitnlm`, `lsqcurvefit`, or `fminsearch` on the Poisson negative log-likelihood. We will use the Gompertz estimates as starting values, with $C_0 = 0.0005$.


Furthermore, since $C$ and $A$ are correlated, and $C$ is small, its standard error can be large and so we must constraining $C \geq 0$ or fit $\operatorname{Log} C$.

## Expectations across sexes and years

- **Males have a higher $A$ and $B$is often similar or slightly lower for males.**
- **$A$ typically falls and $B$ tends to rise**: mortality falls most at younger adult ages, so the log-line gets steeper. The negative $A$-$B$ is called the 'compensation effect'

There exist other models for this modelling to:

- Perks/logistic (they plateau at old ages)
- Kannisto (old ages)
- Heligman–Pollard (8 parameters covering the full age range).

---
# Mortality improvements and forecasting

Forecasting means estimating how fast each age's mortality is falling and carrying that rate forward. The result is very sensitive to which historical years you learn the trend from.

## Measuring Improvement

We have two equivalent definitions, we will chose one and remain with it:

$$
r_{x,t} = 1 - \frac{m_{x,t}}{m_{x,t-1}} \qquad\text{or}\qquad \rho_{x,t} = -\big(\ln m_{x,t} - \ln m_{x,t-1}\big)
$$

For small changes, they are nearly equal since $r = 1-e^{-p}$. Year to year values become very noisy, a bad flu winter followed by a mild one gives a huge apparent improvement which may not necessarily be true. 

What we must do is estimate a smoothed average improvement for each age over a base period, as the slope of a regression of $\ln m$ on $t$:

$$
\ln m_{x,t} = \alpha_x - \rho_x\,(t - t_0) + \varepsilon_{x,t}, \qquad t \in [t_1, t_2]
$$

## Is improvement constant, linear or other?

We can compute $\rho_{x,t}$ over rolling windows and plot them as a heatmap or a line chart. With UK data, we should expect to see:

- Faster improvement from about 1990 to 2011, especially for men aged 50–80, driven by falling cardiovascular deaths.
- A sharp slowdown after about 2011.
- Negative "improvements" in 2020–21.

Improvement is therefore not constant over time and is the central weakness of a simple extrapolation - something we should note in the report.

## A simple projection model

Starting from a base year $T$ and applying each age's average improvement for $h$ years:

$$
\hat m_{x,T+h} = \hat m_{x,T}\, e^{-\rho_x h} \quad\big(\text{equivalently } \hat m_{x,T}(1-r_x)^h\big)
$$

We can apply this to to fitted Gompertz rates in two ways:

1) Apply age specific $\mu(x)$ to the fitted curve $\hat{\mu}(x,T)$
2) Fit Gompertz to every year, then regress in $\ln A(t)$ and $B(t)$ on $t$ and extrapolate both. This keeps the projected curves exactly Gompertz.

## Period vs Cohort Projected tables

Noting here a serious distinction:

- Projected period table for year $T$ : Use $m(x,T+h)$ for every age $x$. This is a snapshot for one future year
- Cohort table for someone aged $x$ in the year $T$ : Use the diagonal $m(x,T) ,  m(x+1,T+1), m(x+2,T+2), \ldots$. This is what an individual actually lives through.

A 65-year-old buying an annuity today should be valued with the cohort diagonal. Report period and cohort life expectancy side by side; the cohort figure will be higher.

## Why indefinite extrapolation breaks down

- With any positive $\rho$, $m$ tends to 0 and life expectancy grows without limit
- Ages improve at different rates, after enough years the projected curve can stop increasing with age causing biological implausibility's such as $m(60)$ falling below $m(50)$
- The base period will dominate, a 1991–2011 base gives far higher improvements than 2011–2019, and including 2020–21 distorts both.

In real actuarial teams, the CMI model blends current improvements into a chosen _long-term rate_, often about 1.5% a year, rather than extrapolating raw trends.

---
# Annuity Valuation

An annuity value is a sum of discounted payments, each weighted by the probability the person is alive to receive it. Interest and mortality are the only two assumptions. Your job is to show how much each one moves the price.

## The building blocks

- Discounting : £1 paid in $t$ years is worth $v^t$ today, where $v = \frac{1}{1+i}$ is the discount factor and $i$ is the annual interest rate. A higher interest rate means a smaller present value, and the effect is strongest for distant payments
- Multi Year Survival : Surviving $t$ years means surviving each single year in turn:

$$
{}_tp_x = \prod_{j=0}^{t-1} p_{x+j} = \frac{l_{x+t}}{l_x}
$$

## Deriving the annuity formula

Let $I_{t} = 1$  if the person aged $x$ years is a live at time $t$ and 0 otherwise. The present value of the payments is a rangom variable $\sum v^{I_t})$ and its expectation is therefore:

$$
a_x = \mathbb{E}\Big[\sum_{t\ge1} v^t I_t\Big] = \sum_{t=1}^{\omega-x} v^t\,{}_tp_x
$$

After the table's last age $\omega$, the sum stops since the survival is beyond its zero.

## Recursion Implementation:

We have a recursive formula as:

$$
a_x = v\,p_x\,\big(1 + a_{x+1}\big), \qquad a_\omega = 0, \qquad \ddot a_x = 1 + a_x
$$

We work backwards from the top age, one loop gives $a(x)$ at for every age at once. $\ddot{a}$ denotes the annuity-due, paid at the start of each year.

## Unit Tests:

- At $i=0$, $a(x)$ equals the curtate life expectancy $e(x)$ 
- $a(x) < \frac{1}{i}$, the value of a perpetuity
- $a(x)$ falls as $i$ rises and usually falls as $x$ rises
- If every $p=1$ up to age $\omega$, $a(x)$ equals the annuity-certain value $\frac{1-v^n}{i}$, with $n = \omega -x$

## Interest-rate sensitivity

Partially differentiating with respect to interest rates:

$$
\frac{\partial a_x}{\partial i} = -\sum_{t} t\,v^{t+1}\,{}_tp_x < 0, \qquad D_x = \frac{\sum_t t\,v^t\,{}_tp_x}{a_x}
$$

The value is decreasing and convex in $i$. $D_x$ is the duration and is the survival-weighted average time to payment. It explains why younger annuitants are more sensitive to interest rates as their payments are further away.

## Constant vs Improving Mortality

- Constant (period) - $p_{x,t}$ comes from one year's table
- Improving (cohort) - $p_{x,t}$ comes from the projected diagonal, so each future year uses that year's rates:

$$
{}_tp_x^{\text{cohort}} = \prod_{j=0}^{t-1}\big(1 - q_{x+j,\,T+j}\big)
$$

Expect improving mortality to raise a(65) by a few percent. The effect is larger at younger ages and lower interest rates, where payments run further into the future. Report the difference as a percentage:

$$
\frac{a_{\text{improving}} - a_{\text{constant}}}{a_{\text{constant}}} \times 100
$$

## Joint sensitivity analysis

For the report, we are going to evaluate $a(65)$ over a grid: interest $i$ from 0% to 6% in steps of 0.5%, and improvement from 0% to 3% a year (or multiples of your base $\rho$). Show the result as a heatmap or contour plot. The point to make is that the effects interact: mortality assumptions matter more when interest rates are low.

## Financial Implications to discuss

- Annuity and pension payments are fixed when the contract is sold. If people live longer than assumed, the provider pays for longer than it reserved for.
- Longevity risk is _systematic_. It cannot be diversified away by writing more policies, because every policy is affected by the same trend. Only the random, individual-level part averages out.
- A small percentage error times a large book is a large sum. A 3% understatement on £10bn of liabilities is £300m.
- How the industry responds: capital requirements (the UK Solvency regime), stress tests, and transferring the risk through longevity swaps, buy-ins and buy-outs.

---
# Validation and Uncertainty

A model earns trust by predicting years it never saw. While compiling this report, we will likely conclude that parameter uncertainty is tiny, because national data are huge, while the choice of trend and model dominates the uncertainty in money terms.

## Out-of-sample testing

- Split by _time_, never at random. For example: train on 1961–2009, test on 2010–2019, and treat 2020–21 as a separate stress test.
- For a stronger test, use a rolling origin: refit with data up to year T, forecast T+1 to T+10, move T forward and repeat. Plot the error against the forecast horizon.
- "Fitting on training data" means estimating everything (Gompertz A and B for each year, their trends, the improvement rates ρ) from the training years only.

## Error measures

|   |   |   |   |
|---|---|---|---|
|Measure|Formula|Strength|Weakness|
|RMSE on the log scale|√ mean (ln m̂ − ln m)²|Treats proportional errors equally across ages|Hard to explain to non-specialists|
|MAE|mean \|m̂ − m\||Robust and in the data's units|Dominated by the oldest ages|
|MAPE|mean \|m̂ − m\| / m|Scale-free percentage|Unstable when m is tiny; penalises over- and under-prediction unequally|
|Mean signed log error (bias)|mean (ln m̂ − ln m)|Shows the _direction_ of error|Positive and negative errors can cancel out|
|Poisson deviance|2Σ [D ln(D/Eμ̂) − (D − Eμ̂)]|Statistically principled; weights by information|Least intuitive|
|Error in e(65) or a(65)|ê − e|Measures what actually matters financially|Errors at different ages can offset|

## Looking for systematic bias

Plot the signed log errors as a heatmap of age by test year. If the training period ended before the post-2011 slowdown, expect the models to have _under_-estimated mortality. That means overstated life expectancy and annuity values, mainly at ages 50–85.

##  Confidence intervals for parameters

- From the regression: β̂ ± t(0.975, n−2) · SE(β̂). In MATLAB, use `coefCI(mdl)`.
- For A = e^β₀, transform the endpoints of the β₀ interval. The result is not symmetric.
- From the Poisson model: Wald intervals from the Fisher information. Profile-likelihood intervals are better for Makeham's C.
- **Interpretation:** if the data-generating process were repeated many times, 95% of intervals built this way would contain the true parameter. Here, the "repetition" is the Poisson randomness in deaths. The interval does not cover the risk that the model itself is wrong.

##  Propagating parameter uncertainty into money

1. Draw N = 5,000 parameter vectors θ* ~ N(θ̂, Σ̂), using `mvnrnd` or `chol(Σ)' * randn`.
2. For each draw, rebuild μ(x), the life table, e(65) and a(65).
3. Report the mean and the 2.5th and 97.5th percentiles, and plot a histogram.
4. Alternative: a parametric bootstrap. Simulate D* ~ Poisson(E · m̂) with `poissrnd`, refit, and recompute.
5. Extension: simulate Lee–Carter k(t) paths with `randn` to get _forecast_ uncertainty. These intervals will be far wider than the parameter intervals.

##  Three kinds of uncertainty

|   |   |   |   |
|---|---|---|---|
|Type|Source|How you measure it|Typical size in this project|
|Statistical (process)|Random timing of individual deaths|Simulate lifetimes with `rand` for a finite portfolio|Large for 100 lives; negligible for a nation|
|Parameter|Estimation error in A, B, C, ρ|CIs, Monte Carlo, bootstrap|Small, because national exposures are in the millions|
|Model|Wrong functional form, base period or projection method|Compare Gompertz, Makeham and Lee–Carter, and different base periods|Usually the largest|

Rank your assumptions by their effect on a(65) with a tornado chart: interest rate, improvement rate, base period, model choice, top-age treatment, and a(x).

##  Why a good historical fit can still forecast badly

- **Structural breaks.** The post-2011 slowdown and COVID-19 change the process itself.
- **Extrapolation.** Forecasting goes beyond the range of the data, where nothing constrains the curve.
- **Overfitting.** Extra parameters fit noise in the training years, not signal.
- **Non-stationarity.** Medical, social and economic drivers of mortality change over time, and the model's form is fixed.
