---
name: fdr-multiple-testing-correction
description: Use when thousands of pathway-level importance scores (VIP, MB-VIP, RGCCA
  loadings, or sparse-DIABLO bootstrap non-zero-loading statistics) must be tested
  against permutation nulls and converted into pathway detection calls with controlled
  false discovery rate.
when_to_use_negative:
- The analysis involves a single pre-specified hypothesis test with no multiplicity
  across pathways, so FDR correction is unnecessary
- The data are real (non-semi-synthetic) datasets with no known target pathway or
  ground-truth labels, so confusion-matrix AUC cannot be computed
- The question is conventional ORA/GSEA-style enrichment of differential molecules
  rather than detection of an enriched pathway from predictive model importance statistics
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PathIntegrate
  role: Produces the MB-VIP/VIP pathway importance statistics whose permutation nulls
    are BH-FDR corrected for pathway detection evaluation
  repo: https://github.com/cwieder/PathIntegrate
- name: DIABLO
  role: Provides RGCCA component loadings and sparse-DIABLO bootstrap non-zero-loading
    statistics that are tested against permutation nulls under the same FDR criterion
- name: SciKitLearn
  role: Used for compatible model optimisation and evaluation when fitting the supervised
    models whose pathway scores feed the detection statistics
- name: Python
  role: Implementation language for the permutation testing, BH correction, and confusion-matrix
    AUC pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pcbi.1011814
    title: 'PathIntegrate: Multivariate modelling approaches for pathway-based multi-omics
      data integration'
  source_content_hash: sha256:183b3128d6869bfef1cffa9ede4edb059f11c9df1381b3cc32e841ba574efa71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cwieder/PathIntegrate
  tools:
  - DIABLO
  - PathIntegrate
  - Python
  - SciKitLearn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# fdr-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg FDR correction to pathway importance statistics derived from permutation-based null distributions (10,000 permutations) when assessing whether PathIntegrate (VIP/MB-VIP) and DIABLO-based pathway models truly detect an enriched target pathway in semi-synthetic multi-omics simulations.

## When to use

Use when thousands of pathway-level importance scores (VIP, MB-VIP, RGCCA loadings, or sparse-DIABLO bootstrap non-zero-loading statistics) must be tested against permutation nulls and converted into pathway detection calls with controlled false discovery rate.

## When NOT to use

- The analysis involves a single pre-specified hypothesis test with no multiplicity across pathways, so FDR correction is unnecessary
- The data are real (non-semi-synthetic) datasets with no known target pathway or ground-truth labels, so confusion-matrix AUC cannot be computed
- The question is conventional ORA/GSEA-style enrichment of differential molecules rather than detection of an enriched pathway from predictive model importance statistics

## Inputs

- Observed pathway importance scores (VIP, MB-VIP, DIABLO pathway relevance from RGCCA loadings, sparse-DIABLO bootstrap non-zero-loading statistic) per pathway
- Permutation-based null distributions of each pathway statistic (10,000 label permutations)
- Effect-size-labelled semi-synthetic multi-omics simulation datasets with an embedded target enriched pathway

## Outputs

- BH-FDR-adjusted p-values / detection calls per pathway per method
- Pathway detection confusion matrices (true/false positives and negatives per method)
- Confusion-matrix AUC per method per effect size, written to pathway_detection_auc.csv

## How to apply

For each pathway importance statistic (MB-VIP/VIP from PathIntegrate Multi-View (MB-PLS) and Single-View models, or pathway relevance scores from DIABLO RGCCA component loadings), first generate a permutation-based null distribution by permuting the outcome labels 10,000 times. Build pathway detection confusion matrices by classifying a pathway as detected when its observed statistic exceeds the permutation null at the Benjamini-Hochberg FDR-controlled threshold. The same detection criterion is applied uniformly to the sparse L1-norm DIABLO variant using its bootstrap non-zero-loading statistic. Finally compute confusion-matrix AUCs per method per effect size, verifying the expected behaviour that detection AUC >= 0.90 even at low effect size 0.25, and write results to pathway_detection_auc.csv. The rationale is that BH correction over 10,000-permutation nulls controls false-positive pathway calls when many pathways are scored simultaneously in low signal-to-noise scenarios.

## Related tools

- **PathIntegrate** (Produces the MB-VIP/VIP pathway importance statistics whose permutation nulls are BH-FDR corrected for pathway detection evaluation) — https://github.com/cwieder/PathIntegrate
- **DIABLO** (Provides RGCCA component loadings and sparse-DIABLO bootstrap non-zero-loading statistics that are tested against permutation nulls under the same FDR criterion)
- **SciKitLearn** (Used for compatible model optimisation and evaluation when fitting the supervised models whose pathway scores feed the detection statistics)
- **Python** (Implementation language for the permutation testing, BH correction, and confusion-matrix AUC pipeline)

## Evaluation signals

- Detection AUC for the target enriched pathway is >= 0.90 for both PathIntegrate and pathway-based DIABLO even at effect size 0.25
- The number of permutation iterations equals 10,000 and BH FDR correction is applied before thresholding pathway statistics into the confusion matrix
- Sparse DIABLO shows a slight improvement over standard DIABLO at high effect sizes in the AUC table
- Results table pathway_detection_auc.csv contains one AUC per method per effect size with no missing entries
- All methods are scored under the same confusion-matrix detection criterion so AUCs are directly comparable

## Limitations

- Ground-truth confusion-matrix AUC evaluation only works on semi-synthetic data with a known embedded target pathway; real datasets lack validated molecular/pathway-level signals
- Pathway coverage (proportion of observable molecules per pathway) and reliance on current pathway annotation knowledge are inherent bottlenecks that can distort which pathways can be detected
- Comparison to conventional pathway analysis approaches (ORA, GSEA, MultiGSEA, IMPaLA) was not performed, so the FDR-based detection benchmark does not situate the statistics against those methods

## Evidence

- [methods] Permutation-based nulls with BH correction: [evidence span withheld — anchor status: not_found]
- [results] High detection AUC at low effect size: [evidence span withheld — anchor status: not_found]
- [abstract] Benefit of pathway grouping in low signal-to-noise: "Using semi-synthetic data we demonstrate the benefit of grouping molecules into pathways to detect signals in low signal-to-noise scenarios"
- [other] Permutation-tested VIP/MB-VIP workflow: [evidence span withheld — anchor status: not_found]
- [readme] SciKitLearn-compatible evaluation: "designed to be compatible with many SciKitLearn [39] functions, enabling fast and efficient model optimisation and evaluation."
