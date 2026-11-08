---
name: msms-validated-annotation-matching
description: Use when comparing metabolite annotation outputs (formula+adduct pairs
  at a stated FDR, e.g. < 10%) from two or more annotation methods against an independent
  MS/MS-validated bulk LC-MS/MS reference ground truth.
when_to_use_negative:
- The annotations are not normalized to formula+adduct identifiers, so they cannot
  be intersected with the LC-MS/MS reference on a common key
- The validation target is spatially resolved peak-shape agreement rather than bulk
  LC-MS/MS ground-truth matching
- The reference identifications are not MS/MS-validated (e.g. only accurate-mass database
  hits), which breaks the ground-truth assumption
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: METASPACE-ML
  role: Machine-learning annotation approach whose formula+adduct outputs at FDR <
    10% are validated against MS/MS-validated bulk LC-MS/MS identifications
  repo: https://github.com/metaspace2020/metaspace
- name: METASPACE
  role: Platform/engine whose rule-based MSM annotations provide the comparison arm
    in the TPR analysis
  repo: https://github.com/metaspace2020/metaspace
- name: pandas
  role: Used for set-based identifier intersection of imaging formula+adduct annotations
    against the LC-MS/MS identifications
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-52213-9
    title: 'METASPACE-ML: Context-specific metabolite annotation for imaging mass
      spectrometry using machine learning'
  source_content_hash: sha256:153d6d6a9676e9a890e22c5cfe02f017248304a623d54eeeb790b8f723d50044
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metaspace2020/metaspace
  tools:
  - METASPACE
  - METASPACE-ML
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# msms-validated-annotation-matching

## Summary

Validates metabolite annotations (formula+adduct pairs) from imaging MS annotation approaches against MS/MS-validated bulk LC-MS/MS identifications by set-based identifier intersection, then computes per-approach median METASPACE annotation power (MAP) and true positive rates. Used in the METASPACE-ML paper to show 91% overall TPR for METASPACE-ML vs the rule-based MSM approach in 10 mouse brain MALDI-FTICR imaging datasets.

## When to use

Use when comparing metabolite annotation outputs (formula+adduct pairs at a stated FDR, e.g. < 10%) from two or more annotation methods against an independent MS/MS-validated bulk LC-MS/MS reference ground truth.

## When NOT to use

- The annotations are not normalized to formula+adduct identifiers, so they cannot be intersected with the LC-MS/MS reference on a common key
- The validation target is spatially resolved peak-shape agreement rather than bulk LC-MS/MS ground-truth matching
- The reference identifications are not MS/MS-validated (e.g. only accurate-mass database hits), which breaks the ground-truth assumption

## Inputs

- Supplementary Data 8: formula+adduct annotations at FDR < 10% by METASPACE-ML and MSM for the 10 mouse brain MALDI-FTICR imaging datasets
- Supplementary Data 9: MS/MS-validated bulk LC-MS/MS metabolite identifications (reference ground truth)
- List of the 10 mouse brain MALDI-FTICR imaging dataset identifiers

## Outputs

- validation_metrics.csv: per-approach median METASPACE annotation power (MAP), overall TPR, and exclusive-ion TPR
- Labeled annotation table: each formula+adduct annotation flagged as MS/MS-validated true positive or not, plus exclusivity flags

## How to apply

Load the annotations file (Supplementary Data 8 in the paper: formula+adduct pairs annotated at FDR < 10% by METASPACE-ML and MSM across 10 mouse brain MALDI-FTICR imaging datasets) and the reference file (Supplementary Data 9: MS/MS-validated bulk LC-MS/MS metabolite identifications). Match each imaging annotation's formula+adduct pair against the LC-MS/MS identifications using pandas set-based identifier intersection, labeling matches as MS/MS-validated true positives. Compute per-approach median MAP across the 10 datasets, then overall TPR per approach, and TPR restricted to ions annotated exclusively by one approach. Correct application reproduces median MAP 0.57 (METASPACE-ML) vs 0.43 (MSM), overall TPR 91%, and exclusive-ion TPR 72% vs 11%; write the metric table to validation_metrics.csv.

## Related tools

- **METASPACE-ML** (Machine-learning annotation approach whose formula+adduct outputs at FDR < 10% are validated against MS/MS-validated bulk LC-MS/MS identifications) — https://github.com/metaspace2020/metaspace
- **METASPACE** (Platform/engine whose rule-based MSM annotations provide the comparison arm in the TPR analysis) — https://github.com/metaspace2020/metaspace
- **pandas** (Used for set-based identifier intersection of imaging formula+adduct annotations against the LC-MS/MS identifications)

## Examples

```
import pandas as pd; ann = pd.read_csv('supp_data8.csv'); msms = pd.read_csv('supp_data9.csv'); ann['tp'] = ann['formula_adduct'].isin(msms['formula_adduct']); ann.to_csv('validation_metrics.csv')
```

## Evaluation signals

- Reproduced overall true positive rate for METASPACE-ML annotations is 91% in the LC-MS/MS bulk validation
- Exclusive-ion TPR matches 72% (METASPACE-ML) vs 11% (MSM) for ions captured by only one approach
- Per-approach median MAP across the 10 datasets matches 0.57 (METASPACE-ML) vs 0.43 (MSM)
- Every matched true positive corresponds to an exact formula+adduct pair present in the MS/MS-validated LC-MS/MS reference (Supplementary Data 9)
- Output table validation_metrics.csv exists with one row per annotation approach and columns for median MAP, overall TPR, and exclusive-ion TPR

## Limitations

- MSM's sensitivity to its rho_spatial score causes it to capture some annotations missed by METASPACE-ML, so exclusive-ion sets are non-empty and one-sided TPRs are expected to diverge strongly
- Validation relies on formula+adduct identity matching; LC-MS/MS ground truth is bulk-level and does not confirm spatial localization of the imaging annotation

## Evidence

- [results] Overall and exclusive-ion TPR reproduced by this skill: [evidence span withheld — anchor status: not_found]
- [supplementary] Input annotation source at FDR < 10%: [evidence span withheld — anchor status: not_found]
- [abstract] Rationale for comparing the two annotation approaches: "METASPACE-ML outperforms its rule-based predecessor, exhibiting higher precision, increased throughput, and enhanced capability in identifying low-intensity and biologically-relevant metabolites."
- [readme] METASPACE platform as annotation engine and knowledgebase: "hosts an engine for metabolite annotation of imaging mass spectrometry data as well as a spatial metabolite knowledgebase of the metabolites from thousands of public datasets"
