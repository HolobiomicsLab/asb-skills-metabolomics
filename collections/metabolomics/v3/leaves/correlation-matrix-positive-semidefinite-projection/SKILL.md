---
name: correlation-matrix-positive-semidefinite-projection
description: Use when correlation/covariance matrices estimated by per-pair numerical
  optimization (e.g. bivariate probit likelihoods via nlm/optimx) fail the positive
  semi-definiteness check and must be repaired before use in Stan, multivariate normal
  draws, or factor models.
when_to_use_negative:
- Input matrices are already valid correlation matrices (unit diagonal, non-negative
  eigenvalues) — projection is unnecessary and can only perturb estimates
- The analysis involves covariance matrices that are not intended to be correlations
  (variances ≠ 1), where nearPD should be called with corr=FALSE rather than forcing
  a correlation structure
- You need a cross-tissue/cross-trait correlation of continuous effect sizes rather
  than binary significance concordance — the bivariate probit framing here models
  IHW-significance overlap, not continuous TWAS effect correlation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: Matrix
  role: Provides Matrix::nearPD() implementing Higham's algorithm for the nearest
    PSD projection of each estimated correlation matrix
- name: optimx
  role: Numerical optimization backend (via nlm) for fitting the bivariate probit
    likelihoods that produce the raw, possibly non-PSD correlation estimates
- name: CmdStanR
  role: Fits the downstream non-centered hierarchical binomial model in Stan that
    consumes the projected correlation matrices
- name: S-PrediXcan
  role: Source of the gene-by-tissue association outputs (IHW α = 0.05 filtered) whose
    overlap counts define the probit-modeled data
  repo: github.com/hakyimlab/MetaXcan
- name: MoTrPAC_Complex_Traits
  role: Repository hosting the R scripts implementing the correlation estimation,
    nearPD projection, and Stan enrichment workflow
  repo: github.com/NikVetr/MoTrPAC_Complex_Traits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45966-w
    title: The impact of exercise on gene regulation in association with complex trait
      genetics
  source_content_hash: sha256:421e353f6772d9bde22704e93cae69afad2c9330476ec6feae8ee737a8fe2c4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CmdStanR
  - DESeq2
  - Matrix
  - MoTrPAC_Complex_Traits
  - PrediXcan
  - S-PrediXcan
  - Stan
  - fgsea
  - optimx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# correlation-matrix-positive-semidefinite-projection

## Summary

Projects estimated gene-wise correlation matrices (tissue Σi, trait Σj, tissue×trait Σi×j) onto the nearest positive semi-definite matrix with Higham's algorithm (R Matrix::nearPD()) so they are valid inputs to downstream hierarchical Bayesian and probit models.

## When to use

Use when correlation/covariance matrices estimated by per-pair numerical optimization (e.g. bivariate probit likelihoods via nlm/optimx) fail the positive semi-definiteness check and must be repaired before use in Stan, multivariate normal draws, or factor models.

## When NOT to use

- Input matrices are already valid correlation matrices (unit diagonal, non-negative eigenvalues) — projection is unnecessary and can only perturb estimates
- The analysis involves covariance matrices that are not intended to be correlations (variances ≠ 1), where nearPD should be called with corr=FALSE rather than forcing a correlation structure
- You need a cross-tissue/cross-trait correlation of continuous effect sizes rather than binary significance concordance — the bivariate probit framing here models IHW-significance overlap, not continuous TWAS effect correlation

## Inputs

- Binary gene significance indicator vectors per tissue×trait cell (S-PrediXcan hits filtered at IHW α = 0.05 with tissue×trait pairs as covariate)
- DEG membership indicators from repfdr nodes 8w_F1_M1 and 8w_F-1_M-1 (sex-consistent 8-week exercise DEGs)
- Symmetric (possibly indefinite) correlation/covariance matrices estimated per pair via bivariate probit likelihood optimization

## Outputs

- Nearest positive semi-definite projected correlation matrices Σi (tissue), Σj (trait), and Σi×j (tissue×trait) with unit diagonals
- Posterior summaries of hierarchical model difference effects (βi, γj, μk) whose validity depends on the corrected matrices

## How to apply

First estimate the gene-wise correlation structure for each marginal (tissue, trait, and tissue×trait cells) by fitting a bivariate probit to binary significance calls, maximizing the likelihood with the nlm algorithm from stats/optimx. Because unconstrained pairwise estimates need not be jointly PSD, apply Higham's nearest positive semi-definite projection to each matrix using Matrix::nearPD() with corr=TRUE so diagonals stay at 1 and symmetry is preserved. Feed the projected matrices into the non-centered hierarchical binomial Stan model (fit via CmdStanR with 4 chains, 2.5e3 warmup and 2.5e3 sampling iterations, delta=0.95) that models intersect counts of IHW α=0.05 S-PrediXcan hits within sex-consistent 8-week DEG sets. Re-check the projected matrices implicitly through MCMC diagnostics: all parameters must achieve r-hat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations; persistent divergences often indicate a badly conditioned input matrix.

## Related tools

- **Matrix** (Provides Matrix::nearPD() implementing Higham's algorithm for the nearest PSD projection of each estimated correlation matrix)
- **optimx** (Numerical optimization backend (via nlm) for fitting the bivariate probit likelihoods that produce the raw, possibly non-PSD correlation estimates)
- **CmdStanR** (Fits the downstream non-centered hierarchical binomial model in Stan that consumes the projected correlation matrices)
- **S-PrediXcan** (Source of the gene-by-tissue association outputs (IHW α = 0.05 filtered) whose overlap counts define the probit-modeled data) — github.com/hakyimlab/MetaXcan
- **MoTrPAC_Complex_Traits** (Repository hosting the R scripts implementing the correlation estimation, nearPD projection, and Stan enrichment workflow) — github.com/NikVetr/MoTrPAC_Complex_Traits

## Examples

```
Sigma_pd <- Matrix::nearPD(Sigma_raw, corr = TRUE, method = "Higham")$mat
```

## Evaluation signals

- Every projected matrix is symmetric with unit diagonal and minimum eigenvalue ≥ 0 (check with eigen() to numerical tolerance)
- Downstream Stan fit passes convergence criteria: r-hat < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations across all 4 chains
- Projection should minimally perturb the original estimates — compare Frobenius norm distance between raw and projected matrices; large shifts indicate unstable bivariate probit fits
- Posterior summaries reproduce the reported confident (>95% posterior mass) positive enrichments: colon, kidneys, small intestines, spleen, hippocampus, lungs, heart, plus Endocrine and Cardiometabolic trait categories

## Limitations

- Pairwise bivariate probit estimates optimized independently can be far from PSD, so the Higham projection may substantially alter implied partial correlations
- Only 99 of 114 traits had nonzero intersect counts, so correlation estimates for the remaining cells are undefined or degenerate
- Convergence of nlm/optimx on the bivariate probit likelihood is not guaranteed for sparse cells; non-convergent fits propagate into non-PSD matrices

## Evidence

- [other] Estimate tissue (Σi), trait (Σj), and tissue×trait (Σi×j) gene-wise correlation matrices under a bivariate probit via the nlm algorithm (stats/optimx), projecting each to the nearest positive semi-definite matrix with Higham's algorithm (Matrix::nearPD()).: [evidence span withheld — anchor status: not_found]
- [methods] filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate): "filtering by significance (IHW α = 0.05, with tissue x trait pairs as a covariate)"
- [methods] transformed into standard normal random variables using qnorm(p-value / 2, lower.tail = F) * sign(log2FC): [evidence span withheld — anchor status: not_found]
- [other] Diagnose MCMC output with the posterior package, requiring r̂ < 1.01, bulk and tail ESS > 500, and < 0.05% divergent iterations.: [evidence span withheld — anchor status: not_found]
