---
name: mcmc-convergence-diagnostics
description: Use when fitting Stan hierarchical/binomial multilevel models (e.g.,
  PrediXcan-DEG enrichment or proportion-positive direction-of-effect models) and
  you must verify chain convergence before extracting marginal posterior means or
  95% posterior-mass flags.
when_to_use_negative:
- When the model is fit by optimization only (e.g., the nlm/optimx bivariate-probit
  correlation estimation step) — no chains or MCMC diagnostics exist to check
- When the goal is frequentist inference (fgsea GSEA with Bonferroni/harmonic-mean
  p-values) rather than posterior-based enrichment calls
- When you intend to report posterior probabilities from a fit that has already failed
  the r-hat/ESS/divergence gates — refit instead
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: CmdStanR
  role: R interface for sampling the hierarchical binomial Stan models with non-centered
    parameterization, 4 chains, 2500/2500 iterations, adapt_delta = 0.95
  repo: https://github.com/NikVetr/MoTrPAC_Complex_Traits
- name: Stan
  role: Bayesian modeling language for the hierarchical enrichment and direction-of-effect
    models whose output is diagnosed
  repo: https://github.com/NikVetr/MoTrPAC_Complex_Traits
- name: posterior
  role: R package used to compute r-hat, bulk/tail ESS, and divergences from the MCMC
    draws
  repo: https://github.com/NikVetr/MoTrPAC_Complex_Traits
- name: Matrix
  role: Matrix::nearPD() (Higham's algorithm) projects correlation matrices (Sigma_i,
    Sigma_j, Sigma_i_x_j; G_SNP from GCTA-GRM) to the nearest PSD matrix so the Stan
    model is well-conditioned
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:0c9c1f491e5ab5faa0cf537f5de7d683fa434958bb2e998d60cfa2fa5eb646e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NikVetr/MoTrPAC_Complex_Traits
  tools:
  - CmdStanR
  - DESeq2
  - GCTA-GRM
  - Matrix
  - Plink
  - PrediXcan
  - S-PrediXcan
  - Stan
  - fgsea
  - posterior
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# mcmc-convergence-diagnostics

## Summary

Rigorous diagnosis of Stan/CmdStanR MCMC output for the paper's hierarchical Bayesian enrichment and direction-of-effect models, requiring r-hat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations before posterior summaries are trusted.

## When to use

Use when fitting Stan hierarchical/binomial multilevel models (e.g., PrediXcan-DEG enrichment or proportion-positive direction-of-effect models) and you must verify chain convergence before extracting marginal posterior means or 95% posterior-mass flags.

## When NOT to use

- When the model is fit by optimization only (e.g., the nlm/optimx bivariate-probit correlation estimation step) — no chains or MCMC diagnostics exist to check
- When the goal is frequentist inference (fgsea GSEA with Bonferroni/harmonic-mean p-values) rather than posterior-based enrichment calls
- When you intend to report posterior probabilities from a fit that has already failed the r-hat/ESS/divergence gates — refit instead

## Inputs

- CmdStanR CmdStanMCMC fit object or posterior MCMC draws (e.g., /data/MCMC_output) from the hierarchical binomial enrichment model
- Stan model fitted with non-centered parameterization, 4 chains, 2500 warmup + 2500 sampling iterations, adapt_delta = 0.95
- Stan parameters of interest: tissue (beta_i), trait (gamma_j), trait-category (mu_k) difference effects and trait x tissue direction-of-effect deviations

## Outputs

- Convergence diagnostic table: r-hat, bulk ESS, tail ESS, and divergent-iteration fraction per parameter
- Posterior summary tables (CSV) of marginal posterior means/quantiles for difference effects, with >95% posterior-mass enrichment/depletion flags
- Ranked trait/tissue list of posterior means (e.g., strongest positive: body fat percentage, asthma, BMI; strongest depletion: standing height, high cholesterol)

## How to apply

Fit the model in Stan via CmdStanR with a non-centered parameterization, 4 randomly initialized chains, 2.5x10^3 warmup and 2.5x10^3 sampling iterations each, and target acceptance rate delta = 0.95 (the higher delta controls divergences in the hierarchical partial-pooling geometry). After sampling, load the draws (/data/MCMC_output) and use the posterior package to compute r-hat, bulk and tail ESS, and count divergent iterations, accepting the fit only if r-hat < 1.01, both ESS metrics exceed 500, and divergences are < 0.05% of iterations. Only after passing these gates extract marginal posteriors for the difference effects (tissue beta_i, trait gamma_j, trait-category mu_k, or per-trait-x-tissue direction deviations) and flag parameters with > 95% posterior mass on one side of 0 as enriched or depleted. If diagnostics fail, re-fit with higher adapt_delta or check the PSD-projected correlation matrices (Matrix::nearPD) fed to the model rather than trusting non-converged summaries.

## Related tools

- **CmdStanR** (R interface for sampling the hierarchical binomial Stan models with non-centered parameterization, 4 chains, 2500/2500 iterations, adapt_delta = 0.95) — https://github.com/NikVetr/MoTrPAC_Complex_Traits
- **Stan** (Bayesian modeling language for the hierarchical enrichment and direction-of-effect models whose output is diagnosed) — https://github.com/NikVetr/MoTrPAC_Complex_Traits
- **posterior** (R package used to compute r-hat, bulk/tail ESS, and divergences from the MCMC draws) — https://github.com/NikVetr/MoTrPAC_Complex_Traits
- **Matrix** (Matrix::nearPD() (Higham's algorithm) projects correlation matrices (Sigma_i, Sigma_j, Sigma_i_x_j; G_SNP from GCTA-GRM) to the nearest PSD matrix so the Stan model is well-conditioned)

## Examples

```
fit <- model$sample(data = stan_data, chains = 4, iter_warmup = 2500, iter_sampling = 2500, adapt_delta = 0.95); stopifnot(max(posterior::rhat(fit$draws())) < 1.01, min(bulk) > 500, min(tail) > 500, n_divergent/iters < 5e-4)
```

## Evaluation signals

- All Stan parameters must pass r-hat < 1.01 and both bulk and tail ESS > 500
- Divergent iterations must be < 0.05% of total sampling iterations
- Posterior summaries must reproduce the reported confident positive enrichments (colon, kidneys, small intestines, spleen, hippocampus, lungs, heart; Endocrine and Cardiometabolic categories)
- Direction-of-effect posterior means must match the reported trait ranking (strongest positive: body fat percentage, asthma, BMI; strongest depletion: standing height, high cholesterol)
- Cells flagged as enriched/depleted must carry > 95% posterior mass on one side of 0

## Limitations

- Some marginal posterior distributions for difference parameters did not reach the 95% posterior mass threshold (e.g., no individual tissues confidently enriched in positive effects for asthma)
- MCMC output depends on well-conditioned inputs: correlation matrices must be projected to positive semi-definiteness via Higham's algorithm before fitting
- Enrichment/depletion effect sizes are small (predominately within 0 +/- 0.3 on the log-odds scale, ~7.5% maximum difference on the probability scale), so convergence gates are essential to avoid mistaking Monte Carlo noise for signal

## Evidence

- [methods] Diagnose MCMC output with the posterior package, requiring r-hat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations.: [evidence span withheld — anchor status: not_found]
- [methods] Fit the hierarchical binomial model in Stan via CmdStanR using a non-centered parameterization: 4 randomly initialized chains, 2.5x10^3 warmup and 2.5x10^3 sampling iterations each, target acceptance rate delta = 0.95.: [evidence span withheld — anchor status: not_found]
- [methods] Run four randomly initialized chains (2.5e3 warmup, 2.5e3 sampling iterations, target acceptance delta = 0.95, non-centered parameterization) and verify convergence with the posterior package.: [evidence span withheld — anchor status: not_found]
- [discussion] Some marginal posterior distributions for difference parameters did not reach the 95% posterior mass threshold.: "though no marginal posterior distributions for difference parameters there reached our 95% posterior mass threshold"
- [discussion] Enrichment/depletion effect sizes were small, predominately within 0 +/- 0.3 on the log-odds scale (~7.5% maximum difference on the probability scale).: [evidence span withheld — anchor status: not_found]
