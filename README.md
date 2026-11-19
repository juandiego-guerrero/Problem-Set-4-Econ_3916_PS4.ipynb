# Problem-Set-4-Econ_3916_PS4.ipynb
Problem Set 4

**Role:** Analyst, Revenue Forecasting

## The mission

I inherited a store-revenue model whose handover note read: *"Predicts monthly revenue by store. R-squared 0.94. Validated."* Since July it has consistently over-forecast the stores that matter most. This project finds out what "validated" meant, establishes what the model can actually do, and hands back a version whose stated accuracy is the accuracy it will have next month.

## Repository contents

| File | Description |
|---|---|
| `Econ_3916_PS4.ipynb` | Full notebook: simulation, diagnostics, model selection, AI phase and handover note |
| `README.md` | This file |

## How to run

1. Open the notebook in Google Colab.
2. Run **Runtime → Restart and run all**. Every random draw comes from one generator (`np.random.default_rng(42)`), so the cells must run once, in order, to reproduce the numbers below. Task 4.2 uses its own fixed seed (`default_rng(0)`).
3. Libraries: `numpy`, `pandas`, `matplotlib`, `scipy`, `statsmodels`, `scikit-learn`. All are preinstalled in Colab.

## Phase 1 — Reproduce the 0.94

- **Simulated panel:** 400 stores. The true drivers are floor area, foot traffic, local median income and years open. Three predictors are pure noise, and one (`trade_area_index`) is a near-linear combination of foot traffic and income. Revenue flattens above 15,000 sq ft, and its variance grows with store size. The full data-generating process is recorded in the notebook.
- **In-sample R² = 0.928.** Held-out R² averaged over 500 random splits is 0.919, a gap of only **+0.008**. The model barely overfits, so overfitting cannot explain the July failure.
- **R² never falls when a predictor is added.** Three pure-noise columns raised R² (+0.00014) while adjusted R² fell (−0.00043).

## Phase 2 — Diagnose the failure

| Violation | Evidence | Effect on estimate | Effect on SE |
|---|---|---|---|
| Non-constant variance | Residual SD rises 15 → 82 across fitted quintiles (Breusch–Pagan p ≈ 10⁻²²) | Unbiased, inefficient | Classical SEs 6–27% too small |
| Missed non-linearity | Arch in residuals vs floor area (RESET p ≈ 10⁻⁵⁶) | Biased | Robust SEs can't fix it |
| Influential observations | 18 stores with Cook's D > 4/n, median 22,793 sq ft | Unstable | Unreliable |

- **The over-forecast is systematic in store size.** The largest decile is over-forecast by **\$41.8k/month** (4.7%), and stores above 25,000 sq ft by **\$123k/month** (12%). The pattern holds out-of-fold.
- **Collinearity is not the cause.** VIFs reach 861, which inflates individual SEs and flips coefficient signs. But dropping `trade_area_index` changes R² by 0.0002 and leaves the big-store over-forecast unchanged.

## Phase 3 — Build something honest

- **Complexity sweep:** held-out RMSE is U-shaped. It turns at all predictors plus squares (43.3) and climbs to 84.3 at the full cubic. Bias dominates to the left of the turn, variance to the right.
- **CV selection:** repeated 10-fold CV picks **all predictors plus squares and cubes**, with RMSE **38.3** (95% CI 26.8–49.8) against 47.4 for the handover model.
- **Selection honesty:** nested CV gives **40.2 \$k/month (95% CI 27.5–52.9), about 7% of mean revenue**. The naive CV score was 5% optimistic.

## Phase 4 — AI expansion

- **Learning curve (Task 4.1):** the handover model converges at 47.7, far above the 28.9 noise floor, which means high bias: better features beat more data. The result agrees with the Phase 3 diagnosis. The unedited AI transcript, the model, the date and my changes are in the notebook.
- **What "validated" meant (Task 4.2):** the analyst ran `SelectKBest` on the full sample before cross-validating. On 300 pure-noise columns that cell reports **R² = 0.09**; inside a `Pipeline` the result is **−0.22**, a gap of **+0.32**. Scaling was a decoy: it is harmless for OLS with an intercept.
- **Handover note (Task 4.3):** under 250 words, at the end of the notebook.

## Key takeaway

The 0.94 was real but irrelevant. It was measured in-sample, "validated" by a leaky pipeline, and blind to a systematic error in exactly the stores that matter most. The replacement model's stated error, about \$40k/month, is the error it should have next month, provided new stores resemble these and are not larger than any in the sample.

## AI use

Claude Opus 5.5 (Anthropic) was used as a coding and writing assistant. The Task 4.1 exchange is reproduced unedited in the notebook, as the brief requires.
