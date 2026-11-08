---
name: contingency-table-construction-oncogene-subgroups
description: Use when binary or k-means-segregated oncogene activation subgroups (phosphoAKT1-high/low
  crossed with MYC-high/low) must be tested for association with ordinal clinical
  variables such as Gleason grade or pathologic stage.
when_to_use_negative:
- The classification variable is continuous densitometric data (phosphoAKT1/MYC Western
  blot values) — use k-means clustering first; contingency tables require the pre-binned
  subgroups.
- Testing association against the 3 metabolic clusters defined by Silhouette analysis
  is fine, but testing against >2 unordered subgroup labels with small expected cell
  counts may violate the 2x2 Fisher setup and needs the 2x4 formulation instead.
- The question concerns metabolite-level differences between subgroups (GSEA enrichment)
  rather than subgroup-vs-clinic association — use enrichment scoring, not contingency
  tables.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: GraphPad Prism
  role: Original Fisher's exact test engine (Prism 5.0) used in the paper to test
    AKT1/MYC subgroup independence from Gleason grade
- name: scipy.stats.fisher_exact
  role: Scriptable equivalent of the Prism Fisher Test for reproducing the 2x2 table
    p-values in Python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1158/0008-5472.can-14-1490
    title: AKT1 and MYC Induce Distinctive Metabolic Fingerprints in Human Prostate
      Cancer
  source_content_hash: sha256:b9465d0d8feac37ae24c95549b6ee68fe703793fce01443021842c9ce868b4cc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - GraphPad Prism
  - scipy.stats.fisher_exact
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# contingency-table-construction-oncogene-subgroups

## Summary

Builds 2x2 and 2x4 contingency tables crossing k-means-defined phosphoAKT1/MYC activation subgroups (e.g. phosphoAKT1-high/MYC-low, 21/60, 35%) against clinical covariates such as Gleason grade, then tests independence with Fisher's exact test (GraphPad Prism 5.0 in the paper; scipy.stats.fisher_exact as a scriptable equivalent).

## When to use

Use when binary or k-means-segregated oncogene activation subgroups (phosphoAKT1-high/low crossed with MYC-high/low) must be tested for association with ordinal clinical variables such as Gleason grade or pathologic stage.

## When NOT to use

- The classification variable is continuous densitometric data (phosphoAKT1/MYC Western blot values) — use k-means clustering first; contingency tables require the pre-binned subgroups.
- Testing association against the 3 metabolic clusters defined by Silhouette analysis is fine, but testing against >2 unordered subgroup labels with small expected cell counts may violate the 2x2 Fisher setup and needs the 2x4 formulation instead.
- The question concerns metabolite-level differences between subgroups (GSEA enrichment) rather than subgroup-vs-clinic association — use enrichment scoring, not contingency tables.

## Inputs

- K-means subgroup counts of phosphoAKT1/MYC densitometric classifications (e.g. 6, 21, 9, 24 of 60 tumors)
- Clinical annotation per tumor: Gleason grade (high 4+3/4+4 vs low 3+3/3+4) and pathologic stage (e.g. all T2)
- 2x2 or 2x4 contingency table cell counts crossing AKT1 status and MYC status against grade

## Outputs

- Contingency tables with margins verified to sum to the cohort size (60)
- Fisher's exact test odds ratios and two-sided p-values per AKT1-vs-grade and MYC-vs-grade table
- An independence verdict at p<0.05 threshold, comparable to Fig. S1A of the source paper

## How to apply

Load the cohort composition — in this study, 60 pathologic stage T2 prostate tumors split into 22 high Gleason (4+3 or 4+4) and 38 low Gleason (3+3 or 3+4), with k-means subgroup counts phosphoAKT1-high/MYC-high = 6, phosphoAKT1-high/MYC-low = 21, phosphoAKT1-low/MYC-high = 9, phosphoAKT1-low/MYC-low = 24. Collapse the 4 subgroups into 2x2 tables (AKT1-high vs AKT1-low, and MYC-high vs MYC-low, each against high vs low Gleason) or keep the full 2x4 layout, verifying that all margins sum to 60 before testing. Run Fisher's exact test on each table with a significance threshold of p<0.05, using scipy.stats.fisher_exact to reproduce the paper's GraphPad Prism 5.0 Fisher Test. Correct application is judged by whether all AKT1-vs-grade and MYC-vs-grade p-values exceed 0.05, matching the reported conclusion that phosphoAKT1 and MYC levels were not associated with Gleason grade (Fig. S1A). Note the contrast case: the same test detected significant differential segregation (p<0.01) of phosphoAKT1-low/MYC-high and phosphoAKT1-high/MYC-low tumors across the 3 Silhouette-defined metabolic clusters, so the skill is valid both for confirming and rejecting independence.

## Related tools

- **GraphPad Prism** (Original Fisher's exact test engine (Prism 5.0) used in the paper to test AKT1/MYC subgroup independence from Gleason grade)
- **scipy.stats.fisher_exact** (Scriptable equivalent of the Prism Fisher Test for reproducing the 2x2 table p-values in Python)

## Examples

```
from scipy.stats import fisher_exact; odds, p = fisher_exact(akt_vs_gleason_2x2); assert p > 0.05, 'expected AKT1/MYC independence from Gleason grade'
```

## Evaluation signals

- All contingency-table margins sum to the full cohort N=60 before any test is run.
- Every Fisher's exact p-value for AKT1-status vs Gleason grade and MYC-status vs Gleason grade exceeds 0.05, reproducing the paper's independence finding (Fig. S1A).
- Subgroup counts match the reported frequencies: 6/60 (10%), 21/60 (35%), 9/60 (15%), 24/60 (40%).
- As a positive control, the Fisher test on phosphoAKT1-low/MYC-high vs phosphoAKT1-high/MYC-low segregation across the 3 Silhouette-defined clusters yields p<0.01.
- Gleason split sums to 22 high + 38 low among the 60 T2 tumors.

## Limitations

- The independence result is specific to the 60 stage T2 tumors analyzed; it cannot be extrapolated to advanced-stage or metastatic cohorts.
- Subgroup labels depend on the upstream k-means clustering of phosphoAKT1/MYC densitometric values; different clustering choices change the table counts and hence the p-values.
- The paper reports this as a null result (no association), which limits statistical power conclusions — non-significance is not evidence of equivalence.

## Evidence

- [results] Levels of phosphoAKT1 and MYC were not associated with the Gleason grade of the tumors (Fig. S1A): "Levels of phosphoAKT1 and MYC were not associated with the Gleason grade of the tumors (Fig. S1A)"
- [results] kmeans subgroups: "K-means clustering of phosphorylated AKT1 and MYC densitometric values (Fig. 1C) was conducted to segregate 4 prostate tumor subgroups"
- [results] subgroup counts: "phosphoAKT1-high/MYC-high (6/60, 10%), phosphoAKT1-high/MYC-low (21/60, 35%), phosphoAKT1-low/MYC-high (9/60, 15%) and phosphoAKT1-low/MYC-low (24/60, 40%)"
- [abstract] independence finding: "the AKT1/MYC status was independent of Gleason grade and pathologic staging"
- [results] positive control: "the phosphoAKT1-low/MYC-high tumors and the phosphoAKT1-high/MYC-low tumors differentially segregated (Fisher test, p<0.01)"
