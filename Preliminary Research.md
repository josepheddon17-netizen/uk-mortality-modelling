# Notation Table

  -------------- ----------------------------------------- -------------------
  Symbol         Meaning                                   Units / range

  $x$            Age last birthday                         years, integer
                                                           0--100+

  $t$            Calendar year                             e.g. 1961--2023

  $D_{x,t}$      Deaths aged x in year t                   count

  $E_{x,t}$      Central exposure (person-years lived at   person-years
                 age x in year t)                          

  $m_{x,t}$      Central death rate, D/E                   per person-year

  $μ(x)$         Force of mortality (instantaneous death   per year
                 rate)                                     

  $q(x), p(x)$   One-year probability of death / survival  0 to 1

  $l(x), d(x)$   Survivors to exact age x, deaths between  people out of
                 x and x+1                                 l(0)=100,000

  $L(x), T(x)$   Person-years lived in \[x, x+1), and from person-years
                 x onwards                                 

  $e(x)$         Remaining life expectancy at age x        years

  $i, v$         Annual interest rate, discount factor v = rate, 0 to 1
                 1/(1+i)                                   

  $a(x)$         EPV of a £1-a-year whole-life             £
                 annuity-immediate                         
  -------------- ----------------------------------------- -------------------

  -----------------------------------------------------------------------------------------------------------------------------------------------------
  \# Mortality Data and Rates
  -----------------------------------------------------------------------------------------------------------------------------------------------------
  \# The Period Life Tables

  \## $m$ and $q$

  A life table follows an imaginary cohort of 100,000 newborns through one year's age-specific death rates. Every column comes from the column before
  it, so the only real modelling choice is how you turn $m$ into $q$. From here, we discuss the difference between $m$ and $q$ :

  \- $m_{x,t}$ is a *rate*: deaths per person-year lived at age $x$. Its denominator is central exposure. In principle it can exceed 1. - $q(x)$ is a
  *probability*: of the people alive at exact age $x$, the share who die before $x+1$. Its denominator is the number alive at the start. It is always
  between 0 and 1. - For small rates they are almost equal. They diverge at high ages, where many people die partway through the year and so contribute
  less than a full year of exposure.

  \## Deriving a conversion formula

  Let $\mathrm{a}(\mathrm{x})$ be the average fraction of the year lived by those who die in $[\mathrm{x}, \mathrm{x}+1)$. Then the person-years lived
  in the interval are:

  $$
  L_x=l_{x+1}+a_x d_x=l_x-\left(1-a_x\right) d_x
  $$

  The central rate is deaths over person-years, $\mathrm{m}=\mathrm{d} / \mathrm{L}$. Substitute $\mathrm{d}=\mathrm{L} \cdot \mathrm{q}$ and divide
  through by I:

  $$
  m_x=\frac{q_x}{1-\left(1-a_x\right) q_x} \quad \Longleftrightarrow \quad q_x=\frac{m_x}{1+\left(1-a_x\right) m_x}
  $$

  The report should feature this. Setting $\mathrm{a}(\mathrm{x})=0.5$ says that deaths are spread evenly across the year of age. That is reasonable
  where mortality changes slowly within a year. It is poor at age 0, and slightly biased at very old ages, where mortality rises steeply within the
  year.

  \## Code Implementation

  $$
  \begin{aligned}
  p_x &= 1 - q_x, \qquad l_0 = 100\,000, \qquad l_{x+1} = l_x\,p_x \\
  d_x &= l_x - l_{x+1} = l_x\,q_x \\
  L_x &= l_x - (1-a_x)\,d_x \\
  T_x &= \sum_{y \ge x} L_y, \qquad \mathring{e}_x = \frac{T_x}{l_x}
  \end{aligned}
  $$

  Checks your code should run (these make good unit tests):

  \- Every $\mathrm{q}(\mathrm{x})$ is in $[0,1]$ and the last one equals 1. - $\mathrm{I}(\mathrm{x})$ is non-increasing, and the
  $\mathrm{d}(\mathrm{x})$ values sum to $\mathrm{I}(0)$. - Each $\mathrm{L}(\mathrm{x})$ sits between $\mathrm{I}(\mathrm{x}+1)$ and
  $\mathrm{I}(\mathrm{x})$. - $\mathrm{e}(\mathrm{x})+\mathrm{x}$ rises with x: a person who has survived to 80 expects to die later than a new born
  does.

  \## Complete vs Curate Life Expectancy

  The $e_{x}$ above is complete, counting fractions of years. The curtate version counts only whole years survived. Annuity work uses curtate-style
  sums: $$
  e_x=\sum_{k \geq 1} k p_x=\sum_{k \geq 1} \frac{l_{x+k}}{l_x}, \quad \dot{e}_x \approx e_x+\frac{1}{2}
  $$ \## Shape of Survival Curve

  On a log scale $\mathrm{m}(\mathrm{x})$ is roughly a rising straight line; $\mathrm{l}(\mathrm{x})$ is the opposite shape. It stays flat near 100,000
  to about age 50, then falls steeply between 70 and 95. Over the last century the curve has become more rectangular: deaths are compressed into a
  narrower band of old ages.

  \## Period vs Cohort life expectancy

  Period e(0) is a summary of one year's death rates. It is not a forecast. It answers the question: "How long would a newborn live if every age kept
  this year's mortality forever?" A baby born today reaches 70 around 2096, by which time the rate at 70 will probably be far lower. When mortality is
  improving, cohort life expectancy is higher than period life expectancy. Period figures also over-react to temporary shocks: the 2020 dip did not
  happen to any real cohort for its whole life.

  \## Benchmarking ONS Figures

  In the UK national life tables for 2022 to 2024, life expectancy at birth was 83.0 years for females and 79.1 years for males; at 65 it was 21.2 and
  18.7 years ([ONS, released 10 Dec
  2025](https://ons.gov.uk/peoplepopulationandcommunity/birthsdeathsandmarriages/lifeexpectancies/bulletins/nationallifetablesunitedkingdom/latest)).
  Single-year figure differ from these because:

  \- ONS pools three consecutive years of data, which smooths out shocks such as the pandemic and bad flu seasons. - Your coverage (England & Wales vs
  UK) or top age may differ. - Your $a(x)$ choices, especially at age 0 and at the oldest ages, may differ from the ONS method. - The exposure
  estimates differ (mid-year population vs HMD exposures built from births by month).
  -----------------------------------------------------------------------------------------------------------------------------------------------------

# Parametric Mortality Laws

We use parametric laws to replace a large number of noisy adult death
rates. This makes mortality smooth and easy to extrapolate to the oldest
ages, and comparable across sexes and years.

## Force of Mortality

The force of mortality, $\mu(x)$, is the instantaneous death rate at
exact age $x$. We link it to the survival by:

$$
\mu(x)=-\frac{d}{d x} \ln l(x), \quad{ }_t p_x=\exp \left(-\int_0^t \mu(x+s) d s\right)
$$

If $\mu$ is constant within each year of age, then
$\mathrm{m}(\mathrm{x})=\mu$. More generally,
$\mathrm{m}(\mathrm{x}) \approx \mu(\mathrm{x}+1 / 2)$. So when you fit
a law to observed m , use the mid-age $\mathrm{x}+0.5$ as the regressor,
or at least say that you used x.

## Gompertz : $\mu_(x) = A e ^{Bx}$

This formula uses:

-   $A$ is the level: the hazard extrapolated to age $0$. It moves the
    whole log-curve up or down and is typically tiny.
-   $B$ is the rate of ageing: the slope of the log-mortality. Each
    extra year of age multiplies mortality by $e^B$. The mortality
    doubling time is In $2 / B$. Typical B is $0.08-0.11$, so mortality
    doubles roughly every $7-9$ years. \`\` By taking logs of both
    sides, we construct our ordinary straight line regression:

$$
\ln \mu(x) = \ln A + Bx \quad\Rightarrow\quad y_x = \beta_0 + \beta_1 x + \varepsilon_x,\quad \hat A = e^{\hat\beta_0},\ \hat B = \hat\beta_1
$$

We may also obtain a closed-form survival function which can be used to
close the life table at high ages:

$$
{}_tp_x = \exp\!\Big(-\frac{A}{B}\,e^{Bx}\big(e^{Bt}-1\big)\Big)
$$

This produces three ways to fit, in increasing rigour:

  ------------ ----------- ------------------------------------------------------------ -----------------------------------
  Method       What it     MATLAB                                                       Pros and cons
               minimises                                                                

  OLS on log m Σ (ln m −   `polyfit(x, log(m), 1)` or `fitlm`                           Simplest, and the brief asks for
               ln μ̂)²                                                                   it. Treats a noisy age-95 rate as
                                                                                        just as reliable as a precise
                                                                                        age-60 rate.

  Weighted     Σ D(x) (ln  `fitlm(..., 'Weights', D)`                                   Weight by D because Var(ln m̂) ≈
  least        m − ln μ̂)²                                                               1/D. A cheap, large improvement.
  squares on                                                                            
  log m                                                                                 

  Poisson      −Σ \[D      `fitglm(x, D, 'Distribution','poisson', 'Offset', log(E))`   The statistically correct model for
  maximum      ln(Eμ) −                                                                 counts. Gives standard errors and
  likelihood   Eμ\]                                                                     deviance directly.
  ------------ ----------- ------------------------------------------------------------ -----------------------------------

The Poisson version is still log-linear in its parameters:
$\ln \mathrm{E}[\mathrm{D}]=\ln \mathrm{E}+\ln \mathrm{A}+\mathrm{Bx}$.
So `fitglm` fits Gompertz exactly, with In E as an offset. Doing all
three and comparing them is an easy way to expand on the report.

## Choosing the age range for Gompertz

We can use Gompertz for describing adult mortality only. So we fit over
roughly ages 30-90 and then show what happens outside that range:

-   **Below about 30:** infant mortality and the accident hump sit far
    above the Gompertz line.
-   **Above about 90:** observed mortality bends below the line
    (late-life deceleration). An exponential will over-predict deaths
    there and under-predict survival at very old ages. Remember that the
    data are also noisiest there.

## Residuals and goodness of fit

-   **Log-scale residual:** $r(x) = \ln m(x) - \ln \hat{\mu}(x)$. Plot
    it against age. Random scatter is good; an S-shape or a U-shape
    means the curve has the wrong form.
-   **Poisson (Pearson) residual:**
    $D - Ẹ\hat{\mu}(x)/ \sqrt{E\hat{\mu}(x)}$. If the model is right,
    these are roughly $\mathrm{N}(0,1)$, so $|\mathrm{r}|>2$ should be
    rare.
-   $\mathrm{RSS}=\Sigma \mathrm{r}^2$ and RMSE
    $=\sqrt{(\mathrm{RSS} / \mathrm{n})}$. Always say which scale you
    used. The log scale and the raw-rate scale give very different
    numbers, and raw-scale RMSE is dominated by the oldest ages.
-   For comparing models with different numbers of parameters, use
    $\mathrm{AlC}=2 \mathrm{k}-2 \mathrm{ln} \mathrm{L}$, BIC, or a
    likelihood-ratio test. RSS alone always favours the bigger model.

## Gompertz - Makeham : $\mu(x) = C + Ae^{Bx}$

We update our equation to have $C$ as an age independent background
hazard. These are accidents and other causes unrelated to ageing. It
matters at 20-45, where the Gompertz term is tiny but barely matters at
80.

Doing this makes our model no longer linear after taking logs, we must
use non-linear fittings such as : `fitnlm`, `lsqcurvefit`, or
`fminsearch` on the Poisson negative log-likelihood. We will use the
Gompertz estimates as starting values, with $C_0 = 0.0005$.

Furthermore, since $C$ and $A$ are correlated, and $C$ is small, its
standard error can be large and so we must constraining $C \geq 0$ or
fit $\log C$.

## Expectations across sexes and years

-   **Males have a higher $A$ and $B$is often similar or slightly lower
    for males.**
-   **$A$ typically falls and $B$ tends to rise**: mortality falls most
    at younger adult ages, so the log-line gets steeper. The negative
    $A$-$B$ is called the 'compensation effect'

There exist other models for this modelling to:

-   Perks/logistic (they plateau at old ages)
-   Kannisto (old ages)
-   Heligman--Pollard (8 parameters covering the full age range).

  ----------------------------------------------------------------------------------------------------------------------
  \# Mortality improvements and forecasting
  ----------------------------------------------------------------------------------------------------------------------
  \# Annuity Valuation

  An annuity value is a sum of discounted payments, each weighted by the probability the person is alive to receive it.
  Interest and mortality are the only two assumptions. Your job is to show how much each one moves the price.

  \## The building blocks

  \- Discounting : £1 paid in $t$ years is worth $v^t$ today, where $v = \frac{1}{1+i}$ is the discount factor and $i$
  is the annual interest rate. A higher interest rate means a smaller present value, and the effect is strongest for
  distant payments - Multi Year Survival : Surviving $t$ years means surviving each single year in turn:

  $$
  {}_tp_x = \prod_{j=0}^{t-1} p_{x+j} = \frac{l_{x+t}}{l_x}
  $$

  \## Deriving the annuity formula

  Let $I_{t} = 1$ if the person aged $x$ years is a live at time $t$ and 0 otherwise. The present value of the payments
  is a rangom variable $\sum v^{I_t})$ and its expectation is therefore:

  $$
  a_x = \mathbb{E}\Big[\sum_{t\ge1} v^t I_t\Big] = \sum_{t=1}^{\omega-x} v^t\,{}_tp_x
  $$

  After the table's last age $\omega$, the sum stops since the survival is beyond its zero.

  \## Recursion Implementation:

  We have a recursive formula as:

  $$
  a_x = v\,p_x\,\big(1 + a_{x+1}\big), \qquad a_\omega = 0, \qquad \ddot a_x = 1 + a_x
  $$

  We work backwards from the top age, one loop gives $a(x)$ at for every age at once. $\ddot{a}$ denotes the
  annuity-due, paid at the start of each year.

  \## Unit Tests:

  \- At $i=0$, $a(x)$ equals the curtate life expectancy $e(x)$ - $a(x) < \frac{1}{i}$, the value of a perpetuity -
  $a(x)$ falls as $i$ rises and usually falls as $x$ rises - If every $p=1$ up to age $\omega$, $a(x)$ equals the
  annuity-certain value $\frac{1-v^n}{i}$, with $n = \omega -x$

  \## Interest-rate sensitivity

  Partially differentiating with respect to interest rates:

  $$
  \frac{\partial a_x}{\partial i} = -\sum_{t} t\,v^{t+1}\,{}_tp_x < 0, \qquad D_x = \frac{\sum_t t\,v^t\,{}_tp_x}{a_x}
  $$

  The value is decreasing and convex in $i$. $D_x$ is the duration and is the survival-weighted average time to payment.
  It explains why younger annuitants are more sensitive to interest rates as their payments are further away.

  \## Constant vs Improving Mortality

  \- Constant (period) - $p_{x,t}$ comes from one year's table - Improving (cohort) - $p_{x,t}$ comes from the projected
  diagonal, so each future year uses that year's rates:

  $$
  {}_tp_x^{\text{cohort}} = \prod_{j=0}^{t-1}\big(1 - q_{x+j,\,T+j}\big)
  $$

  Expect improving mortality to raise a(65) by a few percent. The effect is larger at younger ages and lower interest
  rates, where payments run further into the future. Report the difference as a percentage:

  $$
  \frac{a_{\text{improving}} - a_{\text{constant}}}{a_{\text{constant}}} \times 100
  $$

  \## Joint sensitivity analysis

  For the report, we are going to evaluate $a(65)$ over a grid: interest $i$ from 0% to 6% in steps of 0.5%, and
  improvement from 0% to 3% a year (or multiples of your base $\rho$). Show the result as a heatmap or contour plot. The
  point to make is that the effects interact: mortality assumptions matter more when interest rates are low.

  \## Financial Implications to discuss

  \- Annuity and pension payments are fixed when the contract is sold. If people live longer than assumed, the provider
  pays for longer than it reserved for. - Longevity risk is *systematic*. It cannot be diversified away by writing more
  policies, because every policy is affected by the same trend. Only the random, individual-level part averages out. - A
  small percentage error times a large book is a large sum. A 3% understatement on £10bn of liabilities is £300m. - How
  the industry responds: capital requirements (the UK Solvency regime), stress tests, and transferring the risk through
  longevity swaps, buy-ins and buy-outs.
  ----------------------------------------------------------------------------------------------------------------------

# Validation and Uncertainty

A model earns trust by predicting years it never saw. While compiling
this report, we will likely conclude that parameter uncertainty is tiny,
because national data are huge, while the choice of trend and model
dominates the uncertainty in money terms.

## Out-of-sample testing

-   Split by *time*, never at random. For example: train on 1961--2009,
    test on 2010--2019, and treat 2020--21 as a separate stress test.
-   For a stronger test, use a rolling origin: refit with data up to
    year T, forecast T+1 to T+10, move T forward and repeat. Plot the
    error against the forecast horizon.
-   "Fitting on training data" means estimating everything (Gompertz A
    and B for each year, their trends, the improvement rates ρ) from the
    training years only.

## Error measures

  ----------------- ----------------- ----------------- ------------------
  Measure           Formula           Strength          Weakness

  RMSE on the log   √ mean (ln m̂ − ln Treats            Hard to explain to
  scale             m)²               proportional      non-specialists
                                      errors equally    
                                      across ages       

  MAE               mean \|m̂ − m\|    Robust and in the Dominated by the
                                      data's units      oldest ages

  MAPE              mean \|m̂ − m\| /  Scale-free        Unstable when m is
                    m                 percentage        tiny; penalises
                                                        over- and
                                                        under-prediction
                                                        unequally

  Mean signed log   mean (ln m̂ − ln   Shows the         Positive and
  error (bias)      m)                *direction* of    negative errors
                                      error             can cancel out

  Poisson deviance  2Σ \[D ln(D/Eμ̂) − Statistically     Least intuitive
                    (D − Eμ̂)\]        principled;       
                                      weights by        
                                      information       

  Error in e(65) or ê − e             Measures what     Errors at
  a(65)                               actually matters  different ages can
                                      financially       offset
  ----------------- ----------------- ----------------- ------------------

## Looking for systematic bias

Plot the signed log errors as a heatmap of age by test year. If the
training period ended before the post-2011 slowdown, expect the models
to have *under*-estimated mortality. That means overstated life
expectancy and annuity values, mainly at ages 50--85.

## Confidence intervals for parameters

-   From the regression: β̂ ± t(0.975, n−2) · SE(β̂). In MATLAB, use
    `coefCI(mdl)`.
-   For A = e\^β₀, transform the endpoints of the β₀ interval. The
    result is not symmetric.
-   From the Poisson model: Wald intervals from the Fisher information.
    Profile-likelihood intervals are better for Makeham's C.
-   **Interpretation:** if the data-generating process were repeated
    many times, 95% of intervals built this way would contain the true
    parameter. Here, the "repetition" is the Poisson randomness in
    deaths. The interval does not cover the risk that the model itself
    is wrong.

## Propagating parameter uncertainty into money

1.  Draw N = 5,000 parameter vectors θ\* \~ N(θ̂, Σ̂), using `mvnrnd` or
    `chol(Σ)' * randn`.
2.  For each draw, rebuild μ(x), the life table, e(65) and a(65).
3.  Report the mean and the 2.5th and 97.5th percentiles, and plot a
    histogram.
4.  Alternative: a parametric bootstrap. Simulate D\* \~ Poisson(E · m̂)
    with `poissrnd`, refit, and recompute.
5.  Extension: simulate Lee--Carter k(t) paths with `randn` to get
    *forecast* uncertainty. These intervals will be far wider than the
    parameter intervals.

## Three kinds of uncertainty

  ----------------- ----------------- ----------------- -----------------
  Type              Source            How you measure   Typical size in
                                      it                this project

  Statistical       Random timing of  Simulate          Large for 100
  (process)         individual deaths lifetimes with    lives; negligible
                                      `rand` for a      for a nation
                                      finite portfolio  

  Parameter         Estimation error  CIs, Monte Carlo, Small, because
                    in A, B, C, ρ     bootstrap         national
                                                        exposures are in
                                                        the millions

  Model             Wrong functional  Compare Gompertz, Usually the
                    form, base period Makeham and       largest
                    or projection     Lee--Carter, and  
                    method            different base    
                                      periods           
  ----------------- ----------------- ----------------- -----------------

Rank your assumptions by their effect on a(65) with a tornado chart:
interest rate, improvement rate, base period, model choice, top-age
treatment, and a(x).

## Why a good historical fit can still forecast badly

-   **Structural breaks.** The post-2011 slowdown and COVID-19 change
    the process itself.
-   **Extrapolation.** Forecasting goes beyond the range of the data,
    where nothing constrains the curve.
-   **Overfitting.** Extra parameters fit noise in the training years,
    not signal.
-   **Non-stationarity.** Medical, social and economic drivers of
    mortality change over time, and the model's form is fixed.
