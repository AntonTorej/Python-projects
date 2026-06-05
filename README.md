# Fama-French Three-Factor Model: Replication and Comparison with CAPM

A replication of Fama and French's (1993) three-factor asset pricing model on 25 size/book-to-market sorted U.S. equity portfolios, with extensions including a CAPM comparison, Newey-West HAC standard errors, and the Gibbons-Ross-Shanken (1989) joint test of pricing errors.

## Summary

This project tests whether the Fama-French three-factor model (market, size, value) prices the cross-section of 25 size/book-to-market sorted portfolios over the full available sample from July 1926 to April 2026 (1,198 monthly observations). I find that FF3 substantially improves on CAPM in economic terms — mean time-series R² rises from 0.77 to 0.91, mean absolute alpha falls by approximately 27%, and the cross-sectional R² rises from 0.73 to 0.89. However, both models are formally rejected by the GRS joint test, and a persistent small-growth anomaly survives FF3 — consistent with the literature that motivated subsequent multi-factor extensions.

## Data

All data come from [Kenneth French's online data library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html):

- **`F-F_Research_Data_Factors.csv`** — monthly returns on the three Fama-French factors (Mkt-RF, SMB, HML) and the 1-month Treasury bill rate (RF).
- **`25_Portfolios_5x5.csv`** — monthly value-weighted returns on 25 portfolios formed at the intersection of size and book-to-market quintiles (value-weighted section only).

All series are converted from percent to decimal form. The risk-free rate is subtracted from raw portfolio returns to obtain excess returns prior to estimation.

## Methodology

For each of the 25 test portfolios *i*, I estimate the time-series regression: 

R_i - R_f = α_i + β_MKT·(R_M - R_f) + β_SMB·SMB + β_HML·HML + ε

Standard errors are computed using Newey-West HAC corrections with 6 lags to account for heteroskedasticity and autocorrelation in monthly residuals.

For comparison, I also estimate the analogous CAPM specification with only the market factor on the right-hand side.

The Gibbons-Ross-Shanken (1989) test is applied to both models to jointly evaluate whether all 25 alphas equal zero. This test uses the residual covariance structure across portfolios to assess the significance of the alpha vector.

## Headline Findings

| Metric | CAPM | FF3 |
|--------|------|-----|
| Mean R² (time-series) | 0.768 | 0.907 |
| Cross-sectional R² | 0.727 | 0.888 |
| Mean \|α\| (monthly) | 0.00155 | 0.00113 |
| GRS J-statistic | 86.09 | 83.28 |
| GRS p-value | < 0.001 | < 0.001 |

**Key results:**

1. **FF3 explains substantially more return variation than CAPM.** Adding the size and value factors raises mean time-series R² from 0.77 to 0.91 — approximately 14 percentage points of additional explanatory power.

2. **Cross-sectional R² improves dramatically.** The cross-sectional fit rises from 0.73 (CAPM) to 0.89 (FF3), confirming that SMB and HML capture systematic variation in average returns across portfolios that CAPM cannot explain.

3. **Mean absolute alpha declines under FF3 by approximately 27%** (from 0.00155 to 0.00113 monthly). The three-factor model reduces pricing errors particularly for value portfolios, where HML absorbs the value premium that CAPM was forced to interpret as alpha.

4. **The small-growth anomaly survives FF3.** The SMALL LoBM portfolio retains a significantly negative alpha (-0.7% per month, t ≈ -3.3 under Newey-West) — the model overestimates returns on small growth stocks. This residual anomaly motivated subsequent factor models (Carhart 1997, Fama-French 2015).

5. **Both models are formally rejected by GRS, with comparable test statistics.** Despite FF3's improvements in cross-sectional R² and mean absolute alpha, the GRS J-statistic (83.28 for FF3 vs 86.09 for CAPM) is only modestly lower under FF3. This reflects the long sample's high statistical power: with nearly 1,200 monthly observations, even small alphas become statistically detectable. The rejection of FF3 is driven primarily by the persistent small-growth anomaly rather than by widespread mispricing.

6. **The SMB risk premium is statistically weak over the full sample.** The estimated SMB premium is small (0.08% monthly) and not significantly different from zero (t ≈ 0.9), in contrast to the original Fama-French sample. This is consistent with subsequent literature documenting that the size premium has been weak outside the 1963-1991 window.

## Factor Loading Patterns

The estimated factor loadings behave consistently with the construction of the test portfolios:

- **SMB loadings decline monotonically** from small portfolios (β_SMB ≈ 1.3–1.5) to big portfolios (β_SMB ≈ -0.2), confirming that small-size portfolios are heavily exposed to the small-minus-big factor.
- **HML loadings increase monotonically** from growth portfolios (β_HML ≈ -0.3) to value portfolios (β_HML ≈ 0.9), confirming that value-tilted portfolios load positively on the value factor.
- **Market betas cluster near 1.0** across all portfolios, with modest variation.

## Discussion

The replication confirms the empirical core of Fama and French (1993): the three-factor model represents a major economic improvement over CAPM in explaining the cross-section of size/BM-sorted portfolio returns. The two extra factors substantially raise both time-series and cross-sectional R², and they meaningfully reduce mean absolute alphas across the 25 portfolios.

However, statistical rejection by GRS persists for both models. This divergence between economic improvement (large) and statistical rejection (similar magnitude) is itself informative: with 100 years of monthly data, the GRS test has extreme statistical power. Even small persistent alphas — particularly the small-growth anomaly — are sufficient to reject the null at any conventional significance level. Researchers using shorter samples would likely find FF3 doing more to lower the J-statistic relative to CAPM than this long-sample result suggests.

The weak size premium estimated over the full century is worth flagging: it suggests that the magnitude of the size effect documented in the original 1963-1991 sample may not be representative of the longer run. This is consistent with growing evidence that the size premium has been weak or absent outside its original sample window.

The rejection of FF3 should not be taken as a failure of the project. Rejection of FF3 on these portfolios is the canonical finding in the literature and is precisely what motivated three decades of subsequent research into multi-factor and characteristic-based asset pricing models.

## Limitations

- **In-sample factor construction.** SMB and HML are derived from the same universe of stocks being tested, introducing some in-sample mechanical correlation between factors and test assets.
- **Newey-West lag choice.** Six lags is conventional for monthly data but is somewhat arbitrary; results are not formally robust to alternative lag selections.
- **Sample heterogeneity.** The 100-year sample includes multiple economic regimes (Depression, post-war, modern era). Factor stability across sub-periods is not tested here.
- **Standard errors on CAPM coefficients.** CAPM regressions use plain OLS standard errors in the current code; HAC corrections would be more consistent with the FF3 specification.

## Repository Contents

- `fama_french_replication.ipynb` — main analysis notebook.
- `F-F_Research_Data_Factors.csv` — input data: three Fama-French factors plus risk-free rate.
- `25_Portfolios_5x5.csv` — input data: 25 size/BM sorted portfolios.
- `methods_note.pdf` — short methods note describing the project (forthcoming).

## How to Reproduce

​`​`​`
Requirements:
pandas
numpy
matplotlib
statsmodels
linearmodels
​`​`​`

To reproduce, open `fama_french_replication.ipynb` and run all cells in order. Both data files should be placed in the same directory as the notebook.

## References

- Fama, E. F., & French, K. R. (1993). Common risk factors in the returns on stocks and bonds. *Journal of Financial Economics*, 33(1), 3-56.
- Gibbons, M. R., Ross, S. A., & Shanken, J. (1989). A test of the efficiency of a given portfolio. *Econometrica*, 57(5), 1121-1152.
- Newey, W. K., & West, K. D. (1987). A simple, positive semi-definite, heteroskedasticity and autocorrelation consistent covariance matrix. *Econometrica*, 55(3), 703-708.

---

*This project was completed as an independent replication exercise to build empirical asset pricing skills.*
