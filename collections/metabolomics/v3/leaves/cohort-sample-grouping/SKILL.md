---
name: cohort-sample-grouping
description: Use when cohort metadata (self-reported medication use) is unreliable
  or missing and untargeted MS/MS metabolomics contains drug-associated annotations
  that can be clustered to define exposure strata.
when_to_use_negative:
- When the analysis goal is drug annotation itself rather than exposure-based cohort
  stratification — use the GNPS Drug Library screening workflow alone
- When the result informs immediate clinical decision-making — the article states
  the GNPS Drug Library 'is not intended for immediate clinical decision-making'
- When only parent-drug signals are detectable but only metabolized versions are present
  in samples — exposure will be missed if the parent drug alone is considered
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0605
tools:
- name: GNPS (Global Natural Product Social Molecular Networking)
  role: Provides the GNPS Drug Library MS/MS reference spectra used to annotate drug
    exposures that define the binary sample-by-ARV-exposure matrix for clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-65993-5
    title: A resource to empirically establish drug exposure records directly from
      untargeted metabolomics data
  source_content_hash: sha256:1cd4ab4ce3168c448f0ebf6feca075ca3fe4d31391dfb4b9e8cc6234e72fb30f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GNPS (Global Natural Product Social Molecular Networking)
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cohort-sample-grouping

## Summary

Empirically partitions cohort samples into exposure groups using detected drug signals from untargeted metabolomics instead of self-reported metadata, then tests for outcome associations (e.g., Kruskal-Wallis on N-acyl lipid abundances) across the derived groups.

## When to use

Use when cohort metadata (self-reported medication use) is unreliable or missing and untargeted MS/MS metabolomics contains drug-associated annotations that can be clustered to define exposure strata.

## When NOT to use

- When the analysis goal is drug annotation itself rather than exposure-based cohort stratification — use the GNPS Drug Library screening workflow alone
- When the result informs immediate clinical decision-making — the article states the GNPS Drug Library 'is not intended for immediate clinical decision-making'
- When only parent-drug signals are detectable but only metabolized versions are present in samples — exposure will be missed if the parent drug alone is considered

## Inputs

- Untargeted metabolomics MS/MS feature table (e.g., 322 HNRC fecal metabolomics samples)
- GNPS Drug Library MS/MS reference spectra with machine-readable pharmacologic metadata
- Targeted reference library (e.g., 851-compound N-acyl lipid reference library)
- Sample-level metadata for cohort filtering (e.g., confirmed HIV status, 222 PWH)
- Per-analyte abundance profiles (e.g., nine N-acyl lipids including histamine-C2:0)

## Outputs

- Binary sample-by-ARV-exposure matrix
- Sample cluster assignments (e.g., four ARV exposure groups from Ward linkage on Euclidean distance)
- Per-lipid Kruskal-Wallis test statistics and p-values (significance at p<0.05)
- Combined results file (e.g., nacyl_lipid_arv_analysis.csv)

## How to apply

Annotate the MS/MS feature table (e.g., 322 HNRC fecal samples) against the GNPS Drug Library plus a targeted reference set such as an 851-compound N-acyl lipid library to flag drug-associated signals per sample. Restrict to the confirmed subset (222 people with HIV) and summarize detected ARV-associated signals per sample into a binary sample-by-ARV-exposure matrix. Apply hierarchical clustering with Ward linkage on Euclidean distance over that binary matrix, cutting to a defensible number of groups (four ARV exposure groups in the HNRC example). Extract abundance profiles of target analytes (e.g., nine N-acyl lipids including histamine-C2:0) across the clusters and run Kruskal-Wallis tests per lipid, keeping associations significant at p<0.05. The rationale is that empirically detected exposure strata reveal drug-associated differences (Fig. 2d) that self-reported medication classifications do not capture.

## Related tools

- **GNPS (Global Natural Product Social Molecular Networking)** (Provides the GNPS Drug Library MS/MS reference spectra used to annotate drug exposures that define the binary sample-by-ARV-exposure matrix for clustering)

## Examples

```
from scipy.cluster.hierarchy import linkage, fcluster; groups = fcluster(linkage(exposure_matrix.values, method='ward', metric='euclidean'), t=4, criterion='maxclust')
```

## Evaluation signals

- Cluster assignments partition the confirmed cohort (222 PWH) into the expected number of exposure groups (four) with Ward linkage on Euclidean distance over the binary exposure matrix
- Kruskal-Wallis p-values reproduce the article's Fig. 2d pattern: nine N-acyl lipids including histamine-C2:0 significant at p<0.05 across the four empirically derived groups
- Contrast check: the same per-lipid tests run against self-reported ARV-use classifications yield no significant differences — a divergence that confirms the empirical stratification is capturing signal metadata misses
- Output schema check: nacyl_lipid_arv_analysis.csv contains both sample cluster assignments and per-lipid test statistics

## Limitations

- Stratification depends on detected drug signals; only metabolized versions being present means missed exposure if only the parent drug is considered
- Conventional/self-reported exposure records miss over-the-counter medications, dietary supplements, non-adherence, undocumented drug use, and unrealized food-supply exposures, complicating validation of derived groups
- Empirical drug-exposure readouts are not for immediate clinical decision-making; they support research-grade, data-driven exploration only
- The resource depends on community deposition of reference libraries and expansion of public metabolomics datasets

## Evidence

- [results] Histamine-C2:0 and eight other N-acyl lipids differed significantly (Kruskal-Wallis, p<0.05) across four empirically derived ARV exposure groups: [evidence span withheld — anchor status: not_found]
- [abstract] We uncover drug-associated alterations in microbiota-derived N-acyl lipids that are not captured when stratifying by self-reported medication use.: "We uncover drug-associated alterations in microbiota-derived N-acyl lipids that are not captured when stratifying by self-reported medication use."
- [intro] it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered: "it is common that only metabolized versions of a drug are present in the sample, leading to missed drug exposure if only the parent drug is considered"
- [abstract] While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable.: "While not intended for immediate clinical decision-making, it supports data-driven exploration of drug exposures where traditional records are limited or unreliable."
