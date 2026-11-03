# Measuring Incremental Marketing Lift: Attributed vs. Causal Conversions

Ad platforms count every conversion that follows an ad as caused by it, which overstates what a campaign earned and leads to overspending. A platform's conversion count and the ad's causal effect are different quantities, and the gap between them is where budget decisions go wrong. This project measures that gap on a real holdout experiment. It then uses simulated data, where the true answer is known, to test which estimation methods can be trusted.

![Attributed vs incremental](figures/attributed_vs_incremental.png)

## Key results
**Part 1: Real holdout experiment** (588,101 users; ad vs. a public-service-announcement control)

| | Ad | PSA (control) |
|---|---|---|
| Users | 564,577 | 23,524 |
| Conversion rate | 2.555% | 1.785% |

- **Lift:** the ad raised conversion by **+0.77pp** (95% CI 0.60–0.94pp; z-test p < 0.001), about 43% relative.
- **Attributed vs. incremental:** only **~30% of attributed conversions were incremental**, about 4,343 of 14,423 (95% CI ~3,370–5,320). The platform's count overstates the ad's effect by more than 3x.
- **What the control saw:** the control group saw a PSA, not no ad, so this measures the ad's effect *relative to a PSA*.

**Part 2: Simulation** (true lift = +2pp; 500 replications at three confounding levels)

| Confounding | Naive bias | IPW bias | PSM bias |
|---|---|---|---|
| None | −0.01pp | — | — |
| Moderate | +2.49pp | −0.01pp | +0.01pp |
| Strong | +4.13pp | 0.00pp | +0.01pp |

- **Naive comparison:** when exposure isn't random, comparing exposed and unexposed users overstated the lift by 2.5–4.1pp. Its 95% confidence interval never contained the true value: precise, and wrong.
- **IPW and PSM:** both recovered the true effect within 0.02pp at every level. The cost is more noise, with RMSE roughly doubling as confounding grows.

![Estimates by confounding](figures/estimates_by_confounding.png)

## Approach
- **Two parts:** real data shows the size of the gap, but the true effect is unknown. Simulated data has a known answer, so it shows whether a method gets it right.
- **Estimators compared:**
  - *naive* (exposed vs. unexposed);
  - *randomized holdout*;
  - *IPW* (inverse probability weighting: reweights users so the exposed and unexposed groups look alike on the confounder);
  - *PSM* (propensity score matching: pairs each exposed user with a similar unexposed user).
- **Diagnostics:**
  - *Covariate balance:* the confounder's standardized mean difference drops from 0.71–1.27 to about 0 after adjustment.
  - *Propensity overlap:* confirmed.
  - *Placebo test:* IPW and PSM estimate about 0 on an outcome with no true effect, while the naive method finds a false one.
- **Power analysis:** with the real 24:1 split, the smallest detectable lift is about 0.26pp at the observed 1.79% baseline.

## Limits
- **Real data:** assumes random assignment, no interference between users, and no differential attrition. None of these can be verified from the data alone.
- **Simulation:** this is the best case: one observed confounder, a correctly specified model and a constant effect. IPW and PSM cannot fix confounders that aren't measured.
- **No ROI:** none is claimed, because the dataset has no cost or value-per-conversion data.

## Run it
1. Save the [Kaggle "Marketing A/B Testing" CSV](LINK) as `data/raw/marketing_AB.csv`
2. `pip install -r requirements.txt` (Python 3.11+)
3. `make all`: runs the tests, the simulation (~15–25 min) and both notebooks. Seeds are fixed, so results reproduce.

## Stack
Python · pandas · NumPy · SciPy · statsmodels · matplotlib · seaborn · pytest · Jupyter
