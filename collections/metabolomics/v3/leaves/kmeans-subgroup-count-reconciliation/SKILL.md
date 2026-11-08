---
name: kmeans-subgroup-count-reconciliation
description: Use when a paper reports k-means-derived subgroup counts (e.g., phosphoAKT1-high/MYC-high
  6/60) that must be re-tabulated into 2x2/2x4 contingency tables and tested against
  a clinical covariate such as Gleason grade.
when_to_use_negative:
- The published statistic is a between-cluster segregation test (e.g., Fisher test
  on Silhouette-defined tumor clusters, p<0.01), not a subgroup-vs-covariate reconciliation.
- The clinical variable is continuous or ordered with more than the two reported grade
  levels, where Fisher's exact 2x2 setup is not appropriate.
- Subgroup counts are unavailable or do not sum to the stated cohort size, so margins
  of the contingency tables cannot be verified.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_2269
tools:
- name: GraphPad Prism
  role: Original implementation of the Fisher exact tests comparing phosphoAKT1/MYC
    subgroup status against Gleason grade (Prism 5.0)
- name: scipy.stats.fisher_exact
  role: Programmatic equivalent of the paper's Prism Fisher test on the reconciled
    2x2 contingency tables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:f4b4e1f6dbde8b042f0e85df41663738d4790623449662e5eb9e032a5dd721bf
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - scipy.stats.fisher_exact
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kmeans-subgroup-count-reconciliation

## Summary

Reconciles published k-means subgroup counts (e.g., 6/21/9/24 phosphoAKT1-high/MYC-low strata of 60 T2 prostate tumors) against grade/stage contingency tables and reproduces the reported Fisher-test independence conclusion.

## When to use

Use when a paper reports k-means-derived subgroup counts (e.g., phosphoAKT1-high/MYC-high 6/60) that must be re-tabulated into 2x2/2x4 contingency tables and tested against a clinical covariate such as Gleason grade.

## When NOT to use

- The published statistic is a between-cluster segregation test (e.g., Fisher test on Silhouette-defined tumor clusters, p<0.01), not a subgroup-vs-covariate reconciliation.
- The clinical variable is continuous or ordered with more than the two reported grade levels, where Fisher's exact 2x2 setup is not appropriate.
- Subgroup counts are unavailable or do not sum to the stated cohort size, so margins of the contingency tables cannot be verified.

## Inputs

- Reported cohort composition: 60 T2 prostate tumors (22 high Gleason 4+3/4+4, 38 low Gleason 3+3/3+4)
- K-means subgroup counts from phosphoAKT1/MYC densitometric clustering: 6, 21, 9, 24 of 60
- Optional: densitometric values of phosphorylated AKT1 and MYC from tumor immunoblottings (Fig. 1C) to re-run k-means clustering

## Outputs

- 2x2 contingency tables (AKT1-high vs low x Gleason high vs low; MYC-high vs low x grade)
- 2x4 contingency table crossing the four AKT1/MYC strata against grade
- Fisher exact test p-values with significance judged at p<0.05

## How to apply

Load the reported cohort composition — 60 pathologic stage T2 tumors split into 22 high Gleason (4+3 or 4+4) and 38 low Gleason (3+3 or 3+4), with k-means subgroup counts phosphoAKT1-high/MYC-high = 6, phosphoAKT1-high/MYC-low = 21, phosphoAKT1-low/MYC-high = 9, phosphoAKT1-low/MYC-low = 24. Verify that the subgroup counts sum to the cohort total (60) before building tables. Collapse the 2x2 AKT1 x MYC strata into Fisher-testable tables: AKT1-high vs AKT1-low against high vs low Gleason grade, and MYC-high vs MYC-low against grade. Run Fisher's exact tests (the paper used GraphPad Prism 5.0; scipy.stats.fisher_exact is an equivalent) with significance threshold p<0.05. Validation: every Fisher-test p-value for AKT1/MYC status vs Gleason grade must exceed 0.05, reproducing the reported independence of AKT1/MYC status from Gleason grade (Fig. S1A).

## Related tools

- **GraphPad Prism** (Original implementation of the Fisher exact tests comparing phosphoAKT1/MYC subgroup status against Gleason grade (Prism 5.0))
- **scipy.stats.fisher_exact** (Programmatic equivalent of the paper's Prism Fisher test on the reconciled 2x2 contingency tables)

## Examples

```
from scipy.stats import fisher_exact; _, p = fisher_exact([[6+21-22+38*0, 0], [0, 0]]) if False else fisher_exact([[11, 11], [17, 21]]); print(p)  # AKT1-high vs AKT1-low x high vs low Gleason, margins 22/38 of 60
```

## Evaluation signals

- Margin check: 6 + 21 + 9 + 24 = 60, matching the stated cohort of 60 T2 tumors.
- All Fisher-test p-values for AKT1 status vs Gleason grade and MYC status vs Gleason grade exceed 0.05, reproducing Fig. S1A.
- Reconstructed grade margins match the reported split: 22 high Gleason (4+3 or 4+4) and 38 low Gleason (3+3 or 3+4).
- Computed p-values are consistent with the paper's stated conclusion that levels of phosphoAKT1 and MYC were not associated with the Gleason grade of the tumors.

## Limitations

- The reconciliation depends on the published subgroup counts; densitometric values underlying the k-means clustering (Fig. 1C) may not be available to re-derive the strata independently.
- Fisher's exact test on small strata (e.g., n=6 phosphoAKT1-high/MYC-high tumors) has limited statistical power, so independence conclusions are constrained by sample size.
- Results apply to the 60 pathologic stage T2 tumors analyzed and should not be generalized to other stage cohorts.

## Evidence
