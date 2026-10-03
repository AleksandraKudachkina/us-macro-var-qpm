US Monetary Policy: VAR, QPM-style Model and Shock Analysis

A small macroeconomic modeling project on US data.
The goal is to understand how the **output gap**, **inflation** and the **Federal Funds Rate** interact, and how simple macro models are used for forecasting and policy scenario analysis.

The project follows a typical macro analyst workflow:

**FRED data → output gap → VAR → impulse responses → forecast vs benchmark → QPM → +100 bp shock scenario**

> This is an educational portfolio project, not a full central-bank model. The focus is on understanding the economic logic, the methods and their limitations.

---

## Project structure
├── output/
│ └──figures
├── notebooks/
│ └── 01_us_macro_var_qpm.ipynb # main notebook
├── requirements.txt
└── README.md


---

## Data

Quarterly US data from **FRED** (Federal Reserve Bank of St. Louis), 1990 – 2026Q2.

| Variable | FRED code | Use |
|---|---|---|
| Real GDP | GDPC1 | output gap |
| Potential GDP (CBO) | GDPPOT | output gap |
| PCE price index | PCEPI | inflation (YoY) |
| Federal Funds Rate | FEDFUNDS | policy rate |
| Unemployment rate | UNRATE | context |
| WTI oil price | DCOILWTICO | oil inflation (VAR) |
| Broad dollar index | DTWEXBGS | context |

Monthly and daily series are converted to quarterly averages. Potential GDP projections beyond the sample end are not used.

Output gap:

$$x_t = 100 \cdot (\log Y_t - \log Y_t^*)$$

---

## 1. Macro overview

![Macro overview](figures/01_macro_overview.png)

The data show the main macro episodes: the Global Financial Crisis, the zero lower bound period, the COVID shock, the 2021–2023 inflation surge and the Fed tightening cycle.

![Oil and dollar](figures/02_oil_dollar.png)

---

## 2. VAR model

A 4-variable VAR with Cholesky identification:

**oil inflation → output gap → inflation → Fed Funds Rate**

- Oil prices are ordered first: they behave as an external shock and are not explained by any domestic variable in the model.
- The Fed Funds Rate is ordered last: the Fed can react to current conditions within the quarter, while the economy reacts to policy with a lag.
- All information criteria (AIC, BIC, FPE, HQIC) select **2 lags**. The VAR is stable.

**Key findings from the coefficients:**
- All variables are highly persistent (the policy rate especially: sum of own lags ≈ 0.95, the Fed moves gradually).
- Oil inflation significantly predicts both PCE inflation and the Fed Funds Rate. Including oil is important to separate monetary policy shocks from the Fed's reaction to energy prices.
- The policy rate has no significant effect on inflation within 1–2 quarters, consistent with long lags of monetary policy.

### Impulse responses to a Fed Funds shock

![VAR IRF](figures/03_var_irf.png)

- **Fed Funds Rate:** +0.27 p.p. on impact, peak ≈ 0.58 p.p. after 3–4 quarters, then fades.
- **Inflation:** declines after the shock, minimum ≈ −0.07 p.p. after about 8 quarters.
- **Output gap:** small positive response first, then gradually declines and turns negative by the end of the horizon.

Cholesky shocks depend on the ordering, so these results are interpreted as illustrative rather than precise causal estimates.

---

## 3. Forecast vs random walk

Out-of-sample test: the VAR is estimated on data up to 2023Q2 and forecasts the next 12 quarters. The benchmark is a random walk ("no change" forecast).

| Variable | VAR RMSE | Random walk RMSE | VAR / RW |
|---|---|---|---|
| Oil inflation | 20.66 | 38.74 | 0.53 |
| Output gap | 0.49 | 0.57 | 0.86 |
| Inflation | 0.57 | 1.19 | **0.48** |
| Fed Funds Rate | 1.67 | 0.75 | **2.21** |

![VAR forecast](figures/04_var_forecast.png)

- The VAR beats the random walk for inflation, the output gap and oil.
- The VAR is much worse for the policy rate: it extrapolated the 2022–2023 tightening and predicted rates rising to ~6.7%, while the Fed held rates and then started cutting. A statistical model cannot see forward-looking policy decisions.

---

## 4. QPM-style model

A small semi-structural model with three equations.

**IS curve (demand):**

$$x_t = \alpha_x \, x_{t-1} - \alpha_r \left( i_{t-1} - \pi_{t-1} - r^* \right)$$

**Phillips curve (inflation):**

$$\pi_t = \beta_\pi \, \pi_{t-1} + (1 - \beta_\pi)\,\pi^* + \beta_x \, x_{t-1}$$

**Policy rule (Taylor rule with smoothing):**

$$i_t = \rho \, i_{t-1} + (1 - \rho)\left[ r^* + \pi^* + \phi_\pi (\pi_t - \pi^*) + \phi_x \, x_t \right]$$

### OLS diagnostics and calibration

| Result | Interpretation |
|---|---|
| High persistence: output gap 0.81, inflation 0.89, rate 0.93 | the economy, prices and policy change slowly |
| Long-run Fed response to inflation ≈ 1.07 | above 1, consistent with the Taylor principle |
| Real rate effect on output gap: +0.05, insignificant | wrong sign due to endogeneity: the Fed raises rates when the economy is strong |
| Phillips curve slope: 0.02, insignificant | the US Phillips curve is very flat |

Because OLS cannot properly identify how rates affect the economy, the scenario model is **calibrated** with economically consistent signs:

| Parameter | Value | Meaning |
|---|---|---|
| α_x | 0.65 | output gap persistence |
| α_r | 0.15 | effect of the real rate on demand |
| β_π | 0.80 | inflation persistence |
| β_x | 0.08 | Phillips curve slope |
| ρ | 0.85 | interest-rate smoothing |
| φ_π | 1.50 | response to inflation |
| φ_x | 0.50 | response to output gap |

---

## 5. Scenario: unexpected +100 bp tightening

**Starting point (2026Q2):** output gap 1.39%, inflation 3.55%, Fed Funds Rate 3.63%.

- **Baseline:** inflation gradually returns to ~2% in 12 quarters, the output gap closes and turns slightly negative, the policy rate gradually declines. A "soft landing".
- **Shock:** +100 bp hike in quarter 1. The effect is measured as the difference between the shock scenario and the baseline.

![QPM scenario](figures/05_qpm_shock_scenario.png)

| | Starts reacting | Maximum effect | When |
|---|---|---|---|
| Fed Funds Rate | quarter 1 | +1.00 p.p. | quarter 1 |
| Output gap | quarter 2 | −0.25 p.p. | quarters 4–5 |
| Inflation | quarter 3 | −0.07 p.p. | quarter 9 |

The transmission chain with lags: **rate → demand → prices**.
The inflation effect is small because the Phillips curve is flat. The timing is consistent with the VAR (inflation minimum after ~8–9 quarters in both models).

---

## VAR vs QPM

| | VAR | QPM |
|---|---|---|
| Type | reduced-form, statistical | semi-structural |
| Question | how did variables move together? | what happens under a scenario? |
| Strength | data-driven, good forecast benchmark | clear economic story |
| Weakness | shocks depend on ordering | results depend on calibration |

---

## Limitations

- The output gap uses latest-vintage CBO potential GDP (revised, ex-post measure).
- VAR shocks are identified with Cholesky ordering, not a perfect causal estimate.
- The QPM is small and backward-looking: no inflation expectations, oil or exchange rate channels.
- The neutral real rate is a historical average (−0.77%), likely too low compared to official estimates.

## Next steps

- Bayesian VAR
- A separate small New Keynesian DSGE notebook

---

## How to run

```bash
pip install -r requirements.txt
jupyter notebook notebooks/01_us_macro_var_qpm.ipynb
