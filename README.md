# aleator — Stochastic Test-Driven Development for R

[![R-CMD-check](https://github.com/phosphene/aleator/actions/workflows/ci.yml/badge.svg)](https://github.com/phosphene/aleator/actions/workflows/ci.yml)

**Test probabilistic and Bayesian code the way you test deterministic code.**

Standard `expect_equal(f(x), y)` assertions fail on MCMC, bootstraps, and
any other stochastic output — the result isn't a single number, it's a
distribution. `aleator` gives you the tools to test stochastic systems
rigorously:

- **Controlled seed environments** — `aleator_seed_env()` wraps `withr::with_seed()`
  with cross-platform RNG specification so results are reproducible on any
  machine.
- **Parameter recovery** — `aleator_param_recovery()` generates synthetic data
  from known parameters, fits your model, and verifies the estimates land in
  the credible interval.
- **Convergence diagnostics** — `aleator_convergence_check()` asserts R-hat and
  effective sample size pass configurable thresholds.

Pure functions, one dependency (`withr`), no global state. Drop-in for any
R project using MCMC, Stan, brms, bootstrap, or simulation.

## Installation

```r
remotes::install_github("phosphene/aleator")
```

## Quick Start

```r
library(aleator)

# 1. Reproducible stochastic code
aleator_seed_env(42, rnorm(5))
aleator_seed_env(42, rnorm(5))   # identical every time

# 2. Parameter recovery — does your model recover known truth?
result <- aleator_param_recovery(
  true_params = c(intercept = 2.0, slope = 0.5),
  generate_fn = function(p) {
    x <- rnorm(500)
    y <- p["intercept"] + p["slope"] * x + rnorm(500, sd = 0.3)
    data.frame(x = x, y = y)
  },
  fit_fn = function(d) lm(y ~ x, data = d),
  extract_fn = function(fit) {
    ci <- confint(fit)
    data.frame(
      parameter = c("intercept", "slope"),
      mean = coef(fit),
      lower = ci[, 1],
      upper = ci[, 2]
    )
  }
)
result$all_recovered   # TRUE

# 3. Convergence — did your MCMC actually converge?
check <- aleator_convergence_check(
  rhat_values = c(1.01, 1.00, 1.02),
  ess_values = c(1200, 800, 950),
  param_names = c("alpha", "beta", "sigma")
)
check$all_converged    # TRUE
```

## The Problem STDD Solves

| Deterministic code | Stochastic code |
|---|---|
| `expect_equal(f(x), y)` | `expect_gt(ks_pvalue, 0.05)` |
| One output, exact assertion | Distribution, statistical assertion |
| `set.seed()` at top of script | `withr::with_seed()` per block |
| Deterministic tests | Parameter recovery, distributional checks |

For Bayesian/MCMC code, a test that asserts an exact MCMC draw will fail
on flake. `aleator` gives you the right assertions: recover known parameters,
verify distributions against analytical posteriors, and assert convergence.

## API

| Function | What it does |
|----------|--------------|
| `aleator_seed_env(seed, code, .rng_kind, .rng_normal_kind)` | Run `code` under a controlled seed with cross-platform RNG |
| `aleator_param_recovery(true_params, generate_fn, fit_fn, extract_fn, ci_level, seed)` | Verify a model recovers known parameters within credible intervals |
| `aleator_convergence_check(rhat_values, ess_values, rhat_threshold, ess_threshold, param_names)` | Assert R-hat and ESS pass thresholds |

## Testing

```r
# From the repo root
Rscript -e "testthat::test_local()"
```

Test suite: unit tests (`test-unit-*.R`) + BDD specs (`test-bdd-*.R`) using
testthat's native `describe()`/`it()` — no extra dependencies.

## CI

`.github/workflows/ci.yml` runs in `rocker/r-ver:4.4.0`:
lint (lintr, styler) → test (testthat) → coverage (covr ≥ 80%).

## The Definitory Warrant

This package is open source because definitory power transfers while
sincerity does not. A claim made in public gets its force not from the
claimant's inner state but from the apparatus standing behind it — how
specified it is, how specifiable it remains, and whether it is testable
in some bounded sense. Those are properties of the claim and the
apparatus, not of us. You cannot audit sincerity; you can audit
specification.

This package ships its full specification and its self-tests. Anyone can
verify the apparatus against itself without trusting us at all. Open
source is the form of definitory warrant that does not require belief.

## Docs

- [`docs/ALEATOR_SPEC.md`](docs/ALEATOR_SPEC.md) — full specification of the framework

## License

MIT
