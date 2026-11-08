---
name: lmm-processed-vs-original-data-contrast
description: Use when you have discriminative feature lists (VIP > 1) from PLS-DA
  run on both LMM- and M0-processed metabolomics matrices and need to quantify shared
  vs. uniquely selected metabolites between processing strategies.
when_to_use_negative:
- You only have one processed dataset — the contrast requires feature lists from both
  LMM- and M0-processed PLS-DA.
- Feature importance comes from a method without a VIP score (the selection threshold
  VIP > 1 is specific to PLS-DA/OPLS-DA output).
- The goal is classification performance comparison (accuracy/precision/sensitivity/specificity
  via caret) rather than feature-set overlap.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: R
  role: Environment for computing set intersections, differences, and unions of the
    VIP > 1 metabolite lists and writing the CSV output
  repo: https://www.r-project.org/
- name: mixOmics
  role: Performs the PLS-DA on M0- and LMM-processed training sets whose VIP scores
    define the discriminative metabolite lists being contrasted
- name: lmm2met
  role: Produces the LMM-processed metabolomics matrix by fitting each metabolite
    to a linear mixed-effects model using subject metadata
  repo: https://github.com/kwanjeeraw/lmm2met
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2019.04.009
    title: Accounting for biological variation with linear mixed-effects modelling
      improves the quality of clinical metabolomics data
  source_content_hash: sha256:df6f947b1efeaaeda9ab04eebe8ee159fbf76f12309e9ab9e06c0534fd226c64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - lmm2met
  - mixOmics
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lmm-processed-vs-original-data-contrast

## Summary

Contrasts discriminative metabolite sets (VIP > 1) selected by PLS-DA from LMM-processed versus M0-processed clinical metabolomics data, quantifying shared and uniquely recovered features. This reveals signals (e.g. cholesterol in CRC adipose tissue) that only emerge after linear mixed-effects modelling with patient metadata.

## When to use

Use when you have discriminative feature lists (VIP > 1) from PLS-DA run on both LMM- and M0-processed metabolomics matrices and need to quantify shared vs. uniquely selected metabolites between processing strategies.

## When NOT to use

- You only have one processed dataset — the contrast requires feature lists from both LMM- and M0-processed PLS-DA.
- Feature importance comes from a method without a VIP score (the selection threshold VIP > 1 is specific to PLS-DA/OPLS-DA output).
- The goal is classification performance comparison (accuracy/precision/sensitivity/specificity via caret) rather than feature-set overlap.

## Inputs

- VIP > 1 significant-metabolite list from LMM-processed PLS-DA (e.g. CRC adipose tissue, Table S-3)
- VIP > 1 significant-metabolite list from M0-processed PLS-DA
- Optionally, the processed metabolomics matrices (158 annotated lipids from VAT and SAT of 59 patients)

## Outputs

- venn_intersection_union.csv with metabolite memberships and Venn region counts (LMM-unique, M0-unique, shared)
- Count summary, e.g. 9 metabolites unique to LMM, 6 unique to M0, 60 shared

## How to apply

Load the significant-metabolite lists (VIP > 1) from the LMM-PLS-DA and M0-PLS-DA, e.g. the CRC adipose tissue data of 158 annotated lipids from VAT and SAT of 59 colorectal cancer patients (Table S-3). Compute the set intersection for metabolites discriminative under both processing approaches, and the set differences for metabolites uniquely captured by LMM processing (such as cholesterol-type signals missed before LMM fitting) versus uniquely captured by M0 processing. Tabulate the Venn region counts, confirming |union| = |LMM list| + |M0 list| - |intersection|, and write memberships plus counts to a CSV (e.g. venn_intersection_union.csv). The rationale is that LMM fitting with patient metadata systematically minimizes subject-specific variation, changing which features pass the VIP > 1 threshold in mixOmics PLS-DA.

## Related tools

- **R** (Environment for computing set intersections, differences, and unions of the VIP > 1 metabolite lists and writing the CSV output) — https://www.r-project.org/
- **mixOmics** (Performs the PLS-DA on M0- and LMM-processed training sets whose VIP scores define the discriminative metabolite lists being contrasted)
- **lmm2met** (Produces the LMM-processed metabolomics matrix by fitting each metabolite to a linear mixed-effects model using subject metadata) — https://github.com/kwanjeeraw/lmm2met

## Examples

```
install.packages('devtools', repos="http://cran.rstudio.com/"); library(devtools); install_github('kwanjeeraw/lmm2met')
```

## Evaluation signals

- Set invariant holds: |union| = |LMM list| + |M0 list| - |intersection|
- Every metabolite in the output appears in at least one input VIP > 1 list (no fabricated memberships)
- Region counts match the expected Venn structure in Figure S-2 (9 LMM-unique, 6 M0-unique, 60 shared for the CRC adipose data)
- Feature selection was performed at the stated VIP > 1 threshold from PLS-DA, not an arbitrary cutoff
- Output CSV parses correctly with one row per metabolite and correct membership flags

## Limitations

- Cholesterol was not identified by PLS-DA in the M0 dataset before LMM fitting (signals randomly mixed and indistinguishable), so the M0 list incompletely captures biologically relevant features
- Increased sensitivity of the LMM approach carries the potential introduction of false positives among uniquely selected features
- Further experiments are recommended to validate the results (validation gap)

## Evidence

- [methods] The significant features were selected based on variable importance in projection score (VIP > 1): "The significant features were selected based on variable importance in projection score (VIP > 1)"
- [discussion] List of significant metabolites and VIP scores from the LMM- and M0-PLS-DA of CRC samples: "List of significant metabolites and VIP scores from the LMM- and M0-PLS-DA of CRC samples"
- [results] Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable: "Before LMM fitting, cholesterol signals were randomly mixed and indistinguishable"
- [discussion] subject-specific variation in each metabolite was systematically minimized: "subject-specific variation in each metabolite was systematically minimized"
- [methods] The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA.: "The mixOmics package [34] was applied for PLS-DA and multilevel-based PLS-DA."
- [methods] We developed the R package lmm2met to process a metabolomics matrix with the LMM approach.: "We developed the R package lmm2met to process a metabolomics matrix with the LMM approach."
