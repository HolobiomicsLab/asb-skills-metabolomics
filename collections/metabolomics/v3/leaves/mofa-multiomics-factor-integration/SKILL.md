---
name: mofa-multiomics-factor-integration
description: Use when two or more matched omics feature matrices (e.g. RNA-seq counts,
  methylation beta values, metabolomics peaks) plus cohort labels need integration
  to find latent factors separating conditions, with tuned factor number and factor
  annotation.
when_to_use_negative:
- Only single-omics data are available — use BiomiX's single-omics pipelines (DESeq2/Limma,
  ChAMP, TidyMass/Wilcoxon) instead of MOFA integration
- The goal is supervised prediction of class labels rather than latent-factor exploration
  — MOFA is a middle-integration, unsupervised factor model
- Proteomics or genomics data are the primary inputs — BiomiX does not yet support
  these omics types
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: BiomiX
  role: R-based graphical interface that wraps MOFA integration, factor-number tuning,
    and the factor-interpretation pipeline (clinical correlation, EnrichR/GSEA, PubMed)
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Multi-omics integration engine identifying shared sources of variation and
    discriminant factors across omics views; handles missing data natively
- name: R
  role: Runtime language of BiomiX in which the MOFA integration and statistics are
    executed
- name: EnrichR
  role: Pathway exploration of top contributors of discriminant MOFA factors
- name: GSEA
  role: Biological process exploration of top contributors of discriminant MOFA factors
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:fffd628ee9aea5227041f6b5459bc7c6fe1a2ab5562e6d77970b3c5ab2c5699a
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
  - ChAMP
  - DESeq2
  - EnrichR
  - GSEA
  - Limma
  - MOFA (Multi-Omics Factor Analysis)
  - R
  - TidyMass
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# mofa-multiomics-factor-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Multi-Omics Factor Analysis (MOFA) via BiomiX to integrate omics matrices (e.g. transcriptomics, methylomics, metabolomics), tune the total number of factors, identify cohort-discriminant factors, and interpret them through clinical correlation, EnrichR/GSEA pathway analysis, and PubMed bibliography mining.

## When to use

Use when two or more matched omics feature matrices (e.g. RNA-seq counts, methylation beta values, metabolomics peaks) plus cohort labels need integration to find latent factors separating conditions, with tuned factor number and factor annotation.

## When NOT to use

- Only single-omics data are available — use BiomiX's single-omics pipelines (DESeq2/Limma, ChAMP, TidyMass/Wilcoxon) instead of MOFA integration
- The goal is supervised prediction of class labels rather than latent-factor exploration — MOFA is a middle-integration, unsupervised factor model
- Proteomics or genomics data are the primary inputs — BiomiX does not yet support these omics types

## Inputs

- Normalized/transformed omics feature matrices (transcriptomics, methylomics, metabolomics; optionally imputed or left unimputed with missing values)
- Sample metadata with condition/cohort labels (e.g. mutated vs unmutated IGHV, PTB vs healthy control)
- Clinical annotations for factor correlation (e.g. 11q22.3 deletion, trisomy 12, gender, TP53/SF3B1/del17p13)

## Outputs

- Fitted MOFA model exported as mofa_model.hdf5 (HDF5)
- Per-factor variance-explained and Wilcoxon discrimination statistics (e.g. mofa_factor_statistics.csv)
- Factor annotation table combining clinical correlations, EnrichR/GSEA pathway terms, and PubMed literature hits per discriminant factor (e.g. mofa_factor_annotation_cll.csv)

## How to apply

Load normalized/transformed omics matrices (optionally imputed via NIPALS, random forest, lasso, or mean/median — MOFA can also handle missing values natively) into BiomiX's MOFA module. Run tuning mode by training a series of models with iteratively increasing factor counts until at least three candidate models have a last factor explaining less than 1% of variance, then rank candidates by their ability to discriminate cohorts using the Wilcoxon test on factor values with FDR correction and fit the final model. Identify discriminant factors (e.g. factor 1 in PTB vs HC with p.adj=0.0011, ~2.23% metabolomics / ~33.22% transcriptomics variance; five discriminant factors of ten in the CLL IGHV case) and interpret them via clinical data correlation, EnrichR/GSEA biological process analysis, and the automated PubMed search on top contributors.

## Related tools

- **BiomiX** (R-based graphical interface that wraps MOFA integration, factor-number tuning, and the factor-interpretation pipeline (clinical correlation, EnrichR/GSEA, PubMed)) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Multi-omics integration engine identifying shared sources of variation and discriminant factors across omics views; handles missing data natively)
- **R** (Runtime language of BiomiX in which the MOFA integration and statistics are executed)
- **EnrichR** (Pathway exploration of top contributors of discriminant MOFA factors)
- **GSEA** (Biological process exploration of top contributors of discriminant MOFA factors)

## Examples

```
bash Biomix_linux_exe.sh
```

## Evaluation signals

- The exported mofa_model.hdf5 contains exactly the expected number of factors (e.g. three in the PTB tuning case; ten in the CLL ten-factor model)
- The factor statistics table reproduces reported per-factor variance fractions and adjusted p-values within stated ranges (e.g. factor 1: p.adj=0.0011, ~2.23% metabolomics / ~33.22% transcriptomics variance in PTB)
- The stopping criterion in tuning mode is met: at least three candidate models whose last factor explains less than 1% of variance
- The annotation table records correlations with named clinical variables plus EnrichR/GSEA pathway terms and PubMed literature hits for every discriminant factor
- In the CLL validation, five of the ten factors (factors 1, 3, 4, 5 and 6) discriminate mutated vs unmutated IGHV patients

## Limitations

- BiomiX has limited omics analysers — proteomics and genomics are not yet supported for integration
- Users must carefully review datasets before use; there is no automated validation of inputs
- Mac installation is problematic; Linux or Windows is recommended (Mac to be replaced by a Nextflow workflow)

## Evidence

- [results] train a series of models with iteratively increasing factor counts until the stopping criterion is met: [evidence span withheld — anchor status: not_found]
