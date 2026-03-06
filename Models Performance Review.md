# Models Performance Review

## Coverage

- **Model outputs reviewed:** `OLS`, `OLS3`, `ENET`, `PLS`, `PCR`
- **Prediction horizon reviewed:** 180 monthly out-of-sample periods from **2010-08-31** to **2025-07-31**
- **Full-sample prediction rows per model:** 34,515
- **Common artifact families used:** `r2`, `portfolio`, `benchmark`, `predictions`, `windows`, `importance`, `complexity`
## Key Takeaways

- **Best predictor on the headline squared-error metric:** `ENET`
- **Best investable long-short model:** `ENET`
- **Best long-only model:** `OLS`
- **Most stable model across rolling windows:** `ENET`
- **Weakest completed model in this batch:** `OLS3`

### My takes

1. `ENET` is the strongest completed model overall. It has the best full-sample out-of-sample `R²`, the strongest long-short net Sharpe after 30 bps cost, the highest benchmark information ratio, and the best cumulative long-short path.
2. `OLS` is not the best predictor on `R²`, but it remains a strong long-only model. Its long-only net Sharpe is the highest in this completed batch even though its `R²` is negative. That means portfolio payoff and point-forecast fit are not the same thing in these runs.
3. `PLS` and `PCR` are credible middle-ground models. They do not beat `ENET`, but both deliver usable long-short Sharpe above 1 and materially better results than `OLS3`.
4. `OLS3` is materially weaker than the richer linear alternatives. In this dataset, the 3-factor restricted specification is not capturing enough cross-sectional variation to compete.
5. The model ranking changes depending on the objective. If the goal is **prediction fit**, `ENET` is best. If the goal is **long-only portfolio quality**, `OLS` is strongest among completed runs. If the goal is **best overall tradeable signal**, `ENET` is the best current choice.

## Quick Comparison Table

| Model | R2 OOS Full | R2 OOS Large | R2 OOS Small | Mean Rank IC | Long-only Sharpe | Long-short Sharpe | Benchmark IR |
| ----- | ----------- | ------------ | ------------ | ------------ | ---------------- | ----------------- | ------------ |
| ENET  | 0.0044      | 0.0039       | -0.0003      | 0.0785       | 0.9705           | 1.6960            | 0.2467       |
| OLS   | -0.0143     | -0.0588      | -0.0326      | 0.1044       | 1.2651           | 1.3013            | 0.0341       |
| PCR   | 0.0025      | 0.0004       | -0.0014      | 0.0545       | 0.8250           | 1.1464            | 0.0976       |
| PLS   | 0.0008      | -0.0019      | -0.0034      | 0.0724       | 0.8423           | 1.1394            | 0.1408       |
| OLS3  | -0.0113     | -0.0116      | -0.0124      | 0.0306       | 0.3251           | -0.0542           | -0.2443      |
%% Although short selling is prohibited in VN, for the purpose of comparison I still include it in %%

---
*Quick Notations and Formulas:*
1. Information coeficients (IC)
$$
  IC_t = \mathrm{corr}(\hat y_t,\ y_t)
  $$
  where for each date $t$:
  - $\hat y_t$ = model scores or predicted returns across stocks,
  - $y_t$ = realized returns across the same stocks.
  Then:  $$
  \text{Mean IC} = \frac{1}{T}\sum_{t=1}^{T} IC_t
  $$
1. Information Ratio (IR)
*Benchmark IR (information ratio) is calculated as:*  
$$
AR_t = R^{strat}_t - R^{bench}_t,
\quad
IR = \frac{\overline{AR}}{\sigma(AR)}.
$$*
$R^{bench}_t$ here I take the VN30 index (a portfolio with 30 biggest market capitalization at at the time of the test period, and re-balance every 6 month, equally weighted)

---
## Prediction Quality

![[assets/pipeline3_interim_model_review_2026-03-06/r2_oos_by_model.png]]

![[assets/pipeline3_interim_model_review_2026-03-06/mean_rank_ic_by_model.png]]

![[assets/pipeline3_interim_model_review_2026-03-06/r2_vs_ls_sharpe.png]]

### Prediction metrics table

| Model | R2 OOS Full | Pearson(y,yhat) | Spearman(y,yhat) | Mean Rank IC | Share Positive IC | Sign Accuracy |
| --- | --- | --- | --- | --- | --- | --- |
| ENET | 0.0044 | 0.0585 | 0.0535 | 0.0785 | 71.7% | 51.5% |
| OLS | -0.0143 | 0.0629 | 0.0850 | 0.1044 | 76.7% | 53.6% |
| PCR | 0.0025 | 0.0529 | 0.0515 | 0.0545 | 68.9% | 51.1% |
| PLS | 0.0008 | 0.0551 | 0.0549 | 0.0724 | 70.0% | 51.4% |
| OLS3 | -0.0113 | 0.0182 | 0.0350 | 0.0306 | 56.7% | 52.7% |

###### *Interpretation*

- `ENET` is the only completed model with clearly positive `R²` in both the **full** and **large-firm** samples.
- `PCR` is also positive on full and large samples, but its `R²` is weaker than `ENET`.
- `PLS` is barely positive on full sample and negative on large/small subsamples.
- `OLS` has the **highest mean rank IC** and **highest sign accuracy**, but still negative `R²`. That is a concrete example of why portfolio sorting metrics and squared-error fit can disagree.
- `OLS3` is weak on every predictive dimension reviewed here.
###### *These results so far tell me*

- “Predict the best” as **highest out-of-sample `R²`**, the answer is `ENET`.
- **Best monthly ranking ability**, `OLS` looks better than its `R²` suggests.
- The safest interpretation is that `ENET` is the best balanced predictor, while `OLS` is a noisier but sometimes economically stronger ranking signal.

## Portfolio Performance

![[assets/pipeline3_interim_model_review_2026-03-06/sharpe_by_model.png]]

![[assets/pipeline3_interim_model_review_2026-03-06/cumulative_ls_net30.png]]

![[assets/pipeline3_interim_model_review_2026-03-06/cumulative_long_net30.png]]

### Net 30 bps portfolio summary

| Model | Long Ann Ret | Long Sharpe | Long Cum Ret (x) | Long Max DD | LS Ann Ret | LS Sharpe | LS Cum Ret (x) | LS Max DD | LS Mean Turnover |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| ENET | 28.4% | 0.970 | 41.434 | -51.0% | 40.3% | 1.696 | 160.073 | -26.2% | 1.330 |
| OLS | 28.8% | 1.265 | 43.343 | -36.3% | 32.8% | 1.301 | 69.351 | -37.6% | 1.639 |
| PCR | 22.4% | 0.825 | 19.805 | -63.1% | 26.5% | 1.146 | 33.100 | -50.1% | 1.219 |
| PLS | 23.2% | 0.842 | 21.752 | -57.5% | 26.8% | 1.139 | 34.297 | -48.1% | 1.322 |
| OLS3 | 5.1% | 0.325 | 1.114 | -60.2% | -4.5% | -0.054 | -0.498 | -74.8% | 1.055 |

###### *Interpretation*

- `ENET` is decisively best on the **long-short** portfolio dimension. Its equal-weight long-short strategy posts the highest annualized return, highest Sharpe, and strongest cumulative return after 30 bps cost.
- `OLS` is the best **long-only** model in this completed group by net Sharpe and annualized return.
- `PLS` and `PCR` remain investable in the sense that both produce long-short Sharpe ratios above 1.1 after cost, but neither matches `ENET`.
- `OLS3` is not competitive. Its long-short strategy is negative after cost and its drawdown is the worst of the completed set.

## Benchmark Relative Results

![[assets/pipeline3_interim_model_review_2026-03-06/benchmark_ir_equal.png]]

![[assets/pipeline3_interim_model_review_2026-03-06/benchmark_active_return_equal.png]]

| Model | IR vs EW Benchmark | Mean Active Return | Corr vs Benchmark | IR vs VW Benchmark |
| --- | --- | --- | --- | --- |
| ENET | 0.247 | 2.05% | 0.182 | 0.329 |
| OLS | 0.034 | 0.31% | -0.043 | 0.023 |
| PCR | 0.098 | 0.85% | 0.090 | 0.107 |
| PLS | 0.141 | 1.20% | 0.098 | 0.156 |
| OLS3 | -0.244 | -2.06% | 0.074 | -0.223 |

###### *Interpretation*

- `ENET` is the strongest benchmark-relative strategy by a clear margin.
- `PLS` and `PCR` are second tier but still positive.
- `OLS` is only slightly positive against the equal-weight benchmark.
- `OLS3` is negative against both equal-weight and value-weight benchmark comparisons.

## Rolling Window Stability

![[assets/pipeline3_interim_model_review_2026-03-06/window_stability_heatmap.png]]

| Model | Mean Window R2 | Median Window R2 | Share Positive Windows | Best Window R2 | Worst Window R2 |
| --- | --- | --- | --- | --- | --- |
| OLS | -0.0149 | -0.0011 | 46.7% | 0.0565 | -0.1631 |
| OLS3 | -0.0115 | -0.0038 | 33.3% | 0.0574 | -0.1242 |
| ENET | 0.0013 | 0.0009 | 53.3% | 0.0467 | -0.0307 |
| PLS | -0.0030 | -0.0021 | 40.0% | 0.0486 | -0.0499 |
| PCR | -0.0007 | -0.0043 | 40.0% | 0.0492 | -0.0326 |

###### *Interpretation*

- `ENET` is the only completed model with **positive mean full-sample window `R²`** and more than half of windows positive.
- `OLS` has the best single window but also much worse downside window risk than `ENET`. It is more fragile over time.
- `PCR` and `PLS` are middling and slightly negative on average window `R²`, but less erratic than `OLS`.
- `OLS3` is consistently weak.

## Turnover Cost Sensitivity

![[assets/pipeline3_interim_model_review_2026-03-06/cost_sensitivity_ls_sharpe.png]]

| Model | LS Sharpe 0bps | LS Sharpe 10bps | LS Sharpe 20bps | LS Sharpe 30bps | Long Sharpe 30bps |
| --- | --- | --- | --- | --- | --- |
| OLS | 1.536 | 1.458 | 1.380 | 1.301 | 1.265 |
| OLS3 | 0.098 | 0.047 | -0.003 | -0.054 | 0.325 |
| ENET | 1.916 | 1.843 | 1.770 | 1.696 | 0.970 |
| PLS | 1.336 | 1.271 | 1.205 | 1.139 | 0.842 |
| PCR | 1.330 | 1.269 | 1.208 | 1.146 | 0.825 |

###### *Interpretation*

- `ENET` remains the best long-short model even after 30 bps turnover cost.
- `OLS` starts from a strong gross result but degrades faster than `ENET` once cost is applied.
- `PLS` and `PCR` are moderately resilient to cost, but still well below `ENET`.
- `OLS3` is effectively non-viable once cost is introduced.

## Feature Importance and Model Shape

![[assets/pipeline3_interim_model_review_2026-03-06/feature_importance_heatmap.png]]

![[assets/pipeline3_interim_model_review_2026-03-06/complexity_by_model.png]]

### Top features by model

| Model | Top 1 | Top 2 | Top 3 | Top 4 | Top 5 |
| --- | --- | --- | --- | --- | --- |
| ENET | mom1m (0.789) | Shares_Out (0.566) | cfp (0.039) | ROE_Reported (0.000) | EBITDA (0.000) |
| OLS | FCF (0.120) | US_Bond_10Y (0.079) | std_turn (0.077) | mom6m (0.065) | Hong_Kong_Index (0.061) |
| PCR | Oper_CF (0.075) | agr (0.075) | dollar_vol (0.074) | Cur_Assets (0.074) | Inventory_BS (0.074) |
| PLS | P/B (0.709) | Price (0.107) | Shares_Out (0.061) | FCF (0.038) | std_turn (0.035) |
| OLS3 | me (1.215) | be_me (-0.004) | ret_12_1 (-0.212) |  |  |

### Complexity summary

| Model | Mean Validation Score | Mean Active Complexity | Min Active Complexity | Max Active Complexity |
| --- | --- | --- | --- | --- |
| OLS | 0.298 | 105.000 | 105.000 | 105.000 |
| OLS3 | 0.141 | 3.000 | 3.000 | 3.000 |
| PLS | 0.141 | 2.667 | 1.000 | 6.000 |
| ENET | 0.141 | 22.867 | 5.000 | 54.000 |
| PCR | 0.140 | 20.667 | 1.000 | 49.000 |

---
## *Extra Notes*
#### How Feature Importance was calculated

1. First computes a baseline fit score on `base` using the same `R2OOS` function used elsewhere in the pipeline:

$$
R^2_{\text{base}}
=
1 - \frac{\sum_i (y_i - \hat{y}_i)^2}{\sum_i y_i^2}.
$$

6. Then, for each feature \(f\), it sets that feature to `0.0` in `train`, `validation`, and `base`, refits the model, predicts again, and recomputes the same `R2OOS`.
7. The loss of fit from neutralizing that feature is:

$$
\Delta R^2_f = R^2_{\text{base}} - R^2_{f=0}.
$$

8. Finally, normalizes those losses into the reported variable-importance share:

$$
\mathrm{var\_imp}_f
=
\frac{\Delta R^2_f}{\sum_j \Delta R^2_j}.
$$
#### How Model Complexity was calculated

###### General logic

For each rolling window, the model returns:
- `best_params`: the hyperparameters chosen on the validation set,
- `best_score`: the validation objective value for the selected specification,
- `complexity`: a compact model-shape summary.
###### Complexity by model family

	OLS / OLS3

These are implemented with `HuberRegressor`, not classical closed-form OLS.

- `best_score`: validation **RMSE**
- `n_nonzero_coef`: number of fitted coefficients with absolute value greater than zero after the final `train + validation` refit

Interpretation:
- For `OLS`, this is basically the number of active predictors in the robust linear fit.
- For `OLS3`, this stay at `3` because the model is fixed to `[me, be_me, ret_12_1]`.

	ENET

- `best_score`: validation **RMSE**
- `best_alpha`: chosen penalty strength
- `best_l1_ratio`: chosen Elastic Net mixing weight
- `n_nonzero_coef`: number of coefficients that remain active after shrinkage

	 PLS

- `best_score`: validation **RMSE**
- `n_components`: chosen latent component count

	 PCR

- `best_score`: validation **RMSE**
- `n_components`: chosen principal-component count

	 GBRT

- `best_score`: validation **Huber loss**
- Complexity fields recorded:
  - `best_max_depth`
  - `best_min_samples_split`
  - `best_min_samples_leaf`
  - `best_learning_rate`

	 RF

- `best_score`: validation **MSE**
- Complexity fields recorded:
  - `best_max_depth`
  - `best_max_features
