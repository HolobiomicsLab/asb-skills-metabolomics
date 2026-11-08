---
name: paired-adipose-tissue-lipidomics-context
description: Use when two feature-selection lists (e.g. VIP > 1 metabolites from PLS-DA
  under two processing strategies such as LMM vs M0) must be compared to quantify
  shared and uniquely identified discriminative metabolites.
when_to_use_negative:
- The input is a raw metabolomics feature table rather than already-selected significant
  (VIP > 1) metabolite lists from fitted models
- Only one processing strategy was applied, so no paired LMM-vs-M0 comparison of feature
  lists can be made
- Samples are unpaired or lack a matched-patient design such as paired VAT/SAT tissue
  from the same subjects
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mixOmics
  role: Performs the PLS-DA whose VIP > 1 discriminative features form the compared
    metabolite lists
- name: lmm2met
  role: Produces the LMM-processed metabolomics matrix from which the LMM-PLS-DA feature
    list is derived
  repo: https://github.com/kwanjeeraw/lmm2met
- name: R
  role: Implementation environment for set operations (intersect/union) and writing
    venn_intersection_union.csv
  repo: https://www.r-project.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:6113dd31cc4a8c92358d3038b0dd4a1086c055285d140d83f6312b65a07889f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kwanjeeraw/lmm2met
  tools:
  - R
  - lmm2met
  - mixOmics
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# paired-adipose-tissue-lipidomics-context

## Summary

Compares discriminative-metabolite lists (VIP > 1) obtained from PLS-DA of LMM-processed versus M0-processed paired adipose tissue lipidomics data (158 annotated lipids, VAT and SAT of 59 CRC patients), computing intersection, set differences, and union counts as in Figure S-2.

## When to use

Use when two feature-selection lists (e.g. VIP > 1 metabolites from PLS-DA under two processing strategies such as LMM vs M0) must be compared to quantify shared and uniquely identified discriminative metabolites.

## When NOT to use

- The input is a raw metabolomics feature table rather than already-selected significant (VIP > 1) metabolite lists from fitted models
- Only one processing strategy was applied, so no paired LMM-vs-M0 comparison of feature lists can be made
- Samples are unpaired or lack a matched-patient design such as paired VAT/SAT tissue from the same subjects

## Inputs

- VIP > 1 significant-metabolite list from LMM-processed PLS-DA (CRC adipose tissue, Table S-3)
- VIP > 1 significant-metabolite list from M0-processed PLS-DA (CRC adipose tissue, Table S-3)
- Adipose tissue lipid profiles of 59 colorectal cancer patients (VAT and SAT samples, Liesenfeld et al. 2015)

## Outputs

- venn_intersection_union.csv with metabolite memberships per Venn region
- Region counts: uniquely LMM, uniquely M0, and shared discriminative metabolites
- Venn overlap summary as depicted in Figure S-2

## How to apply

Load the significant-metabolite lists (VIP > 1) from the LMM-PLS-DA and M0-PLS-DA of the CRC adipose tissue dataset (158 annotated lipids from VAT and SAT of 59 colorectal cancer patients, Table S-3), generated with mixOmics on data processed via lmm2met or left unprocessed (M0). Compute the set intersection for metabolites discriminative under both processing approaches, and set differences for metabolites uniquely captured by LMM processing (e.g. cholesterol-type signals missed before LMM fitting) or uniquely captured by M0 processing. Verify the Venn invariant |union| = |LMM list| + |M0 list| - |intersection| and tabulate region counts as in Figure S-2. Expected outcome in the source study: nine metabolites unique to LMM, six unique to M0, and 60 shared across processed datasets.

## Related tools

- **mixOmics** (Performs the PLS-DA whose VIP > 1 discriminative features form the compared metabolite lists)
- **lmm2met** (Produces the LMM-processed metabolomics matrix from which the LMM-PLS-DA feature list is derived) — https://github.com/kwanjeeraw/lmm2met
- **R** (Implementation environment for set operations (intersect/union) and writing venn_intersection_union.csv) — https://www.r-project.org/

## Examples

```
shared <- intersect(lmm_vip_metab, m0_vip_metab); unique_lmm <- setdiff(lmm_vip_metab, m0_vip_metab); write.csv(list(shared=shared, unique_lmm=unique_lmm), 'venn_intersection_union.csv')
```

## Evaluation signals

- Every reported metabolite appears in at least one of the two input lists (no invented features)
- The Venn invariant |union| = |LMM list| + |M0 list| - |intersection| holds exactly
- Feature selection threshold is confirmed as VIP > 1 for both PLS-DA models before comparison
- Output CSV contains both memberships and region counts for all Venn regions (uniquely LMM, uniquely M0, shared)
- Reproducing on the source data recovers the reported counts: 9 unique to LMM, 6 unique to M0, 60 shared

## Limitations

- Cholesterol was not identified by PLS-DA in the M0 dataset before LMM fitting (signals randomly mixed and indistinguishable), so M0 lists may miss biologically relevant signals
- LMM processing may introduce false positives due to increased sensitivity
- Discriminative-feature lists depend on the cross-validated PLS-DA fit, so overlap counts can vary with model training partitions

## Evidence

- [methods] VIP selection threshold: "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [methods] PLS-DA engine: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [methods] Source data: "Lipid profiles of adipose tissue samples [1] are used in this tutorial."
- [discussion] Comparison object: "List of significant metabolites and VIP scores from the LMM- and M0-PLS-DA of CRC samples"
- [readme] Tool availability: "Linear mixed-effects modelling for normalization of clinical metabolomics data by using subject metada."
