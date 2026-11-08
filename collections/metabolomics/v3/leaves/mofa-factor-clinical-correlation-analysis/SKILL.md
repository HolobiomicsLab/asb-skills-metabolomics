---
name: mofa-factor-clinical-correlation-analysis
description: Use when MOFA factor values (latent variables from multi-omics integration)
  must be interpreted against a clinical metadata table, e.g. to rank discriminant
  factors by their association with numeric or binary clinical features.
when_to_use_negative:
- The analysis goal is annotating MOFA factors via literature rather than clinical
  data — BiomiX provides a separate PubMed bibliography mining route and should be
  used instead.
- No clinical metadata (numeric or binary phenotypes) is available for the samples
  in the MOFA model.
- The task is a single-omics differential analysis (e.g. DESeq2/Limma transcriptomics
  or Wilcoxon metabolomics peak testing) rather than interpretation of MOFA integration
  factors.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: BiomiX
  role: Implements the clinical correlation module for MOFA factor interpretation,
    including Pearson and Wilcoxon testing and BH FDR correction
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the factor values matrix that is correlated against clinical data
- name: R
  role: Execution environment in which the correlation analysis module runs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:c861574b5bd91b590cac9792204d494683a923121bb38f698fae8be65562e38f
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# mofa-factor-clinical-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs BiomiX's MOFA factor interpretation module, which correlates discriminant MOFA factor values with patient clinical metadata to identify factors associated with clinical traits. Numeric features are tested with Pearson correlation and binary features with the Wilcoxon rank-sum test, followed by Benjamini–Hochberg FDR correction.

## When to use

Use when MOFA factor values (latent variables from multi-omics integration) must be interpreted against a clinical metadata table, e.g. to rank discriminant factors by their association with numeric or binary clinical features.

## When NOT to use

- The analysis goal is annotating MOFA factors via literature rather than clinical data — BiomiX provides a separate PubMed bibliography mining route and should be used instead.
- No clinical metadata (numeric or binary phenotypes) is available for the samples in the MOFA model.
- The task is a single-omics differential analysis (e.g. DESeq2/Limma transcriptomics or Wilcoxon metabolomics peak testing) rather than interpretation of MOFA integration factors.

## Inputs

- MOFA factor values matrix (samples × factors) from the multi-omics integration step
- Clinical metadata table with numeric and/or binary features per sample
- R / BiomiX execution environment (BiomiX.sh on Linux/Windows)

## Outputs

- mofa_factor_clinical_correlations.csv — per-factor significance table with factor-feature pair, test type, statistic, raw p-value, and BH-adjusted p-value
- Ranked list of discriminant MOFA factors by clinical association for downstream interpretation

## How to apply

Load the MOFA factor values matrix and the clinical metadata table into R within the BiomiX environment. For each factor-feature pair, compute Pearson correlation coefficients and p-values against every numeric clinical feature, and apply the Wilcoxon rank-sum test on the positive/negative factor groups for every binary clinical feature. Pool all raw p-values from both test types and correct them with the Benjamini–Hochberg false discovery rate method. Assemble a per-factor significance table listing each factor-feature pair with its test type, statistic, raw p-value, and BH-adjusted p-value, written to mofa_factor_clinical_correlations.csv, so discriminant MOFA factors can be ranked by clinical association alongside the other BiomiX interpretation routes (pathway analysis and PubMed bibliography mining).

## Related tools

- **BiomiX** (Implements the clinical correlation module for MOFA factor interpretation, including Pearson and Wilcoxon testing and BH FDR correction) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Produces the factor values matrix that is correlated against clinical data)
- **R** (Execution environment in which the correlation analysis module runs)

## Examples

```
Rscript -e 'ct <- cor.test(factor_values[,f], clinical$feature, method="pearson"); p <- wilcox.test(factor_values[clinical$bin==1,f], factor_values[clinical$bin==0,f])'
```

## Evaluation signals

- Output file mofa_factor_clinical_correlations.csv exists and is valid CSV.
- Table contains an entry for every tested factor-feature combination (full Cartesian product of MOFA factors and clinical features).
- A BH-adjusted p-value column is present so factors can be ranked by clinical association.
- Each row records the correct test type: Pearson correlation for numeric clinical features, Wilcoxon rank-sum test for binary features.
- Raw p-values, when pooled and BH-corrected, match the adjusted column values.

## Limitations

- Clinical correlation is only one of BiomiX's three factor annotation approaches (correlation, pathway analysis via EnrichR/GSEA, and PubMed bibliography mining); other tools such as X-omics ACTION only perform correlation analysis, so results from this module alone give a partial interpretation.
- Undefined/other omics contributors were not always recognized by the correlation analysis in earlier BiomiX versions — a bug fixed in version 2.4, so older versions may miss these contributors.
- BiomiX relies on users carefully reviewing datasets before use; there is no automated validation of the clinical metadata.

## Evidence
