---
name: omics-variance-explained-interpretation
description: Use when a MOFA model has been trained on two or more omics matrices
  (e.g. transformed transcriptomics + metabolomics) and you must decide which factor
  discriminates cohorts and quantify how much of each omics' variance that factor
  explains.
when_to_use_negative:
- You have only a single omics data type — BiomiX's integration tuning (and thus this
  skill) presupposes at least two omics matrices to integrate
- You need to annotate factor contributors with proteomics or genomics — the article
  notes BiomiX has limited omics analysers and does not yet support these data types
- The omics matrices have not been QC'd/transformed (e.g. outliers and highly variable
  features not removed via the preview-QC interface) — MOFA variance fractions will
  be driven by noise
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3172
tools:
- name: MOFA (Multi-Omics Factor Analysis)
  role: Fits the multi-omics factor model whose per-factor variance explained per
    view and cohort-discriminant factors are interpreted; handles missing data natively
    on unimputed inputs
  repo: https://github.com/IxI-97/BiomiX
- name: BiomiX
  role: R/Shiny pipeline providing MOFA factor-number tuning mode, Wilcoxon-based
    ranking of candidate models, and export of the model and factor statistics
  repo: https://github.com/BiomiX-consortium/BiomiX2.5
- name: R
  role: Language in which the BiomiX pipeline and MOFA integration are implemented
    and run
  repo: https://github.com/IxI-97/BiomiX
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:5822c3aa9a006197ea0ce25c8f70a28af320ccd3c708c0c9f9d564f5ae1a7f36
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# omics-variance-explained-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret a fitted MOFA multi-omics integration model by reading the per-factor variance explained in each omics view (e.g. metabolomics vs transcriptomics) together with cohort-discrimination statistics, as demonstrated in BiomiX's PTB vs HC case study where factor 1 captured 2.23% metabolomics and 33.22% transcriptomics variance (p.adj = 0.0011).

## When to use

Use when a MOFA model has been trained on two or more omics matrices (e.g. transformed transcriptomics + metabolomics) and you must decide which factor discriminates cohorts and quantify how much of each omics' variance that factor explains.

## When NOT to use

- You have only a single omics data type — BiomiX's integration tuning (and thus this skill) presupposes at least two omics matrices to integrate
- You need to annotate factor contributors with proteomics or genomics — the article notes BiomiX has limited omics analysers and does not yet support these data types
- The omics matrices have not been QC'd/transformed (e.g. outliers and highly variable features not removed via the preview-QC interface) — MOFA variance fractions will be driven by noise

## Inputs

- Transformed (optionally imputed via NIPALS, random forest, lasso, or mean/median) transcriptomics matrix
- Transformed metabolomics matrix
- Sample metadata with condition labels (e.g. PTB vs healthy control)

## Outputs

- Fitted MOFA model exported as mofa_model.hdf5 (exactly three factors in the tuned PTB model)
- Per-factor variance-explained fractions per omics view (e.g. ~2.23% metabolomics / ~33.22% transcriptomics for factor 1)
- Wilcoxon discrimination statistics per factor (p.adj = 0.0011 for factor 1) exported to mofa_factor_statistics.csv

## How to apply

Run BiomiX's MOFA tuning mode: train a series of models with iteratively increasing factor counts until at least three models have a last factor explaining less than 1% of variance, then rank candidate models by Wilcoxon test on factor values with FDR correction and select the top three. Fit the final model (three factors in the PTB case study) and record variance explained per omics view for each factor. Identify the discriminant factor by its adjusted Wilcoxon p-value for cohort separation (factor 1: p.adj = 0.0011, sd = 0.08) and report its per-view variance fractions (~2.23% metabolomics, ~33.22% transcriptomics). Export the model to mofa_model.hdf5 and per-factor variance and discrimination statistics to mofa_factor_statistics.csv so downstream contributors (clinical correlation, pathway analysis, PubMed bibliography mining) can be interpreted against the dominant omics view.

## Related tools

- **MOFA (Multi-Omics Factor Analysis)** (Fits the multi-omics factor model whose per-factor variance explained per view and cohort-discriminant factors are interpreted; handles missing data natively on unimputed inputs) — https://github.com/IxI-97/BiomiX
- **BiomiX** (R/Shiny pipeline providing MOFA factor-number tuning mode, Wilcoxon-based ranking of candidate models, and export of the model and factor statistics) — https://github.com/BiomiX-consortium/BiomiX2.5
- **R** (Language in which the BiomiX pipeline and MOFA integration are implemented and run) — https://github.com/IxI-97/BiomiX

## Examples

```
bash Biomix_linux_exe.sh  # launch BiomiX (R/Shiny), run MOFA tuning mode on transformed transcriptomics + metabolomics, export mofa_model.hdf5 and mofa_factor_statistics.csv
```

## Evaluation signals

- The exported mofa_model.hdf5 contains exactly the number of factors selected by the tuning stopping criterion (three in the PTB case study)
- Factor 1 shows significant cohort discrimination with Wilcoxon FDR-corrected p.adj = 0.0011 (sd = 0.08) reproduced in mofa_factor_statistics.csv
- Reported factor-1 variance fractions match the paper's values: ~2.23% of metabolomics and ~33.22% of transcriptomics total variance (tuned-model totals ~5.05% metabolomics / ~45.77% transcriptomics)
- The tuning procedure terminated only after at least three models had their last factor explaining less than 1% of variance
- Variance-explained and Wilcoxon statistics in mofa_factor_statistics.csv are internally consistent with the model stored in mofa_model.hdf5

## Limitations

- MOFA variance-explained fractions depend on prior data transformation and outlier/highly-variable-feature removal; BiomiX recommends users carefully review the dataset before use and provides no automated validation
- Mac installation has issues; the README suggests using Linux or Windows (Nextflow workflow planned to replace Mac support)
- BiomiX currently supports transcriptomics, metabolomics, and methylomics only — proteomics and genomics integration are not yet implemented
- BiomiX is described as a first attempt at a gold-standard pipeline, not a definitive solution

## Evidence
