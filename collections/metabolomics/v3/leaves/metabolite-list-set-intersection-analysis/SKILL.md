---
name: metabolite-list-set-intersection-analysis
description: Use when you have two or more feature-selection lists (e.g. VIP > 1 metabolites
  from PLS-DA under different data-processing strategies) and need to quantify shared
  vs. uniquely selected features, as in the LMM vs. M0 comparison of the CRC adipose
  lipid data.
when_to_use_negative:
- The input is a raw metabolomics feature matrix rather than already-selected significant-feature
  lists — run PLS-DA with the VIP > 1 filter first
- Feature lists come from different metabolite annotation spaces or different thresholds,
  making set comparison invalid
- The goal is comparing predictive performance (accuracy/precision/sensitivity/specificity)
  of models rather than overlap of selected features
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: lmm2met
  role: R package that produced the LMM-processed metabolomics matrix whose PLS-DA
    feature list feeds the set comparison
  repo: https://github.com/kwanjeeraw/lmm2met
- name: mixOmics
  role: Used to run PLS-DA whose VIP > 1 significant features form the lists being
    intersected
- name: R
  role: Environment in which the set intersection, difference, union and Venn region
    counts are computed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:abac4cb4ba08d51d9ac808231e1b3fc3f6371ec849416d9d807ab1ef860d263f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - lmm2met
  - mixOmics
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# metabolite-list-set-intersection-analysis

## Summary

Computes set intersection, difference, and union of discriminative-metabolite lists (VIP > 1) derived from PLS-DA of LMM-processed versus M0-processed metabolomics data, quantifying which features each processing strategy uniquely captures and tabulating Venn region counts.

## When to use

Use when you have two or more feature-selection lists (e.g. VIP > 1 metabolites from PLS-DA under different data-processing strategies) and need to quantify shared vs. uniquely selected features, as in the LMM vs. M0 comparison of the CRC adipose lipid data.

## When NOT to use

- The input is a raw metabolomics feature matrix rather than already-selected significant-feature lists — run PLS-DA with the VIP > 1 filter first
- Feature lists come from different metabolite annotation spaces or different thresholds, making set comparison invalid
- The goal is comparing predictive performance (accuracy/precision/sensitivity/specificity) of models rather than overlap of selected features

## Inputs

- List of significant metabolites (VIP > 1) from LMM-PLS-DA (e.g. from Table S-3 of the CRC samples)
- List of significant metabolites (VIP > 1) from M0-PLS-DA
- Optionally, additional per-dataset significant-feature lists (union across all processed datasets)

## Outputs

- Set-intersection membership of metabolites discriminative in all input lists
- Per-dataset uniquely selected metabolite lists (set differences)
- Union list with Venn region counts (e.g. venn_intersection_union.csv)

## How to apply

Load the significant-metabolite lists obtained at the VIP > 1 threshold from PLS-DA of the LMM-processed and M0-processed matrices (here the 158 annotated lipids from VAT and SAT of 59 CRC patients, Table S-3). Compute the set intersection to obtain metabolites discriminative under both processing strategies, and set differences to isolate metabolites uniquely selected after LMM fitting (e.g. cholesterol-type signals missed before LMM fitting) versus uniquely selected in the M0 data. Compute the union and verify the invariant |union| = |LMM list| + |M0 list| - |intersection| before writing memberships and region counts (e.g. to venn_intersection_union.csv) for a Venn depiction as in Figure S-2. In the source study this comparison followed PLS-DA performed with the mixOmics package on data processed with the lmm2met R package.

## Related tools

- **lmm2met** (R package that produced the LMM-processed metabolomics matrix whose PLS-DA feature list feeds the set comparison) — https://github.com/kwanjeeraw/lmm2met
- **mixOmics** (Used to run PLS-DA whose VIP > 1 significant features form the lists being intersected)
- **R** (Environment in which the set intersection, difference, union and Venn region counts are computed)

## Examples

```
Rscript -e 'lmm<-readLines("lmm_vip1.txt"); m0<-readLines("m0_vip1.txt"); u<-union(lmm,m0); stopifnot(length(u)==length(lmm)+length(m0)-length(intersect(lmm,m0))); write.csv(data.frame(metabolite=u, in_lmm=u%in%lmm, in_m0=u%in%m0), "venn_intersection_union.csv", row.names=FALSE)'
```

## Evaluation signals

- Every metabolite in the output appears in at least one input significant-metabolite list
- Counts satisfy |union| = |LMM list| + |M0 list| - |intersection|
- Feature selection was performed at the VIP > 1 threshold stated in the methods
- Output CSV contains metabolite memberships for each Venn region plus the region counts, matching the Venn structure (e.g. Figure S-2: 9 unique to LMM, 6 unique to M0, 60 common)

## Limitations

- Overlap analysis alone does not establish which processing strategy is superior; the article recommends further experiments to validate results
- LMM's increased sensitivity may introduce false positives, which would inflate the unique-to-LMM region
- Some discriminative signals (e.g. cholesterol) are absent from the M0 list not because of biology but because signals were 'randomly mixed and indistinguishable' before LMM fitting

## Evidence

- [results] significant metabolites or discriminative metabolites (VIP > 1): "significant metabolites or discriminative metabolites (VIP > 1)"
- [methods] The mixOmics package was applied for PLS-DA and multilevel-based PLS-DA.: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [discussion] List of significant metabolites and VIP scores from the LMM- and M0-PLS-DA of CRC samples: "List of significant metabolites and VIP scores from the LMM- and M0-PLS-DA of CRC samples"
- [results] Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable: "Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable"
- [readme] Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada.: "Linear mixed-effects modeling to improve clinical metabolomics information."
