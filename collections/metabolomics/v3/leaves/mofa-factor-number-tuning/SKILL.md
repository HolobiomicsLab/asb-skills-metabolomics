---
name: mofa-factor-number-tuning
description: Use when integrating two or more transformed omics matrices (e.g. transcriptomics
  and metabolomics) with MOFA in BiomiX and the optimal number of latent factors is
  unknown, and factors must be validated as cohort discriminators.
when_to_use_negative:
- Only a single omics data type is available — MOFA integration needs multi-view matrices
  and BiomiX's pipeline should be run per-omics instead
- An external MOFA model with a fixed factor number already exists — tuning mode would
  redundantly retrain the model series
- The omics type is unsupported by BiomiX (proteomics and genomics are not yet implemented)
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0203
tools:
- name: BiomiX
  role: R pipeline with GUI that runs MOFA integration in tuning mode, applies the
    factor-count stopping criterion, Wilcoxon FDR ranking, and exports the model and
    statistics
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Multi-omics factor model underlying the integration; learns latent factors
    from transcriptomics and metabolomics and can handle missing data natively
- name: R
  role: Language in which BiomiX and its Shiny interfaces are implemented and executed
  repo: https://github.com/IxI-97/BiomiX
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:2f6e7ab2d0eab1374457c1d26a64f669f085e613b078a2040e83dc76bbd47f5d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/IxI-97/BiomiX
  repo_url: https://github.com/IxI-97/BiomiX
  tools:
  - BiomiX
  - MOFA (Multi-Omics Factor Analysis)
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# MOFA factor number tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

BiomiX tuning mode trains a series of MOFA (Multi-Omics Factor Analysis) models on transformed multi-omics matrices (e.g. PTB transcriptomics and metabolomics), iteratively increasing factor counts until a stopping criterion is met, then ranks candidate models by cohort discrimination (Wilcoxon test with FDR correction) to select the final model and its discriminant factors.

## When to use

Use when integrating two or more transformed omics matrices (e.g. transcriptomics and metabolomics) with MOFA in BiomiX and the optimal number of latent factors is unknown, and factors must be validated as cohort discriminators.

## When NOT to use

- Only a single omics data type is available — MOFA integration needs multi-view matrices and BiomiX's pipeline should be run per-omics instead
- An external MOFA model with a fixed factor number already exists — tuning mode would redundantly retrain the model series
- The omics type is unsupported by BiomiX (proteomics and genomics are not yet implemented)

## Inputs

- Transformed transcriptomics matrix (counts or normalized expression, optionally imputed)
- Transformed metabolomics matrix (peak table, optionally imputed)
- Sample metadata with cohort/condition labels (e.g. PTB vs healthy controls)

## Outputs

- Fitted MOFA model file (mofa_model.hdf5) containing the selected number of factors (e.g. exactly three)
- Per-factor variance-explained table per omics view (mofa_factor_statistics.csv)
- Wilcoxon discrimination statistics per factor with FDR-adjusted p-values

## How to apply

Load the transformed (and optionally imputed via NIPALS, random forest, lasso, or mean/median) transcriptomics and metabolomics matrices into the BiomiX R pipeline; MOFA can handle missing values natively where data remain unimputed. Run MOFA integration in tuning mode: train a series of models with iteratively increasing factor counts until the stopping criterion is met — at least three models whose last factor explains less than 1% of variance. Rank candidate models by their ability to discriminate between cohorts using the Wilcoxon test on factor values with False Discovery Rate correction, and select the top three models. Fit the final three-factor MOFA model, recording variance explained per omics view and the cohort-discriminant factor statistics (in the PTB vs HC case study: factor 1, p.adj = 0.0011, capturing 2.23% of metabolomics and 33.22% of transcriptomics total variance; model-level ~5.05% metabolomics / ~45.77% transcriptomics). Export the fitted model to mofa_model.hdf5 and the per-factor variance and Wilcoxon statistics to mofa_factor_statistics.csv.

## Related tools

- **BiomiX** (R pipeline with GUI that runs MOFA integration in tuning mode, applies the factor-count stopping criterion, Wilcoxon FDR ranking, and exports the model and statistics) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Multi-omics factor model underlying the integration; learns latent factors from transcriptomics and metabolomics and can handle missing data natively)
- **R** (Language in which BiomiX and its Shiny interfaces are implemented and executed) — https://github.com/IxI-97/BiomiX

## Evaluation signals

- The exported mofa_model.hdf5 contains exactly the number of factors selected by tuning (three in the PTB case study)
- The factor statistics table reproduces the reported factor-1 values within stated ranges: p.adj = 0.0011, 2.23% metabolomics and 33.22% transcriptomics variance explained
- The stopping criterion was satisfied: at least three trained models whose last factor explains less than 1% of variance
- Model-level variance explained matches the expected order of magnitude (~5.05% metabolomics / ~45.77% transcriptomics)
- Only factors with significant Wilcoxon discrimination (FDR-corrected) are carried forward to interpretation

## Limitations

- Mac installation was still affected by known installation issues; Linux or Windows are recommended instead
- MOFA only handles missing values natively when data remain unimputed; otherwise an imputation method (NIPALS, random forest, lasso, mean/median, 0 values) must be chosen and its bias considered
- Users must carefully review datasets before use; BiomiX performs no automated validation and is described as a first attempt, not a definitive solution

## Evidence
