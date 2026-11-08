---
name: permutation-based-multiple-testing-correction
description: Use when comparing pathway activity levels between two sample groups
  with per-pathway t statistics and a family-wise error correction is needed for many
  correlated pathway tests.
when_to_use_negative:
- Testing a single pre-specified pathway hypothesis where no family-wise correction
  across pathways is needed
- A one-way or multi-factor analysis where a two-group t statistic on mean activity
  is not the test statistic being calibrated
edam_operation: http://edamontology.org/operation_0389
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PLAGE
  role: Web program that performs the SVD-based pathway activity computation, the
    pairwise t-statistic comparisons, and the 10,000-permutation max-statistic calibration
    described here
  repo: http://dulci.biostat.edu/pathways
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1186/1471-2105-6-225
    title: Pathway level analysis of gene expression using singular value decomposition
  source_content_hash: sha256:2898e6d986a1e0c7a523d0802613ab5b0e3435455e51e7614c424e9b905ec26b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PLAGE
  edam_operation: http://edamontology.org/operation_0389
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# permutation-based-multiple-testing-correction

## Summary

Empirical multiple-testing correction that converts per-pathway t statistics to p-values by comparing each observed statistic against the maximum t statistic recorded across all ~400 KEGG/Biocarta pathways in 10,000 sample-label permutations, as used by the PLAGE pipeline.

## When to use

Use when comparing pathway activity levels between two sample groups with per-pathway t statistics and a family-wise error correction is needed for many correlated pathway tests.

## When NOT to use

- Testing a single pre-specified pathway hypothesis where no family-wise correction across pathways is needed
- A one-way or multi-factor analysis where a two-group t statistic on mean activity is not the test statistic being calibrated

## Inputs

- Standardized gene expression matrix (genes × samples; zero mean, unit variance per gene) grouped by sample labels, e.g. Mootha et al. type 2 diabetes muscle data (17 DM2, 17 NGT, 9 IGT) or AGED airway epithelia data (34 current, 18 former, 23 never smokers)
- Observed per-pathway t statistics from pairwise group comparisons of PLAGE activity levels
- Predefined pathway membership sets (~400 Biocarta and KEGG pathways)

## Outputs

- Per-pathway p-values from the empirical max-statistic null distribution, ranked by significance for each pairwise comparison
- Ranked result tables such as table2_top_pathways.tsv (Table 2 smoking comparison) or dm2_ngt_pathway_pvalues.csv (pathway, t statistic, p-value)

## How to apply

First compute an observed two-group t statistic on the mean activity level (the first-row SVD metagene coefficient from PLAGE) of every pathway in the predefined KEGG/Biocarta set (~400 pathways). Then repeat 10,000 times: randomly permute the sample labels, recompute the t statistic for every pathway, and record only the maximum t statistic across pathways for that permutation. Each pathway's p-value is the fraction of recorded maximum statistics that exceed that pathway's observed t statistic; this max-statistic null distribution preserves the empirical inter-pathway correlation structure, so no parametric distribution or step-down scheme is required. Rank pathways by these p-values within each pairwise comparison (e.g. current vs never, former vs never smokers; DM2 vs NGT) and report pathway name, t statistic, and p-value.

## Related tools

- **PLAGE** (Web program that performs the SVD-based pathway activity computation, the pairwise t-statistic comparisons, and the 10,000-permutation max-statistic calibration described here) — http://dulci.biostat.edu/pathways

## Evaluation signals

- Output p-values lie in [0, 1] and equal the fraction of 10,000 recorded maximum t statistics exceeding the observed t statistic
- Permutation counts and the max-across-pathways recording scheme match the article: 10,000 permutations, one maximum t statistic per permutation
- Reproduces Table 2 ordering with 'gamma-Hexachlorocyclohexane degradation' (KEGG) as the top pathway in both current-vs-never and former-vs-never smoker comparisons
- Reproduces the DM2 vs NGT result: Biocarta 'Activation of cAMP-dependent protein kinase, PKA' pathway top-ranked with p ≈ 0.4, consistent with no significant pathway
- Reported p-values are tied to reported t statistics in the output files (dm2_ngt_pathway_pvalues.csv, table2_top_pathways.tsv), i.e. ranking is by empirical p-value not raw t

## Limitations

- Pathway sets continue to expand and evolve — the versions used for the article are only archived at the PLAGE website, so p-values are version-dependent
- With ~400 pathways tested, empirical p-values can be high; e.g. the top DM2 vs NGT pathway reached only p = 0.4, so 'no significant pathway' is an expected outcome
- Calibration is specific to the two-group t statistic on mean pathway activity levels; other test statistics require re-deriving the max-statistic null

## Evidence

- [discussion] we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway: "we perform a large number (10,000 for this article) of comparisons by randomly permuting the sample labels and for each permutation recording the t statistic for the most significant pathway"
- [results] In the smoking study, the pathway most significantly differentially expressed in both current and former smokers as compared to never smokers is 'gamma-Hexachlorocyclohexane degradation' (KEGG).: "the pathway most significantly differentially expressed in both current and former smokers as compared to never smokers is 'gamma-Hexachlorocyclohexane degradation'"
