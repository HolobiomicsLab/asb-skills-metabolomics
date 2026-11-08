---
name: clinical-metadata-association-summarization
description: Use when MOFA factor values and a clinical metadata table (numeric and
  binary patient features) are available and the goal is to rank discriminant MOFA
  factors by their statistical association with clinical traits.
when_to_use_negative:
- MOFA integration has not yet been run and no factor values matrix exists to correlate
- The second table is an omics feature matrix (transcriptomics/metabolomics/methylomics)
  rather than patient clinical metadata
- The interpretation target is pathway enrichment or PubMed bibliography mining rather
  than clinical trait association
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: BiomiX
  role: Runs the MOFA factor-clinical data correlation module, combining Pearson and
    Wilcoxon tests with BH FDR correction as one of its three factor annotation approaches
  repo: https://github.com/IxI-97/BiomiX
- name: MOFA (Multi-Omics Factor Analysis)
  role: Produces the factor values matrix whose discriminant factors are correlated
    against clinical data
- name: R
  role: Environment in which the correlation analysis module is executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s12859-024-06022-y
    title: BiomiX, a user-friendly bioinformatic tool for democratized analysis and
      integration of multiomics data
  source_content_hash: sha256:cee4669c24446267875a211edb9f8b579bb8a425f7335b1cceaf33410a31515c
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# clinical-metadata-association-summarization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs BiomiX's MOFA factor interpretation step in which discriminant MOFA factors are correlated against patient clinical metadata, using Pearson correlation for numeric clinical features and Wilcoxon rank-sum tests for binary features, with Benjamini-Hochberg FDR correction pooling all raw p-values.

## When to use

Use when MOFA factor values and a clinical metadata table (numeric and binary patient features) are available and the goal is to rank discriminant MOFA factors by their statistical association with clinical traits.

## When NOT to use

- MOFA integration has not yet been run and no factor values matrix exists to correlate
- The second table is an omics feature matrix (transcriptomics/metabolomics/methylomics) rather than patient clinical metadata
- The interpretation target is pathway enrichment or PubMed bibliography mining rather than clinical trait association

## Inputs

- MOFA factor values matrix (samples x factors)
- Clinical metadata table with numeric and binary patient features

## Outputs

- Per-factor clinical association table (factor-feature pairs with test type, statistic, raw p-value, BH-adjusted p-value), e.g. mofa_factor_clinical_correlations.csv
- Ranked list of discriminant MOFA factors by clinical association

## How to apply

Load the MOFA factor values matrix and the clinical metadata table into the BiomiX R environment. For each MOFA factor, compute Pearson correlation coefficients and p-values against every numeric clinical feature, and test every binary clinical feature with a Wilcoxon rank-sum test comparing the factor's positive versus negative sample groups. Pool the raw p-values from both the Pearson and Wilcoxon analyses and apply Benjamini-Hochberg false discovery rate correction across them. Assemble a per-factor significance table listing each factor-feature pair with test type, statistic, raw p-value, and BH-adjusted p-value, then rank discriminant factors by their adjusted associations so the top clinical drivers can feed into the downstream pathway and PubMed bibliography interpretation modules.

## Related tools

- **BiomiX** (Runs the MOFA factor-clinical data correlation module, combining Pearson and Wilcoxon tests with BH FDR correction as one of its three factor annotation approaches) — https://github.com/IxI-97/BiomiX
- **MOFA (Multi-Omics Factor Analysis)** (Produces the factor values matrix whose discriminant factors are correlated against clinical data)
- **R** (Environment in which the correlation analysis module is executed)

## Evaluation signals

- Output CSV exists and contains one row for every tested factor-feature combination
- A BH-adjusted p-value column is present so factors can be ranked by adjusted significance
- Numeric clinical features are associated via Pearson correlation while binary features are associated via Wilcoxon test, with test type recorded per row
- Adjusted p-values are derived from pooling raw p-values across both Pearson and Wilcoxon tests (single BH correction)
- Contributors from any omics type, including the 'Undefined' omics, are recognized in the correlation output (BiomiX 2.4 bug fix)

## Limitations

- Bug in BiomiX < 2.4: bibliography and correlation analysis did not properly recognize undefined omics contributors to correlate and interpret
- Clinical correlation is only one of BiomiX's three factor annotation methods; other tools such as X-omics ACTION perform only correlation analysis
- Users are advised to carefully review datasets before use; BiomiX provides no automated validation of inputs

## Evidence

- [abstract] discriminant MOFA factors are correlated with clinical data, and the top contributing pathways are explored: "discriminant MOFA factors are correlated with clinical data, and the top contributing pathways are explored"
- [intro] correlation with clinical data as factor interpretation method: "identification of the biological processes behind the factors of interest through clinical data correlation"
- [discussion] X-omics ACTION only performs correlation analysis to annotate MOFA factors: "while X-omics ACTION only performs correlation analysis to annotate MOFA factors, BiomiX relies on three methods based on different approaches"
