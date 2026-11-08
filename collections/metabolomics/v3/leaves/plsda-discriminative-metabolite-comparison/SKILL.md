---
name: plsda-discriminative-metabolite-comparison
description: Use when two PLS-DA feature-selection runs (e.g., LMM-processed vs M0-processed)
  each yield a VIP > 1 discriminative metabolite list and you must quantify overlap,
  unique selections, and Venn-region counts.
when_to_use_negative:
- The input is a raw metabolomics feature table rather than already-derived VIP-ranked
  discriminative metabolite lists from two PLS-DA runs
- Feature selection used a different significance criterion (e.g., p-values or loadings
  instead of the VIP > 1 threshold), making the two lists non-comparable under this
  skill's validation rules
- The PLS-DA runs were not performed on comparable versions of the same dataset (e.g.,
  different cohorts or platforms), so set membership differences would not be attributable
  to the processing strategy
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mixOmics
  role: Used to perform PLS-DA that generates the VIP-based discriminative metabolite
    lists being compared
- name: R
  role: Environment in which the set intersection, difference, and union operations
    on the VIP lists are computed
  repo: https://www.r-project.org/
- name: lmm2met
  role: R package that performs the LMM processing step producing the LMM-processed
    data whose PLS-DA VIP list is compared against the M0 list
  repo: https://github.com/kwanjeeraw/lmm2met
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:e7c016cb4254062a9e80840cd6d9f3f4c8aaf98fc2234242cf3d38139253636d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - lmm2met
  - mixOmics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# plsda-discriminative-metabolite-comparison

## Summary

Compares discriminative metabolite lists (VIP > 1) selected from PLS-DA of LMM-processed versus M0-processed clinical metabolomics data using set intersection, difference, and union operations to quantify which features each processing strategy uniquely recovers.

## When to use

Use when two PLS-DA feature-selection runs (e.g., LMM-processed vs M0-processed) each yield a VIP > 1 discriminative metabolite list and you must quantify overlap, unique selections, and Venn-region counts.

## When NOT to use

- The input is a raw metabolomics feature table rather than already-derived VIP-ranked discriminative metabolite lists from two PLS-DA runs
- Feature selection used a different significance criterion (e.g., p-values or loadings instead of the VIP > 1 threshold), making the two lists non-comparable under this skill's validation rules
- The PLS-DA runs were not performed on comparable versions of the same dataset (e.g., different cohorts or platforms), so set membership differences would not be attributable to the processing strategy

## Inputs

- List of discriminative metabolites (VIP > 1) from LMM-PLS-DA of the CRC adipose tissue dataset (Table S-3)
- List of discriminative metabolites (VIP > 1) from M0-PLS-DA of the same dataset
- VIP score table from PLS-DA performed with mixOmics on log2-transformed lipid profiles (158 annotated lipids, VAT and SAT, 59 patients)

## Outputs

- venn_intersection_union.csv containing metabolite memberships of the intersection, LMM-unique, M0-unique, and union regions
- Counts of each Venn region (e.g., 9 uniquely LMM, 6 uniquely M0, 60 shared) matching Figure S-2

## How to apply

Load the significant-metabolite lists (VIP > 1) from LMM-PLS-DA and M0-PLS-DA runs — for the CRC adipose tissue case this is the 158 annotated lipids from VAT and SAT of 59 patients (Table S-3). Compute the set intersection to obtain metabolites discriminative under both processing approaches, and the set differences to obtain metabolites uniquely selected by LMM processing (e.g., cholesterol-type signals missed before LMM fitting) and uniquely selected by M0 processing. Take the union and tabulate the counts of each Venn region as depicted in Figure S-2. Perform feature selection at the VIP > 1 threshold stated in the methods (significant features selected based on variable importance in projection score), with PLS-DA carried out in R using the mixOmics package. Write metabolite memberships and region counts to venn_intersection_union.csv and validate with the cardinality identity |union| = |LMM list| + |M0 list| - |intersection|.

## Related tools

- **mixOmics** (Used to perform PLS-DA that generates the VIP-based discriminative metabolite lists being compared)
- **R** (Environment in which the set intersection, difference, and union operations on the VIP lists are computed) — https://www.r-project.org/
- **lmm2met** (R package that performs the LMM processing step producing the LMM-processed data whose PLS-DA VIP list is compared against the M0 list) — https://github.com/kwanjeeraw/lmm2met

## Examples

```
Rscript -e 'i<-intersect(lmm_vip_list, m0_vip_list); u<-union(lmm_vip_list, m0_vip_list); write.csv(data.frame(intersection=i, union=u), "venn_intersection_union.csv")'
```

## Evaluation signals

- Cardinality invariant holds: |union| = |LMM list| + |M0 list| - |intersection|
- Every metabolite in the output appears in at least one input VIP > 1 list
- Feature selection in both input lists was performed at the VIP > 1 threshold stated in the methods
- Region counts match the published Venn structure in Figure S-2 (9 uniquely LMM, 6 uniquely M0, 60 shared across all processed datasets)
- venn_intersection_union.csv contains both metabolite memberships and per-region counts

## Limitations

- Cholesterol was not identified by PLS-DA in the M0 dataset before LMM fitting (signals were randomly mixed and indistinguishable), so M0 lists can miss physiologically important features
- The increased sensitivity of the LMM approach carries the potential introduction of false positives into the LMM-unique region
- Further experiments are recommended to validate the results — the discriminative-metabolite overlap is a computational comparison, not biological validation

## Evidence

- [results] VIP > 1 threshold used to identify significant/discriminative metabolites: "significant metabolites or discriminative metabolites (VIP > 1)"
- [methods] VIP > 1 feature selection criterion: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [methods] PLS-DA performed with mixOmics: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [results] Cholesterol missed by M0 processing: "Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable"
- [readme] lmm2met installation from GitHub README: "install_github('kwanjeeraw/lmm2met')"
