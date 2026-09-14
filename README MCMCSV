# Stochastic Volatility Model via MCMC

A Bayesian stochastic volatility (SV) model fit to daily market returns using a
hand-written Metropolis-within-Gibbs sampler — no probabilistic-programming
libraries (PyMC/Stan). The sampler, the log-posterior, and the diagnostics are
built from scratch in NumPy.

## The model

Returns are modelled as having a time-varying, unobserved (latent) volatility
that follows a mean-reverting AR(1) process in log-variance:

```
r_t   = exp(h_t / 2) · ε_t,        ε_t ~ N(0, 1)          (observed returns)
h_t   = μ + φ (h_{t-1} − μ) + η_t,  η_t ~ N(0, σ²)          (latent log-variance)
```

- `r_t` — observed daily return
- `h_t` — latent log-variance on day `t` (never observed; inferred)
- `μ` — long-run mean log-variance, `φ` — persistence, `σ` — volatility of volatility

The goal is to recover both the hidden volatility path `h_{1:T}` **and** the
parameters `(μ, φ, σ)` from the returns alone.

## Method

The joint posterior `p(h, μ, φ, σ | r)` has no closed form, so it is sampled with
MCMC. Each sweep alternates two blocks:

1. **Latent path** — single-site Metropolis update of each `h_t` in turn. Because
   `h_t` enters the posterior only through its own observation term and its two
   AR(1) transitions, the acceptance ratio is computed locally (3 terms), giving a
   large speed-up over recomputing the full-path likelihood.
2. **Parameters** — a joint random-walk Metropolis update of `(μ, φ, σ)`, holding
   the path fixed.

The intractable normalising constant cancels in every acceptance ratio, so only
the unnormalised log-posterior is ever evaluated.

## Validation

The sampler is validated on **simulated data with a known ground truth** before
being applied to real returns: a path and parameters are generated, and the
sampler is confirmed to recover them (both the parameters and the hidden path).

## Results (daily market excess returns, Fama–French, 2004–2024)

Recovered parameters land in the empirically expected ranges for daily equity
volatility:

| parameter | estimate | typical empirical range |
|-----------|----------|-------------------------|
| μ         | ≈ −9.1   | −9 to −10 |
| φ         | ≈ 0.965  | 0.95–0.98 (high persistence) |
| σ         | ≈ 0.16   | 0.1–0.3 |

The implied long-run volatility is `exp(μ/2)·√252 ≈ 17%` annualised, consistent
with the market. The estimated volatility path spikes at the 2008 financial
crisis and the March 2020 COVID crash.

## Forecasting

Volatility (not return direction) is forecastable because it is persistent. From
the last estimated state, volatility is projected forward via

```
E[h_{T+k}] = μ + φ^k (ĥ_T − μ)
```

which yields a forecast **distribution** for future returns, `r_{T+k} ~ N(0, e^{h_{T+k}})`
— i.e. a forecast of the *range* / risk, not the direction. The model makes no
claim to predict whether the market rises or falls, by construction (`E[r_t] = 0`).

## Data

Daily Fama–French research factors (`Mkt-RF` column) from the
[Kenneth French data library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html).

## Files

- `sv_mcmc.py` — data loading, log-posterior, sampler, and plots.

## Limitations & possible extensions

- The basic model assumes Gaussian return shocks; real returns are fat-tailed
  (a Student-t observation density would be a natural extension).
- No leverage effect (volatility responding asymmetrically to negative returns).
- Convergence should be confirmed with multiple chains and effective-sample-size
  diagnostics for production use.

## Requirements

`numpy`, `pandas`, `matplotlib`
