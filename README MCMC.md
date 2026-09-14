# Stochastic Volatility Model via MCMC

A Bayesian stochastic volatility (SV) model fit to daily market returns using a
hand-written Metropolis-within-Gibbs sampler — no probabilistic-programming
libraries (PyMC/Stan). The sampler, the log-posterior, and the diagnostics are
built from scratch in NumPy.

## The model

Returns are modelled as having a time-varying, unobserved (latent) volatility
that follows a mean-reverting AR(1) process in log-variance:

$$
r_t = e^{h_t/2}\,\varepsilon_t, \qquad \varepsilon_t \sim \mathcal{N}(0,1)
$$

$$
h_t = \mu + \phi\,(h_{t-1} - \mu) + \eta_t, \qquad \eta_t \sim \mathcal{N}(0,\sigma^2)
$$

where $r_t$ is the observed daily return, $h_t$ is the latent log-variance on day
$t$ (never observed; inferred), $\mu$ is the long-run mean log-variance, $\phi$ is
the persistence, and $\sigma$ is the volatility of volatility. The conditional
variance of the return is $\mathrm{Var}(r_t \mid h_t) = e^{h_t}$.

The goal is to recover both the hidden volatility path $h_{1:T}$ **and** the
parameters $(\mu, \phi, \sigma)$ from the returns alone.

## Method

The joint posterior

$$
p(h_{1:T}, \mu, \phi, \sigma \mid r_{1:T}) \;\propto\;
p(r_{1:T} \mid h_{1:T})\;
p(h_{1:T} \mid \mu, \phi, \sigma)\;
p(\mu, \phi, \sigma)
$$

factors into an observation likelihood, an AR(1) dynamics prior on the latent
path, and a prior on the parameters. It has no closed form, so it is sampled with
MCMC. Each sweep alternates two blocks:

1. **Latent path** — a single-site Metropolis update of each $h_t$ in turn. Since
   $h_t$ enters the posterior only through its own observation term and its two
   adjacent AR(1) transitions, the acceptance ratio is evaluated *locally* from
   three terms rather than by recomputing the full-path likelihood — a substantial
   speed-up.
2. **Parameters** — a joint random-walk Metropolis update of $(\mu, \phi, \sigma)$
   with the path held fixed.

The intractable normalising constant cancels in every acceptance ratio, so only
the unnormalised log-posterior is ever evaluated. The single-site proposal for
$h_t$ uses the local log-density

$$
\ell_{\text{local}}(h_t) = -\tfrac{h_t}{2} - \frac{r_t^2}{2e^{h_t}}
- \frac{\big(h_t - \mu - \phi(h_{t-1}-\mu)\big)^2}{2\sigma^2}
- \frac{\big(h_{t+1} - \mu - \phi(h_t-\mu)\big)^2}{2\sigma^2},
$$

and the move is accepted with probability
$\alpha = \min\!\big(1,\ \exp[\,\ell_{\text{local}}(h_t') - \ell_{\text{local}}(h_t)\,]\big)$.

## Validation

The sampler is validated on **simulated data with a known ground truth** before
being applied to real returns: a path and parameters are generated, and the
sampler is confirmed to recover both the parameters and the hidden path.

## Results

Fit to daily market excess returns ($\mathrm{Mkt}\text{-}\mathrm{RF}$, Fama–French,
2004–2024). The recovered parameters land in the empirically expected ranges for
daily equity volatility:

| parameter | estimate | typical empirical range |
|-----------|----------|-------------------------|
| $\mu$     | $\approx -9.1$  | $-9$ to $-10$ |
| $\phi$    | $\approx 0.965$ | $0.95$–$0.98$ (high persistence) |
| $\sigma$  | $\approx 0.16$  | $0.1$–$0.3$ |

The implied long-run volatility is $e^{\mu/2}\sqrt{252} \approx 17\%$ annualised,
consistent with the market. The estimated volatility path spikes at the 2008
financial crisis and the March 2020 COVID crash.

## Forecasting

Volatility — not return direction — is forecastable, because it is persistent.
From the last estimated state, the log-variance is projected forward by its
mean-reverting expectation

$$
\mathbb{E}[h_{T+k} \mid r] = \mu + \phi^{k}\,(\hat h_T - \mu),
$$

which yields a forecast *distribution* for the future return,
$r_{T+k} \sim \mathcal{N}\!\big(0,\ e^{h_{T+k}}\big)$ — a forecast of the **range**
(risk), not the direction. The model makes no claim to predict whether the market
rises or falls, by construction, since $\mathbb{E}[r_t] = 0$.

## Data

Daily Fama–French research factors ($\mathrm{Mkt}\text{-}\mathrm{RF}$ column) from the
[Kenneth French data library](https://mba.tuck.dartmouth.edu/pages/faculty/ken.french/data_library.html).

## Files

- `sv_mcmc.py` — data loading, log-posterior, sampler, and plots.

## Limitations and possible extensions

- The basic model assumes Gaussian return shocks; real returns are fat-tailed, so
  a Student-$t$ observation density would be a natural extension.
- No leverage effect (volatility responding asymmetrically to negative returns).
- For production use, convergence should be confirmed with multiple chains
  (e.g. the Gelman–Rubin $\hat R$ statistic) and the precision quantified with the
  effective sample size $N_{\text{eff}} = N/\tau$, where $\tau$ is the integrated
  autocorrelation time of the chain.

## Requirements

`numpy`, `pandas`, `matplotlib`
